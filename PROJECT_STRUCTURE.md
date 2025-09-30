# TREM-Electron 專案結構文件 (Project Structure Documentation)

## 專案概述 (Project Overview)

TREM (Taiwan Real-time Earthquake Monitoring / 臺灣即時地震監測) 是一個基於 Electron 的開源地震速報軟體，提供即時的地震資訊、強震即時警報和各地震度資訊。

TREM is an open-source earthquake monitoring application built with Electron, providing real-time earthquake information, early warning alerts, and seismic intensity data across locations.

## 技術架構 (Technical Architecture)

### 核心技術棧 (Core Technology Stack)
- **Electron**: 桌面應用程式框架 (Desktop application framework)
- **MapLibre GL**: 地圖渲染引擎 (Map rendering engine)
- **WebSocket**: 即時資料通訊 (Real-time data communication)
- **Material Design Icons**: UI 圖示系統 (UI icon system)

### 主要依賴 (Main Dependencies)
- `maplibre-gl`: 互動式地圖顯示
- `ws`: WebSocket 客戶端
- `uuid`: 唯一識別碼生成
- `chroma-js`: 顏色處理工具
- `electron-fcm-push-receiver`: FCM 推播通知

## 目錄結構 (Directory Structure)

```
src/
├── main.js                 # Electron 主進程入口 (Main process entry)
├── preload.js             # 預載腳本 (Preload script)
├── views/                 # 視圖層 (View layer)
│   └── index.html        # 主要 UI 界面
├── scripts/              # 核心邏輯層 (Core logic layer)
│   ├── renderer.js       # 渲染進程主邏輯 (Renderer process main logic)
│   ├── api.js           # API 通訊模組 (API communication module)
│   ├── constants.js     # 常數定義 (Constants definitions)
│   ├── factory.js       # UI 元素工廠 (UI element factory)
│   ├── controls.js      # 控制器 (Controllers)
│   ├── file.js          # 檔案操作 (File operations)
│   ├── classes/         # 自訂類別 (Custom classes)
│   │   ├── eew.js      # 地震預警類別 (EEW class)
│   │   ├── wave.js     # 地震波類別 (Seismic wave class)
│   │   └── circle.js   # 圓形標記類別 (Circle marker class)
│   └── helpers/         # 輔助函式 (Helper functions)
│       ├── map.js      # 地圖操作輔助 (Map operations)
│       ├── ui.js       # UI 操作輔助 (UI operations)
│       ├── colors.js   # 顏色處理 (Color processing)
│       ├── audio.js    # 音效處理 (Audio processing)
│       ├── pga.js      # PGA 計算 (PGA calculations)
│       ├── utils.js    # 通用工具 (General utilities)
│       ├── distance.js # 距離計算 (Distance calculations)
│       └── domhelper.js # DOM 操作輔助 (DOM operations helper)
├── styles/              # 樣式層 (Style layer)
│   ├── index.css       # 主樣式
│   ├── map.css         # 地圖樣式
│   ├── nav.css         # 導航樣式
│   ├── panels.css      # 面板樣式
│   ├── components/     # 元件樣式
│   ├── markers/        # 標記樣式
│   ├── panels/         # 各面板專用樣式
│   └── theme/          # 主題樣式
├── Resources/           # 資源檔案 (Resource files)
│   ├── Resources.js    # 資源載入器
│   ├── area.json       # 區域資料
│   ├── region.json     # 地區資料
│   └── GeoJSON/        # 地理資料
├── Utils/              # 工具函式 (Utilities)
│   └── Utils.js
├── assets/             # 靜態資源 (Static assets)
│   ├── TREM.ico       # 應用程式圖示
│   └── json/          # JSON 資料檔
├── audio/              # 音效檔案 (Audio files)
└── image/              # 圖片檔案 (Image files)
```

## 核心模組說明 (Core Modules Description)

