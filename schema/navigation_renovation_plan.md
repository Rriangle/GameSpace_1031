# GamiPort 導覽改造計劃
## Navigation Renovation Plan

**文件日期**: 2025-10-29
**目標**: 修改 GamiPort 全域導覽結構（Sidebar + Topbar）

---

## 📋 改造需求總覽

### Sidebar 側邊欄變更
1. ❌ **刪除**: "小遊戲" 導覽項目 (lines 27-33 in _Sidebar.cshtml)
2. ✅ **新增**: "每日簽到" (Daily Sign-in) - 使用 Bootstrap Icon `bi-calendar-check-fill`
3. ✅ **新增**: "寵物冒險" (Pet Adventure) - 使用 SVG `wwwroot/images/paw.svg`
4. 🔄 **更新**: 其他現有導覽項目的圖標以保持一致性

### Topbar 頂部欄變更
1. 🔄 **修改**: "連結 B" → "我的錢包" (My Wallet) - 使用 Bootstrap Icon `bi-wallet`

---

## 📁 影響檔案清單

### 需要修改的全域共用檔案 (Global Shared Files)
這些是**全域共用樣式檔案**，修改前必須 **PAUSE-AND-ASK**：

1. **`Views/Shared/_Sidebar.cshtml`** ⚠️ GLOBAL
   - 路徑: `C:\Users\n2029\Desktop\work-1029\GamiPort\GamiPort\Views\Shared\_Sidebar.cshtml`
   - 用途: 主側邊欄導覽結構
   - 影響範圍: 整個 GamiPort 應用程式的左側導覽
   - 需要變更:
     - 刪除 "小遊戲" 項目 (lines 27-33)
     - 新增 "每日簽到" 項目
     - 新增 "寵物冒險" 項目
     - 更新圖標: 總覽、商城、交友、論壇、使用者

2. **`Views/Shared/Components/TopbarLevel1/Default.cshtml`** ⚠️ GLOBAL
   - 路徑: `C:\Users\n2029\Desktop\work-1029\GamiPort\GamiPort\Views\Shared\Components\TopbarLevel1\Default.cshtml`
   - 用途: 頂部導覽欄 ViewComponent
   - 影響範圍: 整個 GamiPort 應用程式的頂部導覽
   - 需要變更:
     - 修改 line 29: "連結 B" → "我的錢包" (with icon)

### 相關檔案 (不需修改，僅供參考)
- `Views/Shared/_Layout.cshtml` - 主版面配置（包含 TopbarLevel1 ViewComponent）
- `wwwroot/css/site.css` - 可能包含 sidebar/topbar 樣式
- `wwwroot/images/` - SVG 圖標資源 (已確認存在)

---

## 🎯 詳細變更規格

### 變更 1: 刪除 "小遊戲" 側邊欄項目
**檔案**: `Views/Shared/_Sidebar.cshtml`
**位置**: Lines 27-33
**動作**: 刪除以下程式碼區塊

```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="Home" asp-action="Index">
    <i class="bi bi-grid"></i>
    <span class="label">
        小遊戲
    </span>
</a>
```

**原因**:
- 根據需求，"小遊戲" 功能將與 "寵物冒險" 合併到前端
- 不再需要獨立的 "小遊戲" 導覽入口

**影響**:
- 移除後，用戶無法從側邊欄直接訪問 `MiniGame/Home/Index`
- 需要確保 MiniGame 功能可以通過其他方式訪問（例如通過新的 "寵物冒險" 或 "每日簽到" 入口）

---

### 變更 2: 新增 "每日簽到" 側邊欄項目
**檔案**: `Views/Shared/_Sidebar.cshtml`
**位置**: 插入到 "商城" 項目之後（建議 line 26 之後）
**動作**: 新增以下程式碼

```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="DailySignIn" asp-action="Index">
    <i class="bi bi-calendar-check-fill"></i>
    <span class="label">
        每日簽到
    </span>
</a>
```

