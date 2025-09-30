# 貢獻指南 / Contributing Guide

感謝您有興趣為 TREM (Taiwan Real-time Earthquake Monitoring) 做出貢獻！本文件將幫助您了解專案結構和開發流程。

Thank you for your interest in contributing to TREM! This document will help you understand the project structure and development workflow.

## 📋 目錄 / Table of Contents

- [專案概述](#專案概述--project-overview)
- [技術架構](#技術架構--technical-architecture)
- [專案結構](#專案結構--project-structure)
- [開發環境設置](#開發環境設置--development-setup)
- [資料流程](#資料流程--data-flow)
- [主要功能模組](#主要功能模組--main-modules)
- [程式碼規範](#程式碼規範--code-style)
- [提交貢獻](#提交貢獻--submitting-contributions)

## 專案概述 / Project Overview

TREM 是一個基於 Electron 的開源地震速報軟體，專為台灣地區設計。主要功能包括：

- 🗃️ **地震報告查看** - 互動式地圖顯示地震資訊
- ⚠️ **地震預警接收** - 即時接收強震即時警報 (EEW)
- 📊 **即時震度監測** - 顯示各地測站即時震度 (RTS)
- 🗺️ **國外地震資訊** - 支援日本、韓國、中國等地的地震資訊

TREM is an open-source earthquake early warning software built on Electron, designed for Taiwan. Main features include earthquake reports, real-time warnings (EEW), real-time seismic intensity monitoring (RTS), and international earthquake information.

## 技術架構 / Technical Architecture

### 核心技術棧 / Core Technologies

- **框架 / Framework**: Electron 27.x
- **地圖庫 / Map Library**: MapLibre GL
- **即時通訊 / Real-time Communication**: WebSocket
- **推送通知 / Push Notifications**: Firebase Cloud Messaging (FCM)
- **樣式 / Styling**: CSS (Material Design inspired)

### 系統架構圖 / System Architecture

```
┌─────────────────────────────────────────────┐
│           Electron Main Process             │
│  - Window Management (main.js)              │
│  - IPC Communication                        │
│  - Push Notification Service                │
└─────────────────────────────────────────────┘
                      │
                      ├─ IPC
                      │
┌─────────────────────────────────────────────┐
│        Electron Renderer Process            │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │  UI Layer (index.html)               │  │
│  │  - Reports Panel                     │  │
│  │  - Settings Panel                    │  │
│  │  - Map Container                     │  │
│  └──────────────────────────────────────┘  │
│                      │                      │
│  ┌──────────────────────────────────────┐  │
│  │  Application Layer (renderer.js)     │  │
│  │  - Event Handling                    │  │
│  │  - UI Updates                        │  │
│  │  - Data Processing                   │  │
│  └──────────────────────────────────────┘  │
│                      │                      │
│  ┌──────────────────────────────────────┐  │
│  │  API Layer (api.js)                  │  │
│  │  - WebSocket Connection              │  │
│  │  - HTTP Requests                     │  │
│  │  - Event Emission                    │  │
│  └──────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
                      │
                      ├─ WebSocket / HTTP
                      │
┌─────────────────────────────────────────────┐
│         External Services                   │
│  - ExpTech API (ws.exptech.com.tw)         │
│  - RTS Data                                │
│  - EEW Data                                │
│  - Earthquake Reports                      │
└─────────────────────────────────────────────┘
```

## 專案結構 / Project Structure

```
TREM-electron/
├── src/
│   ├── main.js                 # Electron 主程序入口 / Main process entry
│   ├── preload.js             # Preload 腳本 / Preload script
│   │
│   ├── views/
│   │   └── index.html         # 主視窗 HTML / Main window HTML
│   │
│   ├── scripts/               # 渲染程序腳本 / Renderer scripts
│   │   ├── renderer.js        # 渲染程序主入口 / Renderer entry
│   │   ├── api.js            # API 客戶端 / API client
│   │   ├── constants.js      # 常數定義 / Constants
│   │   ├── controls.js       # UI 控制 / UI controls
│   │   ├── factory.js        # DOM 元素工廠 / DOM factory
│   │   ├── file.js          # 檔案處理 / File handling
│   │   │
│   │   ├── classes/          # 類別定義 / Class definitions
│   │   │   ├── circle.js    # 圓形圖層類別 / Circle layer
│   │   │   ├── eew.js       # 地震預警類別 / EEW class
│   │   │   └── wave.js      # 地震波類別 / Wave class
│   │   │
│   │   └── helpers/          # 工具函數 / Helper functions
│   │       ├── audio.js     # 音訊播放 / Audio playback
│   │       ├── colors.js    # 顏色處理 / Color handling
│   │       ├── distance.js  # 距離計算 / Distance calculation
│   │       ├── domhelper.js # DOM 操作 / DOM manipulation
│   │       ├── ipc.js       # IPC 通訊 / IPC communication
│   │       ├── map.js       # 地圖操作 / Map operations
│   │       ├── pga.js       # 地動加速度 / Peak Ground Acceleration
│   │       ├── ui.js        # UI 工具 / UI utilities
│   │       └── utils.js     # 通用工具 / General utilities
│   │
│   ├── styles/               # 樣式檔案 / Stylesheets
│   │   ├── index.css        # 主樣式 / Main styles
│   │   ├── components/      # 元件樣式 / Component styles
│   │   ├── panels/          # 面板樣式 / Panel styles
│   │   ├── markers/         # 標記樣式 / Marker styles
│   │   └── theme/           # 主題樣式 / Theme styles
│   │
│   ├── assets/              # 靜態資源 / Static assets
│   │   ├── json/           # GeoJSON 資料 / GeoJSON data
│   │   └── audio/          # 音訊檔案 / Audio files
│   │
│   ├── audio/              # 警報音效 / Alert sounds
│   ├── image/              # 圖片資源 / Image assets
│   │
│   ├── Resources/          # 資源檔案 / Resource files
│   │   ├── GeoJSON/       # 地理資料 / Geographic data
│   │   └── Resources.js   # 資源載入器 / Resource loader
│   │
│   └── Utils/             # 工具類別 / Utility classes
│       └── Utils.js
│
├── LICENSE                # AGPL-3.0 授權條款 / License
├── README.md             # 專案說明 / Project README
└── CONTRIBUTING.md       # 本文件 / This file
```

## 開發環境設置 / Development Setup

### 前置需求 / Prerequisites

- Node.js >= 16.x
- npm >= 8.x
- Git

### 安裝步驟 / Installation Steps

1. **克隆儲存庫 / Clone the repository**
   ```bash
   git clone https://github.com/Howard4995/TREM-electron.git
   cd TREM-electron
   ```

2. **安裝相依套件 / Install dependencies**
   ```bash
   cd src
   npm install
   ```

3. **啟動開發模式 / Start development mode**
   ```bash
   npm run dev
   ```

4. **程式碼檢查 / Lint code**
   ```bash
   npm run lint
   ```

5. **建置應用程式 / Build application**
   ```bash
   npm run dist
   ```

### 開發模式 / Development Mode

- `npm run dev` - 啟動應用程式並開啟開發者工具
- `npm start` - 正常啟動應用程式
- `npm run lint` - 執行 ESLint 檢查

## 資料流程 / Data Flow

### 1. WebSocket 連線流程 / WebSocket Connection Flow

```javascript
// api.js
constructor(key) {
  // 初始化 WebSocket 配置
  this.wsConfig = {
    uuid: `TREM/v${constants.AppVersion} (${localStorage.uuid})`,
    function: "subscriptionService",
    value: ["trem-rts-v2", "trem-eew-v1", "report-v1", "tsunami-v1"],
    key: this.key
  };
  this.initWebSocket();
}
```

### 2. 資料接收與處理 / Data Reception and Processing

```
WebSocket Message → api.js (parse & emit) → renderer.js (handle event) → UI Update
```

### 3. 主要事件類型 / Main Event Types

- **RTS (Real-Time Seismograph)** - 即時測站震度資料
- **EEW (Earthquake Early Warning)** - 地震預警資料
- **Report** - 地震報告資料
- **NTP** - 時間同步資料

### 4. 地圖渲染流程 / Map Rendering Flow

```javascript
// 接收資料 → 處理資料 → 更新地圖圖層
renderRtsData(stations, map)     // 渲染 RTS 資料
renderEewData(eewData, map)       // 渲染 EEW 資料
renderReportData(report, map)     // 渲染報告資料
```

## 主要功能模組 / Main Modules

### API 模組 (api.js)

負責與後端服務通訊，提供以下功能：

- WebSocket 連線管理
- HTTP API 請求
- 事件發送機制
- 資料快取

主要方法：
- `initWebSocket()` - 初始化 WebSocket 連線
- `getReports(fetchCwb)` - 獲取地震報告
- `getRts(time, force)` - 獲取 RTS 資料
- `getEarthquake(time, type, force)` - 獲取地震資訊

### 地圖模組 (helpers/map.js)

處理地圖相關功能：

- `setMapLayers(map)` - 設置地圖圖層
- `setDefaultMapView(map)` - 設置預設視圖
- `renderRtsData(stations, map)` - 渲染 RTS 資料
- `renderEewData(eew, map)` - 渲染 EEW 資料
- `setAreaIntensity(id, intensity, map)` - 設置區域震度
- `clearAreaIntensity(id, map)` - 清除區域震度

### UI 模組 (helpers/ui.js)

管理使用者介面：

- `switchView(view, map)` - 切換視圖（報告、設定等）
- `resetReportViewport(transition)` - 重置報告視圖

### 類別模組 / Classes

#### EEW 類別 (classes/eew.js)
處理地震預警顯示和動畫

#### Circle 類別 (classes/circle.js)
在地圖上繪製圓形圖層（震央、波形等）

#### Wave 類別 (classes/wave.js)
處理地震波的傳播動畫

## 程式碼規範 / Code Style

### JavaScript 規範

本專案使用 ESLint 進行程式碼檢查，配置檔案位於 `src/.eslintrc.json`。

主要規範：
- 使用 2 空格縮排
- 使用單引號
- 變數命名使用 camelCase
- 類別命名使用 PascalCase
- 常數命名使用 UPPER_CASE
- 檔案末尾保留空行

### CSS 規範

本專案使用 Stylelint 進行樣式檢查，配置檔案位於 `src/.stylelintrc.json`。

主要規範：
- 使用 Material Design 設計原則
- 支援亮色/暗色主題
- 使用 CSS 變數管理顏色和尺寸

### 註解規範 / Comment Guidelines

- 使用 JSDoc 格式註解函數和類別
- 關鍵邏輯需要添加說明註解
- 註解使用中文或英文均可

範例：
```javascript
/**
 * 渲染地震預警資料
 * @param {Object} eew - 地震預警資料
 * @param {MapLibreMap} map - 地圖實例
 */
const renderEewData = (eew, map) => {
  // 實作內容
};
```

## 提交貢獻 / Submitting Contributions

### 開發流程 / Development Workflow

1. **Fork 專案**
   - 點擊右上角的 "Fork" 按鈕

2. **創建分支**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **進行修改**
   - 遵循程式碼規範
   - 添加必要的註解
   - 測試您的修改

4. **提交變更**
   ```bash
   git add .
   git commit -m "feat: 添加某某功能"
   ```

5. **推送到 GitHub**
   ```bash
   git push origin feature/your-feature-name
   ```

6. **建立 Pull Request**
   - 前往原始儲存庫
   - 點擊 "New Pull Request"
   - 詳細描述您的修改

### Commit 訊息規範

使用 Conventional Commits 格式：

- `feat:` 新功能
- `fix:` 錯誤修復
- `docs:` 文件更新
- `style:` 程式碼格式調整
- `refactor:` 程式碼重構
- `test:` 測試相關
- `chore:` 建置或輔助工具的變動

範例：
```
feat: 新增地震報告匯出功能
fix: 修復地圖縮放問題
docs: 更新 API 文件
```

## 重要資源 / Important Resources

### 外部 API 來源 / External API Sources

- **ExpTech API**: `https://data.exptech.com.tw/api/v1/`
  - `/trem/rts` - RTS 資料
  - `/eq/info` - 地震資訊
  - `/eq/report` - 地震報告

- **WebSocket**: `wss://ws.exptech.com.tw/websocket`
  - 訂閱服務：RTS, EEW, 報告, 海嘯

### 資料來源 / Data Sources

- 中央氣象局 (CWA)
- 日本防災科研 (NIED)
- 日本氣象廳 (JMA)
- 韓國氣象廳 (KMA)
- 中國福建省地震局
- 中國四川省地震局

### 相關文件 / Related Documentation

- [TREM 文件](https://hackmd.io/@n5w-HNYMQUmvhV6t1kor5g/Bkqtwduo9)
- [MapLibre GL JS](https://maplibre.org/maplibre-gl-js/docs/)
- [Electron 文件](https://www.electronjs.org/docs/latest)

## 注意事項 / Important Notes

1. **測試環境** - 請在測試環境中驗證所有修改
2. **API Key** - 開發時需要有效的 API Key
3. **資料授權** - 所有地震資料僅供研究、學術及教育用途
4. **效能考量** - 注意地圖渲染和動畫效能
5. **相容性** - 確保跨平台相容性 (Windows, macOS, Linux)

## 獲取幫助 / Getting Help

如果您有任何問題：

- 提交 [Issue](https://github.com/Howard4995/TREM-electron/issues)
- 加入 [Discord 社群](https://discord.gg/5dbHqV8ees)
- 參考現有的 [Pull Requests](https://github.com/Howard4995/TREM-electron/pulls)

## 授權條款 / License

本專案採用 AGPL-3.0 授權條款，詳見 [LICENSE](LICENSE) 檔案。

貢獻即表示您同意您的貢獻將在相同授權條款下發布。

---

感謝您的貢獻！🎉

Thank you for your contribution! 🎉
