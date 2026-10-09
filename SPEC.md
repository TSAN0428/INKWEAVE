# 墨織 InkWeave — 小說設定工坊 規格書

> 版本：v2.0　｜　最後更新：2026-10-09
> 本文件是本專案的**唯一事實來源（Single Source of Truth）**。
> AI 協作者在實作任何功能前，必須先閱讀本文件；若需求與本文件衝突，以本文件為準，並提出修改建議而非自行變更。
> 修改歷程見〈附錄 B：AI 協作紀錄〉。

---

## 0. 給 AI 的開發規則（每次開發前必讀）

1. **最終成品是單一 `index.html` 檔案**：所有 CSS 寫在 `<style>`、所有 JavaScript 寫在 `<script>` 內。不得拆分成多個檔案，不得需要建置步驟（不用 npm、Vite、打包工具）。
2. **不使用任何外部資源**：不引用 CDN 函式庫、外部字型、外部圖片。只用瀏覽器原生 API（DOM、SVG、IndexedDB、Canvas）。
3. **使用原生 JavaScript（ES2020+）**，不使用 React、Vue、jQuery 等框架。
4. **只做本文件描述的功能**。不要自行加入未列出的功能；若認為有必要，先說明理由並等待確認。
5. **純前端**：不得連線任何伺服器、不得呼叫外部 API、不得放入任何 API 金鑰。
6. **所有資料存取必須經過「資料層」函式**（第 5.4 節）。畫面程式不得直接操作 IndexedDB。
7. **防止 XSS**：所有使用者輸入的文字放進 HTML 前，必須經過 `escapeHtml()`。
8. **介面文字一律使用繁體中文（台灣用語）**；程式碼、變數名稱使用英文。
9. **刪除操作必須跳出確認對話框**，並依第 6 節的連動規則處理關聯資料。
10. 程式碼依第 10.1 節的區塊順序排列，每個區塊以註解標題分隔，方便後續修改時定位。
11. 修改資料結構時，必須同步更新：第 5 節資料模型、匯出／匯入、範例資料。
12. 每完成一個里程碑（第 11 節），逐項檢查該里程碑的驗收條件，並在〈附錄 B〉新增一筆紀錄。

---

## 1. 產品概述

### 1.1 一句話描述
**墨滴是一個給小說寫作者的設定管理工具**：以「章節」為骨架，把角色、角色關係、時間線與伏筆串在一起，讓作者隨時掌握故事全貌。

### 1.2 解決的問題
| 寫作痛點 | 墨滴的解法 |
|---|---|
| 角色設定散落各處，寫到後面忘記口頭禪、動機 | 角色卡集中管理，並自動列出相關關係、事件、伏筆 |
| 人物關係隨劇情變化，難以追蹤 | 關係可設定「從第幾章開始」，拖動章節滑桿即可看關係演變 |
| 倒敘、插敘讓時間順序混亂 | 時間線同時記錄「故事時間」與「出現章節」，可切換兩種排序 |
| 埋了伏筆卻忘記回收 | 伏筆追蹤器自動提醒超過 N 章未回收的伏筆 |

### 1.3 目標使用者
- 正在寫中長篇小說的業餘／網路小說作者
- 寫奇幻、武俠、群像劇等角色多、設定多的類型

### 1.4 產品原則
- **不寫正文**：本工具只管理「設定」，正文由使用者用自己習慣的工具撰寫。
- **資料在本機**：無需註冊登入，資料存在使用者瀏覽器的 IndexedDB。
- **開箱即用**：一鍵載入範例作品即可體驗全部功能。
- **單檔即用**：下載 `index.html` 雙擊即可開啟，也可放上 GitHub Pages。

---

## 2. 技術規格

| 項目 | 做法 |
|---|---|
| 檔案 | 單一 `index.html`（內嵌 `<style>` 與 `<script>`） |
| 語言 | 原生 HTML / CSS / JavaScript（ES2020+），`'use strict'` |
| 畫面渲染 | 以樣板字串產生 HTML，設定到容器的 `innerHTML`；使用者文字必須經 `escapeHtml()` |
| 事件處理 | **事件委派**：在根容器監聽 `click` / `input` / `change`，依元素的 `data-action` 屬性分派 |
| 路由 | 以 `location.hash` 切換頁面，監聽 `hashchange` |
| 資料儲存 | IndexedDB，自行撰寫小型存取函式（第 5.3 節） |
| 關係圖 | 原生 **SVG** 繪製，自行實作節點拖曳 |
| 時間線雙軸圖 | 原生 **SVG** 繪製 |
| 圖片壓縮 | Canvas 縮圖後轉成 data URL |
| 字型 | 系統字型：`system-ui, "Noto Sans TC", "Microsoft JhengHei", sans-serif` |
| 樣式 | CSS 變數（`:root`）定義顏色，深色模式以 `[data-theme="dark"]` 覆寫 |
| ID | `crypto.randomUUID()`；不可用時（例：以 `file://` 開啟的部分瀏覽器）使用後備產生器 |
| 部署 | GitHub Pages（直接從 `main` 分支根目錄發布） |

