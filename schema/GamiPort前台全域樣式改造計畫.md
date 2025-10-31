# GamiPort 前台全域樣式改造計畫

**建立日期**: 2025-10-30
**目標**: 在不影響其他 Area 的前提下，為 MiniGame Area 提供專屬導航入口
**原則**: 只修改全域共用樣式檔案，MiniGame Area 內部保持獨立

---

## 一、改造目標概述

根據圖片標註（`3 - 複製.jpg`），本次改造需要達成以下目標：

### 1.1 左側 Sidebar 改造
- **「小遊戲」項目優化**:
  - 在「小遊戲」右側添加 **「此團隊」** 標籤（小字體，灰色，表示這是本團隊負責的區域）
  - 在「小遊戲」下方展開 **兩個子選單**：
    1. **每日簽到** - 使用 Bootstrap Icon `bi bi-calendar-check-fill`
    2. **寵物冒險** - 使用自訂 SVG `wwwroot/images/paw.svg`

- **Icon 更新** - 替換為更語義化的圖示：
  - 總覽：`overview.svg`
  - 商城：`bi bi-shop-window`
  - 小遊戲：保持原 `bi bi-grid`（或可改為 `bi bi-joystick`）
  - 交友：`add_friends.svg`
  - 論壇：`forum_icon.svg`
  - 使用者：`bi bi-person`

### 1.2 右上角 TopbarLevel1 改造
- **新增「我的錢包」連結**：
  - 位置：登入按鈕左側
  - Icon：`bi bi-wallet`
  - 功能：顯示通知徽章（紅色數字，表示錢包異動通知）
  - 連結目標：`/MiniGame/Wallet/Index`（導向 MiniGame Area 的錢包頁面）

---

## 二、改造範圍與影響分析

### 2.1 需要修改的檔案

| 檔案路徑 | 修改內容 | 影響範圍 | 風險等級 |
|---------|---------|---------|---------|
| `Views/Shared/_Sidebar.cshtml` | 1. 更新所有 icon<br>2. 為「小遊戲」添加團隊標籤<br>3. 添加兩個子選單項目 | **全域** - 影響所有頁面的左側導航 | ⚠️ 中等 |
| `Views/Shared/_TopbarLevel1.cshtml` | 添加「我的錢包」連結與通知徽章 | **全域** - 影響所有頁面的頂部導航 | ⚠️ 中等 |
| `wwwroot/css/sidebar.css` | 添加子選單縮排樣式、團隊標籤樣式 | 全域 Sidebar 樣式 | ⚠️ 低 |
| `wwwroot/css/topbar1.css` | 添加錢包通知徽章樣式 | 全域 Topbar 樣式 | ⚠️ 低 |

### 2.2 不需要修改的區域
- ✅ `Areas/MiniGame/**` - MiniGame Area 內部檔案完全不受影響
- ✅ 其他 Area（Forum, OnlineStore, social_hub, MemberManagement）
- ✅ `_Layout.cshtml` - 主版面配置不變
- ✅ `_TopbarLevel2.cshtml`, `_Footer.cshtml` - 其他共用組件不變

### 2.3 影響評估

#### 正面影響
1. **導航更直觀** - 用戶可直接從 Sidebar 進入簽到和寵物冒險功能
2. **錢包可見性提升** - 頂部錢包連結方便用戶隨時查看點數
3. **團隊識別** - 「此團隊」標籤清楚標示 MiniGame Area 的負責歸屬
4. **語義化 Icon** - 新 icon 更符合功能意義（如錢包用錢包圖示、論壇用論壇圖示）

#### 潛在風險與緩解
1. **風險**: 其他團隊成員可能不理解為何 Sidebar 只有「小遊戲」有子選單
   - **緩解**: 「此團隊」標籤明確標示，且其他 Area 若需要也可仿照添加

2. **風險**: 錢包連結導向 MiniGame Area，可能與其他 Area 的導航邏輯不一致
   - **緩解**: 這是正常的跨 Area 導航，符合專案設計原則

3. **風險**: CSS 樣式可能影響其他 Area 的未來擴展
   - **緩解**: 使用特定 class 命名（如 `.minigame-submenu`, `.team-badge`），避免全局影響

---

## 三、詳細實作計畫

### 階段一：更新 Sidebar 的 Icon（低風險）

**檔案**: `Views/Shared/_Sidebar.cshtml`

**變更內容**:
- 總覽：`<i class="bi bi-speedometer2"></i>` → `<img src="~/images/overview.svg" alt="" width="16">`
- 商城：`<i class="bi bi-collection"></i>` → `<i class="bi bi-shop-window"></i>`
- 交友：`<i class="bi bi-diagram-3"></i>` → `<img src="~/images/add_friends.svg" alt="" width="16">`
- 論壇：`<i class="bi bi-grid"></i>` → `<img src="~/images/forum_icon.svg" alt="" width="16">`
- 使用者：`<i class="bi bi-grid"></i>` → `<i class="bi bi-person"></i>`

