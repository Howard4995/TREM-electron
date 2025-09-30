# TREM-Electron 開發者指南 (Developer Guide)

## 快速開始 (Quick Start)

### 前置需求 (Prerequisites)
- Node.js (v14 或更高版本)
- npm 或 yarn
- Git

### 設定開發環境 (Setup Development Environment)

1. **克隆專案** (Clone the project)
```bash
git clone https://github.com/Howard4995/TREM-electron.git
cd TREM-electron/src
```

2. **安裝依賴** (Install dependencies)
```bash
npm install
```

3. **執行開發模式** (Run in development mode)
```bash
npm run dev
```

## 程式碼架構理解 (Code Architecture Understanding)

### 進程模型 (Process Model)

TREM 使用 Electron 的多進程架構：

1. **主進程 (Main Process)** - `main.js`
   - 建立和管理視窗
   - 處理系統層級操作（托盤、通知）
   - 管理 IPC 通訊

2. **渲染進程 (Renderer Process)** - `scripts/renderer.js`
   - UI 渲染和互動
   - 地圖顯示
   - 資料處理和顯示

### 關鍵程式碼流程 (Key Code Flows)

#### 1. 應用程式啟動流程 (Application Startup Flow)

```
main.js
  ├── createWindow() - 建立主視窗
  ├── 載入 index.html
  └── 初始化系統托盤

index.html
  ├── 載入樣式和 MapLibre GL
  └── 載入 renderer.js

renderer.js
  ├── 初始化地圖 (Map initialization)
  ├── 初始化 API WebSocket 連線
  ├── 設定事件監聽器
  └── 開始資料更新循環
```

#### 2. 即時震度資料更新流程 (RTS Data Update Flow)

```javascript
// 在 renderer.js 中
setInterval(async () => {
  const rts = await api.getRts(Date.now());
  renderRtsData(rts.station, map);
}, 1000);

// api.js - 從快取或伺服器獲取資料
getRts(time, force = false) {
  // 1. 檢查快取
  // 2. 如果沒有快取或強制更新，從伺服器獲取
  // 3. 更新快取
  // 4. 返回資料
}

// helpers/map.js - 渲染資料到地圖
renderRtsData(rts, map) {
  // 1. 更新每個測站的標記
  // 2. 根據模式（震度/PGA-PGV）設定顏色
  // 3. 更新最大震度
  // 4. 播放警報音效（如需要）
}
```

#### 3. 地震預警處理流程 (EEW Processing Flow)

```javascript
// WebSocket 接收到 EEW 資料
api.on(constants.Events.CwbEew, (eew) => {
  // 1. 建立 EEW 實例
  const eewInstance = new EEW({
    id: eew.id,
    origin_time: eew.eq.time,
    // ... 其他參數
  });

  // 2. 計算並渲染 S 波圓圈
  eewInstance.draw(map);

  // 3. 播放警報音效
  new Audio(`audio/Alert.wav`).play();

  // 4. 顯示 UI 通知
  // ...
});
```

#### 4. 地震報告顯示流程 (Report Display Flow)

```javascript
// 1. 定期獲取報告
const updateReports = async () => {
  const reports = await api.getReports();
  
  // 2. 過濾和排序報告
  const filteredReports = reports.filter(/* 條件 */);
  
  // 3. 建立報告列表項目
  for (const report of filteredReports) {
    const item = new ElementBuilder()
      .setClass(["report-item"])
      .addChildren(/* UI 元素 */);
    
    // 4. 新增點擊事件處理
    item.addEventListener("click", () => {
      showReportDetail(report);
    });
  }
};
```

## 重要類別和函式說明 (Important Classes and Functions)

### API Class (api.js)

```javascript
class api extends EventEmitter {
  constructor(key) {
    // 初始化 WebSocket 配置
  }

  initWebSocket() {
    // 建立 WebSocket 連線
    // 處理訊息接收
    // 發射對應事件
  }

  getRts(time, force = false) {
    // 獲取即時震度資料
    // 使用 Cache API 進行快取
  }

  getReports(fetchCwb = false) {
    // 獲取地震報告
  }
}
```