---

## 3. 資訊架構

```
作品列表（首頁）
 └─ 作品工作區（側邊欄導覽）
     ├─ 總覽
     ├─ 章節
     ├─ 角色（列表 → 角色卡詳細頁）
     ├─ 關係圖
     ├─ 時間線
     ├─ 伏筆
     └─ 作品設定（含匯出、刪除）
 全域：搜尋（Ctrl/⌘ + K）、深色模式切換
```

### 3.1 路由表

| Hash | 頁面 |
|---|---|
| `#/` 或空白 | 作品列表 |
| `#/works/{workId}` | 作品總覽 |
| `#/works/{workId}/chapters` | 章節管理 |
| `#/works/{workId}/characters` | 角色列表 |
| `#/works/{workId}/characters/{characterId}` | 角色卡詳細頁 |
| `#/works/{workId}/relations` | 角色關係圖 |
| `#/works/{workId}/timeline` | 時間線 |
| `#/works/{workId}/foreshadows` | 伏筆追蹤器 |
| `#/works/{workId}/settings` | 作品設定 |
| 其他／找不到資料 | 「找不到頁面」，提供回首頁連結 |

### 3.2 版面
- **桌面（≥ 768px）**：左側固定側邊欄（作品名稱＋導覽，目前頁面高亮），右側內容區。
- **手機（< 768px）**：頂部列＋漢堡選單展開導覽；頁面不得出現水平捲動（關係圖畫布內部可捲動除外）。
- 全站共用元件：按鈕、對話框（`<dialog>`）、提示訊息（toast）、章節下拉選單、角色多選、頭像。

---

## 4. 功能規格

> 每個功能標示優先度：**【必做】** 為核心功能，必須完成；**【選做】** 在必做全部完成後再實作。

### 4.1 作品列表（首頁）【必做】
- 以卡片顯示所有作品：標題、簡介（截斷 2 行）、章節數、角色數、最後更新時間。
- 「新增作品」：輸入標題（必填）、簡介（選填）。
- 「載入範例作品」：建立第 9 節的範例作品（可重複載入，每次產生新副本）。
- 「匯入備份」：選擇 JSON 檔，依第 8 節規則匯入。
- 「匯出全部」：下載所有作品的備份檔。
- 無作品時顯示空狀態引導：「建立第一部作品」或「載入範例作品」。
- 頁尾提示：「資料儲存在此瀏覽器中，清除瀏覽器資料會遺失，請定期匯出備份。」
- 啟動時呼叫 `navigator.storage.persist()`（若支援）請求持久儲存。

### 4.2 作品總覽【必做】
- 統計卡片：章節數、角色數、事件數、伏筆（未回收／已回收）。
- 「目前寫到」：顯示 `currentChapterId` 對應章節，可直接切換。
- **⚠️ 逾期伏筆清單**：列出所有逾期伏筆（定義見 7.3），點擊跳到伏筆頁。
- 【選做】最近更新的 5 筆項目（角色／事件／伏筆），點擊跳轉。

### 4.3 章節管理（骨架）【必做】
所有功能都以章節作為對照軸。章節**只記錄標題與摘要，不寫正文**。
- 列表依 `order` 排序，顯示「第 N 章」＋標題＋摘要。
- 新增章節（預設加在最後）、編輯、刪除。
- 調整順序：上移／下移按鈕；調整後 `order` 重新正規化為 1, 2, 3…
- 每章可展開顯示：本章發生的事件、本章埋下的伏筆、本章回收的伏筆。
- 「第 N 章」編號一律由 `order` 即時計算，不另存。
- 全站選擇章節的下拉選單統一格式：`第 N 章｜標題`。
- 【選做】拖曳排序。

### 4.4 角色卡【必做】

**角色列表頁**
- 卡片網格：頭像（無圖時顯示姓名首字＋角色代表色底）、姓名、標籤。
- 搜尋（姓名、別名）與標籤篩選。
- 新增角色（只需輸入姓名即可建立，其餘到詳細頁填寫）。

**角色詳細頁欄位**
| 欄位 | 型別 | 說明 |
|---|---|---|
| 姓名 | 文字，必填 | |
| 別名 | 文字陣列 | 綽號、稱號（以逗號或 Enter 新增） |
| 頭像 | 圖片 | 見下方圖片規則 |
| 代表色 | 色碼 | `<input type="color">`，用於關係圖節點、頭像底色 |
| 標籤 | 文字陣列 | 例：主角、反派、配角 |
| 外貌 | 長文字 | |
| 性格 | 長文字 | |
| 口頭禪 | 長文字 | |
| 動機 | 長文字 | |
| 秘密 | 長文字 | 預設**模糊遮罩**（CSS `filter: blur`），點擊才顯示 |
| 自訂欄位 | 鍵值對陣列 | 使用者自行新增／刪除，例：「年齡：17」、「能力：御風」 |
| 備註 | 長文字 | |