**理由**: 這些是純視覺優化，不影響功能邏輯。

---

### 階段二：添加「小遊戲」的團隊標籤與子選單（中風險）

**檔案**: `Views/Shared/_Sidebar.cshtml`

**變更內容**:

#### 2.1 在「小遊戲」項目添加團隊標籤
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="Home" asp-action="Index">
    <i class="bi bi-grid"></i>
    <span class="label">小遊戲</span>
    <span class="badge bg-secondary ms-auto team-badge">此團隊</span>
</a>
```

#### 2.2 在「小遊戲」項目後添加子選單
```html
<!-- MiniGame 子選單 -->
<div class="minigame-submenu ms-3">
    <a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
       asp-area="MiniGame" asp-controller="SignIn" asp-action="Index">
        <i class="bi bi-calendar-check-fill"></i>
        <span class="label">每日簽到</span>
    </a>
    <a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
       asp-area="MiniGame" asp-controller="Pet" asp-action="Index">
        <img src="~/images/paw.svg" alt="" width="16" class="flex-shrink-0">
        <span class="label">寵物冒險</span>
    </a>
</div>
```

**理由**:
- 子選單方便用戶直接進入簽到和寵物功能，無需先進入 MiniGame 首頁
- 團隊標籤清楚標示這是本團隊負責的區域
- 使用 `asp-area`, `asp-controller`, `asp-action` 確保路由正確

**影響**:
- 所有頁面的 Sidebar 都會顯示這兩個子選單項目
- 如果用戶不在 MiniGame Area，點擊會導航到 MiniGame Area

---

### 階段三：添加「我的錢包」連結到右上角（中風險）

**檔案**: `Views/Shared/_TopbarLevel1.cshtml`

**變更內容**:

在登入按鈕前添加「我的錢包」連結：

```html
<ul class="navbar-nav ms-auto align-items-lg-center">
    <li class="nav-item d-none d-md-block"><a class="nav-link" href="#">連結 A</a></li>
    <li class="nav-item d-none d-md-block"><a class="nav-link" href="#">連結 B</a></li>

    <!-- ⭐ 新增：我的錢包 -->
    <li class="nav-item d-none d-md-block position-relative">
        <a class="nav-link d-flex align-items-center gap-1"
           asp-area="MiniGame" asp-controller="Wallet" asp-action="Index">
            <i class="bi bi-wallet"></i>
            <span>我的錢包</span>
            <!-- 通知徽章（需後端提供未讀數量） -->
            <span class="badge bg-danger rounded-pill wallet-badge">3</span>
        </a>
    </li>

    <li class="nav-item ms-2"><a class="btn btn-primary" href="#">登入</a></li>
</ul>
```

**理由**:
- 錢包是高頻功能，放在頂部方便用戶隨時查看點數
- 通知徽章可顯示錢包異動（如獲得點數、優惠券到期提醒）

**影響**:
- 所有頁面頂部都會顯示「我的錢包」連結
- 需要後端 API 提供未讀通知數量（暫時先寫死數字 3 作為展示）

---

### 階段四：添加必要的 CSS 樣式（低風險）

#### 4.1 Sidebar 樣式 (`wwwroot/css/sidebar.css`)

添加以下樣式：

```css
/* MiniGame 子選單縮排 */
.minigame-submenu {
    padding-left: 1rem;
}

.minigame-submenu .list-group-item {
    padding: 0.5rem 0.75rem;
    font-size: 0.9rem;
    border-left: 3px solid transparent;
}

.minigame-submenu .list-group-item:hover {
    border-left-color: var(--bs-primary);
    background-color: rgba(var(--bs-primary-rgb), 0.05);
}

.minigame-submenu .list-group-item.active {
    border-left-color: var(--bs-primary);
    background-color: rgba(var(--bs-primary-rgb), 0.1);
}

/* 團隊標籤 */
.team-badge {
    font-size: 0.7rem;
    padding: 0.15rem 0.4rem;
}
```

#### 4.2 Topbar 樣式 (`wwwroot/css/topbar1.css`)

添加以下樣式：

```css
/* 錢包通知徽章 */
.wallet-badge {
    position: absolute;
    top: -5px;
    right: -10px;
    font-size: 0.65rem;
    padding: 0.2rem 0.4rem;
    line-height: 1;
    min-width: 18px;
    text-align: center;
}

