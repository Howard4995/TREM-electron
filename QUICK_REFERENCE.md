# TREM-Electron 快速參考卡 (Quick Reference Card)

## 常用命令 (Common Commands)

```bash
# 安裝依賴 (Install dependencies)
cd src && npm install

# 開發模式 (Development mode)
npm run dev

# 正式執行 (Production run)
npm start

# 程式碼檢查 (Lint)
npm run lint

# 建置應用程式 (Build)
npm run dist
```

## 關鍵檔案位置 (Key File Locations)

| 檔案 | 用途 |
|------|------|
| `src/main.js` | Electron 主進程入口 |
| `src/views/index.html` | 主要 UI 界面 |
| `src/scripts/renderer.js` | 渲染進程主邏輯 |
| `src/scripts/api.js` | API 通訊模組 |
| `src/scripts/constants.js` | 常數定義 |
| `src/scripts/classes/eew.js` | 地震預警類別 |
| `src/scripts/helpers/map.js` | 地圖操作輔助 |
| `src/scripts/helpers/ui.js` | UI 操作輔助 |

## 常用 API (Common APIs)

### API 模組
```javascript
// 初始化 API
const api = new api(apiKey);

// 獲取即時震度資料
const rts = await api.getRts(Date.now());

// 獲取地震報告
const reports = await api.getReports();

// 監聽事件
api.on('rts', (data) => { /* ... */ });
api.on('eew-cwb', (eew) => { /* ... */ });
api.on('report', (report) => { /* ... */ });
```

### 地圖操作
```javascript
// 初始化地圖
const map = new Map({
  container: 'map',
  center: [120.5, 23.6],
  zoom: 6.75
});

// 設定圖層
setMapLayers(map);

// 渲染即時震度
renderRtsData(rts.station, map);

// 調整視圖
map.fitBounds(constants.TaiwanBounds, {
  padding: 24,
  animate: true
});
```

### EEW 處理
```javascript
// 建立 EEW 實例
const eewInstance = new EEW({
  id: eew.id,
  origin_time: eew.eq.time,
  lat: eew.eq.lat,
  lon: eew.eq.lon,
  depth: eew.eq.depth,
  mag: eew.eq.mag,
  // ...
}, map, eewList);

// 繪製到地圖
eewInstance.draw(map);

// 移除
eewInstance.remove();
```

### UI 元素建立
```javascript
// 使用 ElementBuilder
const element = new ElementBuilder()
  .setClass(['my-class'])
  .setContent('內容')
  .setAttribute('data-id', '123')
  .addEventListener('click', handleClick)
  .addChildren(childElement);
```

## 重要常數 (Important Constants)

### 震度分級
```javascript
constants.Intensities = [
  { value: 0, label: "0", text: "０級" },
  { value: 1, label: "1", text: "１級" },
  // ... (0-7級)
];
```

### 事件類型
```javascript
constants.Events = {
  Report: "report",
  TremEew: "trem-eew",
  CwbEew: "eew-cwb",
  Rts: "rts",
  Ntp: "ntp"
};
```

### 預設設定
```javascript
constants.DefaultSettings = {
  RtsMode: "i",              // 震度模式
  UseSiteEffect: "true",     // 場址效應
  MapAnimation: "true",      // 地圖動畫
  // ...
};
```

## 資料格式 (Data Formats)

### RTS 資料
```json
{
  "Alert": false,
  "station": {
    "uuid-1": {
      "i": 2.5,      // 震度
      "pga": 15.2,   // 最大地動加速度
      "pgv": 1.2     // 最大地動速度
    }
  }
}
```

### EEW 資料
```json
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
  }
}
```

### 地震報告
```json
{
  "identifier": "report-id",
  "time": 1234567890000,
  "location": "台灣東部海域",
  "depth": 10,
  "magnitudeType": "ML",
  "magnitudeValue": 5.5,
  "data": [
    {
      "area": "區域",
      "areaIntensity": 4,
      "stations": [...]
    }
  ]
}
```

## 顏色參考 (Color Reference)

### 震度顏色
| 震度 | 顏色 | RGB |
|------|------|-----|
| 0-1 | 深灰 | rgb(35, 40, 49) |
| 2 | 藍色 | rgb(86, 125, 188) |
| 3 | 綠色 | rgb(55, 168, 106) |
| 4 | 黃色 | rgb(247, 220, 111) |
| 5- | 橘色 | rgb(242, 147, 5) |
| 5+ | 深橘 | rgb(242, 100, 25) |
| 6- | 紅色 | rgb(211, 47, 47) |
| 6+ | 深紅 | rgb(158, 0, 93) |
| 7 | 紫色 | rgb(97, 0, 97) |

