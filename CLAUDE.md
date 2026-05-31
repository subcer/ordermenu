# 點餐管理系統 — 專案說明

咖啡廳專用輕量 POS，純前端部署在 GitHub Pages，資料存 Firebase Realtime Database。

## 技術架構

- **前端**：純 HTML + CSS + Vanilla JavaScript（無框架、無 npm）
- **資料庫**：Firebase Realtime Database v8.10.1（即時同步）
- **驗證**：Firebase Authentication（Email / Password）
- **圖表**：Chart.js v4.4.0
- **語音**：Web Speech API（Chrome / Edge）
- **部署**：GitHub Pages（main branch）
- **字型 / 圖示**：Google Fonts（Noto Sans TC、Manrope）、Material Symbols、Font Awesome

## 檔案結構

```
ordermenu/
├── index.html     # 所有 UI（單頁應用）
├── app.js         # 全部邏輯（~2000行）
├── style.css      # 全部樣式
├── favicon.svg    # 咖啡杯圖示
├── robots.txt     # 禁止爬蟲
└── CLAUDE.md      # 本文件
```

## Firebase 資料結構

```
/cafe_orders/          ← 餐桌即時狀態
  {tableId}/
    name: "吧台"
    status: "empty" | "ordering" | "served" | "paid"
    order: 10          ← 拖曳排序用
    seatedAt: timestamp
    paidAt: timestamp
    paidTotal: number
    paidFlag: boolean  ← 獨立收款確認（不關桌）
    items/
      {itemId}/
        name, price, qty, note, done: boolean

/cafe_menu/            ← 菜單品項
  {itemId}/
    name, price, category, order
    options: [{label, choices[]}]
    voiceAliases: string[]   ← 語音同音字別名
    hidden: boolean          ← 隱藏品項

/cafe_daily/           ← 日結紀錄
  {YYYY-MM-DD}/
    revenue, tableCount
    itemSales/
      {safeKey}/  ← 品名特殊字替換為底線
        name, qty, revenue

/cafe_settings/
  tablePresets: string[]   ← 快速新增桌名預設

/cafe_waitlist/        ← 候補名單
  {wlId}/
    name, phone, partySize, note
    addedAt: timestamp
    status: "waiting" | "calling"

/cafe_custom_modifiers/ ← 語音關鍵字（甜度/溫度）
```

## 重要設計原則

- **Firebase key 限制**：key 不能含 `. # $ / [ ]`，日結品項用 `item.name.replace(/[.#$\/\[\]]/g, '_')` 當 key，同時把原始 name 存在 value 裡
- **多路徑原子更新**：換桌等需要同時改多個節點時，用 `firebase.database().ref().update(updates)` 一次寫入
- **Auth 保護**：所有 Firebase 監聽器在 `onAuthStateChanged` 裡的 `if(user)` 內啟動，登出時全部 `.off()`
- **不使用任何 npm / build tool**，所有 CDN 直接在 index.html 的 `<script>` 載入

## 主要功能模組（app.js）

| 功能 | 相關函式 |
|---|---|
| 餐桌渲染 | `renderTables()`, `buildTableCard()` |
| 桌況 Modal | `openTableModal()`, `updateModalContent()`, `closeTableModal()` |
| 狀態按鈕 | `btnStartTimer`, `btnMarkServed`, `btnMarkPaidVisual`, `btnLeave` |
| 品項操作 | `addItem()`, `toggleItemDone()`, `deleteItem()` |
| 菜單快選 | `renderMenuPicker()` |
| 語音點餐 | `startVoiceOrder()`, `parseVoiceText()` |
| 菜單管理 | `renderMenuItemsList()`, `startEditMenuItem()`, `saveEditMenuItem()` |
| 拖曳排序 | `menuDragStart/Over/Drop()`, `tableDragStart/Over/Drop()` |
| 候補名單 | `renderWaitlist()`, `openWaitlistModal()`, `addWaitlistEntry()` |
| 換桌 | `openTransferModal()`, `executeTransfer()` |
| 日結 | `doSettlement()` |
| 報表 | `setReportMode()`, `renderRankingReport()` |
| 計時提醒 | `checkTableTimers()`, `elapsedMinutes()` |
| 桌名預設 | `renderTablePresets()`, `savePresets()` |

## 狀態流程

```
空桌 → [開始計時] → 點餐中 → [已出餐] → 已出餐 → [離開] → 已結帳(關桌)
                                          ↑
                               加點時自動退回點餐中

全部品項打勾 → 自動變已出餐
點已出餐按鈕 → 全部品項自動打勾

獨立 [已結帳] 按鈕 → 設 paidFlag=true，卡片變色但不關桌
```

## CSS 設計 Tokens（style.css 頂部）

```css
--primary: #984215       /* 主色咖啡褐 */
--green:   #446345       /* 已出餐 */
--orange:  #b85a2b       /* 點餐中 */
--red:     #ba1a1a       /* 危險操作 */
```

卡片狀態色：`status-empty / ordering / served / paid`  
收款確認色：`paid-flag-ordering`（琥珀黃）、`paid-flag-served`（青綠）

## 注意事項

- 語音辨識只支援 Chrome / Edge，需麥克風權限
- 品名含特殊字（如 `.`）的品項在日結時會轉換 key，`renderRankingReport` 要讀 `data.name` 而非直接用 key
- Modal 在平板橫式時改為置中顯示（`@media orientation:landscape and max-height:768px`）
- 所有 Modal 開啟時避免 auto-focus，防止平板軟鍵盤自動彈出
- `robots.txt` 已設定禁止所有爬蟲
