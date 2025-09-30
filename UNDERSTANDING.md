# 專案理解總結 (Project Understanding Summary)

## 文檔概覽 (Documentation Overview)

本專案已建立完整的文檔系統，幫助開發者快速理解 TREM-Electron 的架構和運作方式。

### 📚 文檔指南 (Documentation Guide)

#### 新手入門 (For Beginners)
1. 先閱讀 [README.md](README.md) 了解專案基本資訊
2. 查看 [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) 理解專案結構
3. 參考 [QUICK_REFERENCE.md](QUICK_REFERENCE.md) 學習常用操作

#### 深入開發 (For Development)
1. 閱讀 [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md) 設定開發環境
2. 研究 [ARCHITECTURE.md](ARCHITECTURE.md) 了解系統架構
3. 使用 [QUICK_REFERENCE.md](QUICK_REFERENCE.md) 作為速查手冊

## TREM-Electron 核心概念 (Core Concepts)

### 🎯 專案目標
TREM (Taiwan Real-time Earthquake Monitoring / 臺灣即時地震監測) 是一個開源的地震監測應用程式，提供：
- 即時地震資訊
- 地震預警 (EEW)
- 各地震度監測
- 地震報告檢視

### 🏗️ 技術架構

```
Electron 應用程式
├── 主進程 (Main Process)
│   └── 視窗管理、系統整合
│
└── 渲染進程 (Renderer Process)
    ├── UI 層 (View Layer)
    ├── 邏輯層 (Logic Layer)
    └── 資料層 (Data Layer)
        ├── WebSocket 連線
        └── REST API 請求
```

### 🔄 資料流程

1. **即時震度資料 (RTS)**
   - 每秒從 API 獲取資料
   - 更新地圖上的測站標記
   - 根據震度值顯示對應顏色

2. **地震預警 (EEW)**
   - WebSocket 即時接收預警
   - 計算 S 波到達時間
   - 繪製影響範圍圓圈
   - 播放警報音效

3. **地震報告 (Report)**
   - 定期獲取最新報告
   - 顯示報告列表
   - 提供詳細資訊檢視

### 📦 核心模組

| 模組 | 檔案 | 功能 |
|------|------|------|
| API 模組 | `api.js` | WebSocket 連線、資料獲取、事件發射 |
| 地圖模組 | `helpers/map.js` | 地圖渲染、圖層管理、標記更新 |
| EEW 類別 | `classes/eew.js` | 預警資訊處理、圓圈繪製 |
| UI 模組 | `helpers/ui.js` | 視圖切換、面板管理 |
| 常數定義 | `constants.js` | 設定值、事件類型、API 端點 |

## 🚀 快速開始 (Quick Start)

### 安裝與執行
```bash
# 1. 克隆專案
git clone https://github.com/Howard4995/TREM-electron.git

# 2. 安裝依賴
cd TREM-electron/src
npm install

# 3. 執行開發模式
npm run dev
```

### 主要設定項目
- **API Key**: 在設定頁面輸入 ExpTech API 金鑰
- **RTS 模式**: 選擇震度 (i) 或 PGA/PGV (a) 顯示模式
- **音效音量**: 調整警報音效大小
- **地圖動畫**: 開啟/關閉地圖過渡動畫

## 📊 關鍵資料格式 (Key Data Formats)

### 即時震度資料 (RTS)
```javascript
{
  Alert: boolean,           // 是否警報
  station: {
    "uuid": {
      i: number,           // 震度 (0-9)
      pga: number,         // 最大地動加速度
      pgv: number          // 最大地動速度
    }
  }
}
```

### 地震預警 (EEW)
```javascript
{
  type: "eew-cwb",          // 預警來源
  id: string,               // 預警 ID
  serial: number,           // 報數
  eq: {
    lat: number,            // 緯度
    lon: number,            // 經度
    depth: number,          // 深度 (km)
    mag: number,            // 規模
    time: number,           // 發生時間
    loc: string             // 位置描述
  }
}
```

## 🎨 UI 視圖說明 (UI Views)

### 主要視圖
1. **地震報告** (`reports`) - 顯示最近的地震報告列表
2. **天氣** (`forecast`) - 天氣資訊顯示
3. **溫度** (`temperature`) - 溫度分布圖
4. **空氣品質** (`aqi`) - AQI 資訊
5. **設定** (`settings`) - 應用程式設定

### 地圖顯示模式
- **震度模式 (i)**: 以震度值顯示測站顏色
- **PGA/PGV 模式 (a)**: 以地動加速度/速度顯示顏色

## 🔧 開發工具 (Development Tools)

### 程式碼檢查
```bash
npm run lint
```