**原因**:
- 新增每日簽到功能的入口
- 使用 Bootstrap Icon `bi-calendar-check-fill` 清楚表達簽到概念

**影響**:
- 增加一個新的導覽項目
- 需要確保 MiniGame Area 中存在 `DailySignInController` 和對應的 `Index` Action
- ⚠️ **注意**: 目前不清楚 MiniGame Area 中是否已經有 DailySignInController，需要後續確認

**路由假設**: `asp-area="MiniGame" asp-controller="DailySignIn" asp-action="Index"`
- 這會路由到: `/MiniGame/DailySignIn/Index`
- 需要在 MiniGame Area 中實現對應的 Controller

---

### 變更 3: 新增 "寵物冒險" 側邊欄項目
**檔案**: `Views/Shared/_Sidebar.cshtml`
**位置**: 插入到 "每日簽到" 項目之後
**動作**: 新增以下程式碼

```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="PetAdventure" asp-action="Index">
    <img src="~/images/paw.svg" alt="寵物冒險" width="16" height="16" class="flex-shrink-0" />
    <span class="label">
        寵物冒險
    </span>
</a>
```

**原因**:
- 新增寵物冒險功能的入口
- 使用自訂 SVG 圖標 `paw.svg` (已確認存在於 wwwroot/images/)

**影響**:
- 增加一個新的導覽項目
- 需要確保 MiniGame Area 中存在 `PetAdventureController` 和對應的 `Index` Action
- ⚠️ **注意**: 目前不清楚 MiniGame Area 中是否已經有 PetAdventureController，需要後續確認

**路由假設**: `asp-area="MiniGame" asp-controller="PetAdventure" asp-action="Index"`
- 這會路由到: `/MiniGame/PetAdventure/Index`
- 需要在 MiniGame Area 中實現對應的 Controller

**SVG 使用說明**:
- 使用 `<img>` 標籤而非 `<i>` 標籤
- 指定 `width="16" height="16"` 以匹配 Bootstrap Icon 的預設大小
- 添加 `class="flex-shrink-0"` 確保圖標不會因 flexbox 壓縮
- 使用 `~/images/paw.svg` 路徑（ASP.NET Core 會解析為 `/images/paw.svg`）

---

### 變更 4: 更新其他側邊欄圖標
**檔案**: `Views/Shared/_Sidebar.cshtml`
**目的**: 統一圖標風格，根據使用者提供的規格更新

#### 4.1 總覽 (Overview) - Lines 14-19
**現有圖標**: `<i class="bi bi-speedometer2"></i>`
**目標圖標**: SVG `overview.svg`

**變更前**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2" href="#">
    <i class="bi bi-speedometer2"></i>
    <span class="label">
        總覽
    </span>
</a>
```

**變更後**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2" href="#">
    <img src="~/images/overview.svg" alt="總覽" width="16" height="16" class="flex-shrink-0" />
    <span class="label">
        總覽
    </span>
</a>
```

#### 4.2 商城 (Store) - Lines 20-26
**現有圖標**: `<i class="bi bi-collection"></i>`
**目標圖標**: `<i class="bi bi-shop-window"></i>`

**變更前**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="OnlineStore" asp-controller="Home" asp-action="Index">
    <i class="bi bi-collection"></i>
    <span class="label">
        商城
    </span>
</a>
```

**變更後**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="OnlineStore" asp-controller="Home" asp-action="Index">
    <i class="bi bi-shop-window"></i>
    <span class="label">
        商城
    </span>
</a>
```

#### 4.3 交友 (Friends) - Lines 34-40
**現有圖標**: `<i class="bi bi-diagram-3"></i>`
**目標圖標**: SVG `add_friends.svg`

**變更前**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="social_hub" asp-controller="Home" asp-action="Index">
    <i class="bi bi-diagram-3"></i>
    <span class="label">
        交友
    </span>