### 1. Main Process (main.js)
主進程負責：
- 建立應用程式視窗 (Create application window)
- 管理系統托盤 (Manage system tray)
- 處理進程間通訊 (Handle IPC communication)
- 管理應用程式生命週期 (Manage app lifecycle)
- 推播通知整合 (Push notification integration)

### 2. Renderer Process (scripts/renderer.js)
渲染進程主邏輯包含：
- 初始化地圖 (Initialize map)
- 處理即時震度資料 (Handle real-time seismic data)
- 管理地震報告顯示 (Manage earthquake reports)
- 處理 EEW（地震預警）事件 (Handle EEW events)
- UI 狀態管理 (UI state management)

### 3. API Module (scripts/api.js)
API 模組提供：
- WebSocket 連線管理 (WebSocket connection management)
- 即時震度資料（RTS）獲取 (Real-time seismic data retrieval)
- 地震報告資料獲取 (Earthquake report data retrieval)
- 資料快取機制 (Data caching mechanism)
- 事件發射器（EventEmitter）模式 (Event emitter pattern)

### 4. Classes (scripts/classes/)

#### EEW Class (eew.js)
地震預警物件，負責：
- 管理預警資訊顯示 (Manage warning information display)
- 計算 S 波到達時間 (Calculate S-wave arrival time)
- 渲染預警圓圈和影響範圍 (Render warning circles and affected areas)

#### Wave Class (wave.js)
地震波物件，用於：
- 視覺化地震波傳播 (Visualize seismic wave propagation)
- 動態更新波形 (Dynamically update waveforms)

#### Circle Class (circle.js)
圓形標記物件，用於：
- 在地圖上繪製圓形 (Draw circles on map)
- 顯示影響範圍 (Show affected areas)

### 5. Helpers (scripts/helpers/)

#### Map Helper (map.js)
地圖操作輔助函式：
- `setMapLayers()`: 設定地圖圖層 (Set map layers)
- `renderRtsData()`: 渲染即時震度資料 (Render RTS data)
- `renderEewData()`: 渲染地震預警資料 (Render EEW data)
- `setDefaultMapView()`: 設定預設地圖視圖 (Set default map view)

#### UI Helper (ui.js)
UI 操作輔助函式：
- `switchView()`: 切換不同視圖（報告、天氣、設定等） (Switch between views)
- 管理面板顯示狀態 (Manage panel display state)
- 處理地圖視窗調整 (Handle map viewport adjustments)

#### Colors Helper (colors.js)
顏色處理函式：
- `getIntensityColor()`: 根據震度取得顏色 (Get color by intensity)
- `getAccerateColor()`: 根據 PGA/PGV 取得顏色 (Get color by PGA/PGV)

## 資料流程 (Data Flow)

### 1. 即時震度資料流 (Real-time Seismic Data Flow)
```
WebSocket Server → API Module (api.js) 
  → Event Emission ('rts') 
  → Renderer (renderer.js) 
  → Map Helper (map.js) 
  → 更新地圖標記 (Update map markers)
```

### 2. 地震預警流程 (EEW Flow)
```
WebSocket Server → API Module 
  → Event Emission ('eew-cwb' or 'trem-eew') 
  → EEW Class Instance Creation 
  → 渲染預警圈和資訊 (Render warning circles and info)
  → 播放警報音效 (Play alert sound)
```

### 3. 地震報告流程 (Earthquake Report Flow)
```
API Request → api.getReports() 
  → Cache Check 
  → Fetch from Server 
  → 渲染報告列表 (Render report list)
  → 使用者點擊 (User click) 
  → 顯示詳細資料 (Show details)
```

## 主要功能 (Main Features)

### 1. 地震報告 (Earthquake Reports)
- 顯示最近的地震報告列表 (Display recent earthquake reports)
- 互動式地圖顯示震央位置 (Interactive map showing epicenter)
- 各地震度資訊 (Regional intensity information)
- 地震波到達時間圈 (Seismic wave arrival time circles)

### 2. 地震預警 (EEW - Earthquake Early Warning)
- 支援多來源預警 (Multiple source support):
  - 中央氣象局 (CWB)
  - TREM 自有預警系統
  - 國際預警系統（日本、韓國等）