**使用方式:**
```javascript
const api = new api(apiKey);

// 監聽事件
api.on('rts', (data) => {
  // 處理 RTS 資料
});

api.on('eew-cwb', (eew) => {
  // 處理地震預警
});
```

### EEW Class (classes/eew.js)

```javascript
class EEW {
  constructor(options) {
    // 儲存地震資訊
    // 初始化圖層和標記
  }

  draw(map) {
    // 繪製 S 波圓圈
    // 繪製震央標記
    // 更新資訊顯示
  }

  getMaxIntensity() {
    // 計算預估最大震度
  }

  remove() {
    // 移除所有圖層和標記
  }
}
```

**使用情境:**
- 接收到 EEW 資料時建立實例
- 每秒更新以顯示 S 波擴散
- 取消或更新時移除舊實例

### ElementBuilder (helpers/domhelper.js)

用於快速建立 DOM 元素的工具類別：

```javascript
const element = new ElementBuilder()
  .setClass(['my-class'])
  .setContent('內容')
  .setAttribute('data-id', '123')
  .addEventListener('click', handleClick)
  .addChildren(childElement);
```

### Map Helpers (helpers/map.js)

```javascript
// 設定地圖圖層
setMapLayers(map) {
  // 新增底圖圖層
  // 新增邊界圖層
  // 設定樣式
}

// 渲染即時震度
renderRtsData(rts, map) {
  // 更新測站標記
  // 計算最大震度
  // 處理警報
}

// 渲染地震預警
renderEewData(eew, map) {
  // 建立 EEW 實例
  // 繪製圓圈和標記
}
```

## 資料結構 (Data Structures)

### RTS 資料格式 (RTS Data Format)
```javascript
{
  "Alert": false,  // 是否有警報
  "station": {
    "uuid-1": {
      "i": 2.5,      // 震度
      "pga": 15.2,   // 最大地動加速度
      "pgv": 1.2,    // 最大地動速度
      "station": "測站名稱"
    },
    // ... 更多測站
  }
}
```

### EEW 資料格式 (EEW Data Format)
```javascript
{
  "type": "eew-cwb",
  "id": "eew-id",
  "serial": 1,
  "eq": {
    "lat": 24.5,
    "lon": 121.5,
    "depth": 10,
    "mag": 5.5,
    "time": 1234567890000,
    "loc": "台灣東部海域"
  },
  "author": "eew-cwb"
}
```

### 地震報告格式 (Report Data Format)
```javascript
{
  "identifier": "report-id",
  "time": 1234567890000,
  "location": "台灣東部海域",
  "depth": 10,
  "magnitudeType": "ML",
  "magnitudeValue": 5.5,
  "epicenterLat": 24.5,
  "epicenterLon": 121.5,
  "data": [
    {
      "area": "區域名稱",
      "areaIntensity": 4,
      "stations": [
        {
          "station": "測站名稱",
          "stationIntensity": 4
        }
      ]
    }
  ]
}
```

## 常用操作模式 (Common Operation Patterns)

### 1. 新增新的資料來源

```javascript
// 1. 在 constants.js 新增事件類型
Events: {
  MyNewSource: "my-new-source",
}

// 2. 在 api.js 處理新的訊息類型
this.ws.on("message", (raw) => {
  const data = JSON.parse(raw);
  
  switch (data.type) {
    case "my-new-source":
      this.emit(constants.Events.MyNewSource, data);
      break;
  }
});

// 3. 在 renderer.js 監聽事件
api.on(constants.Events.MyNewSource, (data) => {
  // 處理資料
});
```

### 2. 新增設定選項

```javascript
// 1. 在 constants.js 新增預設值
DefaultSettings: {
  MyNewSetting: "default-value",
}

// 2. 在 views/index.html 新增 UI 控制
<input class="setting" 
       type="checkbox" 
       data-setting="MyNewSetting">

// 3. 在 controls.js 中使用
const myValue = localStorage.getItem("MyNewSetting") 
                ?? constants.DefaultSettings.MyNewSetting;
```