### 建置應用程式
```bash
npm run dist
```

### 除錯模式
- 執行 `npm run dev` 自動開啟 DevTools
- 使用 Console 檢視日誌訊息
- 檢查 Network 監控 API 請求

## 📈 效能優化建議 (Performance Tips)

1. **快取策略**: 使用 Cache API 減少重複請求
2. **批次更新**: 避免頻繁的 DOM 操作
3. **事件防抖**: 對高頻事件使用防抖處理
4. **圖層管理**: 適時移除不需要的地圖圖層
5. **精確計算**: 根據需求選擇是否啟用精確數學計算

## 🔐 安全注意事項 (Security Notes)

- API 金鑰儲存在 localStorage，不會暴露在日誌中
- WebSocket 使用 WSS 加密連線
- 資料進行類型和範圍驗證
- 避免使用 eval() 等不安全函式

## 🌐 資料來源 (Data Sources)

### API 端點
- WebSocket: `wss://ws.exptech.com.tw/websocket`
- RTS 資料: `https://data.exptech.com.tw/api/v1/trem/rts`
- 地震報告: `https://data.exptech.com.tw/api/v1/eq/report`

### 預警來源
- 中央氣象局 (CWB)
- TREM 自有系統
- 日本防災科研 (NIED)
- 日本氣象廳 (JMA)
- 韓國氣象廳 (KMA)
- 中國地震局 (福建、四川)

## 📝 程式碼風格 (Code Style)

- 使用 ESLint 進行程式碼檢查
- 遵循現有的命名慣例
- 保持函式簡潔，單一職責
- 適當添加註解說明複雜邏輯
- 使用有意義的變數和函式名稱

## 🐛 常見問題 (Common Issues)

### WebSocket 連線失敗
- 檢查網路連線
- 確認 API 金鑰正確
- 查看 Console 錯誤訊息

### 地圖顯示空白
- 確認 MapLibre GL 載入成功
- 檢查地圖樣式設定
- 查看 Network 請求狀態

### 測站資料不更新
- 檢查 RTS API 請求
- 確認時間同步正確
- 驗證快取機制運作

## 📚 延伸閱讀 (Further Reading)

### 專案文檔
- [專案結構詳細說明](PROJECT_STRUCTURE.md)
- [開發者完整指南](DEVELOPER_GUIDE.md)
- [系統架構圖表](ARCHITECTURE.md)
- [快速參考卡片](QUICK_REFERENCE.md)

### 外部資源
- [Electron 官方文件](https://www.electronjs.org/docs)
- [MapLibre GL 文件](https://maplibre.org/maplibre-gl-js-docs/api/)
- [TREM 社群 Discord](https://discord.gg/5dbHqV8ees)

## 🤝 貢獻指南 (Contributing)

1. Fork 專案到您的帳號
2. 建立新的功能分支
3. 遵循程式碼風格指南
4. 撰寫清楚的 commit 訊息
5. 測試您的變更
6. 提交 Pull Request

## 📄 授權資訊 (License)

本專案採用 AGPL-3.0 授權。詳細資訊請參閱 [LICENSE](LICENSE) 檔案。

## 🔗 相關專案 (Related Projects)

- 原始專案: [ExpTechTW/TREM](https://github.com/ExpTechTW/TREM)
- 新版本 (Tauri): [ExpTechTW/TREM-tauri](https://github.com/ExpTechTW/TREM-tauri)
- 輕量版: [ExpTechTW/TREM-Lite](https://github.com/ExpTechTW/TREM-Lite)

---

## 📌 專案狀態 (Project Status)

> **重要提示**: TREM Electron 版本已被標記為棄用（deprecated），目前正在進行全面重寫。
> 
> 新版本將使用 Tauri 框架開發，提供更好的效能和更小的應用程式體積。
> 
> 如需最新版本，請參考 [TREM-tauri](https://github.com/ExpTechTW/TREM-tauri)。

---

## 💡 學習路徑建議 (Learning Path)

### 第一階段：基礎理解
1. 閱讀 README.md 了解專案背景
2. 瀏覽 PROJECT_STRUCTURE.md 熟悉目錄結構
3. 安裝並執行專案，體驗功能

### 第二階段：深入研究
1. 研讀 ARCHITECTURE.md 理解系統設計
2. 閱讀 DEVELOPER_GUIDE.md 學習開發流程
3. 追蹤程式碼執行流程，理解資料流向

### 第三階段：實踐應用
1. 嘗試修改現有功能
2. 新增自訂功能或資料來源
3. 參與社群討論，分享經驗

---

**祝您學習愉快！如有任何問題，歡迎在 GitHub Issues 中提出。** 🎉