- 即時 S 波到達時間計算 (Real-time S-wave arrival calculation)
- 視覺化預警範圍 (Visual warning area)
- 多層級警報音效 (Multi-level alert sounds)

### 3. 即時震度監測 (Real-time Seismic Monitoring)
- 全台測站即時震度顯示 (Real-time intensity display for all stations)
- 支援兩種顯示模式 (Two display modes):
  - 震度模式 (Intensity mode)
  - PGA/PGV 模式 (Acceleration/Velocity mode)
- 測站資料快取 (Station data caching)

### 4. 其他功能 (Other Features)
- 天氣資訊 (Weather information)
- 溫度資訊 (Temperature information)
- 空氣品質指標 (AQI - Air Quality Index)
- 設定管理 (Settings management)

## 設定系統 (Settings System)

設定儲存於 `localStorage`，主要設定項目包括：

- `ApiKey`: API 金鑰
- `RtsMode`: 即時震度顯示模式 (i: 震度, a: PGA/PGV)
- `UseSiteEffect`: 啟用場址效應
- `UseNewDecayFormula`: 使用新衰減公式
- `HideStationEEW`: 隱藏測站（EEW 模式）
- `HideStationReport`: 隱藏測站（報告模式）
- `MapAnimation`: 地圖動畫
- `ReportShowCWB`: 顯示中央氣象局報告
- `ReportShowTYA`: 顯示其他來源報告

詳細設定請參考 `constants.js` 中的 `DefaultSettings` 物件。

## 事件系統 (Event System)

應用程式使用事件驅動架構，主要事件包括：

- `rts`: 即時震度資料更新
- `ntp`: 時間同步
- `report`: 地震報告
- `eew-cwb`: 中央氣象局預警
- `trem-eew`: TREM 預警
- `eew-nied`: 日本防災科研預警
- `eew-jma`: 日本氣象廳預警
- `eew-kma`: 韓國氣象廳預警

## 開發指南 (Development Guide)

### 安裝依賴 (Install Dependencies)
```bash
cd src
npm install
```

### 執行開發模式 (Run Development Mode)
```bash
npm run dev
```

### 執行程式 (Run Application)
```bash
npm start
```

### 程式碼檢查 (Lint Code)
```bash
npm run lint
```

### 建置應用程式 (Build Application)
```bash
npm run dist
```

## API 端點 (API Endpoints)

- RTS (即時震度): `https://data.exptech.com.tw/api/v1/trem/rts`
- 地震資訊: `https://data.exptech.com.tw/api/v1/eq/info`
- 地震報告: `https://data.exptech.com.tw/api/v1/eq/report`
- WebSocket: `wss://ws.exptech.com.tw/websocket`

## 震度分級 (Intensity Scale)

系統使用以下震度分級：
- 0: 0級（無感）
- 1: 1級（微震）
- 2: 2級（輕震）
- 3: 3級（弱震）
- 4: 4級（中震）
- 5-: 5弱（強震）
- 5+: 5強（強震）
- 6-: 6弱（烈震）
- 6+: 6強（烈震）
- 7: 7級（劇震）

## 注意事項 (Important Notes)

1. 此專案已被標記為棄用（deprecated），正在進行全面重寫
2. 即時測站資訊僅供參考，實際以中央氣象局為主
3. 此軟體僅供研究、學術及教育用途
4. API 金鑰需要向 ExpTech 申請

## 相關連結 (Related Links)

- 原始專案: [ExpTechTW/TREM](https://github.com/ExpTechTW/TREM)
- 新版本 (Tauri): [ExpTechTW/TREM-tauri](https://github.com/ExpTechTW/TREM-tauri)
- 輕量版: [ExpTechTW/TREM-Lite](https://github.com/ExpTechTW/TREM-Lite)
- 官方網站: [https://exptech.com.tw/](https://exptech.com.tw/)

## 授權 (License)

本專案採用 AGPL-3.0 授權