</a>
```

**變更後**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="social_hub" asp-controller="Home" asp-action="Index">
    <img src="~/images/add_friends.svg" alt="交友" width="16" height="16" class="flex-shrink-0" />
    <span class="label">
        交友
    </span>
</a>
```

#### 4.4 論壇 (Forum) - Lines 41-47
**現有圖標**: `<i class="bi bi-grid"></i>`
**目標圖標**: SVG `forum_icon.svg`

**變更前**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="Forum" asp-controller="Home" asp-action="Index">
    <i class="bi bi-grid"></i>
    <span class="label">
        論壇
    </span>
</a>
```

**變更後**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="Forum" asp-controller="Home" asp-action="Index">
    <img src="~/images/forum_icon.svg" alt="論壇" width="16" height="16" class="flex-shrink-0" />
    <span class="label">
        論壇
    </span>
</a>
```

#### 4.5 使用者 (Users) - Lines 48-54
**現有圖標**: `<i class="bi bi-grid"></i>`
**目標圖標**: `<i class="bi bi-person"></i>`

**變更前**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MemberManagement" asp-controller="Home" asp-action="Index">
    <i class="bi bi-grid"></i>
    <span class="label">
        使用者
    </span>
</a>
```

**變更後**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MemberManagement" asp-controller="Home" asp-action="Index">
    <i class="bi bi-person"></i>
    <span class="label">
        使用者
    </span>
</a>
```

---

### 變更 5: 修改 "連結 B" 為 "我的錢包"
**檔案**: `Views/Shared/Components/TopbarLevel1/Default.cshtml`
**位置**: Line 29
**動作**: 修改程式碼

**變更前**:
```html
<li class="nav-item d-none d-md-block"><a class="nav-link" href="#">連結 B</a></li>
```

**變更後**:
```html
<li class="nav-item d-none d-md-block">
    <a class="nav-link d-flex align-items-center gap-1" asp-area="MiniGame" asp-controller="Wallet" asp-action="Index">
        <i class="bi bi-wallet"></i>
        <span>我的錢包</span>
    </a>
</li>
```

**原因**:
- 將通用的 "連結 B" 替換為具體功能 "我的錢包"
- 使用 Bootstrap Icon `bi-wallet` 清楚表達錢包概念
- 添加適當的路由到 MiniGame Area 的 Wallet Controller

**影響**:
- Topbar 導覽更加語意化
- 需要確保 MiniGame Area 中存在 `WalletController` 和對應的 `Index` Action
- ⚠️ **注意**: 目前不清楚 MiniGame Area 中是否已經有 WalletController，需要後續確認

**路由假設**: `asp-area="MiniGame" asp-controller="Wallet" asp-action="Index"`
- 這會路由到: `/MiniGame/Wallet/Index`
- 需要在 MiniGame Area 中實現對應的 Controller

**樣式說明**:
- 使用 `d-flex align-items-center gap-1` 確保圖標和文字垂直對齊
- 保留 `d-none d-md-block` 以確保在小螢幕上隱藏（響應式設計）

---

## 🎯 最終導覽結構

### Sidebar 最終結構
```
主連結
├── 總覽 (overview.svg)
├── 商城 (bi-shop-window)
├── 每日簽到 (bi-calendar-check-fill) ⭐ NEW
├── 寵物冒險 (paw.svg) ⭐ NEW
├── 交友 (add_friends.svg)
├── 論壇 (forum_icon.svg)
└── 使用者 (bi-person)

其他
├── 設定 (bi-gear)
└── 說明 (bi-question-circle)

❌ 已刪除: 小遊戲 (bi-grid)
```

### Topbar 最終結構
```
左側: Logo + GamiPort
右側:
├── 購物車 (with badge)
├── 連結 A
├── 我的錢包 (bi-wallet) ⭐ CHANGED (原 "連結 B")
└── 使用者選單 (dropdown)
```

---

## ⚠️ 風險與注意事項