### 3. 新增地圖圖層

```javascript
// 在 helpers/map.js 的 setMapLayers 函式中
map.addSource('my-source', {
  type: 'geojson',
  data: myGeoJsonData
});

map.addLayer({
  id: 'my-layer',
  type: 'fill',
  source: 'my-source',
  paint: {
    'fill-color': '#088',
    'fill-opacity': 0.8
  }
});
```

## 除錯技巧 (Debugging Tips)

### 1. 開發者工具
在開發模式下，Electron 會自動開啟 DevTools：
```javascript
if (_devMode) {
  MainWindow.webContents.openDevTools();
}
```

### 2. 日誌記錄
使用 console 方法記錄資訊：
```javascript
console.debug("[API] Socket --> open");
console.log("%c[Reports] Refreshing...", "color: cornflowerblue");
console.warn("[Reports] Skipped refresh");
console.error("[Reports] Refresh failed");
```

### 3. 事件追蹤
追蹤 API 事件：
```javascript
api.on('*', (event, data) => {
  console.log(`Event: ${event}`, data);
});
```

### 4. 地圖除錯
檢查地圖狀態：
```javascript
console.log(map.getStyle());  // 獲取樣式
console.log(map.getCenter()); // 獲取中心點
console.log(map.getZoom());   // 獲取縮放級別
```

## 效能優化建議 (Performance Optimization)

### 1. 快取策略
- 使用 Cache API 儲存頻繁請求的資料
- 設定合理的快取時間

### 2. 地圖渲染優化
- 使用 `UsePreciseMath` 設定控制精確度
- 適時移除不需要的圖層和標記

### 3. 事件處理優化
- 避免在高頻事件中進行複雜計算
- 使用防抖（debounce）或節流（throttle）

## 常見問題 (Common Issues)

### Q: WebSocket 連線失敗
A: 檢查網路連線和 API 金鑰是否正確

### Q: 地圖顯示空白
A: 確認 MapLibre GL 樣式載入正確，檢查控制台錯誤訊息

### Q: 測站資料不更新
A: 檢查 RTS 資料請求是否成功，確認時間同步正確

### Q: 音效無法播放
A: 確認音效檔案路徑正確，檢查瀏覽器音效權限

## 測試建議 (Testing Recommendations)

### 手動測試檢查清單
- [ ] 地圖正常載入和互動
- [ ] 測站資料正常顯示
- [ ] EEW 警報正常運作
- [ ] 地震報告顯示正確
- [ ] 音效正常播放
- [ ] 設定可以儲存和載入
- [ ] 視圖切換正常

### 模擬資料測試
可以在 DevTools 中手動觸發事件：
```javascript
// 模擬 EEW
api.emit('eew-cwb', {
  type: "eew-cwb",
  id: "test-eew",
  // ... 測試資料
});

// 模擬 RTS 警報
api.emit('rts', {
  Alert: true,
  station: {
    // ... 測試資料
  }
});
```

## 貢獻指南 (Contribution Guidelines)

1. Fork 專案並建立新分支
2. 遵循現有的程式碼風格
3. 新增適當的註解說明
4. 測試您的變更
5. 提交 Pull Request

### 程式碼風格
- 使用 ESLint 進行程式碼檢查
- 遵循現有的命名慣例
- 保持函式簡潔，單一職責

## 相關資源 (Related Resources)

- [Electron 文件](https://www.electronjs.org/docs)
- [MapLibre GL 文件](https://maplibre.org/maplibre-gl-js-docs/api/)
- [TREM API 文件](https://exptech.com.tw/api)
- [Material Symbols](https://fonts.google.com/icons)

## 聯絡方式 (Contact)

- GitHub Issues: [提交問題](https://github.com/Howard4995/TREM-electron/issues)
- Discord: [加入社群](https://discord.gg/5dbHqV8ees)
