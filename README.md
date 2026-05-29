# DotNet7Web

這是一個用於學習目的的 .NET 7 Web (Razor Pages) 專案。

## 專案環境
- **開發工具**: Visual Studio 2022 for Mac
- **框架**: .NET 7

## 如何啟動專案

### 透過 IDE 啟動 (建議)

如果您是使用 **Visual Studio 2022 for Mac** (或其他支援的 IDE)：
1. 雙擊專案目錄下的 `DotNet7Web.sln` 解決方案檔案來開啟專案。
2. 點擊上方的「執行」按鈕 (Run) 或按下 `Cmd + Enter` 即可編譯並啟動專案。
3. 瀏覽器將會自動開啟並導向本機的開發伺服器位址。

### 透過命令列啟動 (CLI)

如果您偏好使用終端機 (Terminal) 或 .NET CLI 啟動本專案，請確保您已安裝 .NET 7 SDK。

1. **進入專案根目錄**:
   ```bash
   cd /path/to/DotNet7Web
   ```
   *(請將路徑替換為您實際存放專案的位置)*

2. **還原 NuGet 套件**:
   ```bash
   dotnet restore
   ```

3. **啟動專案**:
   ```bash
   dotnet run --project DotNet7Web/DotNet7Web.csproj
   ```

4. **在瀏覽器中查看**:
   專案啟動後，終端機會輸出類似以下的訊息：
   ```text
   info: Microsoft.Hosting.Lifetime[14]
         Now listening on: https://localhost:xxxx
   info: Microsoft.Hosting.Lifetime[14]
         Now listening on: http://localhost:yyyy
   ```
   請複製輸出中的 `https://localhost:xxxx` 網址並貼上至您的瀏覽器中即可瀏覽網站。

### 停止專案
在終端機中按下 `Ctrl + C` 即可停止正在執行的應用程式。
