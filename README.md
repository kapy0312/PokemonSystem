# 寶可夢餵養中心 (Pokemon Feeding Center)

一套為班級教學打造的**遊戲化加分系統**。每位學生配對一隻寶可夢，透過累積「精華能量」驅動寶可夢從蛋孵化、歷經四階進化，並在進化關鍵時刻透過 QTE 連擊機制激發專注力與反應力。

---

## 目錄

- [核心特色](#核心特色)
- [系統架構](#系統架構)
- [技術棧](#技術棧)
- [目錄結構](#目錄結構)
- [Google Sheets 資料結構](#google-sheets-資料結構)
- [寶可夢圖鑑（24 系列）](#寶可夢圖鑑24-系列)
- [安裝與部署](#安裝與部署)
- [API 文件](#api-文件)
- [遊戲機制說明](#遊戲機制說明)
- [前端架構](#前端架構)
- [音效引擎](#音效引擎)
- [資源清單](#資源清單)

---

## 核心特色

### 遊戲化進化機制
- 每隻寶可夢從「寶可夢蛋」出發，歷經**四個進化階段**
- 涵蓋多種特殊形態：**MEGA 進化、超極巨化、洗翠型態、異色型態**
- 統一進化門檻：`[30, 120, 280, 440, 560]` 能量值
- 24 個寶可夢系列可供領養，依屬性配對專屬彩蛋顏色

### QTE 連擊系統
- 能量跨越門檻時觸發全螢幕 QTE 動畫
- 規則：**5 秒內連點 8 下**
- 成功 → 進化完成，播放勝利音效
- 失敗 → 能量自動回滾至進化前狀態，播放失敗音效

### 五段餵養序列
- 系統將「可餵能量」自動均分為 **5 份**，逐次注入
- 每次點擊「注入能量」消耗一份，提供儀式感與節奏感

### 即時資料同步
- 所有讀寫透過 Google Apps Script API 同步至 Google Sheets
- **樂觀更新 (Optimistic UI)**：前端即時反映，API 失敗時自動 Rollback
- **防競爭寫入**：GAS 端使用 `LockService` 防止並發衝突

### 防快取圖片載入
- 圖片 URL 附加時間戳 `?t={timestamp}` 強制繞過 CDN 快取

---

## 系統架構

```
┌─────────────────────────────────┐
│       GitHub Pages              │
│    index.html (React + HTM)     │
│  - 學生卡片列表                 │
│  - 寶可夢進化詳情 Modal         │
│  - QTE 進化動畫                 │
│  - 音效引擎                     │
└───────────┬─────────────────────┘
            │ fetch() GET / POST
            ▼
┌─────────────────────────────────┐
│    Google Apps Script (GAS)     │
│         Code.gs                 │
│  doGet()  → getAll              │
│  doPost() → addStudent          │
│           → deleteStudent       │
│           → updateStudentName   │
│           → updateEnergy        │
└───────────┬─────────────────────┘
            │ SpreadsheetApp API
            ▼
┌─────────────────────────────────┐
│         Google Sheets           │
│  PokemonData  │  StudentData    │
└─────────────────────────────────┘
```

---

## 技術棧

### 前端

| 技術 | 版本 | 說明 |
|------|------|------|
| HTML5 / CSS3 | — | 基礎結構與自訂動畫 (keyframes) |
| [Tailwind CSS](https://cdn.tailwindcss.com) | 最新 CDN | 響應式 Utility-first UI 框架 |
| [React](https://unpkg.com/react@18/umd/react.production.min.js) | 18 (CDN) | 組件化 UI 與 Hooks 狀態管理 |
| [HTM](https://unpkg.com/htm@3.1.1/dist/htm.js) | 3.1.1 (CDN) | 無需編譯器的 JSX 語法支援 |

### 後端

| 技術 | 說明 |
|------|------|
| Google Apps Script (GAS) | 後端邏輯層，以 `doGet` / `doPost` 提供 REST-like API |
| Google Sheets | 輕量級雲端資料庫，存放學生與寶可夢資料 |

### 部署

| 服務 | 用途 |
|------|------|
| GitHub Pages | 前端靜態頁面免費託管 |
| Google Apps Script Web App | 後端 API 端點（免費雲端函式） |

---

## 目錄結構

```
PokemonSystem/
├── index.html                # 前端單頁應用程式 (React + HTM，無需建置流程)
├── assets/
│   ├── bg.png                # 網頁背景圖
│   ├── egg_B.png             # 藍色蛋（水系）
│   ├── egg_BL.png            # 黑色蛋（鋼系）
│   ├── egg_G.png             # 綠色蛋（草系）
│   ├── egg_O.png             # 橘色蛋（火系／一般系）
│   ├── egg_P.png             # 紫色蛋（龍系／超能力系）
│   ├── egg_R.png             # 紅色蛋（火系）
│   ├── egg_Y.png             # 黃色蛋（電系）
│   ├── Ditto2.png            # 百變怪 第 2 階（客製圖）
│   ├── Ditto3.png            # 百變怪 第 3 階（客製圖）
│   ├── Ditto4.png            # 百變怪 第 4 階（客製圖）
│   ├── Jynx3.png             # 迷唇姐 第 3 階（客製圖）
│   ├── Jynx4.png             # 迷唇姐 第 4 階（客製圖）
│   ├── home.mp3              # 主畫面背景音樂（循環，音量 15%）
│   ├── evolution.mp3         # 進化事件背景音樂（音量 40%）
│   ├── click.mp3             # 點擊音效（每次獨立實例）
│   ├── victory1.mp3          # 進化成功音效
│   └── fail.mp3              # 進化失敗音效
└── .cursor/                  # Cursor AI 設定檔（可忽略）
```

> **注意**：後端 `Code.gs` 部署於 Google Apps Script 雲端，不在本機儲存庫中。

---

## Google Sheets 資料結構

### PokemonData 工作表

| 欄位 | 索引 | 說明 |
|------|------|------|
| ID | `[0]` | 寶可夢識別碼（如 `P001`） |
| 主名稱 | `[1]` | 系列名稱（如 `皮卡丘系列`） |
| 已被領養 | `[2]` | 核取方塊，`true` 表示已被領養 |
| 圖1～圖5 | `[3]` ~ `[7]` | 各進化階段圖片 URL |
| 限1～限5 | `[8]` ~ `[12]` | 各階段進化能量門檻 (Number) |
| 名1～名5 | `[13]` ~ `[17]` | 各階段寶可夢名稱 |

**統一進化門檻（約 7 個月課程進度）：`[30, 120, 280, 440, 560]`**

### StudentData 工作表

| 欄位 | 索引 | 說明 |
|------|------|------|
| ID | `[0]` | 學生識別碼（`STU` + 時間戳 + 亂數） |
| 小朋友名字 | `[1]` | 學生姓名 |
| 持有寶可夢名稱 | `[2]` | 對應寶可夢主名稱 |
| 目前精華能量 | `[3]` | 累積總能量 (Number) |
| 目前可餵的精華能量 | `[4]` | 本次可注入的能量 (Number) |

---

## 寶可夢圖鑑（24 系列）

依屬性配對專屬彩色蛋圖示，圖片主要來自 [PokeAPI 官方素材](https://github.com/PokeAPI/sprites)。

| ID | 系列名稱 | 蛋色 | 第1階 | 第2階 | 第3階 | 第4階（特殊形態）|
|----|----------|------|-------|-------|-------|------------------|
| P001 | 皮卡丘系列 | 黃 | 皮丘 | 皮卡丘 | 雷丘 | 超極巨化皮卡丘 |
| P002 | 小火龍系列 | 紅 | 小火龍 | 火恐龍 | 噴火龍 | 超級噴火龍X |
| P003 | 妙蛙種子系列 | 綠 | 妙蛙種子 | 妙蛙草 | 妙蛙花 | 超級妙蛙花 |
| P004 | 傑尼龜系列 | 藍 | 傑尼龜 | 卡咪龜 | 水箭龜 | 超級水箭龜 |
| P005 | 木守宮系列 | 綠 | 木守宮 | 森林蜥蜴 | 蜥蜴王 | 超級蜥蜴王 |
| P006 | 幼基拉斯系列 | 綠 | 幼基拉斯 | 沙基拉斯 | 班基拉斯 | 超級班基拉斯 |
| P007 | 可可多拉系列 | 黑 | 可可多拉 | 可多拉 | 波士可多拉 | 超級波士可多拉 |
| P008 | 甲賀忍蛙系列 | 藍 | 呱呱泡蛙 | 呱頭蛙 | 甲賀忍蛙 | 小智版甲賀忍蛙 |
| P009 | 火球鼠系列 | 橘 | 火球鼠 | 火岩鼠 | 火暴獸 | 洗翠火暴獸 |
| P010 | 黏黏寶系列 | 紫 | 黏黏寶 | 黏美兒 | 黏美龍 | 洗翠黏美龍 |
| P011 | 木木梟系列 | 綠 | 木木梟 | 投羽梟 | 狙射樹梟 | 洗翠狙射樹梟 |
| P012 | 稚山雀系列 | 黑 | 稚山雀 | 藍鴉 | 鋼鎧鴉 | 超極巨化鋼鎧鴉 |
| P013 | 敲音猴系列 | 綠 | 敲音猴 | 啪咚猴 | 轟擂金剛猩 | 超極巨化轟擂金剛猩 |
| P014 | 炎兔兒系列 | 橘 | 炎兔兒 | 騰蹴小將 | 閃焰王牌 | 超極巨化閃焰王牌 |
| P015 | 呆呆獸系列 | 紫 | 呆呆獸 | 呆殼獸 | 呆呆王 | 超級呆殼獸 |
| P016 | 利歐路系列 | 藍 | 利歐路 | 路卡利歐 | 超級路卡利歐 | 波導勇者路卡利歐 |
| P017 | 鯉魚王系列 | 藍 | 鯉魚王 | 暴鯉龍 | 紅色暴鯉龍（異色） | 霸主暴鯉龍 |
| P018 | 圓陸鯊系列 | 藍 | 圓陸鯊 | 尖牙陸鯊 | 烈咬陸鯊 | 超級烈咬陸鯊 |
| P019 | 綠毛蟲系列 | 綠 | 綠毛蟲 | 鐵甲蛹 | 巴大蝶 | 超極巨化巴大蝶 |
| P020 | 伊布系列 | 橘 | 伊布 | 火伊布 | 太陽伊布 | 月亮伊布 |
| P021 | 火爆猴系列 | 橘 | 猴怪 | 火爆猴 | 異色火爆猴 | 棄世猴 |
| P022 | 百變怪系列 | 紫 | 百變怪 | 百變怪(鯉魚王) | 百變怪(呆呆獸) | 百變怪(夢幻)（客製圖）|
| P023 | 耿鬼系列 | 紫 | 鬼斯 | 鬼斯通 | 耿鬼 | 超級耿鬼 |
| P024 | 迷唇姐系列 | 紫 | 迷唇娃 | 迷唇姐 | 超級迷唇姐（客製圖）| 波波先生（客製圖）|

---

## 安裝與部署

### 1. 後端：Google Apps Script 設定

1. 建立一個 [Google Sheet](https://sheets.google.com) 作為資料庫，記下試算表 ID
2. 前往 [Google Apps Script](https://script.google.com)，建立新專案
3. 將 `Code.gs` 內容貼入編輯器，並更新以下常數：

```javascript
const SPREADSHEET_ID = '你的試算表 ID';
```

4. 執行 `initSetup()` 函式，初始化工作表結構與標題列
5. 執行 `initPokemonDataOnly()` 函式，批次寫入 24 隻寶可夢資料
6. 點選「部署」→「管理部署作業」→「新增部署」
   - 類型：**網頁應用程式**
   - 執行身分：**我**
   - 誰可以存取：**任何人**
7. 複製部署後的「網頁應用程式 URL」

### 2. 前端：設定 API 端點

開啟 `index.html`，找到以下常數並替換：

```javascript
const GAS_DEPLOY_URL = '貼上你的 GAS 網頁應用程式 URL';
```

### 3. 部署至 GitHub Pages

1. 將整個儲存庫推送至 GitHub
2. 進入儲存庫 → `Settings` → `Pages`
3. `Source` 設為 `Deploy from a branch`，選擇 `main`，根目錄 `/ (root)`
4. 儲存後等待部署，網址格式為：
   ```
   https://<your-github-username>.github.io/<repo-name>/
   ```

### 4. 本機執行

直接用瀏覽器開啟 `index.html`（需連網以載入 CDN 資源與 GAS API）。

---

## API 文件

基底 URL：`GAS_DEPLOY_URL`

### GET 請求

| 參數 | 說明 |
|------|------|
| `action=getAll` | 取得所有寶可夢與學生資料，附加 `&t={timestamp}` 防快取 |

**回應範例：**
```json
{
  "status": 200,
  "data": {
    "pokemons": [
      {
        "id": "P001",
        "mainName": "皮卡丘系列",
        "isAdopted": false,
        "images": ["egg_Y_url", "pichu_url", "pikachu_url", "raichu_url", "gmax_url"],
        "thresholds": [30, 120, 280, 440, 560],
        "stageNames": ["寶可夢蛋", "皮丘", "皮卡丘", "雷丘", "超極巨化皮卡丘"]
      }
    ],
    "students": [
      {
        "id": "STU17430000001234",
        "name": "小明",
        "pokemonName": "皮卡丘系列",
        "energy": 150,
        "feedableEnergy": 30
      }
    ]
  }
}
```

### POST 請求

所有 POST 請求的 body 均為 JSON。

#### `addStudent` — 新增學生

```json
{ "action": "addStudent", "name": "小明" }
```
- 從未被領養的寶可夢中**隨機分配**一隻
- 使用 `LockService` 防止並發搶佔
- 回傳：`{ "success": true, "studentId": "STU..." }`

#### `deleteStudent` — 刪除學生

```json
{ "action": "deleteStudent", "id": "STU..." }
```
- 同步將對應寶可夢的「已被領養」欄位改回 `false`
- 回傳：`{ "success": true }`

#### `updateStudentName` — 修改學生姓名

```json
{ "action": "updateStudentName", "id": "STU...", "name": "新名字" }
```
- 回傳：`{ "success": true, "id": "STU...", "newName": "新名字" }`

#### `updateEnergy` — 更新能量值

```json
{ "action": "updateEnergy", "id": "STU...", "energy": 150, "feedableEnergy": 0 }
```
- 使用 `LockService` 防止並發寫入
- 回傳：`{ "success": true }`

---

## 遊戲機制說明

### 能量與進化階段

```
energy:  0  ──[30]──  stage1  ──[120]──  stage2  ──[280]──  stage3  ──[440]──  stage4  ──[560]──  MAX
```

- `getStageIndex(energy, thresholds)` 根據當前能量計算所在階段（0～4）
- 能量每次跨越門檻即觸發進化流程

### 五段餵養序列

```
feedableEnergy = 30
amount    = floor(30 / 5) = 6   (每段)
remainder = 30 % 5        = 0   (最後一段補差)

點擊順序: +6 → +6 → +6 → +6 → +6
```

### QTE 連擊流程

```
觸發條件: energy 跨越 thresholds[n]
          ↓
    全螢幕 QTE Modal 出現
    ┌────────────────────────┐
    │  目前寶可夢 → 進化目標   │
    │  閃爍速度: 0.6s → 0.1s  │  (每點擊一下加快)
    │  計時器: 5 秒倒數        │
    │  計數器: X / 8           │
    └────────────────────────┘
          ↓ 8 下 & 時間內          ↓ 超時
       進化成功                   進化失敗
    播放勝利音效               播放失敗音效
    energy 保留               energy rollback
```

### 樂觀更新與 Rollback

```
使用者操作
    │
    ▼ 立即更新 UI（零延遲）
    │
    ▼ 非同步呼叫 GAS API
    ├── 成功 → 繼續
    └── 失敗 → fetchAllData() 重新載入，UI 自動回滾
```

---

## 前端架構

### 主要狀態 (React Hooks)

```javascript
data            // { students: [], pokemons: [] }  — 後端全量資料
loading         // boolean                         — 初始載入狀態
syncing         // boolean                         — API 同步中（顯示 overlay）
selectedStudent // Student | null                  — 目前開啟 Modal 的學生
newStudentName  // string                          — 新增學生輸入欄
feedSeq         // { active, chunksLeft, amount, remainder, originalState }
qte             // { active, clicks, timeLeft, targetStage }
```

### 自訂 CSS 動畫

| 類別 | 效果 | 使用場景 |
|------|------|----------|
| `.shake` | 0.3s 震動循環 | 注入能量按鈕 |
| `.animate-pulse-slow` | 3s 透明度+縮放脈動 | Modal 寶可夢圖片 |
| `.animate-flash-toggle` | 0.1～0.6s 可變頻率閃爍 | QTE 進化動畫 |
| `.animate-spin-slow` | 2s 持續旋轉 | 讀取中圖示 |
| `.evolution-silhouette` | `brightness(0) invert(1)` + 白色光暈 | QTE 剪影效果 |

---

## 音效引擎

```javascript
AudioController = {
  homeBgm:  new Audio('assets/home.mp3'),       // volume: 0.15, loop: true
  evoBgm:   new Audio('assets/evolution.mp3'),  // volume: 0.40
  bgmTimer: null                                // 事件結束後恢復計時器
}
```

| 方法 | 行為 |
|------|------|
| `playClick()` | 建立新 Audio 實例播放，確保連點不卡頓 |
| `playHomeBgm()` | 播放主畫面 BGM 循環 |
| `playEvoBgm()` | 停止主畫面 BGM，播放進化 BGM |
| `playSuccess()` | 播放勝利音效，5 秒後恢復主畫面 BGM |
| `playFail()` | 播放失敗音效，5 秒後恢復主畫面 BGM |

> 瀏覽器自動播放限制：所有音效均附加 `.catch(() => {})` 避免 Uncaught Promise 錯誤。

---

## 資源清單

### 圖片來源

- **官方素材（PokeAPI Sprites）**
  - 一般圖鑑圖：`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/{id}.png`
  - 異色圖：`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/shiny/{id}.png`

- **客製圖片（本機 assets，透過 jsDelivr CDN 供應）**
  - 格式：`https://cdn.jsdelivr.net/gh/kapy0312/PokemonSystem@main/assets/{filename}.png`
  - 包含：7 種彩色蛋、百變怪 2-4 階、迷唇姐 3-4 階

### 蛋色對應屬性

| 蛋色 | 檔案 | 屬性 |
|------|------|------|
| 黃色 | `egg_Y.png` | 電系 |
| 紅色 | `egg_R.png` | 火系 |
| 橘色 | `egg_O.png` | 火系／一般系 |
| 綠色 | `egg_G.png` | 草系／岩石系（綠外觀）|
| 藍色 | `egg_B.png` | 水系／格鬥系 |
| 黑色 | `egg_BL.png` | 鋼系／深色外觀 |
| 紫色 | `egg_P.png` | 龍系／超能力系／幽靈系 |