**自動關聯區（唯讀，詳細頁下方）**
- 「人際關係」：列出涉及此角色的所有關係，及其在「目前章節」時的狀態。
- 「參與事件」：列出 `characterIds` 包含此角色的事件，依故事時間排序。
- 「相關伏筆」：列出 `characterIds` 包含此角色的伏筆及狀態。
- 「出場章節」：由參與事件推算（見 7.4）。
- 每一項可點擊跳轉到對應頁面。

**圖片規則**
- 接受 jpg / png / webp / gif。
- 上傳後以 Canvas **壓縮**：最長邊 512px，輸出 `image/webp`（瀏覽器不支援時用 `image/jpeg`），品質 0.8。
- 壓縮後的 data URL 存於 `images` 資料表，角色只存 `avatarImageId`。
- 可移除或更換頭像；更換或移除時刪除舊圖片紀錄。

**編輯方式**
- 欄位直接在頁面上編輯，停止輸入 600ms 後自動儲存，並顯示「已儲存」提示。
- 自動儲存時**不得重新渲染整頁**（避免輸入框失去焦點）。

### 4.5 角色關係圖【必做】

**資料概念**
- 一條「關係」連接兩個角色，包含一或多個「階段（phase）」，代表關係隨章節變化。
- 每個階段有「起始章節」；有效範圍是「起始章節」到「下一個階段起始章節的前一章」。
- 例：A 與 B：第 1 章起「師徒」→ 第 7 章起「敵對」。
- 階段類型可為 `ended`，代表關係從該章起結束（圖上不顯示此線）。

**畫面（SVG）**
- 角色為節點：圓形，內含頭像（SVG `<image>` 搭配圓形 `clipPath`）或姓名首字，邊框為代表色，下方顯示姓名。
- 關係為連線：顏色依關係類型（5.2），線段中點顯示標籤文字。
- `directed = true` 的關係顯示箭頭（SVG `<marker>`）。
- 同一對角色若有兩條關係，連線以弧線錯開，不可重疊。
- **章節滑桿**（`<input type="range">`）：位於畫布上方，範圍為第 1 章到最後一章，預設為目前章節，旁邊顯示「第 N 章｜標題」。拖動時即時重繪該章節的關係狀態（計算規則見 7.1）。
- 「顯示全部」核取方塊：勾選時忽略章節，顯示每條關係的最新階段。
- 圖例：列出各關係類型的顏色與中文名稱。
- 沒有章節時，滑桿隱藏，顯示提示「新增章節後即可依章節檢視關係變化」。

**互動**
- 節點可用滑鼠或觸控（Pointer Events）拖曳；放開時將位置存入角色的 `graphPosition`。
- 沒有位置的角色，自動以圓形排列在畫布中央。
- 點節點：右側面板顯示角色摘要與「開啟角色卡」連結。
- 點連線：右側面板顯示該關係的所有階段，可新增／編輯／刪除階段（手機版改為下方面板）。
- 「新增關係」按鈕：選擇角色 A、角色 B、是否有方向，並設定第一個階段。
- 【選做】「自動排列」按鈕：將所有節點重新排成圓形。
- 【選做】畫布縮放與平移（滾輪縮放、拖曳空白處平移）。

### 4.6 時間線【必做】

**資料概念**
每個事件有兩個座標：
- **故事時間**：事情在故事世界中真正發生的時間。為支援架空曆法，分成兩個欄位：
  - `storyTimeLabel`：顯示文字，例：「霧曆 88 年冬」、「十年前」。
  - `storyTimeSort`：排序用數字，越小越早，可用小數插入兩個事件之間。
- **敘事位置**：讀者在哪一章讀到這件事（`chapterId`；為空代表「背景事件，正文未呈現」），同一章內依 `orderInChapter` 排序。

**檢視模式（頁面上方分頁切換）**
1. **故事順序**【必做】：依 `storyTimeSort` 排列的垂直時間軸，每個事件標示「出現於第 N 章」或「背景事件」。
2. **敘事順序**【必做】：依章節 `order` → `orderInChapter` 排列，以章節分組，每個事件標示故事時間；倒敘事件（見 7.2）顯示「↩ 倒敘」徽章。
3. **雙軸對照圖**【選做】：SVG 繪製，左欄為故事順序、右欄為敘事順序，同一事件以線相連；交叉的連線代表倒敘／插敘。滑鼠移到事件上時，高亮該事件的連線。

**功能**
- 新增／編輯／刪除事件：標題（必填）、描述、故事時間文字、排序數字、出現章節、參與角色（多選）、標籤。
- 編輯排序數字時，在旁邊提示相鄰事件的數字（例：「前一個：30，後一個：40」），方便插入。
- 篩選：依角色、依標籤。
- 事件卡片顯示參與角色的小頭像。

### 4.7 伏筆追蹤器【必做】

**欄位**
| 欄位 | 說明 |
|---|---|
| 標題 | 必填，例：「斷掉的玉珮」 |
| 描述 | 伏筆內容與預計的真相 |
| 埋設章節 | 必填 |
| 預計回收章節 | 選填 |
| 實際回收章節 | 狀態為「已回收」時必填 |
| 狀態 | `open` 未回收／`resolved` 已回收／`abandoned` 已放棄 |
| 相關角色 | 多選 |
| 相關事件 | 多選 |
| 回收方式備註 | 選填 |