/* 錢包連結 hover 效果 */
.nav-link:has(.bi-wallet):hover {
    color: var(--bs-primary) !important;
}
```

---

## 四、實作步驟與檢查點

### Step 1: 備份現有檔案 ✅
```powershell
Copy-Item "Views/Shared/_Sidebar.cshtml" "Views/Shared/_Sidebar.cshtml.bak"
Copy-Item "Views/Shared/_TopbarLevel1.cshtml" "Views/Shared/_TopbarLevel1.cshtml.bak"
```

### Step 2: 更新 Sidebar Icon ⏳
- 修改 `_Sidebar.cshtml` 中的所有 icon
- **檢查點**: 重新整理任一頁面，確認 icon 顯示正常

### Step 3: 添加「小遊戲」子選單 ⏳
- 添加團隊標籤和兩個子選單項目
- **檢查點**:
  1. Sidebar 顯示「此團隊」標籤
  2. 子選單顯示「每日簽到」和「寵物冒險」
  3. 點擊子選單可正確導航到 MiniGame Area

### Step 4: 添加「我的錢包」連結 ⏳
- 修改 `_TopbarLevel1.cshtml`
- **檢查點**:
  1. 右上角顯示「我的錢包」連結
  2. 通知徽章顯示數字
  3. 點擊可導航到錢包頁面

### Step 5: 添加 CSS 樣式 ⏳
- 更新 `sidebar.css` 和 `topbar1.css`
- **檢查點**: 子選單縮排正確，徽章位置正確

### Step 6: 跨瀏覽器測試 ⏳
- Chrome、Edge、Firefox
- 響應式測試（桌面、平板、手機）

### Step 7: 其他 Area 驗證 ⏳
- 進入 OnlineStore、Forum、social_hub 等其他 Area
- 確認導航正常，無樣式錯亂

---

## 五、後續擴展建議

### 5.1 動態通知數量
目前「我的錢包」的通知徽章數字是寫死的（3），建議：
- 在 `_Layout.cshtml` 或 `_TopbarLevel1` 的 ViewComponent 中，從後端 API 獲取未讀通知數量
- 使用 SignalR 實時更新徽章數字（當錢包異動時即時通知）

### 5.2 子選單展開/收合
目前子選單是固定展開的，若未來「小遊戲」功能更多，建議：
- 添加 JavaScript 實現點擊「小遊戲」時展開/收合子選單
- 使用 localStorage 記住用戶的展開狀態

### 5.3 Active 狀態標記
建議在 `_Sidebar.cshtml` 中添加邏輯，當用戶在 MiniGame Area 時：
- 高亮「小遊戲」主項目
- 根據當前 Controller 高亮對應的子選單項目（SignIn 或 Pet）

### 5.4 其他 Area 的子選單
如果其他團隊也需要子選單（如 OnlineStore 可能需要「商品列表」、「訂單查詢」等），可參考本次實作方式，統一添加。

---

## 六、風險評估與回滾計畫

### 6.1 回滾計畫
如果改造後出現問題，可立即回滾：

```powershell
# 復原 Sidebar
Copy-Item "Views/Shared/_Sidebar.cshtml.bak" "Views/Shared/_Sidebar.cshtml" -Force

# 復原 TopbarLevel1
Copy-Item "Views/Shared/_TopbarLevel1.cshtml.bak" "Views/Shared/_TopbarLevel1.cshtml" -Force

# 重啟應用
dotnet build
dotnet run
```

### 6.2 常見問題與解決

| 問題 | 可能原因 | 解決方案 |
|-----|---------|---------|
| Icon 不顯示 | SVG 路徑錯誤 | 檢查 `~/images/` 路徑是否正確 |
| 子選單路由 404 | Controller/Action 不存在 | 確認 MiniGame Area 已實作對應的 Controller |
| 徽章位置偏移 | CSS 樣式衝突 | 檢查 `topbar1.css` 的 `.wallet-badge` 樣式 |
| 手機版 Sidebar 錯亂 | Offcanvas 樣式衝突 | 測試 `#mainOffcanvas` 中的 Sidebar 顯示 |

---

## 七、協作溝通建議

### 7.1 告知其他團隊成員
修改完成後，建議在團隊會議或 Slack 中說明：
1. 為何在 Sidebar 添加「小遊戲」子選單（方便用戶導航）
2. 「此團隊」標籤的用意（標示負責區域）
3. 其他 Area 若需要類似功能，可參考此實作

### 7.2 文檔更新
- 更新 `CLAUDE.md` 的「UI Frameworks」章節，說明新的導航結構
- 在 `schema/` 目錄保留本改造計畫文件，供未來參考

---

## 八、總結

### 改造目標
✅ 在全域 Sidebar 添加 MiniGame 專屬子選單（每日簽到、寵物冒險）
✅ 在全域 Topbar 添加「我的錢包」快捷連結
✅ 更新所有導航 Icon 為更語義化的圖示
✅ 保持 MiniGame Area 內部獨立，不受全域樣式影響

### 關鍵原則
1. **最小化影響** - 只修改必要的全域檔案
2. **明確標示** - 使用「此團隊」標籤標示 MiniGame 歸屬
3. **向下相容** - 不破壞其他 Area 的現有功能
4. **可擴展性** - 其他團隊可參考此模式添加自己的子選單

### 預期效果
用戶可以：
- 直接從 Sidebar 進入簽到和寵物功能
- 隨時從頂部查看錢包狀態
- 更直觀地理解各功能的意義（通過語義化 Icon）

---

**文件版本**: v1.0
**建立者**: Claude Code
**審核狀態**: ⏳ 待審核