### 1. Controller 依賴性 ⚠️ HIGH RISK
**問題**: 新增的導覽項目假設 MiniGame Area 中存在以下 Controller：
- `DailySignInController` (每日簽到)
- `PetAdventureController` (寵物冒險)
- `WalletController` (我的錢包)

**風險**: 如果這些 Controller 不存在，點擊導覽連結會導致 404 錯誤

**解決方案**:
1. **選項 A**: 修改導覽前，先確認 MiniGame Area 中是否存在這些 Controller
2. **選項 B**: 使用臨時的 `href="#"` 或路由到 MiniGame 首頁，等 Controller 實現後再更新
3. **選項 C**: 同時在 MiniGame Area 中創建基本的 Controller stub

**建議**: 在實際修改導覽前，先檢查 `Areas/MiniGame/Controllers/` 目錄中是否存在這些 Controller

### 2. 全域檔案修改影響 ⚠️ HIGH IMPACT
**問題**: `_Sidebar.cshtml` 和 `TopbarLevel1/Default.cshtml` 是全域共用檔案

**影響範圍**:
- 所有使用 `_Layout.cshtml` 的頁面
- 整個 GamiPort 應用程式的導覽體驗
- 所有已登入和未登入的使用者

**風險**:
- 如果修改錯誤，可能導致整個網站的導覽失效
- 可能影響到其他 Area 的功能（例如 OnlineStore, Forum, social_hub 等）

**緩解措施**:
- 修改前先備份原始檔案
- 修改後立即測試所有導覽連結
- 在開發環境中測試，確認無誤後再部署到生產環境

### 3. SVG 圖標樣式一致性 ⚠️ MEDIUM RISK
**問題**: 混合使用 Bootstrap Icons (`<i>`) 和 SVG (`<img>`)

**風險**:
- SVG 圖標的大小、顏色可能與 Bootstrap Icons 不一致
- 可能需要額外的 CSS 調整

**緩解措施**:
- 為 SVG 圖標指定固定尺寸 `width="16" height="16"`
- 添加 `class="flex-shrink-0"` 防止 flexbox 壓縮
- 測試時檢查視覺一致性
- 如有需要，可能需要添加 CSS filter 或調整 SVG 內部的 fill 顏色

### 4. 響應式設計 ⚠️ LOW RISK
**問題**: Topbar 在小螢幕上使用 offcanvas，會顯示 `_Sidebar`

**影響**: 修改 Sidebar 後，offcanvas 中的導覽也會改變

**確認事項**:
- 確保新增的項目在 mobile view (offcanvas) 中顯示正常
- 測試不同螢幕尺寸下的顯示效果

---

## 📋 實施步驟 (Step-by-Step Implementation)

### Phase 1: 準備與驗證
1. ✅ **已完成**: 確認所有 SVG 圖標檔案存在
   - paw.svg ✓
   - forum_icon.svg ✓
   - add_friends.svg ✓
   - overview.svg ✓

2. ⏳ **待確認**: 檢查 MiniGame Area 中的 Controller
   - 檢查是否存在: `Areas/MiniGame/Controllers/DailySignInController.cs`
   - 檢查是否存在: `Areas/MiniGame/Controllers/PetAdventureController.cs`
   - 檢查是否存在: `Areas/MiniGame/Controllers/WalletController.cs`
   - 如果不存在，需要決定使用臨時路由或創建 Controller stub

3. ⏳ **建議**: 備份原始檔案
   - 備份 `_Sidebar.cshtml`
   - 備份 `TopbarLevel1/Default.cshtml`

### Phase 2: 修改 _Sidebar.cshtml (⚠️ PAUSE-AND-ASK)
**在此階段前必須 PAUSE-AND-ASK 用戶**

