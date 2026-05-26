# 點餐管理系統

咖啡廳專用的輕量 POS 點餐系統，使用 Firebase Realtime Database 儲存資料，部署在 GitHub Pages 上，無需後端伺服器。

---

## 功能總覽

| 功能 | 說明 |
|---|---|
| 餐桌管理 | 新增 / 刪除餐桌，追蹤狀態（空桌 → 點餐中 → 已出餐 → 已結帳） |
| 菜單管理 | 品項分類、定價、選項設定（如溫度、甜度） |
| 語音點餐 | 中文語音辨識，支援模糊比對與同音字別名 |
| 快速點餐 | 菜單快選 chips，按分類分組顯示 |
| 用餐計時 | 入座後自動計時，90 分鐘變橘、120 分鐘變紅並彈窗提醒 |
| 日結功能 | 執行日結，記錄每日營業額與品項銷量 |
| 營業報表 | 週報 / 月報 / 自訂區間，長條圖顯示趨勢 |
| 品項排行 | 依杯數或營業額排列各品項銷售排名 |
| 登入驗證 | Firebase Email / Password 驗證，未登入無法存取資料 |
| PWA 支援 | 可加入手機桌面，全螢幕使用 |

---

## 技術架構

- **前端**：純 HTML + CSS + Vanilla JavaScript（無框架）
- **資料庫**：Firebase Realtime Database（即時同步）
- **驗證**：Firebase Authentication（Email / Password）
- **圖表**：Chart.js v4.4.0
- **語音**：Web Speech API（`SpeechRecognition`，需 Chrome / Edge）
- **部署**：GitHub Pages（免費靜態主機）

---

## 建置步驟

### 1. Fork / Clone 這個 Repo

```bash
git clone https://github.com/你的帳號/ordermenu.git
cd ordermenu
```

或直接在 GitHub 上點「Fork」。

---

### 2. 建立 Firebase 專案

1. 前往 [Firebase Console](https://console.firebase.google.com/)
2. 點「新增專案」→ 輸入專案名稱（如 `my-cafe-pos`）
3. 可關閉 Google Analytics（非必要）→ 點「建立專案」

---

### 3. 建立 Realtime Database

1. 左側選單 → **Build → Realtime Database**
2. 點「建立資料庫」
3. 選擇**位置**（建議亞洲用 `asia-southeast1`）
4. 模式選「**以鎖定模式啟動**」→ 點「啟用」

**設定安全性規則：**

進入 Realtime Database → 選「規則」分頁，將內容改為：

```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```

點「發布」儲存。

---

### 4. 啟用 Firebase Authentication

1. 左側選單 → **Build → Authentication**
2. 點「開始使用」
3. 選「Sign-in method」分頁
4. 點「電子郵件 / 密碼」→ 啟用第一個開關 → 儲存

**新增使用者帳號：**

1. 選「Users」分頁
2. 點「新增使用者」
3. 輸入 Email 和密碼 → 新增

---

### 5. 取得 Firebase 設定

1. 左側齒輪圖示 → **專案設定**
2. 往下滑到「您的應用程式」區塊
3. 若尚未新增應用程式，點「`</>`」（Web 應用程式）圖示
4. 輸入應用程式暱稱 → 點「註冊應用程式」
5. 複製 `firebaseConfig` 物件：

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  databaseURL: "https://your-project-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "000000000000",
  appId: "1:000000000000:web:xxxxxxxxxxxx"
};
```

---

### 6. 更新 app.js

打開 `app.js`，找到最上方的 `firebaseConfig`，替換成上一步複製的內容：

```javascript
// ── Firebase ──
const firebaseConfig = {
  apiKey: "貼上你的 apiKey",
  authDomain: "貼上你的 authDomain",
  databaseURL: "貼上你的 databaseURL",
  projectId: "貼上你的 projectId",
  storageBucket: "貼上你的 storageBucket",
  messagingSenderId: "貼上你的 messagingSenderId",
  appId: "貼上你的 appId"
};
```

> ⚠️ **注意**：`databaseURL` 必須填寫，否則無法連線到 Realtime Database。  
> 格式通常為：`https://[專案ID]-default-rtdb.[地區].firebasedatabase.app`

---

### 7. 部署到 GitHub Pages

**方法一：使用 GitHub 網頁介面**

1. 將修改後的 `app.js` 推送到你的 repo
2. 進入 repo → Settings → Pages
3. Source 選「Deploy from a branch」
4. Branch 選 `main`，資料夾選 `/ (root)` → 儲存
5. 幾分鐘後即可在 `https://你的帳號.github.io/ordermenu/` 存取

**方法二：使用指令**

```bash
git add .
git commit -m "設定 Firebase 設定"
git push origin main
```

---

### 8. 測試

1. 開啟你的 GitHub Pages 網址
2. 以步驟 4 建立的帳號登入
3. 新增一個餐桌，試著點餐看看

---

## 語音點餐說明

- 需使用 **Chrome 或 Edge** 瀏覽器
- 點「語音點餐」後說出品項，例如：`美式一杯拿鐵兩杯`
- 說完後確認 UI 顯示的品項是否正確，再按「全部加入訂單」

**提高辨識準確率的技巧：**

| 情境 | 建議 |
|---|---|
| 多品項 | 品名後接數量，如「美式一杯拿鐵兩杯」 |
| 有選項 | 一次只說一個品項，在同句說備註，如「拿鐵一杯少糖」 |
| 外來語品名 | 在菜單編輯中設定「語音別名」，填入語音實際聽到的字 |

---

## 自訂語音別名

若品名包含外來語或特殊字（如「荷芙莎」常被聽成「和服紗」）：

1. 開啟「菜單」→ 找到該品項 → 點編輯圖示
2. 在「語音別名」欄位填入語音實際聽到的字
3. 多個別名用逗號分隔，如：`和服紗、河芙莎`
4. 儲存後即生效

---

## 注意事項

- Firebase API Key 會公開在前端程式碼中，這是正常的。安全性由 Firebase Security Rules 和 Authentication 保護，只要 Rules 設定為 `auth != null`，未登入者即無法讀寫資料。
- 語音辨識功能需要麥克風權限，且僅支援 Chrome / Edge（不支援 Safari）。
- 排行報表從第一次「執行日結」後開始累積品項資料，舊的日結紀錄不含品項明細。
- 建議使用平板或桌機操作，手機版已針對 RWD 優化但螢幕較小。

---

## Firebase 資料結構

```
/cafe_orders/          ← 目前所有桌次狀態
  table_xxx/
    name: "吧台"
    status: "ordering"
    seatedAt: 1234567890
    items/
      item_xxx/
        name: "美式"
        price: 110
        qty: 1
        note: "少糖"
        done: false

/cafe_menu/            ← 菜單品項
  item_xxx/
    name: "拿鐵"
    price: 140
    category: "CLASSIC"
    options: [...]
    voiceAliases: ["拿鐵"]

/cafe_daily/           ← 每日日結紀錄
  2025-01-01/
    revenue: 2800
    tableCount: 8
    itemSales/
      美式: { qty: 5, revenue: 550 }

/cafe_settings/        ← 系統設定
  tablePresets: ["桌1","桌2","吧台","外帶"]

/cafe_custom_modifiers/ ← 自訂語音關鍵字清單
```
