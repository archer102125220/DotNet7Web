# DotNet7Web 專案結構與設定檔說明

這份文件旨在說明本 .NET 7 Web (Razor Pages) 專案的資料夾結構以及各個重要設定檔的用途，幫助開發者快速了解專案架構。

## 📁 專案根目錄 (Solution Root)

```text
DotNet7Web/ (Solution Folder)
├── DotNet7Web.sln          # Visual Studio 方案檔
├── global.json             # 指定 .NET SDK 版本
├── .gitignore              # Git 忽略檔案清單
├── README.md               # 專案說明文件
├── note/                   # 存放開發筆記與技術文件 (例如此文件)
└── DotNet7Web/             # 實際的專案原始碼目錄 (Project Folder)
```

### 重要檔案說明
- **`DotNet7Web.sln`**: Visual Studio 方案檔 (Solution File)，用來組織和管理一個或多個專案。
- **`global.json`**: 用來指定專案建置時應使用的 .NET SDK 版本，確保開發團隊使用一致的 SDK，避免版本差異帶來的問題。

---

## 📂 專案原始碼目錄 (Project Folder)

進入 `DotNet7Web/` 資料夾，這是專案的實際內容：

```text
DotNet7Web/ (Project Folder)
├── DotNet7Web.csproj               # 專案設定檔
├── Program.cs                      # 應用程式進入點與服務註冊
├── appsettings.json                # 應用程式全域設定檔
├── appsettings.Development.json    # 開發環境專用的設定檔
├── Pages/                          # 存放 Razor Pages 視圖與程式碼
├── wwwroot/                        # 存放靜態資源檔案
├── Properties/                     # 專案屬性與啟動設定
├── bin/                            # 編譯後的輸出檔 (二進位檔)
└── obj/                            # 編譯過程中的暫存檔
```

### 1. 核心設定與程式進入點
- **`DotNet7Web.csproj`**: 
  專案檔 (Project File)，基於 XML 格式，定義了專案的目標框架 (Target Framework, 例如 `net7.0`)、需要的 NuGet 套件參考 (Packages)、以及其他建置設定。
- **`Program.cs`**: 
  .NET 6+ 之後採用的頂層語句 (Top-level statements) 進入點檔案。負責：
  - 建立 `WebApplicationBuilder`。
  - 註冊相依性注入 (DI, Dependency Injection) 的服務 (如 `AddRazorPages()`)。
  - 配置 HTTP 請求管線 (Middleware Pipeline, 如路由、錯誤處理、靜態檔案中介軟體)。
  - 啟動應用程式 (`app.Run()`)。

### 2. 環境設定檔 (appsettings)
- **`appsettings.json`**: 
  應用程式的全域組態設定檔，存放如資料庫連線字串 (Connection Strings)、Log 紀錄層級 (Logging Level)、自定義的應用程式變數等。
- **`appsettings.Development.json`**: 
  針對開發環境 (`ASPNETCORE_ENVIRONMENT=Development`) 的設定檔。這裡的設定會覆蓋 `appsettings.json` 中的同名設定，適合放一些開發期間專用或敏感的除錯設定。

### 3. Razor Pages 與前端資源
- **`Pages/`**: 
  Razor Pages 專案的核心資料夾，用於處理網頁請求。
  - 包含 `.cshtml` (HTML 與 Razor 語法) 及 `.cshtml.cs` (Code-behind, 負責處理頁面邏輯的 C# 類別)。
  - `Pages/Shared/` 資料夾通常存放共用的版面配置 (如 `_Layout.cshtml`) 或部分檢視。
- **`wwwroot/`**: 
  網頁的靜態檔案根目錄 (Web Root)。
  - 放置 CSS 樣式表 (`css/`)、JavaScript 檔案 (`js/`)、圖片 (`lib/`) 及第三方函式庫 (如 Bootstrap, jQuery)。
  - 只有放在此資料夾中的檔案，才能直接透過 URL 被客戶端瀏覽器存取。

### 4. 開發與建置目錄
- **`Properties/launchSettings.json`**: 
  (位於 `Properties/` 內) 定義了專案在開發環境啟動時的設定檔 (Profiles)。包含啟動的 URL、使用的 Port (如 HTTPS/HTTP)、以及設定環境變數 (例如 `ASPNETCORE_ENVIRONMENT` 設為 `Development`)。此檔案**僅於本機開發時生效**，不會佈署到正式環境。
- **`bin/` 與 `obj/`**: 
  - `obj/` (Object): 存放建置過程中的暫存與中間產物，幫助加速後續的編譯。
  - `bin/` (Binary): 存放最終編譯出來的組件 (`.dll`) 以及執行檔 (`.exe`)，是真正要發佈/執行的內容。這兩個資料夾通常會被加入 `.gitignore` 中。
