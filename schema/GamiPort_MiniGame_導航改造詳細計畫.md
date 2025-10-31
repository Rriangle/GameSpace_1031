# GamiPort MiniGame 導航改造詳細計畫

**版本**: v1.0
**建立日期**: 2025-10-29
**作者**: Claude Code
**專案**: GamiPort MiniGame Area 導航結構重構

---

## 📋 目錄

1. [改造目標](#改造目標)
2. [現況分析](#現況分析)
3. [改造方案](#改造方案)
4. [檔案修改清單](#檔案修改清單)
5. [影響評估](#影響評估)
6. [實施步驟](#實施步驟)
7. [技術規格](#技術規格)
8. [風險與注意事項](#風險與注意事項)

---

## 改造目標

### 主要需求

**Sidebar 導航變更**:
- ❌ **刪除**: "小遊戲" 導航項目
- ✅ **新增**: "每日簽到" 導航項目
- ✅ **新增**: "寵物冒險" 導航項目

**Topbar 導航變更**:
- ✅ **新增**: "我的錢包" 導航項目（放在使用者下拉選單中）

### 設計原則

1. **最小影響原則**: 只修改必要的全域共用檔案
2. **一致性原則**: 遵循現有的 UI/UX 設計模式
3. **可擴展原則**: 為未來功能擴展預留空間
4. **審核優先原則**: 每個全域檔案修改前必須 pause-and-ask

---

## 現況分析

### A. Sidebar 現狀

**檔案位置**: `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Views\Shared\_Sidebar.cshtml`

**目前結構** (Lines 27-33):
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="Home" asp-action="Index">
    <i class="bi bi-grid"></i>
    <span class="label">
        小遊戲
    </span>
</a>
```

**問題分析**:
- "小遊戲" 單一導航項目無法區分不同功能模組
- 使用者需要點擊後才能看到子功能，不夠直觀
- 無法快速進入每日簽到或寵物冒險功能

**導航架構**:
- 使用 Bootstrap 5.3.3 list-group 組件
- 圖示使用 Bootstrap Icons 1.11.3
- ASP.NET Core Area 路由模式

---

### B. Topbar 現狀

**檔案位置**: `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Views\Shared\Components\TopbarLevel1\Default.cshtml`

**目前結構**:
- Logo（左對齊）
- 購物車連結（附 badge）
- 連結 A, B（僅平板以上顯示）
- 使用者下拉選單（已登入時顯示）
  - 檢視個人資料
  - 編輯個人資料
  - 編輯個人密碼
  - 登出

**問題分析**:
- 缺少錢包入口，使用者無法快速查看餘額
- 與 OnlineStore 購物車功能不對等（購物車有專屬位置，錢包沒有）

**ViewComponent 資料流**:
- `TopbarLevel1ViewComponent.cs` → `Default.cshtml`
- 使用 `ICurrentUserService` 取得使用者資訊
- 條件式渲染（已登入 vs 未登入）

---

### C. MiniGame Area 現狀

**Controllers 狀態**:
| Controller | 狀態 | 用途 |
|---|---|---|
| HomeController.cs | ✅ 已存在 | MiniGame Area 首頁 |
| WalletController.cs | ❌ 不存在 | 我的錢包 |
| SignInController.cs | ❌ 不存在 | 每日簽到 |
| PetController.cs | ❌ 不存在 | 寵物冒險 |

**Views 狀態**:
- 僅有 `Home/Index.cshtml`（顯示 "小遊戲" 文字）
- 缺少 Wallet/, SignIn/, Pet/ 子資料夾

**Database Models**:
- ✅ UserWallet, WalletHistory（錢包相關）
- ✅ UserSignInStat, SignInRule（簽到相關）
- ✅ Pet, PetLevelRewardSetting（寵物相關）
- ✅ MiniGame（小遊戲記錄）

**路由配置**:
- ✅ Program.cs 已配置 Area 路由模式
- Pattern: `{area:exists}/{controller=Home}/{action=Index}/{id?}`

---

## 改造方案

### 方案 A: Sidebar 改造（建議採用）

**設計決策**: 刪除 "小遊戲" 單一項目，替換為 "每日簽到" 和 "寵物冒險" 兩個獨立導航項目

**優點**:
- ✅ 提供直接導航路徑，減少點擊次數
- ✅ 符合使用者心智模型（每日簽到和寵物是獨立功能）
- ✅ 為未來新增更多 MiniGame 功能預留空間
- ✅ 與 Bahamut 風格一致（功能分類清晰）

**HTML 結構**:

```html
<!-- 刪除此段 (Lines 27-33) -->
<!-- 原 "小遊戲" 導航項目 -->

<!-- 新增：每日簽到 -->
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="SignIn" asp-action="Index">
    <i class="bi bi-calendar-check"></i>
    <span class="label">
        每日簽到
    </span>
</a>

<!-- 新增：寵物冒險 -->
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="Pet" asp-action="Index">
    <i class="bi bi-heart-fill"></i>
    <span class="label">
        寵物冒險
    </span>
</a>
```

**圖示選擇理由**:
- `bi-calendar-check`: 日曆打勾符號，直觀表達 "簽到" 概念
- `bi-heart-fill`: 愛心符號，表達寵物陪伴與互動概念

**替代圖示方案**:
- 每日簽到: `bi-calendar-event`, `bi-calendar2-check`
- 寵物冒險: `bi-star`, `bi-map`, `bi-puzzle`

---

### 方案 B: Topbar 改造（建議採用）

**設計決策**: 在使用者下拉選單中新增 "我的錢包" 項目

**優點**:
- ✅ 遵循 "使用者相關功能集中管理" 的 UX 原則
- ✅ 與現有 "個人資料"、"編輯密碼" 等項目同層級
- ✅ 減少 topbar 視覺負擔
- ✅ 只對已登入使用者顯示（符合業務邏輯）

**HTML 結構** (插入位置: Line 60 之後, 密碼編輯項目之後):

```html
<li>
    <a class="dropdown-item" asp-area="MiniGame" asp-controller="Wallet" asp-action="Index">
        <i class="bi bi-wallet2 me-2"></i> 我的錢包
    </a>
</li>
```

**圖示選擇理由**:
- `bi-wallet2`: 經典錢包圖示，最直觀

**替代圖示方案**:
- `bi-coin`: 錢幣圖示
- `bi-cash-coin`: 現金+錢幣組合
- `bi-piggy-bank`: 存錢筒圖示

**插入位置示意** (Line 47-65):
```html
<li>
    <a class="dropdown-item" asp-area="Login" asp-controller="Introduce" asp-action="Details">
        <i class="bi bi-person-badge me-2"></i> 檢視個人資料
    </a>
</li>
<li>
    <a class="dropdown-item" asp-area="Login" asp-controller="Introduce" asp-action="Edit">
        <i class="bi bi-pencil-square me-2"></i> 編輯個人資料
    </a>
</li>
<li>
    <a class="dropdown-item" asp-area="Login" asp-controller="Password" asp-action="Edit">
        <i class="bi bi-pencil-square me-2"></i> 編輯個人密碼
    </a>
</li>

<!-- ========== 新增位置：此處插入 "我的錢包" ========== -->
<li>
    <a class="dropdown-item" asp-area="MiniGame" asp-controller="Wallet" asp-action="Index">
        <i class="bi bi-wallet2 me-2"></i> 我的錢包
    </a>
</li>
<!-- ========== 新增位置結束 ========== -->

<li><hr class="dropdown-divider" /></li>
<li>
    <form asp-area="Login" asp-controller="Login" asp-action="Logout" method="post" class="px-2 py-1 mb-0">
        @Html.AntiForgeryToken()
        <button type="submit" class="dropdown-item text-danger w-100">
            <i class="bi bi-box-arrow-right me-2"></i> 登出
        </button>
    </form>
</li>
```

---

### 方案 C: 替代方案（不建議）

**方案 C-1**: 在 topbar 新增獨立 "我的錢包" 按鈕（與購物車並列）

**缺點**:
- ❌ 增加 topbar 視覺複雜度
- ❌ 與購物車功能性質不同（購物車有即時更新 badge 需求，錢包沒有）
- ❌ 需要額外 CSS 調整

**方案 C-2**: 在 sidebar 新增 "錢包" 項目

**缺點**:
- ❌ 錢包是使用者個人功能，不適合與主功能導航並列
- ❌ 不符合 "個人功能集中管理" 的 UX 原則

---

## 檔案修改清單

### 全域共用檔案（需要 pause-and-ask）

#### 檔案 1: _Sidebar.cshtml ⚠️

**路徑**: `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Views\Shared\_Sidebar.cshtml`

**修改類型**: 刪除 + 新增

**原始內容** (Lines 27-33):
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="Home" asp-action="Index">
    <i class="bi bi-grid"></i>
    <span class="label">
        小遊戲
    </span>
</a>
```

**修改後內容**:
```html
<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="SignIn" asp-action="Index">
    <i class="bi bi-calendar-check"></i>
    <span class="label">
        每日簽到
    </span>
</a>

<a class="list-group-item list-group-item-action d-flex align-items-center gap-2"
   asp-area="MiniGame" asp-controller="Pet" asp-action="Index">
    <i class="bi bi-heart-fill"></i>
    <span class="label">
        寵物冒險
    </span>
</a>
```

**變更說明**:
- 刪除原 "小遊戲" 導航項目（7 行）
- 新增 "每日簽到" 導航項目（7 行）
- 新增 "寵物冒險" 導航項目（7 行）
- 淨增加: 7 行

**影響範圍**: 全站所有頁面的 sidebar 導航

---

#### 檔案 2: TopbarLevel1/Default.cshtml ⚠️

**路徑**: `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Views\Shared\Components\TopbarLevel1\Default.cshtml`

**修改類型**: 新增

**插入位置**: Line 60 之後（編輯個人密碼項目之後）

**新增內容**:
```html
<li>
    <a class="dropdown-item" asp-area="MiniGame" asp-controller="Wallet" asp-action="Index">
        <i class="bi bi-wallet2 me-2"></i> 我的錢包
    </a>
</li>
```

**變更說明**:
- 在使用者下拉選單中新增 "我的錢包" 項目
- 新增 4 行

**影響範圍**: 全站所有頁面的 topbar 使用者選單（僅已登入使用者可見）

---

### MiniGame Area 檔案（使用者負責，無需審核）

#### 檔案 3: SignInController.cs ✅

**路徑**: `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Areas\MiniGame\Controllers\SignInController.cs`

**狀態**: 需要建立

**內容**:
```csharp
using Microsoft.AspNetCore.Mvc;

namespace GamiPort.Areas.MiniGame.Controllers
{
    [Area("MiniGame")]
    public class SignInController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }
    }
}
```

---

#### 檔案 4: PetController.cs ✅

**路徑**: `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Areas\MiniGame\Controllers\PetController.cs`

**狀態**: 需要建立

**內容**:
```csharp
using Microsoft.AspNetCore.Mvc;

namespace GamiPort.Areas.MiniGame.Controllers
{
    [Area("MiniGame")]
    public class PetController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }
    }
}
```

---

#### 檔案 5: WalletController.cs ✅

**路徑**: `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Areas\MiniGame\Controllers\WalletController.cs`

**狀態**: 需要建立

**內容**:
```csharp
using Microsoft.AspNetCore.Mvc;

namespace GamiPort.Areas.MiniGame.Controllers
{
    [Area("MiniGame")]
    public class WalletController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }
    }
}
```

---

#### 檔案 6-8: View 檔案 ✅

**需要建立的 View 檔案**:

1. `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Areas\MiniGame\Views\SignIn\Index.cshtml`
2. `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Areas\MiniGame\Views\Pet\Index.cshtml`
3. `C:\Users\n2029\Desktop\work2\GamiPort\GamiPort\Areas\MiniGame\Views\Wallet\Index.cshtml`

**基本內容模板**:
```html
@{
    ViewData["Title"] = "[功能名稱]";
}

<div class="container mt-4">
    <h1>@ViewData["Title"]</h1>
    <p>功能開發中...</p>
</div>
```

---

## 影響評估

### A. 功能影響

| 項目 | 影響等級 | 說明 |
|---|---|---|
| 全站導航結構 | 🟡 中等 | Sidebar 導航項目變更，使用者需要適應新導航位置 |
| 使用者體驗 | 🟢 正面 | 提供更直觀的功能入口，減少點擊次數 |
| 現有功能 | 🟢 無影響 | 不影響現有商城、論壇、社交等功能 |
| 路由系統 | 🟢 無影響 | 使用既有 Area 路由模式，不需修改路由配置 |
| 資料庫 | 🟢 無影響 | 所有必要 Models 已存在，不需 Migration |

---

### B. 技術影響

| 項目 | 影響範圍 | 風險等級 |
|---|---|---|
| CSS 樣式 | 無影響 | 🟢 低風險 - 使用現有 Bootstrap 類別 |
| JavaScript | 無影響 | 🟢 低風險 - 無新增 JS 邏輯需求 |
| 認證授權 | 無影響 | 🟢 低風險 - 繼承現有認證機制 |
| 效能 | 無影響 | 🟢 低風險 - 僅靜態 HTML 變更 |
| 瀏覽器相容性 | 無影響 | 🟢 低風險 - 使用 Bootstrap 5.3.3 標準組件 |
| 響應式設計 | 無影響 | 🟢 低風險 - Bootstrap RWD 自動處理 |

---

### C. 使用者影響

**正面影響**:
- ✅ 降低認知負擔：功能分類更清晰
- ✅ 提高效率：減少導航點擊次數
- ✅ 符合預期：符合 "每日簽到" 和 "寵物" 為獨立功能的心智模型
- ✅ 視覺一致性：圖示風格與現有導航一致

**可能的負面影響**:
- ⚠️ 習慣變更：熟悉舊版 "小遊戲" 入口的使用者需要重新學習
- ⚠️ 教學需求：可能需要公告說明導航變更

**緩解措施**:
- 📢 發布公告說明導航變更
- 📖 更新使用說明文件
- 💡 考慮在 Home/Index 頁面加入導航提示

---

### D. 開發團隊影響

| 項目 | 影響 |
|---|---|
| 前端開發 | 需要實作 SignIn, Pet, Wallet 三個頁面的 UI |
| 後端開發 | 需要實作對應的 API 端點與業務邏輯 |
| 測試團隊 | 需要測試新導航路徑與功能正確性 |
| 文件團隊 | 需要更新使用者手冊與開發者文件 |

---

## 實施步驟

### Phase 1: 準備階段（開發環境）✅

**Step 1.1**: 建立 MiniGame Area Controllers ✅ 無需審核
- SignInController.cs
- PetController.cs
- WalletController.cs

**Step 1.2**: 建立對應 View 資料夾與基本頁面 ✅ 無需審核
- Views/SignIn/Index.cshtml
- Views/Pet/Index.cshtml
- Views/Wallet/Index.cshtml

**Step 1.3**: 本地測試路由是否正常 ✅
- 啟動專案
- 測試 URL:
  - `/MiniGame/SignIn/Index`
  - `/MiniGame/Pet/Index`
  - `/MiniGame/Wallet/Index`

---

### Phase 2: 全域檔案修改（需要 pause-and-ask）⚠️

**Step 2.1**: 修改 Sidebar 導航 ⚠️ **需要審核**

**修改內容**:
- 檔案: `_Sidebar.cshtml`
- 動作: 刪除 "小遊戲" 項目（Lines 27-33）
- 動作: 新增 "每日簽到" 項目
- 動作: 新增 "寵物冒險" 項目

**Why（為何要做）**:
- 提供更清晰的功能分類
- 讓使用者可以直接進入每日簽到與寵物功能，無需額外點擊

**What（要做什麼）**:
- 將單一 "小遊戲" 導航分拆為兩個獨立項目
- 使用語意化圖示 (calendar-check, heart-fill)
- 指向新建立的 SignInController 與 PetController

**Impact（影響）**:
- 影響範圍: 全站所有頁面的 sidebar 顯示
- 視覺變化: sidebar 增加 1 個導航項目（原 1 個變 2 個）
- 功能變化: 使用者點擊後直接進入功能頁面，而非中繼頁面
- 風險: 🟢 低風險 - 僅 HTML 結構變更，不影響其他功能

**PAUSE-AND-ASK**: 等待使用者確認後才執行

---

**Step 2.2**: 修改 Topbar 使用者選單 ⚠️ **需要審核**

**修改內容**:
- 檔案: `TopbarLevel1/Default.cshtml`
- 動作: 在 Line 60 後新增 "我的錢包" 項目

**Why（為何要做）**:
- 提供快速進入錢包的入口
- 與 "個人資料"、"編輯密碼" 等使用者功能分組一致
- 符合使用者對個人財務資訊管理的預期位置

**What（要做什麼）**:
- 在使用者下拉選單中新增 1 個 dropdown-item
- 使用 `bi-wallet2` 圖示
- 指向 WalletController.Index

**Impact（影響）**:
- 影響範圍: 全站 topbar 使用者下拉選單（僅已登入使用者可見）
- 視覺變化: 下拉選單增加 1 個項目（4 個變 5 個）
- 功能變化: 使用者可從 topbar 直接進入錢包頁面
- 風險: 🟢 低風險 - 僅新增項目，不影響現有選單項目

**PAUSE-AND-ASK**: 等待使用者確認後才執行

---

### Phase 3: 功能開發（後續工作）✅

**Step 3.1**: 開發 SignIn 功能頁面
- SignInController 完整實作
- SignIn/Index.cshtml 日曆式簽到 UI
- 整合 UserSignInStat, SignInRule 資料庫邏輯

**Step 3.2**: 開發 Pet 功能頁面
- PetController 完整實作
- Pet/Index.cshtml 寵物互動 UI
- 整合 Pet, PetLevelRewardSetting 資料庫邏輯

**Step 3.3**: 開發 Wallet 功能頁面
- WalletController 完整實作
- Wallet/Index.cshtml 錢包管理 UI
- 整合 UserWallet, WalletHistory 資料庫邏輯

---

### Phase 4: 測試與部署 ✅

**Step 4.1**: 整合測試
- 測試所有導航連結正確性
- 測試響應式設計（Desktop, Tablet, Mobile）
- 測試認證授權流程

**Step 4.2**: UAT 使用者驗收測試
- 邀請使用者測試新導航結構
- 收集回饋並調整

**Step 4.3**: 部署上線
- 備份現有程式碼
- 部署至 Staging 環境測試
- 部署至 Production 環境
- 監控錯誤日誌

---

## 技術規格

### A. 路由規格

| Controller | Area | Route | HTTP Method |
|---|---|---|---|
| SignInController | MiniGame | /MiniGame/SignIn/Index | GET |
| PetController | MiniGame | /MiniGame/Pet/Index | GET |
| WalletController | MiniGame | /MiniGame/Wallet/Index | GET |

**路由模式**: `{area:exists}/{controller=Home}/{action=Index}/{id?}`

---

### B. 認證授權規格

**建議配置**:

所有 MiniGame Area Controllers 都應加上 `[Authorize]` 屬性：

```csharp
[Area("MiniGame")]
[Authorize]
public class SignInController : Controller
{
    // ...
}
```

**原因**:
- 每日簽到、寵物、錢包都是使用者個人功能
- 未登入使用者應導向登入頁面

---

### C. CSS 規格

**不需要新增 CSS**，使用現有 Bootstrap 5.3.3 類別：

| 元素 | CSS Classes |
|---|---|
| Sidebar 項目 | `list-group-item list-group-item-action d-flex align-items-center gap-2` |
| Sidebar 圖示 | `bi bi-[icon-name]` |
| Sidebar 文字 | `<span class="label">` |
| Topbar 選單項目 | `dropdown-item` |
| Topbar 圖示 | `bi bi-[icon-name] me-2` |

---

### D. 圖示規格

使用 **Bootstrap Icons 1.11.3**:

| 功能 | 圖示 | 類別 |
|---|---|---|
| 每日簽到 | 日曆打勾 | `bi-calendar-check` |
| 寵物冒險 | 愛心填滿 | `bi-heart-fill` |
| 我的錢包 | 錢包 | `bi-wallet2` |

---

### E. 響應式規格

| 斷點 | Sidebar 行為 | Topbar 行為 |
|---|---|---|
| < 576px (xs) | 摺疊至 Offcanvas | 顯示漢堡選單 + Logo + 購物車 |
| 576-768px (sm) | 摺疊至 Offcanvas | 顯示漢堡選單 + Logo + 購物車 |
| 768-992px (md) | 固定顯示 | 顯示所有項目（含 Link A/B） |
| 992-1200px (lg) | 固定顯示 | 顯示所有項目 |
| ≥ 1200px (xl) | 固定顯示 | 顯示所有項目 |

---

## 風險與注意事項

### 高風險項目 🔴

**無高風險項目** - 本次改造僅涉及導航結構變更，不涉及資料庫、API、或核心業務邏輯

---

### 中風險項目 🟡

**風險 1**: 使用者習慣變更

**描述**: 熟悉舊版 "小遊戲" 入口的使用者可能一時找不到功能

**緩解措施**:
- ✅ 發布公告說明變更
- ✅ 在 MiniGame/Home/Index 頁面保留導航說明
- ✅ 考慮過渡期保留舊入口（Home/Index）並顯示導航提示

**應變方案**:
- 如果使用者反饋負面，可快速 rollback 到舊版導航

---

**風險 2**: Controller 尚未實作完整功能

**描述**: SignInController, PetController, WalletController 僅有空殼，功能未完整實作

**緩解措施**:
- ✅ 在 Index.cshtml 顯示 "功能開發中" 訊息
- ✅ 優先實作基本 UI 與資料顯示
- ✅ 分階段開發功能（先讀取，後新增/編輯）

**應變方案**:
- 如果開發進度落後，可暫時隱藏對應導航項目（CSS `display: none`）

---

### 低風險項目 🟢

**風險 3**: HTML 語法錯誤

**描述**: 手動修改 HTML 可能引入語法錯誤

**緩解措施**:
- ✅ 使用 Edit 工具精確替換
- ✅ 本地測試後再部署
- ✅ 使用 Browser Dev Tools 檢查 HTML 結構

---

**風險 4**: 路由衝突

**描述**: 新增 Controller 可能與現有路由衝突

**緩解措施**:
- ✅ 使用既有 Area 路由模式
- ✅ 測試所有新增路由是否正常
- ✅ 檢查 Program.cs 路由配置

---

## 附錄

### A. 檔案清單總覽

**需要修改的全域檔案** (2 個，需要 pause-and-ask):
1. `GamiPort\Views\Shared\_Sidebar.cshtml`
2. `GamiPort\Views\Shared\Components\TopbarLevel1\Default.cshtml`

**需要建立的 MiniGame Area 檔案** (6 個，無需審核):
1. `Areas\MiniGame\Controllers\SignInController.cs`
2. `Areas\MiniGame\Controllers\PetController.cs`
3. `Areas\MiniGame\Controllers\WalletController.cs`
4. `Areas\MiniGame\Views\SignIn\Index.cshtml`
5. `Areas\MiniGame\Views\Pet\Index.cshtml`
6. `Areas\MiniGame\Views\Wallet\Index.cshtml`

**不需要修改的檔案**:
- Program.cs (路由配置已支援)
- sidebar.css (CSS 已足夠)
- topbar1.css (CSS 已足夠)
- 所有 JavaScript 檔案
- 資料庫 Models（已存在）

---

### B. 參考資料

**相關文件**:
- `schema/README_合併版.md` - MiniGame Area 功能規格
- `schema/GamiPort前台風格布局改造建議.md` - 前台風格設計指引
- `schema/巴哈姆特風格布局特色完整分析.md` - Bahamut 風格參考

**技術文件**:
- Bootstrap 5.3.3 Documentation: https://getbootstrap.com/docs/5.3/
- Bootstrap Icons 1.11.3: https://icons.getbootstrap.com/
- ASP.NET Core Areas: https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/areas

---

### C. 版本歷史

| 版本 | 日期 | 作者 | 變更說明 |
|---|---|---|---|
| v1.0 | 2025-10-29 | Claude Code | 初始版本，定義完整改造計畫 |

---

## 總結

本改造計畫旨在優化 GamiPort MiniGame Area 的使用者導航體驗，主要變更包括：

1. **Sidebar**: 刪除 "小遊戲" 單一項目，替換為 "每日簽到" 和 "寵物冒險" 兩個獨立導航
2. **Topbar**: 在使用者下拉選單中新增 "我的錢包" 項目

改造影響範圍明確，風險等級低，實施步驟清晰。所有全域檔案修改都遵循 **pause-and-ask** 原則，確保每一步都經過使用者確認。

**下一步行動**: 等待使用者確認本計畫後，開始執行 Phase 1（建立 Controllers 與 Views）。

---

**文件結束**