**畫面**
- 上方統計：未回收、已回收、已放棄、逾期的數量。
- 分頁：全部／未回收／逾期／已回收／已放棄。
- 列表項目：標題、埋設章、「已埋 N 章」（7.3）、狀態徽章。
- **逾期**項目以警示色標示，並排在列表最上方。
- 「標記為已回收」快捷按鈕：跳出對話框選擇實際回收章節。
- 沒有章節時，顯示提示「請先新增章節才能記錄伏筆」，並提供前往章節頁的按鈕。
- 【選做】「伏筆跨度圖」：橫軸為章節，每條伏筆是從埋設章到回收章（未回收則到目前章）的橫條，逾期以警示色顯示。

### 4.8 作品設定【必做】
- 編輯作品標題、簡介。
- 「目前寫到第幾章」（`currentChapterId`）；未設定時視為最後一章。
- 伏筆逾期門檻 N（整數，1～999，預設 10）。
- 匯出此作品（JSON，含圖片）。
- 刪除作品：需輸入作品標題確認，連同所有關聯資料一起刪除。

### 4.9 全域搜尋【選做】
- 在作品工作區按 `Ctrl/⌘ + K` 或點頂部搜尋按鈕，開啟搜尋對話框。
- 搜尋範圍：此作品的角色（姓名、別名）、章節（標題、摘要）、事件（標題、描述）、伏筆（標題、描述）。
- 結果依類型分組；支援鍵盤上下選擇、Enter 跳轉、Esc 關閉。
- 不分大小寫的子字串比對即可。

### 4.10 深色模式【選做】
- 頂部切換按鈕，循環切換：淺色 → 深色 → 跟隨系統。
- 偏好存於 `localStorage`（key：`inkweave-theme`），讀寫需包在 `try/catch` 中，失敗時退回「跟隨系統」。

---

## 5. 資料模型

### 5.1 資料結構（以 JSDoc 註解寫在程式中）

```js
/**
 * @typedef {Object} Work 作品
 * @property {string} id
 * @property {string} title
 * @property {string} description
 * @property {string|null} currentChapterId   null = 視為最後一章
 * @property {number} foreshadowWarnThreshold 預設 10
 * @property {string} createdAt               ISO 8601
 * @property {string} updatedAt
 */

/**
 * @typedef {Object} Chapter 章節
 * @property {string} id
 * @property {string} workId
 * @property {number} order      1, 2, 3…（正規化後連續）
 * @property {string} title
 * @property {string} summary
 * @property {string} createdAt
 * @property {string} updatedAt
 */

/**
 * @typedef {Object} Character 角色
 * @property {string} id
 * @property {string} workId
 * @property {string} name
 * @property {string[]} aliases
 * @property {string|null} avatarImageId
 * @property {string} color                   '#RRGGBB'
 * @property {string[]} tags
 * @property {string} appearance
 * @property {string} personality
 * @property {string} catchphrase
 * @property {string} motivation
 * @property {string} secret
 * @property {{id:string, key:string, value:string}[]} customFields
 * @property {string} notes
 * @property {{x:number, y:number}|null} graphPosition
 * @property {string} createdAt
 * @property {string} updatedAt
 */

/**
 * @typedef {Object} StoredImage 圖片
 * @property {string} id
 * @property {string} workId
 * @property {string} dataUrl     壓縮後的 data URL
 * @property {string} createdAt
 */

/**
 * @typedef {'family'|'romance'|'crush'|'friend'|'ally'|'rival'|'enemy'|'mentor'|'subordinate'|'other'|'ended'} RelationType
 */

/**
 * @typedef {Object} RelationPhase 關係階段
 * @property {string} id
 * @property {string|null} startChapterId   null = 從故事開始
 * @property {RelationType} type
 * @property {string} label                  線上顯示文字，例：「青梅竹馬」
 * @property {string} note
 */

/**
 * @typedef {Object} Relationship 關係
 * @property {string} id
 * @property {string} workId
 * @property {string} sourceId      Character.id
 * @property {string} targetId      Character.id
 * @property {boolean} directed     true = source → target 單向
 * @property {RelationPhase[]} phases  至少 1 筆
 * @property {string} createdAt
 * @property {string} updatedAt
 */

/**
 * @typedef {Object} TimelineEvent 時間線事件
 * @property {string} id
 * @property {string} workId
 * @property {string} title
 * @property {string} description
 * @property {string} storyTimeLabel
 * @property {number} storyTimeSort
 * @property {string|null} chapterId   null = 背景事件
 * @property {number} orderInChapter   同章內排序，從 1 起
 * @property {string[]} characterIds
 * @property {string[]} tags
 * @property {string} createdAt
 * @property {string} updatedAt
 */

/**
 * @typedef {Object} Foreshadow 伏筆
 * @property {string} id
 * @property {string} workId
 * @property {string} title
 * @property {string} description
 * @property {string} plantedChapterId
 * @property {string|null} plannedPayoffChapterId
 * @property {string|null} payoffChapterId  status = 'resolved' 時必填
 * @property {'open'|'resolved'|'abandoned'} status
 * @property {string[]} characterIds
 * @property {string[]} eventIds
 * @property {string} payoffNote
 * @property {string} createdAt
 * @property {string} updatedAt
 */
```

