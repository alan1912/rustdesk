<p align="center">
  <img src="res/logo-header.svg" alt="RustDesk - Your remote desktop">
</p>

## 環境變數

### RUSTDESK_SHARED_2FA
2FA Token

### RUSTDESK_ID_SERVER
自建伺服器

### RUSTDESK_RELAY_SERVER
自建中繼伺服器

### RUSTDESK_KEY
自建伺服器KEY

## 更動檔案說明
### libs/hbb_common/src/config.rs

本檔案更動的重點在於進一步強化安全性控制，限制使用者端可自訂的功能，並將關鍵安全相關選項鎖定為指定值，避免被竄改。  
下方這四個參數（設定 HashMap）為本次調整的主要配置項目，其用途如下：

- **OVERWRITE_SETTINGS**  
  全局覆寫設定。用來鎖定主要功能選項（如硬體解碼、剪貼簿、檔案傳輸、啟用/關閉 2FA、白名單...等），避免被使用者修改。這確保了只有預期的功能可以開啟或關閉，其餘皆以預設值強制設定。

- **OVERWRITE_LOCAL_SETTINGS**  
  用於覆寫本地端（local）設定，例如停用自動檢查更新、預設主題樣式等。防止使用者在本機調整這些選項，確保所有部署端設定一致。

- **HARD_SETTINGS**  
  更高優先權的硬性限制設定。例如只允許被連入、不允許帳號登入、禁用安裝等，屬於無法在介面調整的強制性選項，主要用於加強核心安全防護。

- **BUILTIN_SETTINGS**  
  內建預設設定，通常用來隱藏部分設定頁籤或 UI 元素，例如隱藏印表機設定分頁等。這能避免使用者誤觸或調整某些預設功能。

### src/auth_2fa.rs

本檔案負責 RustDesk 專案中的兩步驟驗證（2FA，Two-Factor Authentication）相關邏輯，包含 TOTP 產生、驗證與相關資料加解密等功能。  
其中重點調整為：  
- 在 `get_2fa` 函式內，強制回傳預先寫死於編譯時期（透過環境變數 `RUSTDESK_SHARED_2FA` 設定）的 2FA 金鑰，無視外部設定
。
### flutter/lib/desktop/pages/desktop_setting_page.dart

本檔案負責桌面端安全性與權限相關的設定 UI，提供用戶在圖形介面上一站式管理各項安全選項。  
其核心作用包含：

- 整合並顯示所有安全設定（如功能啟用/停用、2FA、防止未授權存取等），方便用戶或管理員管理系統安全。
- 對於無法在伺服器後端（如 libs/hbb_common/src/config.rs）直接強制鎖定的設定（例如：二步驟驗證 (2FA) 強制開啟/鎖定、密碼驗證方式限制等），在 UI 層進行畫面元素鎖定或遮蔽，確保端點用戶無法操作或變更這些關鍵選項。

### .github/workflows/flutter-boxful-build.yml

此檔案為 GitHub Actions 的 CI 設定檔，主要用於自動化建置 RustDesk 桌面版的 Windows 與 macOS 平台安裝檔。  
內容結構以 `.github/workflows/flutter-build.yml` 為基礎，但僅保留 Windows 和 macOS 相關的 build 流程，移除其他（如 Linux、Android 等）平台，快速獨立出單純只需部署指定桌面平台的建置腳本。

## 編譯
清除快取
```
cargo clean
```
### MAC
前置安裝
[參考文件](https://rustdesk.com/docs/en/dev/build/osx/)

* 安裝 Rust
* 安裝 vcpkg
* 安裝 Xcode Command Line Tools
* 安裝 flutter 3.24.5
```
# 可以使用 fvm 安裝
brew install fvm
fvm install 3.24.5
fvm global 3.24.5
export PATH="$HOME/fvm/default/bin:$PATH"
flutter --version
```
* 安裝 flutter_rust_bridge_codegen

***前置安裝完畢後，執行以下指令編譯***

生成 target/release/librustdesk.dylib（macOS 動態庫）
```
# 沒有 2fa 情境
cargo build --features flutter --lib --release

# 有 2fa 情境
RUSTDESK_SHARED_2FA='your_2fa_token' cargo build --features flutter --lib --release
```

建構 Flutter 版本應用程式
```
# 沒有 2fa 情境
python3 ./build.py --flutter

# 有 2fa 情境
RUSTDESK_SHARED_2FA='your_2fa_token' python3 ./build.py --flutter
```

建構完成檔案會出現在專案目錄下 `flutter/build/macos/Build/Products/Release/RustDesk.app`

***建置出來的要執行以下指令才可以在 mac 開啟***

重新簽名應用程式
```
codesign --force --deep --sign - flutter/build/macos/Build/Products/Release/RustDesk.app
```

開啟應用程式
```
open flutter/build/macos/Build/Products/Release/RustDesk.app
```

### Windows
[參考文件](https://rustdesk.com/docs/en/dev/build/windows/)