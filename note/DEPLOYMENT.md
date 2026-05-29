# DotNet7Web 部署指南 (Deployment Guide)

這份文件旨在提供 `DotNet7Web` 專案在不同環境下的部署策略與步驟。由於本專案為標準的 .NET 7 Web 應用程式，您可以選擇多種方式進行部署。

## 1. 發布 (Publish) 專案

在部署之前，必須先將專案編譯並發布為可執行的檔案或 DLL。

### 使用 .NET CLI 發布

在專案根目錄執行以下指令：

```bash
dotnet publish DotNet7Web/DotNet7Web.csproj -c Release -o ./publish
```

* `-c Release`: 以 Release 模式建置，進行最佳化。
* `-o ./publish`: 指定輸出的資料夾位置。

發布完成後，`./publish` 資料夾內會包含所有運行應用程式所需的檔案。

---

## 2. 環境變數設定

在生產環境 (Production) 中，通常需要設定環境變數來覆蓋預設的開發設定。

### ASPNETCORE_ENVIRONMENT

請確保在伺服器上設定 `ASPNETCORE_ENVIRONMENT` 變數為 `Production`。

* **Linux / macOS**:
  ```bash
  export ASPNETCORE_ENVIRONMENT=Production
  ```
* **Windows (PowerShell)**:
  ```powershell
  $env:ASPNETCORE_ENVIRONMENT="Production"
  ```

當設定為 `Production` 時，應用程式會自動讀取 `appsettings.Production.json` 的設定檔，並關閉詳細的開發者例外狀況網頁 (Developer Exception Page)。

---

## 3. 部署策略

### 方案 A: 使用 Docker 容器化 (推薦)

Docker 提供了一致且可移植的執行環境，非常適合現代應用程式部署。

1. **建立 Dockerfile** (如果專案尚未包含):
   在解決方案的根目錄建立 `Dockerfile`:
   ```dockerfile
   FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
   WORKDIR /app
   EXPOSE 80
   EXPOSE 443

   FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
   WORKDIR /src
   COPY ["DotNet7Web/DotNet7Web.csproj", "DotNet7Web/"]
   RUN dotnet restore "DotNet7Web/DotNet7Web.csproj"
   COPY . .
   WORKDIR "/src/DotNet7Web"
   RUN dotnet build "DotNet7Web.csproj" -c Release -o /app/build

   FROM build AS publish
   RUN dotnet publish "DotNet7Web.csproj" -c Release -o /app/publish /p:UseAppHost=false

   FROM base AS final
   WORKDIR /app
   COPY --from=publish /app/publish .
   ENTRYPOINT ["dotnet", "DotNet7Web.dll"]
   ```

2. **建置 Docker Image**:
   ```bash
   docker build -t dotnet7web-image .
   ```

3. **執行 Container**:
   ```bash
   docker run -d -p 8080:80 --name dotnet7web-app -e ASPNETCORE_ENVIRONMENT=Production dotnet7web-image
   ```

### 方案 B: Linux (Nginx + Kestrel)

在 Linux 伺服器上，通常會使用 Kestrel 作為內部應用程式伺服器，並使用 Nginx 或 Apache 作為反向代理 (Reverse Proxy) 來處理外部請求。

1. **將發布的檔案複製到伺服器**:
   將 `./publish` 資料夾內容複製到 `/var/www/dotnet7web`。

2. **設定 Systemd 服務**:
   建立服務設定檔 `/etc/systemd/system/dotnet7web.service`:
   ```ini
   [Unit]
   Description=DotNet7Web Application

   [Service]
   WorkingDirectory=/var/www/dotnet7web
   ExecStart=/usr/bin/dotnet /var/www/dotnet7web/DotNet7Web.dll
   Restart=always
   # 如果 dotnet 崩潰，10 秒後重啟
   RestartSec=10
   KillSignal=SIGINT
   SyslogIdentifier=dotnet7web
   User=www-data
   Environment=ASPNETCORE_ENVIRONMENT=Production

   [Install]
   WantedBy=multi-user.target
   ```
   啟用並啟動服務：
   ```bash
   sudo systemctl enable dotnet7web.service
   sudo systemctl start dotnet7web.service
   ```

3. **設定 Nginx 反向代理**:
   編輯 `/etc/nginx/sites-available/default` 或建立新設定檔：
   ```nginx
   server {
       listen 80;
       server_name example.com;

       location / {
           proxy_pass         http://127.0.0.1:5000;
           proxy_http_version 1.1;
           proxy_set_header   Upgrade $http_upgrade;
           proxy_set_header   Connection keep-alive;
           proxy_set_header   Host $host;
           proxy_cache_bypass $http_upgrade;
           proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header   X-Forwarded-Proto $scheme;
       }
   }
   ```
   重新載入 Nginx：
   ```bash
   sudo nginx -s reload
   ```

### 方案 C: Windows IIS 或 Windows 服務

如果您部署在 Windows 伺服器環境，可以使用 IIS 或設定為 Windows 服務。

**1. 使用 IIS 部署:**
   - 在伺服器上安裝對應版本的 **.NET Hosting Bundle**。
   - 在 IIS 管理員中新增網站。
   - 將實體路徑指向您發布的 `./publish` 資料夾。
   - 確保應用程式集區 (Application Pool) 的 .NET CLR 版本設定為「沒有 Managed 程式碼 (No Managed Code)」。
   - 確保 `IIS_IUSRS` 具有該資料夾的讀取與執行權限。

**2. 註冊為 Windows 服務:**
   如果不需要 IIS 作為反向代理，可以將應用程式直接註冊為 Windows 服務運行。
   - 需要在專案中安裝 `Microsoft.Extensions.Hosting.WindowsServices` 套件，並在 `Program.cs` 加上 `.UseWindowsService()`。
   - 使用 PowerShell 或 `sc.exe` 將發布的執行檔建立為服務。

---

## 4. 注意事項

- **資料庫遷移**: 如果專案使用了 Entity Framework Core 等 ORM 框架，請記得在部署前或部署指令碼中執行資料庫遷移 (`dotnet ef database update` 或產生 SQL 腳本)。
- **HTTPS/SSL**: 建議在反向代理 (Nginx/IIS) 層設定 SSL 憑證，確保資料傳輸安全。
- **日誌管理**: 在生產環境中，建議設定更完善的日誌機制 (如 Serilog, NLog)，並將日誌輸出到檔案或集中式日誌伺服器，以便後續維運與除錯。