### 5.2 關係類型常數 `RELATION_TYPES`

| type | 中文 | 顏色 | 新增時預設有方向 |
|---|---|---|---|
| family | 親情 | `#F59E0B` | 否 |
| romance | 戀人 | `#EC4899` | 否 |
| crush | 暗戀 | `#F472B6` | 是 |
| friend | 朋友 | `#10B981` | 否 |
| ally | 盟友 | `#3B82F6` | 否 |
| rival | 對手 | `#8B5CF6` | 否 |
| enemy | 敵對 | `#EF4444` | 否 |
| mentor | 師徒 | `#14B8A6` | 是 |
| subordinate | 從屬 | `#64748B` | 是 |
| other | 其他 | `#9CA3AF` | 否 |
| ended | 關係結束 | （不顯示） | — |

> 顏色需在淺色與深色背景下都清楚可見；若需微調，需同步更新此表。

### 5.3 IndexedDB 結構
- 資料庫名稱：`inkweave`，版本：`1`
- Object stores（`keyPath: 'id'`，皆建立 `workId` 索引）：

| store | 內容 |
|---|---|
| `works` | Work（無 `workId` 索引） |
| `chapters` | Chapter |
| `characters` | Character |
| `images` | StoredImage |
| `relationships` | Relationship |
| `events` | TimelineEvent |
| `foreshadows` | Foreshadow |

**底層存取函式**（以 Promise 包裝原生 IndexedDB）：
`dbOpen()`、`dbGetAll(store)`、`dbPut(store, obj)`、`dbDelete(store, id)`、`dbTx(storeNames, async (tx) => {...})`（多表操作用同一個 transaction）。

### 5.4 資料層（Repository）
- **啟動時**將所有 store 讀進記憶體中的 `state` 物件（`state.works`、`state.chapters`…），畫面一律從 `state` 讀取。
- **寫入時**由資料層函式同時更新 `state` 與 IndexedDB，完成後呼叫 `render()` 重繪（自動儲存除外，見 4.4）。
- 每種資料提供：`createX(input)`、`updateX(id, patch)`、`removeX(id)`，以及查詢函式，例：`getChaptersByWork(workId)`（已依 order 排序）。
- `create` / `update` 自動填入 `createdAt` / `updatedAt`，並同步更新所屬作品的 `updatedAt`。
- 連動刪除、匯入等多表操作必須使用 `dbTx()`；失敗時不得留下部分寫入的資料，並重新從 IndexedDB 載入 `state`。

---

## 6. 刪除連動規則

| 刪除對象 | 連動處理 |
|---|---|
| 作品 | 刪除該作品所有章節、角色、圖片、關係、事件、伏筆 |
| 章節 | **若有伏筆的 `plantedChapterId` 指向此章，阻止刪除**，提示使用者先處理這些伏筆。否則：事件的 `chapterId` → `null`；關係階段的 `startChapterId` 指向此章者 → 改為下一章（沒有下一章則刪除該階段，若關係因此沒有任何階段則刪除整條關係）；伏筆的 `plannedPayoffChapterId` / `payoffChapterId` 指向此章 → `null`（若 `resolved` 的伏筆因此缺少回收章，狀態改回 `open`）；`Work.currentChapterId` 指向此章 → `null`；剩餘章節 `order` 重新正規化 |
| 角色 | 刪除所有 `sourceId` 或 `targetId` 為此角色的關係；從所有事件、伏筆的 `characterIds` 移除；刪除其頭像圖片 |
| 事件 | 從所有伏筆的 `eventIds` 移除 |
| 關係階段 | 若為最後一個階段，連同整條關係刪除（對話框需說明） |

刪除確認對話框需列出受影響的數量，例：「刪除此角色將同時移除 3 條關係，並從 5 個事件中移除。」

---

## 7. 核心計算邏輯

以下為純函式（不讀寫 DOM 與資料庫），集中放在程式的「計算邏輯」區塊。

### 7.1 某章節時的關係狀態 `getActivePhase(relationship, chapterOrder, chapters)`
1. 將 `phases` 依起始章節的 `order` 排序（`startChapterId = null` 視為 order 0）。
2. 取「起始 order ≤ `chapterOrder`」的最後一個階段。
3. 若沒有符合的階段（關係尚未開始），或該階段 `type = 'ended'`，回傳 `null`（圖上不顯示）。

### 7.2 倒敘判斷 `markFlashbacks(eventsInNarrativeOrder)`
依敘事順序逐一檢查：若事件的 `storyTimeSort` **小於**目前為止出現過的最大值，標記為倒敘。`chapterId = null` 的事件不參與判斷。

