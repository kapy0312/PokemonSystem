# 寶可夢餵養中心 (Pokemon System)

這是一個為小朋友精心設計的遊戲化加分系統，結合寶可夢進化概念，讓小朋友在「注入能量」的互動過程中體驗陪伴寶可夢成長的樂趣，並透過 QTE 連擊機制激發反應力與專注力。

---

## 核心特色

### 遊戲化升級機制
- 從寶可夢蛋開始，歷經**四階進化**，見證寶可夢的華麗轉變
- 支援多種特殊進化形態：**MEGA 進化、超極巨化、洗翠型態**
- 每隻寶可夢有獨立的能量閾值，達標後觸發進化

### QTE 連擊系統 (Quick Time Event)
- 進化關鍵時刻觸發 QTE 動畫，需在 **5 秒內連點 8 下**
- 成功：寶可夢進化，播放勝利音效
- 失敗：能量歸零，需重新累積

### 獨立音效引擎
- 主畫面 BGM (`home.mp3`) — 循環播放，音量 15%
- 進化事件 BGM (`evolution.mp3`) — 進化觸發時立即切換
- 點擊音效 (`click.mp3`) — 每次點擊獨立實例，連點不卡頓
- 勝利音效 (`victory1.mp3`) / 失敗音效 (`fail.mp3`)
- 進化事件結束後，延遲 5 秒自動恢復主畫面 BGM

### 即時資料同步
- 所有資料透過 **Google Apps Script (GAS)** 即時同步至 **Google Sheets**
- **樂觀更新 (Optimistic UI)**：前端即時更新，API 失敗時自動 Rollback
- **Cache-Busting**：圖片 URL 加上時間戳 (`?t={timestamp}`)，避免 CDN 快取問題

---

## 技術棧

### 前端

| 技術 | 說明 |
|------|------|
| HTML5 / CSS3 | 基礎結構與樣式 |
| [Tailwind CSS](https://cdn.tailwindcss.com) (CDN) | 響應式 UI 框架 |
| [React 18](https://unpkg.com/react@18/umd/react.production.min.js) (CDN) | UI 組件化與狀態管理 |
| [HTM 3.1.1](https://unpkg.com/htm@3.1.1/dist/htm.js) (CDN) | 無需編譯器的 JSX 語法支援 |

### 後端

| 技術 | 說明 |
|------|------|
| Google Apps Script (GAS) | 後端邏輯層，處理資料讀寫 |
| Google Sheets | 輕量級資料庫，儲存學生與寶可夢資料 |

### 部署

| 服務 | 用途 |
|------|------|
| GitHub Pages | 前端靜態頁面託管 |
| Google Apps Script Web App | 後端 API 端點 |

---

## 目錄結構

```
PokemonSystem/
├── index.html              # 前端單頁應用程式入口 (React + HTM)
├── assets/
│   ├── bg.png              # 網頁背景圖片
│   ├── 蛋.png              # 預設寶可夢蛋圖片
│   ├── egg_B.png           # 藍色蛋 (水系)
│   ├── egg_BL.png          # 黑色蛋 (鋼系)
│   ├── egg_G.png           # 綠色蛋 (草系)
│   ├── egg_O.png           # 橘色蛋 (火系/一般系)
│   ├── egg_P.png           # 紫色蛋 (龍系/超能力系)
│   ├── egg_R.png           # 紅色蛋 (火系)
│   ├── egg_Y.png           # 黃色蛋 (電系)
│   ├── Ditto2.png          # 百變怪第二階段
│   ├── Ditto3.png          # 百變怪第三階段
│   ├── Ditto4.png          # 百變怪第四階段
│   ├── home.mp3            # 主畫面背景音樂
│   ├── click.mp3           # 點擊音效
│   ├── evolution.mp3       # 進化過程背景音樂
│   ├── fail.mp3            # 進化失敗音效
│   └── victory1.mp3        # 進化成功音效
└── .cursor/                # Cursor AI 設定檔
```

> **注意**：後端 `Code.gs` 部署於 Google Apps Script 雲端，不在本機儲存庫中。

---

## 安裝與部署

### 1. 後端：Google Apps Script (GAS) 設定

1. 建立一個 Google Sheet 作為資料庫
2. 前往 [Google Apps Script](https://script.google.com)，建立新專案並綁定上述 Sheet
3. 將後端程式碼貼入 GAS 專案
4. 點擊「部署」→「新增部署作業」，類型選「網頁應用程式」
5. 執行身分選「我」，存取權選「任何人」
6. 部署後複製「網頁應用程式 URL」
7. 將 URL 貼至 `index.html` 第 148 行：

```javascript
const GAS_DEPLOY_URL = '您的 GAS 發佈網址'; // 替換此行
```

### 2. 前端：GitHub Pages 部署

1. 將 `index.html` 與 `assets/` 資料夾上傳至 GitHub 儲存庫
2. 進入儲存庫 → `Settings` → `Pages`
3. `Build and deployment` 的 `Source` 設為 `Deploy from a branch`
4. 選擇 `main` 分支，根目錄 `/ (root)`，點擊「Save」
5. 部署完成後，網址格式為：
   ```
   https://<your-github-username>.github.io/<your-repository-name>/
   ```

### 3. 本機執行

直接用瀏覽器開啟 `index.html` 即可（需連網以載入 CDN 資源）。

---

## API 端點 (GAS)

| action | 說明 |
|--------|------|
| `getAll` | 取得所有學生與寶可夢資料 |
| `addStudent` | 新增學生與對應寶可夢 |
| `deleteStudent` | 刪除學生資料 |
| `updateStudentName` | 修改學生姓名 |
| `updateEnergy` | 更新能量值 |

---

## 主要狀態結構

```javascript
data:            { students: [], pokemons: [] }     // 所有資料
selectedStudent: { id, name, energy, ... }          // 當前選中學生
feedSeq:         { active, chunksLeft, ... }        // 餵食序列狀態
qte:             { active, clicks, timeLeft, ... }  // QTE 挑戰狀態
```
