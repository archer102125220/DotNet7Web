# 使用 asdf 在 macOS 安裝與管理 .NET SDK

這份文件記錄了如何在 macOS 環境下，使用套件版本管理工具 [asdf](https://asdf-vm.com/) 來安裝和管理 .NET SDK。

## 1. 系統環境準備

請確保您的 Mac 上已經安裝了 `asdf`。如果您尚未安裝，可以透過 Homebrew 來安裝：

```bash
# 透過 Homebrew 安裝 asdf
brew install asdf

# 請根據您使用的 Shell (如 zsh 或 bash)，將 asdf 載入腳本加入至環境變數設定檔中（例如 ~/.zshrc 或 ~/.bash_profile）
# (詳細請參考 asdf 官方安裝文件)
```

## 2. 加入 .NET Core 外掛 (Plugin)

首先，您需要將 .NET Core 的外掛加入 asdf。執行以下指令：

```bash
asdf plugin add dotnet-core https://github.com/emersonmx/asdf-dotnet-core.git
```
*(提示：通常只需執行 `asdf plugin add dotnet-core`，asdf 也會自動找到官方列表中的路徑)*

## 3. 安裝 .NET SDK

### 查看可用的版本
您可以列出所有可透過 asdf 安裝的 .NET 版本：

```bash
asdf list all dotnet-core
```
這會列出非常多版本（包括不同的 Patch 號）。建議找出您需要的 Major 版本最新版（如 `8.0.xxx` 或 `7.0.xxx`）。

### 執行安裝
以本專案使用的 .NET 7.0 SDK 為例 (例如 `7.0.317`)，請執行：

```bash
asdf install dotnet-core 7.0.317
```

> **注意**：下載與解壓縮過程可能會需要一些時間，請耐心等候。

## 4. 設定與切換版本

安裝完成後，您必須告訴 asdf 在哪裡使用這個版本。您可以選擇設定為全域 (Global) 或專案目錄 (Local) 層級。

### 設定為全域 (Global)
如果您希望整個系統預設都使用這個版本：

```bash
asdf set -u dotnet-core 7.0.317
```
這會在您的主目錄 (`~/.tool-versions`) 中記錄該版本（`-u` 等同於 `--home`）。

### 設定為單一專案 (Local)
如果您只希望在這個特定專案中使用，請進入專案根目錄後執行：

```bash
asdf set dotnet-core 7.0.317
```
這會在當前專案資料夾下建立或更新 `.tool-versions` 檔案，確保團隊其他開發者如果也用 asdf，能自動匹配對應的版本。

## 5. 驗證安裝

完成設定後，檢查 `dotnet` 是否已成功透過 asdf 的 shim 載入，並確認版本號：

```bash
dotnet --version
```
如果出現對應的 SDK 版本號 (例如 `7.0.317`)，代表您已成功透過 asdf 裝好並啟用了 .NET SDK！

## 6. 使用 `global.json` 鎖定專案 SDK 版本

除了 asdf 的 `.tool-versions` 之外，.NET 官方提供了一個更原生的方式來控制專案使用的 SDK 版本，那就是 `global.json`。

### `global.json` 的作用
`global.json` 檔案放在專案目錄，用來告訴 `dotnet` CLI 在建置或執行此專案時，必須使用哪個特定版本的 .NET SDK。這能確保團隊中所有開發者和 CI/CD 流程都使用完全一樣的 SDK 版本，避免因為 SDK 版本差異導致不可預期的建置或編譯錯誤。以這份專案為例，它鎖定了 `7.0.317` 版的 SDK。

### 設定與使用方式
您可以直接在專案根目錄建立或編輯 `global.json` 檔案。例如，本專案的 `global.json` 內容如下：

```json
{
  "sdk": {
    "version": "7.0.317"
  }
}
```

如果您想透過指令快速產生這個檔案，可以在專案目錄下執行：
```bash
dotnet new globaljson --sdk-version 7.0.317
```
設定好之後，只要在該目錄下執行任何 `dotnet` 相關指令，CLI 就會自動優先使用 `global.json` 所定義的 SDK 版本。

> **⚠️ 重要注意 (針對 asdf 使用者)：**
> 因為 asdf 會將不同版本的 SDK 安裝在彼此隔離的資料夾中，所以請務必確保專案目錄下的 `.tool-versions` (由 `asdf set dotnet-core <version>` 產生) 與 `global.json` 內指定的版本 **完全一致**。
> 如果兩者不一致，asdf 可能會將指令導向錯誤版本的 SDK 資料夾，導致 `dotnet` 找不到 `global.json` 所要求版本的 SDK 而報錯。

---

## 補充筆記

### 1. 全域工具路徑
如果您習慣透過 `dotnet tool install -g <tool_name>` 安裝全域命令列工具 (例如 `dotnet-ef`)，它們預設會安裝在 `~/.dotnet/tools`。

針對 asdf 使用者，若系統找不到安裝好的全域工具，建議將以下路徑加入到您的 `~/.zshrc` 或 `~/.bash_profile` 中：

```bash
export PATH="$HOME/.dotnet/tools:$PATH"
```

### 2. IDE 找不到 .NET SDK (例如 VS Code 或 Rider)
如果您使用的編輯器或 IDE 找不到透過 asdf 安裝的 .NET SDK，您需要設定 `DOTNET_ROOT` 環境變數。根據您的使用情境，有以下兩種設定方式：

#### 做法一：單一靜態設定 (傳統寫法)
如果您只使用一個版本的 .NET SDK，請在您的 Shell 設定檔 (`~/.zshrc` 等) 中加入：

```bash
export DOTNET_ROOT=$(asdf where dotnet-core)
```
> **缺點**：這個寫法會在終端機啟動時將路徑固定。如果您同時開發多個專案，且各專案要求不同的 `.NET` 版本，當您切換資料夾時，這個變數不會自動更新，可能會導致 IDE 抓錯 SDK 版本。

#### 做法二：動態自動切換 (官方推薦寫法)
為了解決多專案版本不一致的問題，您可以透過 `asdf-dotnet-core` 提供的指令碼來動態載入環境變數。請在您的 Shell 設定檔中改為加入：

```bash
# 針對 Zsh 使用者
. ~/.asdf/plugins/dotnet-core/set-dotnet-home.zsh

# 針對 Bash 使用者
# . ~/.asdf/plugins/dotnet-core/set-dotnet-home.bash
```
> **優點**：當您使用 `cd` 切換到不同專案目錄時，系統會根據當下的 `.tool-versions` 自動調整 `DOTNET_ROOT`，比直接寫死 `export` 更加靈活且穩定。

設定完成後，請記得執行 `source ~/.zshrc`（或對應的設定檔）使變數生效，並重新啟動您的 IDE 讓工具抓取最新設定。
