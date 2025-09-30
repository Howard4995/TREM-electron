# TREM-Electron 程式碼架構圖 (Code Architecture Diagram)

## 整體架構圖 (Overall Architecture)

```
┌─────────────────────────────────────────────────────────────────┐
│                        TREM Application                          │
│                                                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    Main Process (main.js)                  │  │
│  │                                                             │  │
│  │  - Window Management                                        │  │
│  │  - System Tray                                              │  │
│  │  - IPC Communication                                        │  │
│  │  - Push Notifications (FCM)                                 │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                    │
│                              │ IPC                                │
│                              ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                 Renderer Process (renderer.js)             │  │
│  │                                                             │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐  │  │
│  │  │   UI Layer  │  │  Logic Layer │  │   Data Layer     │  │  │
│  │  │             │  │              │  │                  │  │  │
│  │  │ - index.html│──│- renderer.js │──│- API Module     │  │  │
│  │  │ - Styles    │  │- Controls    │  │- WebSocket      │  │  │
│  │  │ - Map UI    │  │- Factory     │  │- Cache (IndexDB)│  │  │
│  │  └─────────────┘  └──────────────┘  └──────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ WebSocket/HTTPS
                              ▼
                    ┌──────────────────────┐
                    │   ExpTech Servers    │
                    │                      │
                    │ - WebSocket Server   │
                    │ - RTS API           │
                    │ - Report API        │
                    │ - EEW Service       │
                    └──────────────────────┘
```

## 資料流架構 (Data Flow Architecture)

```
External Sources                 Application                    Display
─────────────────               ──────────────                ────────────

┌─────────────┐
│ WebSocket   │──┐
│  Server     │  │
└─────────────┘  │
                 │    ┌──────────────┐    ┌──────────────┐
┌─────────────┐  ├───→│  API Module  │───→│   Renderer   │
│   RTS API   │──┤    │              │    │              │
└─────────────┘  │    │ - Parse data │    │ - Update UI  │
                 │    │ - Cache data │    │ - Render map │
┌─────────────┐  │    │ - Emit event │    │ - Play sound │
│ Report API  │──┤    └──────────────┘    └──────────────┘
└─────────────┘  │            │                    │
                 │            │                    │
┌─────────────┐  │            ▼                    ▼
│ FCM Push    │──┘    ┌──────────────┐    ┌──────────────┐
└─────────────┘       │  Event Bus   │    │   Map View   │
                      │              │    │              │
                      │ - rts        │    │ - Markers    │
                      │ - eew        │    │ - Circles    │
                      │ - report     │    │ - Layers     │
                      │ - ntp        │    └──────────────┘
                      └──────────────┘
```

## 模組依賴關係 (Module Dependencies)

```
renderer.js (主要入口 / Main Entry)
    │
    ├── api.js (資料層 / Data Layer)
    │   ├── constants.js (常數定義)
    │   └── WebSocket (外部連接)
    │
    ├── helpers/ (輔助函式 / Helper Functions)
    │   ├── map.js
    │   │   ├── colors.js
    │   │   └── classes/
    │   │       ├── eew.js
    │   │       ├── wave.js
    │   │       └── circle.js
    │   │
    │   ├── ui.js
    │   │   └── domhelper.js
    │   │
    │   ├── audio.js
    │   ├── pga.js
    │   ├── utils.js
    │   └── distance.js
    │
    ├── factory.js (UI 元素工廠)
    │   ├── domhelper.js
    │   └── Resources/Resources.js
    │
    └── controls.js (控制器 / Controllers)
        └── file.js
```

## 類別關係圖 (Class Diagram)