### 7.3 伏筆逾期 `isOverdue(foreshadow, work, chapters)`
令 `current` = `currentChapterId` 對應的 order（未設定則為最大 order）。
`status = 'open'` 且符合**任一條件**即為逾期：
- `current − planted.order ≥ work.foreshadowWarnThreshold`
- 有設定 `plannedPayoffChapterId`，且 `current > plannedPayoff.order`

另提供 `chaptersSincePlanted(foreshadow, work, chapters) = current − planted.order`，用於顯示「已埋 N 章」。

### 7.4 角色出場章節 `getCharacterChapters(characterId, events, chapters)`
回傳該角色參與的事件中，所有不為 `null` 的 `chapterId`，去除重複並依 order 排序。

### 7.5 自我檢查【選做】
網址加上 `?selftest` 時，在主控台以 `console.assert` 執行 7.1～7.4 的測試案例（每個函式至少 3 個案例），並輸出「自我檢查：通過 N／共 M」。

---

## 8. 匯出／匯入【必做】

### 8.1 匯出格式
檔名：全部匯出為 `inkweave-backup-YYYYMMDD-HHmm.json`；單一作品為 `inkweave-<作品標題>-YYYYMMDD.json`。
以 `Blob` + `URL.createObjectURL` + `<a download>` 觸發下載。

```json
{
  "app": "inkweave",
  "schemaVersion": 1,
  "exportedAt": "2026-10-09T12:00:00.000Z",
  "works": [],
  "chapters": [],
  "characters": [],
  "images": [],
  "relationships": [],
  "events": [],
  "foreshadows": []
}
```
各陣列內容與第 5 節資料結構完全相同（圖片本身就是 data URL，可直接匯出）。

### 8.2 匯入規則
1. 驗證 `app === "inkweave"` 且 `schemaVersion` 為支援的版本，否則顯示錯誤並中止。
2. **一律以新副本匯入，不覆蓋既有資料**：為所有紀錄產生新 ID，並同步替換所有參照（`workId`、`chapterId`、`characterIds`、`eventIds`、`avatarImageId`、`sourceId`、`targetId`、`phases[].startChapterId`、`currentChapterId` 等）。
3. 作品標題若與既有作品重複，加上「（匯入）」後綴。
4. 整個匯入在單一 transaction 中完成；任何錯誤則全部取消，並顯示錯誤訊息。
5. 完成後顯示「已匯入 N 部作品」。

> 範例作品的載入也使用同一套「產生新 ID 並替換參照」的流程。

---

## 9. 範例作品【必做】

以常數 `SAMPLE_WORK` 寫在程式中（格式與匯出檔相同）。內容為**原創設定**，不得使用任何既有作品的內容。

**作品**：《霧港信使》（奇幻冒險），目前章節設為第 8 章，逾期門檻 5。

**章節（10 章）**：從主角收到神秘信件開始，到第 10 章揭露寄件人身分。

**角色（6 位，皆有自訂欄位，不附頭像圖片）**
- 林霧（主角，信使）、沈遙（青梅竹馬）、白鴉（神秘委託人）、顧長風（信使公會會長）、阿七（街頭情報販子）、夜鶯（敵對組織殺手）

**關係（至少 7 條，至少 2 條有階段變化）**
- 林霧 ↔ 顧長風：第 1 章起「師徒」→ 第 7 章起「敵對」
- 夜鶯 ↔ 林霧：第 3 章起「敵對」→ 第 9 章起「盟友」
- 沈遙 → 林霧：「暗戀」（有方向）
- 其餘自行設計

**時間線事件（至少 12 個）**
- 至少 3 個倒敘事件（例：第 6 章才揭露的「十年前的港口大火」）
- 至少 1 個背景事件（`chapterId = null`）
- 故事時間使用架空曆法，例：「霧曆 88 年冬」

**伏筆（至少 6 個）**
- 已回收 2 個、已放棄 1 個、未回收 3 個
- 未回收中至少 1 個**逾期**（例：第 1 章埋下的「信封上的鴉羽印記」）

---

## 10. 程式結構與部署

### 10.1 `index.html` 內部結構
```
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>墨滴 InkWeave｜小說設定工坊</title>
  <style>
    /* 1. CSS 變數（淺色／深色） */
    /* 2. 基礎樣式與版面（側邊欄、頂部列、內容區） */
    /* 3. 共用元件（按鈕、表單、卡片、對話框、徽章、toast） */
    /* 4. 各頁面樣式（依頁面分段） */
    /* 5. 響應式（@media） */
  </style>
</head>
<body>
  <div id="app"></div>
  <dialog id="modal"></dialog>
  <div id="toast"></div>
  <script>
    'use strict';
    // ===== 1. 常數（RELATION_TYPES 等） =====
    // ===== 2. 工具函式（escapeHtml、uuid、debounce、formatDate、downloadJson） =====
    // ===== 3. IndexedDB 底層存取 =====
    // ===== 4. 資料層（state 與 create/update/remove） =====
    // ===== 5. 計算邏輯（第 7 節） =====
    // ===== 6. 匯出／匯入、範例作品 =====
    // ===== 7. 共用 UI（對話框、toast、章節選單、頭像） =====
    // ===== 8. 各頁面 render 函式 =====
    // ===== 9. 路由 =====
    // ===== 10. 事件委派 =====
    // ===== 11. 啟動（init） =====
  </script>
</body>
</html>
```