```javascript
// 獲取震度顏色
const color = colors.getIntensityColor(intensity);

// 獲取 PGA 顏色
const color = colors.getAccerateColor(pga, pgv);
```

## IPC 通訊 (IPC Communication)

### 主進程到渲染進程
```javascript
// main.js
MainWindow.webContents.send('channel-name', data);
```

### 渲染進程到主進程
```javascript
// renderer.js
ipcRenderer.send('channel-name', data);

// 監聽回應
ipcRenderer.on('response-channel', (event, data) => {
  // 處理資料
});
```

### 常用通道
- `win:minimize` - 最小化視窗
- `win:maximize` - 最大化視窗
- `win:close` - 關閉視窗
- `report:clear.station` - 清除報告測站
- `report:unhide.marker` - 顯示報告標記

## localStorage 鍵值 (localStorage Keys)

| 鍵 | 用途 |
|----|------|
| `uuid` | 使用者唯一識別碼 |
| `ApiKey` | API 金鑰 |
| `RtsMode` | RTS 顯示模式 (i/a) |
| `UseSiteEffect` | 場址效應開關 |
| `MapAnimation` | 地圖動畫開關 |
| `AudioVolume` | 音效音量 (0-100) |

## 音效檔案 (Audio Files)

| 檔案 | 用途 |
|------|------|
| `Alert.wav` | 警報音效 |
| `EEW.wav` | EEW 提示音 |
| `Report.wav` | 報告提示音 |
| `PGA1.wav` / `PGA2.wav` | PGA 警報 |
| `Shindo0.wav` - `Shindo2.wav` | 震度音效 |
| `Update.wav` | 更新提示音 |

```javascript
// 播放音效
new Audio(`audio/Alert.wav`).play();

// 使用輔助函式
playAudio('Alert', volume);
```

## 地圖圖層 ID (Map Layer IDs)

- `taiwan-boundary` - 台灣邊界
- `taiwan-fill` - 台灣填充
- `eew-circle-*` - EEW 圓圈
- `eew-marker-*` - EEW 標記
- `report-cross-*` - 報告十字標記
- `report-wave-*` - 報告波形圓圈

## 除錯快捷鍵 (Debug Shortcuts)

```javascript
// 開啟開發者工具
Ctrl/Cmd + Shift + I

// 重新載入
Ctrl/Cmd + R

// 強制重新載入
Ctrl/Cmd + Shift + R
```

## 常見錯誤處理 (Common Error Handling)

```javascript
// WebSocket 錯誤
ws.on('error', (error) => {
  console.error('[API] Socket error:', error);
});

// API 請求錯誤
try {
  const data = await api.getRts(time);
} catch (error) {
  console.error('[API] Request failed:', error);
}

// 地圖錯誤
map.on('error', (e) => {
  console.error('[Map] Error:', e.error);
});
```

## 效能優化提示 (Performance Tips)

1. **使用 requestAnimationFrame** 進行動畫
2. **批次更新 DOM** 減少重繪
3. **使用 Cache API** 快取資料
4. **避免在迴圈中使用 querySelector**
5. **使用事件委派** 處理大量元素

## 有用的工具函式 (Utility Functions)

```javascript
// 距離計算
const dist = Distance(lat1, lon1, lat2, lon2);

// 震度轉整數
const intValue = convertToIntensityInteger(intensity);

// 取得規模等級
const magLevel = getMagnitudeLevel(magnitude);

// 取得深度等級
const depthLevel = getDepthLevel(depth);

// 格式化時間
const timeStr = toFormattedTimeString(timestamp);
```

## 專案結構速查 (Quick Structure Reference)

```
src/
├── main.js              # 主進程
├── views/index.html     # UI
├── scripts/
│   ├── renderer.js      # 渲染邏輯
│   ├── api.js          # API 層
│   ├── constants.js    # 常數
│   ├── classes/        # 類別
│   └── helpers/        # 輔助函式
├── styles/             # 樣式
└── Resources/          # 資源
```

## 相關連結 (Quick Links)

- [完整專案文檔](PROJECT_STRUCTURE.md)
- [開發者指南](DEVELOPER_GUIDE.md)
- [架構圖表](ARCHITECTURE.md)
- [Electron 文件](https://www.electronjs.org/docs)
- [MapLibre GL 文件](https://maplibre.org/maplibre-gl-js-docs/api/)

---

**提示**: 將此文件加入書籤以便快速查閱！
