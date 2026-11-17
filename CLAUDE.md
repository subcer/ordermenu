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
├── app.js         # 全部邏輯（~2700行）
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
    fixed: boolean     ← 固定桌（離開時清空桌子，帳記到 cafe_daily_queue）
    seatedAt: timestamp
    paidAt: timestamp  ← 一般桌按「離開」的時間，日結依此分日期
    paidTotal: number
    items/
      {itemId}/        ← 數量 >1 加點時會拆成多筆 qty:1
        name, price, qty, note
        done: boolean  ← 已出餐打勾
        paid: boolean  ← 品項層級收款（部分收款用）

/cafe_daily_queue/     ← 固定桌離開後、尚未日結的帳
  {queueId}/
    tableId, tableName, total, items, seatedAt
    completedAt: timestamp  ← 日結依此分日期

/cafe_menu/            ← 菜單品項
  {itemId}/
    name, price, category, order
    options: [{label, choices[]}]
    voiceAliases: string[]   ← 語音同音字別名
    hidden: boolean          ← 隱藏品項

/cafe_daily/           ← 日結紀錄
  {YYYY-MM-DD}/
    date, settledAt, revenue, tableCount
    tables: [{name, total, fixed?}]
    itemSales/
      {safeKey}/  ← 品名特殊字替換為底線
        name, qty, revenue
    manual: true   ← 手動補登的紀錄
    note           ← 補登備註
    unmerged: [...]  ← 補登金額與系統不符時，系統算出的資料存這裡備查，不計入 revenue

/cafe_daily_backup/    ← 日結紀錄被累加/覆蓋前的舊版本
  {YYYY-MM-DD}/{timestamp}: 舊的 cafe_daily 紀錄

/cafe_settings/
  tablePresets: string[]   ← 快速新增桌名預設
  menuCategories: string[]
  tableSortMode: "manual" | "timer"
  settleLock: timestamp    ← 日結跨裝置鎖（60 秒後視為殘留）
  autoSettleDate: "YYYY-MM-DD"  ← 23:50 自動日結已執行的日期

/cafe_waitlist/        ← 候補名單
  {wlId}/
    name（姓＋稱呼）, lastName, title, phone, partySize, note
    addedAt: timestamp
    status: "waiting" | "calling"

/cafe_custom_modifiers/ ← 語音關鍵字（甜度/溫度）
```

## 重要設計原則

- **Firebase key 限制**：key 不能含 `. # $ / [ ]`，日結品項用 `item.name.replace(/[.#$\/\[\]]/g, '_')` 當 key，同時把原始 name 存在 value 裡
- **多路徑原子更新**：換桌、對調、日結等需要同時改多個節點時，用 `firebase.database().ref().update(updates)` 一次寫入
- **Auth 保護**：所有 Firebase 監聽器在 `onAuthStateChanged` 裡的 `if(user)` 內啟動，登出時全部 `.off()`
- **不使用任何 npm / build tool**，所有 CDN 直接在 index.html 的 `<script>` 載入
- **改 app.js / style.css 後要更新 index.html 的 `?v=` 版本號**，否則平板會讀到快取的舊檔

## 主要功能模組（app.js）

| 功能 | 相關函式 |
|---|---|
| 餐桌渲染 | `renderTables()`, `buildTableCard()` |
| 桌況 Modal | `openTableModal()`, `updateModalContent()`, `closeTableModal()` |
| 狀態按鈕 | `btnStartTimer`, `btnMarkServed`, `btnMarkPaidVisual`, `btnLeave`, `btnRevertPaid` |
| 品項操作 | `addItem()`, `addItemUnits()`, `toggleItemDone()`, `toggleItemPaid()`, `deleteItem()` |
| 收款／找零 | `paidSummary()`, `updateTotalBar()`, `updateChangeCalc()` |
| 菜單快選 | `renderMenuPicker()`, `openOptionPicker()` |
| 語音點餐 | `initSpeechRecognition()`, `parseVoiceText()` |
| 菜單管理 | `renderMenuItemsList()`, `startEditMenuItem()`, `saveEditMenuItem()` |
| 拖曳排序 | `menuDragStart/Over/Drop()`, `tableDragStart/Over/Drop()` |
| 候補名單 | `renderWaitlist()`, `openWaitlistModal()`（新增／編輯）, `addWaitlistEntry()` |
| 換桌／對調 | `openTransferModal()`, `executeTransfer()`, `executeSwap()` |
| 日結 | `collectPending()`, `runSettlement()`, `planDailyWrite()`, `openSettlementModal()`, `doSettlement()` |
| 自動日結 | `checkCatchupSettlement()`（開啟時補前幾天）, `checkAutoSettlement()`（23:50） |
| 補登日結 | `saveBackfill()`, `renderHistoryList()` |
| 固定桌 | `buildQueueCard()`, `revertQueueEntry()` |
| 報表 | `setReportMode()`, `renderRankingReport()` |
| 計時提醒 | `checkTableTimers()`, `showTimerAlert()`, `elapsedMinutes()` |
| 確認視窗 | `showConfirm()`, `closeConfirm()` |
| 桌名預設 | `renderTablePresets()`, `savePresets()` |

## 狀態流程

```
空桌 → [開始計時] → 點餐中 → [已出餐] → 已出餐 → [離開] → 已結帳(關桌)
                                          ↑                 │
                               加點時自動退回點餐中      [退回] → 已出餐

全部品項打勾 → 自動變已出餐
點已出餐按鈕 → 全部品項自動打勾

收款是品項層級（item.paid），不影響桌況：
  每個品項的 $ 按鈕 → 單品收款
  [已結帳] 按鈕 → 全部品項一次標記收款／取消
  離開時若有未收款會跳確認
```

## 日結規則

- 待日結的帳 = `cafe_orders` 裡 status 為 paid 的桌 ＋ `cafe_daily_queue` 全部
- 依離開時間（`paidAt` / `completedAt`）分組，各自記到該日期
- 該日期已有紀錄：一般紀錄累加；補登紀錄金額相同改用系統明細，不同則保留補登、系統資料存 `unmerged`
- 寫入前把舊紀錄存到 `cafe_daily_backup`，寫入後移除已日結的桌與 queue，所以不會重複計算
- 開啟系統時若有昨天以前的待日結帳，先跳預覽讓使用者確認；23:50 後由一台裝置自動日結

## CSS 設計 Tokens（style.css 頂部）

```css
--primary: #984215       /* 主色咖啡褐 */
--green:   #446345       /* 已出餐 */
--orange:  #b85a2b       /* 點餐中 */
--red:     #ba1a1a       /* 危險操作 */
--blue:    #2563a8       /* 部分收款 */
```

卡片狀態色：`status-empty / ordering / served / paid`  
全部收款色：`paid-flag-ordering`（琥珀黃）、`paid-flag-served`（青綠）  
部分收款色：`partial-paid-ordering / partial-paid-served`（藍）

## 注意事項

- 語音辨識只支援 Chrome / Edge，需麥克風權限
- 品名含特殊字（如 `.`）的品項在日結時會轉換 key，`renderRankingReport` 要讀 `data.name` 而非直接用 key
- Modal 在平板橫式時改為置中顯示（`@media (orientation: landscape) and (max-height: 768px)`）
- 所有 Modal 開啟時避免 auto-focus，防止平板軟鍵盤自動彈出
- 用餐提醒若遇到確認視窗開著，會排隊等視窗關閉再顯示（`_queuedTimerAlerts`）
- `robots.txt` 已設定禁止所有爬蟲