```
┌─────────────────────────┐
│   EventEmitter          │
│   (Node.js built-in)    │
└───────────┬─────────────┘
            │
            │ extends
            ▼
┌─────────────────────────┐
│      API Class          │
│                         │
│ Properties:             │
│ - key                   │
│ - ws (WebSocket)        │
│ - wsConfig              │
│                         │
│ Methods:                │
│ + initWebSocket()       │
│ + getRts()              │
│ + getReports()          │
│ + getEarthquake()       │
│ + requestReplay()       │
└─────────────────────────┘


┌─────────────────────────┐
│      EEW Class          │
│                         │
│ Properties:             │
│ - id                    │
│ - serial                │
│ - origin_time           │
│ - eq (震央資訊)         │
│ - circles (圓圈)        │
│ - markers (標記)        │
│                         │
│ Methods:                │
│ + draw(map)             │
│ + getMaxIntensity()     │
│ + remove()              │
└─────────────────────────┘


┌─────────────────────────┐
│     Wave Class          │
│                         │
│ Properties:             │
│ - center (中心點)       │
│ - radius (半徑)         │
│ - circles (圓圈物件)    │
│                         │
│ Methods:                │
│ + update(time)          │
│ + remove()              │
└─────────────────────────┘


┌─────────────────────────┐
│    Circle Class         │
│                         │
│ Properties:             │
│ - map                   │
│ - id                    │
│ - center                │
│ - radius                │
│ - source                │
│ - layer                 │
│                         │
│ Methods:                │
│ + updateRadius(radius)  │
│ + remove()              │
└─────────────────────────┘


┌─────────────────────────┐
│  ElementBuilder Class   │
│                         │
│ Properties:             │
│ - element (DOM)         │
│                         │
│ Methods:                │
│ + setClass(classes)     │
│ + setContent(content)   │
│ + setAttribute(k, v)    │
│ + addEventListener()    │
│ + addChildren(child)    │
└─────────────────────────┘
```

## 事件流程圖 (Event Flow Diagram)

```
WebSocket Message Received
         │
         ▼
┌──────────────────┐
│  Parse Message   │
│  (api.js)        │
└────────┬─────────┘
         │
         ▼
    ┌────────────┐
    │Event Type? │
    └─┬────┬───┬─┘
      │    │   │
  ┌───▼┐ ┌─▼─┐ ┌▼───────┐
  │RTS │ │EEW│ │Report  │
  └┬───┘ └─┬─┘ └┬───────┘
   │       │     │
   │       │     │ emit event
   ▼       ▼     ▼
┌────────────────────────┐
│   Event Listeners      │
│   (renderer.js)        │
└──┬──────────┬────────┬─┘
   │          │        │
   ▼          ▼        ▼
┌──────┐  ┌──────┐  ┌──────────┐
│Update│  │Show  │  │Display   │
│Map   │  │Alert │  │Report    │
│Markers  │UI    │  │Details   │
└──────┘  └──────┘  └──────────┘
```

## 地圖渲染流程 (Map Rendering Flow)

```
Initialize Map (MapLibre GL)
         │
         ▼
┌──────────────────────┐
│  setMapLayers()      │
│  - Base layer        │
│  - Boundary layer    │
│  - Custom styles     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  Add Station Markers │
│  (for each station)  │
└──────────┬───────────┘
           │
           ▼
     ┌─────────────┐
     │RTS Update?  │◄───── Every 1 second
     └──────┬──────┘
            │
            ▼
┌──────────────────────┐
│ renderRtsData()      │
│ - Update marker color│
│ - Update intensity   │
│ - Check alert        │
└──────────┬───────────┘
           │
      ┌────▼─────┐
      │Alert?    │
      └─┬────────┘
        │ Yes
        ▼
┌──────────────────────┐
│ Play Alert Sound     │
│ Highlight Markers    │
└──────────────────────┘
```

## 設定系統流程 (Settings System Flow)

```
User Changes Setting
         │
         ▼
┌──────────────────────┐
│  Update localStorage │
│  (controls.js)       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  Trigger Change      │
│  Event               │
└──────────┬───────────┘
           │
      ┌────▼─────────────┐
      │What Setting?     │
      └─┬──────┬────────┬┘
        │      │        │
    ┌───▼┐  ┌──▼──┐  ┌─▼────┐
    │RTS │  │Map  │  │Audio │
    │Mode│  │Anim │  │Vol   │
    └─┬──┘  └──┬──┘  └─┬────┘
      │        │        │
      ▼        ▼        ▼
  ┌────────────────────────┐
  │  Apply Changes         │
  │  - Update display      │
  │  - Refresh map         │
  │  - Adjust volume       │
  └────────────────────────┘
```

## 快取策略 (Caching Strategy)

```
Data Request
     │
     ▼
┌─────────────┐
│Check Cache? │
└──────┬──────┘
       │
   ┌───▼─────────┐
   │Cache Exists?│
   └─┬─────────┬─┘
     │         │
    Yes        No
     │         │
     ▼         ▼
┌─────────┐ ┌──────────────┐
│Return   │ │Fetch from    │
│Cached   │ │Server        │
│Data     │ └──────┬───────┘
└─────────┘        │
                   ▼
              ┌──────────────┐
              │Update Cache  │
              │(Cache API)   │
              └──────┬───────┘
                     │
                     ▼
              ┌──────────────┐
              │Return Data   │
              └──────────────┘
```

