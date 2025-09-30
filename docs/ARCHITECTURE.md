# TREM 架構文件 / Architecture Documentation

本文件詳細說明 TREM (Taiwan Real-time Earthquake Monitoring) 的技術架構、設計理念和實作細節。

This document provides detailed information about TREM's technical architecture, design principles, and implementation details.

## 目錄 / Table of Contents

1. [系統概述](#系統概述--system-overview)
2. [技術選型](#技術選型--technology-stack)
3. [核心架構](#核心架構--core-architecture)
4. [資料模型](#資料模型--data-models)
5. [模組說明](#模組說明--module-description)
6. [效能優化](#效能優化--performance-optimization)

## 系統概述 / System Overview

### 設計目標 / Design Goals

- **即時性 / Real-time**: 毫秒級的資料更新和顯示
- **可靠性 / Reliability**: 穩定的 WebSocket 連線和錯誤恢復機制
- **跨平台 / Cross-platform**: 支援 Windows、macOS、Linux
- **離線支援 / Offline Support**: 使用 Cache API 儲存歷史資料
- **效能優化 / Performance**: 高效的地圖渲染和動畫處理

### 應用程式類型 / Application Type

TREM 是一個桌面應用程式，採用 Electron 架構：
- **Main Process**: 管理視窗、系統整合、IPC 通訊
- **Renderer Process**: 處理 UI、資料可視化、使用者互動

## 技術選型 / Technology Stack

### 核心框架 / Core Framework

| 技術 | 版本 | 用途 |
|------|------|------|
| Electron | 27.x | 桌面應用程式框架 |
| Node.js | >= 16.x | JavaScript 執行環境 |
| MapLibre GL | 3.5.x | 地圖渲染引擎 |

### 主要依賴 / Main Dependencies

```json
{
  "maplibre-gl": "^3.5.2",           // 地圖渲染
  "ws": "^8.14.2",                    // WebSocket 客戶端
  "uuid": "^9.0.1",                   // UUID 生成
  "chroma-js": "^2.4.2",              // 顏色處理
  "electron-fcm-push-receiver": "^2.1.7"  // FCM 推送通知
}
```

### 開發工具 / Development Tools

- **ESLint**: JavaScript 程式碼檢查
- **Stylelint**: CSS 樣式檢查
- **Electron Builder**: 應用程式打包

## 核心架構 / Core Architecture

### 1. 程序模型 / Process Model

```
┌──────────────────────────────────────┐
│         Main Process                 │
│  ┌────────────────────────────────┐  │
│  │  Window Management             │  │
│  │  - BrowserWindow               │  │
│  │  - Window Controls             │  │
│  │  - Login Items                 │  │
│  └────────────────────────────────┘  │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  IPC Main                      │  │
│  │  - win:minimize                │  │
│  │  - win:maximize                │  │
│  │  - win:screenshot              │  │
│  └────────────────────────────────┘  │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  Push Notification Service     │  │
│  │  - FCM Setup                   │  │
│  │  - Notification Handling       │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
              │
              │ IPC Channel
              ↓
┌──────────────────────────────────────┐
│       Renderer Process               │
│  ┌────────────────────────────────┐  │
│  │  Application Core              │  │
│  │  - renderer.js (Entry)         │  │
│  │  - Map Initialization          │  │
│  │  - Event Loop                  │  │
│  └────────────────────────────────┘  │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  API Client                    │  │
│  │  - WebSocket Connection        │  │
│  │  - HTTP Requests               │  │
│  │  - Event Emitter               │  │
│  └────────────────────────────────┘  │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  Map Engine                    │  │
│  │  - MapLibre GL                 │  │
│  │  - Layers Management           │  │
│  │  - Markers & Popups            │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

### 2. 資料流架構 / Data Flow Architecture

```
External API
    │
    ↓
┌─────────────────┐
│  WebSocket      │ ← 訂閱 RTS, EEW, Report
│  Connection     │
└─────────────────┘
    │
    ↓
┌─────────────────┐
│  api.js         │
│  EventEmitter   │ ← emit('rts', data)
└─────────────────┘   emit('eew', data)
    │                 emit('report', data)
    ↓
┌─────────────────┐
│  renderer.js    │
│  Event Handler  │ ← on('rts', handler)
└─────────────────┘   on('eew', handler)
    │                 on('report', handler)
    ↓
┌─────────────────┐
│  Map Helpers    │
│  - renderRts    │ ← 處理資料並更新地圖
│  - renderEew    │
│  - renderReport │
└─────────────────┘
    │
    ↓
┌─────────────────┐
│  MapLibre GL    │ ← 渲染到螢幕
│  Map Instance   │
└─────────────────┘
```

### 3. 事件驅動架構 / Event-Driven Architecture

TREM 採用事件驅動架構，主要事件流如下：

```javascript
// 1. WebSocket 接收訊息
ws.on('message', (raw) => {
  const data = JSON.parse(raw);
  // 2. 發送對應事件
  this.emit(data.type, data);
});

// 3. 監聽事件並處理
api.on('rts', (data) => renderRtsData(data, map));
api.on('eew', (data) => renderEewData(data, map));
api.on('report', (data) => handleReport(data));
```

## 資料模型 / Data Models

### 1. RTS (Real-Time Seismograph) 資料

```javascript
{
  station: [
    {
      id: "string",           // 測站 ID
      lat: number,            // 緯度
      lon: number,            // 經度
      intensity: number,      // 震度 (0-9)
      pga: number,           // 地動加速度
      pgv: number,           // 地動速度
      timestamp: number      // 時間戳記
    }
  ]
}
```

### 2. EEW (Earthquake Early Warning) 資料

```javascript
{
  id: "string",              // 地震 ID
  number: number,            // 報號
  type: "string",            // 類型 (eew-cwb, trem-eew, etc.)
  epicenterLat: number,      // 震央緯度
  epicenterLon: number,      // 震央經度
  depth: number,             // 深度 (km)
  magnitude: number,         // 規模
  location: "string",        // 地點描述
  originTime: number,        // 發震時刻
  model: "string",           // 模型 (EEW, PLUM, etc.)
  earthquakeNo: number       // 地震編號
}
```

### 3. Report (地震報告) 資料

```javascript
{
  id: "string",              // 報告 ID
  earthquakeNo: number,      // 地震編號
  reportType: "string",      // 報告類型
  reportImageURI: "string",  // 報告圖片 URL
  epicenterLat: number,      // 震央緯度
  epicenterLon: number,      // 震央經度
  depth: number,             // 深度 (km)
  magnitude: number,         // 規模
  location: "string",        // 地點描述
  originTime: number,        // 發震時刻
  reportTime: number,        // 報告時間
  web: "string",             // 網頁連結
  area: [                    // 各地震度
    {
      areaName: "string",
      areaIntensity: number,
      stations: [...]
    }
  ]
}
```

## 模組說明 / Module Description

### 1. API 模組 (api.js)

#### 職責 / Responsibilities
- 管理 WebSocket 連線
- 處理 HTTP API 請求
- 實作事件發送機制
- 提供資料快取功能

#### 核心方法 / Core Methods

```javascript
class api extends EventEmitter {
  constructor(key) {
    // 初始化 API key 和 WebSocket 配置
  }

  initWebSocket() {
    // 建立 WebSocket 連線
    // 處理連線、斷線、重連邏輯
  }

  async getReports(fetchCwb = false) {
    // 獲取地震報告列表
    // 支援快取機制
  }

  getRts(time, force = false) {
    // 獲取 RTS 資料
    // 支援快取機制
  }

  getEarthquake(time, type = "all", force = false) {
    // 獲取特定地震資訊
    // 支援快取機制
  }
}
```

#### 快取策略 / Caching Strategy

使用瀏覽器 Cache API 儲存 HTTP 請求結果：

```javascript
caches.open('earthquake').then((cache) => {
  cache.match(url).then((response) => {
    if (!force && response != undefined) {
      // 使用快取資料
    } else {
      // 發送新請求並更新快取
    }
  });
});
```

### 2. 地圖模組 (helpers/map.js)

#### 職責 / Responsibilities
- 初始化地圖圖層
- 處理地圖事件
- 渲染即時資料
- 管理標記和圖形

#### 圖層架構 / Layer Architecture

```javascript
// 地圖圖層由下到上：
1. 縣市邊界圖層 (county)
2. 鄉鎮邊界圖層 (town)  
3. 區域震度圖層 (area)
4. RTS 標記圖層 (markers)
5. EEW 圓形圖層 (circles)
6. 震央標記 (epicenter marker)
```

#### 關鍵功能 / Key Functions

```javascript
// 設置地圖圖層
const setMapLayers = (map) => {
  map.addSource('tw_county', { ... });
  map.addLayer({ id: 'county', ... });
  // ...
};

// 渲染 RTS 資料
const renderRtsData = (stations, map) => {
  stations.forEach(station => {
    // 更新或新增標記
  });
};

// 渲染 EEW 資料
const renderEewData = (eew, map) => {
  if (eew.id in eewList) {
    eewList[eew.id].update(eew);
  } else {
    eewList[eew.id] = new EEW(eew, map);
  }
};
```

### 3. UI 模組 (helpers/ui.js)

#### 職責 / Responsibilities
- 管理視圖切換
- 處理面板顯示
- 控制標記可見性

#### 視圖系統 / View System

```javascript
const views = {
  Reports: "reports",      // 地震報告列表
  Report: "report",        // 單一報告詳細資訊
  Forecast: "forecast",    // 預報（未實作）
  Temperature: "temperature", // 溫度（未實作）
  AQI: "aqi",             // 空氣品質（未實作）
  Settings: "settings"     // 設定
};
```

### 4. 類別模組 / Classes

#### EEW 類別 (classes/eew.js)

處理地震預警的顯示和動畫：

```javascript
class EEW {
  constructor(data, map, eewList) {
    this.data = data;
    this.map = map;
    // 建立震央標記
    // 建立 S 波圓圈
    // 建立 P 波圓圈
    // 開始動畫
  }

  update(data, eewList) {
    // 更新資料
    // 更新圓圈半徑
  }

  remove() {
    // 移除標記和圓圈
  }
}
```

#### Circle 類別 (classes/circle.js)

在地圖上繪製圓形圖層：

```javascript
class Circle {
  constructor(map, options) {
    this.id = options.id;
    this.center = options.center;
    this.radius = options.radius;
    // 建立 GeoJSON 圓形
    // 新增到地圖
  }

  update(radius) {
    // 更新圓形半徑
  }

  remove() {
    // 從地圖移除
  }
}
```

## 效能優化 / Performance Optimization

### 1. 地圖渲染優化

- **圖層合併**: 合併相似圖層減少渲染次數
- **Feature State**: 使用 MapLibre 的 Feature State 避免重新繪製整個圖層
- **動畫節流**: 限制動畫更新頻率

```javascript
// 使用 Feature State 更新震度而非重繪圖層
map.setFeatureState({
  source: 'tw_area',
  id: areaId
}, { intensity: value });
```

### 2. 記憶體管理

- **物件池化**: 重用 DOM 元素和地圖物件
- **及時清理**: 移除不需要的標記和圖層
- **快取控制**: 限制快取大小

```javascript
// 清理舊的 RTS 標記
for (const [id, marker] of rtsMarkers.entries()) {
  if (!newStationIds.has(id)) {
    marker.remove();
    rtsMarkers.delete(id);
  }
}
```

### 3. WebSocket 優化

- **自動重連**: 斷線後自動重新連線
- **心跳機制**: 保持連線活躍
- **批量處理**: 合併處理多個訊息

```javascript
this.ws.once('close', () => {
  this.ws.removeAllListeners();
  delete this.ws;
  this.initWebSocket(); // 自動重連
});
```

### 4. 資料快取策略

- **HTTP 快取**: 使用 Cache API 儲存 API 回應
- **本地儲存**: 使用 localStorage 儲存設定
- **過期策略**: 實作快取過期機制

## 安全性考量 / Security Considerations

### 1. API Key 保護

- API Key 儲存在 localStorage
- 不在程式碼中硬編碼
- 使用者可自行設定

### 2. 資料驗證

- 驗證 WebSocket 訊息格式
- 檢查 API 回應狀態
- 過濾無效資料

### 3. XSS 防護

- 避免使用 innerHTML 插入未驗證內容
- 使用 DOM API 建立元素
- 清理使用者輸入

## 開發建議 / Development Recommendations

### 1. 新增功能

1. 在 `constants.js` 定義相關常數
2. 在 `api.js` 實作 API 方法（如需要）
3. 在 `helpers/` 建立處理邏輯
4. 在 `renderer.js` 整合功能
5. 更新 UI 和樣式

### 2. 除錯技巧

- 使用 `console.debug()` 輸出除錯訊息
- 檢查 DevTools 的 Network 標籤
- 監控 WebSocket 訊息
- 使用 Electron DevTools

### 3. 測試建議

- 測試 WebSocket 斷線重連
- 驗證快取機制
- 檢查不同震度的顯示
- 測試跨平台相容性

## 相關資源 / Related Resources

- [MapLibre GL JS 文件](https://maplibre.org/maplibre-gl-js/docs/)
- [Electron 文件](https://www.electronjs.org/docs/latest)
- [ExpTech API 文件](https://exptech.com.tw/)

---

最後更新：2024