**修改順序建議**:
1. 更新 "總覽" 圖標 (line 15: bi-speedometer2 → overview.svg)
2. 更新 "商城" 圖標 (line 22: bi-collection → bi-shop-window)
3. **刪除** "小遊戲" 區塊 (lines 27-33)
4. 在 "商城" 後**新增** "每日簽到" 項目
5. 在 "每日簽到" 後**新增** "寵物冒險" 項目
6. 更新 "交友" 圖標 (line 36: bi-diagram-3 → add_friends.svg)
7. 更新 "論壇" 圖標 (line 43: bi-grid → forum_icon.svg)
8. 更新 "使用者" 圖標 (line 50: bi-grid → bi-person)

**每一步驟前都應該**:
- 說明即將做什麼
- 說明為什麼要這麼做
- 說明影響是什麼
- 等待使用者同意

### Phase 3: 修改 TopbarLevel1/Default.cshtml (⚠️ PAUSE-AND-ASK)
**在此階段前必須 PAUSE-AND-ASK 用戶**

1. 修改 line 29: "連結 B" → "我的錢包"
   - 更新文字內容
   - 新增 wallet 圖標
   - 新增路由到 MiniGame/Wallet/Index

### Phase 4: 測試與驗證
1. 啟動應用程式
2. 測試所有 Sidebar 連結
3. 測試 Topbar "我的錢包" 連結
4. 測試 mobile view (offcanvas)
5. 檢查圖標顯示是否一致
6. 確認 SVG 圖標大小和對齊正確

---

## 🔍 後續工作 (MiniGame Area 實作)

以下是在 MiniGame Area 中需要實作的 Controller（如果尚不存在）：

### 1. DailySignInController
**路徑**: `Areas/MiniGame/Controllers/DailySignInController.cs`
**用途**: 處理每日簽到功能
**最小實作**:
```csharp
[Area("MiniGame")]
public class DailySignInController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```

### 2. PetAdventureController
**路徑**: `Areas/MiniGame/Controllers/PetAdventureController.cs`
**用途**: 處理寵物冒險功能（可能整合小遊戲）
**最小實作**:
```csharp
[Area("MiniGame")]
public class PetAdventureController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```

### 3. WalletController
**路徑**: `Areas/MiniGame/Controllers/WalletController.cs`
**用途**: 處理使用者錢包功能
**最小實作**:
```csharp
[Area("MiniGame")]
public class WalletController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```

**注意**: 這些 Controller 的詳細實作應該根據 `schema/README_合併版.md` 中的需求來完成，目前僅提供最小可行的結構以確保導覽連結不會 404。

---

## 📊 總結

### 修改檔案數量
- **2 個全域共用檔案** 需要修改 (⚠️ HIGH IMPACT)
- **3 個 MiniGame Area Controller** 可能需要創建 (如果不存在)

### 變更項目數量
- **1 個刪除** (小遊戲)
- **2 個新增** (每日簽到, 寵物冒險)
- **6 個圖標更新** (總覽, 商城, 交友, 論壇, 使用者, 我的錢包)

### 風險評估
- **HIGH RISK**: Controller 依賴性 - 需要確認 MiniGame Controllers 存在
- **HIGH IMPACT**: 全域檔案修改 - 影響整個應用程式
- **MEDIUM RISK**: SVG 樣式一致性 - 需要測試視覺效果
- **LOW RISK**: 響應式設計 - 現有結構應該能處理

### 建議實施策略
1. **先驗證**: 確認 MiniGame Area 中的 Controller 存在性
2. **後修改**: 使用 PAUSE-AND-ASK 流程逐步修改全域檔案
3. **立即測試**: 每次修改後立即測試導覽功能
4. **漸進式部署**: 在開發環境驗證後再考慮生產部署

---

## ✅ Next Actions

1. **立即行動**: 檢查 MiniGame Area 中的 Controller 是否存在
2. **等待確認**: 向使用者說明即將開始修改全域檔案 (_Sidebar.cshtml)
3. **實施修改**: 按照 Phase 2 和 Phase 3 的步驟進行修改
4. **測試驗證**: 完成後進行完整測試

---

**文件版本**: 1.0
**最後更新**: 2025-10-29
**狀態**: ✅ 計劃完成，等待實施