### 10.2 GitHub 儲存庫內容
```
（儲存庫根目錄）
├─ index.html      ← 作品本體（唯一程式檔）
├─ SPEC.md         ← 本規格書
├─ README.md
└─ screenshots/    ← 介面截圖（README 與繳交使用）
```

### 10.3 部署到 GitHub Pages
1. 將檔案推送到 `main` 分支。
2. 儲存庫 Settings → Pages → Source 選「Deploy from a branch」，Branch 選 `main`、資料夾選 `/ (root)`。
3. 等待約 1 分鐘，開啟 `https://<帳號>.github.io/<儲存庫名稱>/` 確認可正常操作。

### 10.4 README 內容
- 專案簡介與截圖
- 線上網址（GitHub Pages 連結）
- 功能列表
- 使用方式（直接開啟網址，或下載 `index.html` 用瀏覽器開啟）
- 資料儲存說明（存在本機瀏覽器，記得匯出備份）
- 註明本專案以 AI 輔助開發（vibe coding），並連結到 `SPEC.md`

---

## 11. 開發里程碑

每個里程碑可作為一次（或數次）AI 開發對話的單位。完成後在此勾選，並在〈附錄 B〉記錄。

### M0 骨架
- [x] `index.html` 可直接用瀏覽器開啟，無主控台錯誤
- [x] 依 10.1 的區塊結構建立程式骨架
- [x] 側邊欄、頂部列、手機漢堡選單
- [x] Hash 路由：所有頁面可切換（先顯示頁面標題即可），未知路徑顯示 404
- [x] 共用元件：按鈕、對話框、toast

### M1 資料層
- [x] IndexedDB 底層函式、啟動載入 `state`
- [x] 所有資料的 create / update / remove
- [x] 第 6 節刪除連動規則
- [x] 第 7 節計算邏輯

### M2 作品與章節
- [x] 作品列表：新增、卡片顯示、空狀態
- [x] 作品設定：編輯、目前章節、逾期門檻、刪除
- [x] 章節管理：新增、編輯、刪除（含阻止規則）、上下移排序、展開顯示關聯項目

### M3 角色卡
- [x] 角色列表、搜尋、標籤篩選
- [x] 詳細頁全部欄位、自動儲存（輸入時不失焦）、秘密遮罩、自訂欄位
- [x] 頭像上傳、壓縮、更換、移除
- [x] 自動關聯區（無資料時顯示空狀態）

### M4 伏筆追蹤器
- [x] 新增／編輯／刪除、狀態切換、快捷回收
- [x] 分頁篩選、逾期標示與置頂、「已埋 N 章」
- [x] 總覽頁統計與逾期清單

### M5 時間線
- [x] 事件新增／編輯／刪除、相鄰數字提示
- [x] 故事順序檢視
- [x] 敘事順序檢視（含倒敘徽章）
- [x] 角色、標籤篩選

### M6 角色關係圖
- [x] SVG 節點、連線、顏色、箭頭、標籤、圖例、雙關係弧線
- [x] 節點拖曳（滑鼠與觸控）並儲存位置
- [x] 章節滑桿與「顯示全部」
- [x] 關係與階段的新增／編輯／刪除

### M7 匯出入與範例
- [x] 匯出全部、匯出單一作品
- [x] 匯入（新 ID 對應、單一 transaction、錯誤處理）
- [x] 範例作品《霧港信使》符合第 9 節所有條件
- [x] 載入範例後，所有頁面皆有內容可展示

### M8 選做功能
- [x] 雙軸對照圖（4.6）
- [x] 全域搜尋（4.9）
- [x] 深色模式（4.10）
- [x] 伏筆跨度圖（4.7）
- [x] 總覽「最近更新」（4.2）
- [x] 關係圖自動排列、縮放平移（4.5）
- [x] 章節拖曳排序（4.3）
- [x] 自我檢查（7.5）

### M9 收尾與發布
- [x] 第 12 節手動驗收清單全部通過
- [x] 手機寬度（375px）逐頁檢查
- [ ] 上傳 GitHub、開啟 GitHub Pages，線上網址可正常操作
- [ ] README、截圖完成（README 已完成，截圖待補）

---

## 12. 手動驗收清單

發布前依序操作，全部通過才算完成：