## UI 視圖切換流程 (View Switching Flow)

```
User Clicks Nav Button
         │
         ▼
┌──────────────────────┐
│  switchView()        │
│  (helpers/ui.js)     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  Hide All Panels     │
└──────────┬───────────┘
           │
           ▼
    ┌──────────────┐
    │Which View?   │
    └─┬──────┬───┬─┘
      │      │   │
   ┌──▼┐  ┌─▼─┐ ┌▼────┐
   │Rep│  │Set│ │Fore│
   │ort│  │tng│ │cast│
   └─┬─┘  └─┬─┘ └┬────┘
     │      │     │
     ▼      ▼     ▼
┌──────────────────────┐
│  Show Selected Panel │
│  - Activate button   │
│  - Adjust map view   │
│  - Load data         │
└──────────────────────┘
```

## 地震預警處理詳細流程 (Detailed EEW Processing)

```
EEW Message Received
         │
         ▼
┌─────────────────────┐
│Parse EEW Data       │
│- ID, Serial         │
│- Epicenter (lat/lon)│
│- Magnitude, Depth   │
│- Origin Time        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│Check Existing EEW   │
│Same ID?             │
└─┬─────────────────┬─┘
  │                 │
 Yes                No
  │                 │
  ▼                 ▼
┌──────────┐    ┌──────────────┐
│Update    │    │Create New    │
│Existing  │    │EEW Instance  │
└────┬─────┘    └──────┬───────┘
     │                 │
     └────────┬────────┘
              │
              ▼
┌─────────────────────┐
│Calculate S-wave     │
│Arrival Time & Radius│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│Draw on Map          │
│- Epicenter Marker   │
│- S-wave Circle      │
│- Intensity Zones    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│Update UI            │
│- Show Alert Banner  │
│- Display Info       │
│- Play Alert Sound   │
└─────────────────────┘
```

## 重要演算法 (Key Algorithms)

### 1. 震度顏色映射 (Intensity to Color)
```
Intensity Value → Color Mapping
    0     → rgb(35, 40, 49)     (深灰)
    1     → rgb(35, 40, 49)     (深灰)
    2     → rgb(86, 125, 188)   (藍色)
    3     → rgb(55, 168, 106)   (綠色)
    4     → rgb(247, 220, 111)  (黃色)
    5-    → rgb(242, 147, 5)    (橘色)
    5+    → rgb(242, 100, 25)   (深橘)
    6-    → rgb(211, 47, 47)    (紅色)
    6+    → rgb(158, 0, 93)     (深紅)
    7     → rgb(97, 0, 97)      (紫色)
```

### 2. S 波半徑計算 (S-wave Radius Calculation)
```
時間差 = 當前時間 - 地震發生時間
S 波速度 = 3.5 km/s (預設)
距離 = sqrt((測站經度 - 震央經度)² + (測站緯度 - 震央緯度)²)
S 波半徑 = 時間差 × S 波速度
```

### 3. 預估震度計算 (Estimated Intensity)
```
基於規模和距離的震度預估:
Intensity = f(Magnitude, Depth, Distance, Site_Effect)

其中:
- Magnitude: 地震規模
- Depth: 震源深度
- Distance: 測站距離
- Site_Effect: 場址效應修正
```

## 效能關鍵點 (Performance Critical Points)

1. **地圖標記更新** (每秒執行)
   - 使用 `requestAnimationFrame` 優化渲染
   - 批次更新減少重繪

2. **WebSocket 訊息處理** (高頻)
   - 使用事件驅動避免阻塞
   - 非同步處理資料

3. **快取機制** (減少網路請求)
   - Cache API 儲存 RTS 資料
   - IndexedDB 儲存報告資料

4. **DOM 操作** (減少重排)
   - 使用 DocumentFragment
   - 批次更新樣式

## 安全考量 (Security Considerations)

```
┌──────────────────────────────┐
│    Security Measures         │
├──────────────────────────────┤
│                              │
│ 1. API Key Storage           │
│    ├─ localStorage (client)  │
│    └─ Not exposed to logs    │
│                              │
│ 2. WebSocket Communication   │
│    ├─ WSS (Encrypted)        │
│    └─ Origin validation      │
│                              │
│ 3. Data Validation           │
│    ├─ Type checking          │
│    └─ Range validation       │
│                              │
│ 4. Content Security          │
│    ├─ No eval()              │
│    └─ Sanitize user input    │
│                              │
└──────────────────────────────┘
```