1. 以無痕視窗開啟網址，首頁顯示空狀態，主控台無錯誤。
2. 點「載入範例作品」，進入作品後，總覽顯示統計數字與至少 1 個逾期伏筆。
3. 關係圖拖動章節滑桿從第 1 章到第 10 章，林霧與顧長風的連線從「師徒」變成「敵對」。
4. 時間線切到「敘事順序」，至少 3 個事件顯示「↩ 倒敘」。
5. 新增一個角色並上傳頭像，重新整理頁面後資料與頭像仍在。
6. 在角色詳細頁連續輸入文字，輸入框不會失去焦點，並顯示「已儲存」。
7. 刪除一個有關係的角色，確認對話框顯示受影響數量，刪除後關係圖不再出現相關連線。
8. 嘗試刪除埋有伏筆的章節，系統阻止並說明原因。
9. 匯出該作品，再匯入，作品列表出現「（匯入）」副本，內容完整（含頭像）。
10. 將逾期門檻改為 20，總覽的逾期伏筆數量隨之變化。
11. 在角色名稱輸入 `<b>測試</b>`，畫面顯示原始文字而非粗體（XSS 防護）。
12. 手機寬度下所有頁面可正常操作、無水平捲動。

---

## 13. 未來擴充（本版不實作）

- **矛盾偵測**：角色增加「死亡章節」欄位，若在死亡後的事件中出現則警告。
- **出場統計圖**：每個角色在各章節的出場熱度，提醒配角消失太久。
- **雲端同步**：登入並跨裝置同步。
- **地點管理**：地點卡與地圖。

---

## 附錄 A：給 AI 的指令範本

開始一個里程碑時：

```
請先閱讀 SPEC.md，特別是第 0 節的開發規則。
現在要實作 M3「角色卡」，請依照第 4.4 節完成，並修改現有的 index.html。
完成後逐項檢查第 11 節 M3 的驗收條件，並告訴我在附錄 B 要新增的紀錄。
不要修改與本里程碑無關的部分。
```

修正問題時：

```
請先閱讀 SPEC.md。目前的問題是：<描述現象與重現步驟>。
依 SPEC.md 第 <X> 節，預期行為應該是：<描述>。
請找出原因並修正 index.html，不要改動其他功能。
```

調整設計或功能時：

```
請先閱讀 SPEC.md。我想調整：<描述想改的地方與原因>。
請先說明這個修改會影響規格書的哪些章節，等我確認後再同時修改 SPEC.md 與 index.html。
```

---

## 附錄 B：AI 協作紀錄

記錄每一次與 AI 討論、修改的過程，作為「反覆調整與完善作品」的證明。

| # | 日期 | 我的想法或要求 | AI 的回應或修改 | 結果 |
|---|---|---|---|---|
| 1 | 2026-10-09 | 我喜歡小說，想做小說類的 Web App | 提出讀者、寫作者、互動體驗三類共 8 個方向 | 選擇讀者與寫作者方向 |
| 2 | 2026-10-09 | 延伸讀者與寫作者方向 | 提出「讀寫書房」概念，用靈感庫串連閱讀室與寫作室，並列出延伸功能 | — |
| 3 | 2026-10-09 | 詢問資料要存在哪裡、能否放上 GitHub | 比較 localStorage、IndexedDB、雲端資料庫，建議用瀏覽器儲存並加上匯出備份 | 確定純前端、資料存本機 |
| 4 | 2026-10-09 | 決定專注寫作室：角色卡、關係圖、時間線、伏筆追蹤器 | 建議以「章節」作為串連骨架、關係隨章節變化、故事時間與敘事順序分開、加入範例作品與匯出備份 | 全部採用 |
| 5 | 2026-10-09 | 請 AI 產生規格書 | 產出規格書 v1（Vite + React + TypeScript） | SPEC v1.0 |
| 6 | 2026-10-09 | 對照作業要求檢查規格書 | 發現作業要求單一 `index.html`，與 v1 技術選型衝突 | 決定改版 |
| 7 | 2026-10-09 | 依作業要求修改規格書 | 改為原生 JS 單檔、自寫 IndexedDB 存取與 SVG 關係圖、功能分必做／選做、新增手動驗收清單與本協作紀錄 | SPEC v2.0 |
| 8 | 2026-10-09 | 依 SPEC.md 以 Vibe Coding 實作 index.html | AI 依里程碑 M0～M8 完成所有必做與選做功能：單一 index.html、原生 JS、自寫 IndexedDB 資料層、SVG 關係圖與雙軸對照圖、範例作品《霧港信使》 | M0～M8 完成 |
| 9 | 2026-10-09 | 逐項執行第 12 節手動驗收清單 | 在瀏覽器實測 12 項驗收（含 375px 手機寬度逐頁檢查、XSS、匯出再匯入），並以 `?selftest` 執行自我檢查 | 12 項全部通過；自我檢查 19／19 |
| 10 | 2026-10-09 | 修正測試中發現的問題 | ① 範例資料時間戳記相同，重新整理後角色順序會亂 → 改為依序遞增的時間戳記 ② 關係圖在手機直式畫面太小 → 依節點範圍自動調整 viewBox ③ 總覽逾期清單標題與說明擠在同一行 → 修正樣式 | 已修正 |
| 11 | 2026-10-09 | 確認 AI 是否加入規格外的功能 | AI 主動列出額外加入的小功能：關係面板「刪除整條關係」、敘事順序中同章事件上移／下移、側邊欄數量與逾期徽章、表單驗證（回收章不得早於埋設章、同一關係不可有同章起始的階段、第一個階段不可為「關係結束」） | 待確認（保留或移除） |
| 12 | | | | |
