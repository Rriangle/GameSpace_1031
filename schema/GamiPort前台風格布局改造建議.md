# GamiPort 前台風格布局改造建議

> **撰寫日期**: 2025-10-29 (更新版)
> **改造對象**: GamiPort Client Frontend
> **參考標準**: 巴哈姆特設計系統 + Bootstrap 5.3.3
> **改造目標**: 打造簡潔、現代、用戶友善的遊戲平台介面
> **更新說明**: 整合 OnlineStore（電商）、Support（客訴）、Chat（聊天）最新功能發現

---

## 目錄

1. [現況評估與問題診斷](#1-現況評估與問題診斷)
2. [整體改造策略](#2-整體改造策略)
3. [色彩系統重構](#3-色彩系統重構)
4. [頂部導航列改造](#4-頂部導航列改造)
5. [側邊欄優化](#5-側邊欄優化)
6. [Layout Shell 架構調整](#6-layout-shell-架構調整)
7. [首頁 (Home/Index) 重新設計](#7-首頁-homeindex-重新設計)
8. [卡片系統建立](#8-卡片系統建立)
9. [表單系統改造](#9-表單系統改造)
10. [Modal 系統建立](#10-modal-系統建立)
11. [按鈕系統標準化](#11-按鈕系統標準化)
12. [通知系統實作](#12-通知系統實作)
13. [響應式設計強化](#13-響應式設計強化)
14. [社交功能 (Chat/Dock) 優化](#14-社交功能-chatdock-優化)
15. [載入與錯誤狀態](#15-載入與錯誤狀態)
16. [動畫與過渡效果](#16-動畫與過渡效果)
17. [輔助功能改善](#17-輔助功能改善)
18. [效能優化建議](#18-效能優化建議)
19. [實作優先級與時程](#19-實作優先級與時程)
20. [驗收標準 (Acceptance Criteria)](#20-驗收標準-acceptance-criteria)

---

## 1. 現況評估與問題診斷

### 1.1 當前 GamiPort 架構分析

根據 exploration agent 的深入分析，目前 GamiPort 具有以下結構：

**✅ 已有良好基礎**:
- Bootstrap 5.3.3 已引入
- Bootstrap Icons 1.11.3 已配置
- 三層式 Layout 結構 (TopbarLevel1 → TopbarLevel2 + MainContent + RightRail → Footer)
- CSS 變數系統 (`:root` 定義於 `layout-shell.css`)
- 側邊欄響應式機制 (collapse/offcanvas)
- **SignalR 8.0 完整整合**（ChatHub、SupportHub）
- ViewComponent 架構 (TopbarLevel1ViewComponent)
- **OnlineStore 電商模組**（購物車 178行、結帳 409行、ECPay 416行）
- **Support 客訴系統**（SupportController 151行、SupportHub 167行、SignalR 跨站）
- **Chat 聊天系統**（ChatService、FloatingDock 613行、ProfanityFilter 104行）
- **QuickFab 浮動按鈕系統**（244行，統一樣式 + 入場動畫）
- **IAppCurrentUser 統一登入介面**（36行，跨模組認證）

**❌ 需要改進的地方**:

#### 1.1.1 色彩系統問題
- **未建立品牌主色**: 缺乏一致的主色調 (不像巴哈姆特的青綠色)
- **CSS 變數不完整**: `:root` 僅定義尺寸變數，缺少色彩、陰影等
- **Bootstrap 預設色**: 仍使用 Bootstrap 預設配色，缺乏品牌識別

**診斷**:
```css
/* 當前 layout-shell.css 僅有尺寸變數 */
:root {
  --topbar-h: 56px;
  --sidebar-w-open: 260px;
  /* ... 缺少色彩變數! */
}
```

#### 1.1.2 頂部導航問題
- **缺少品牌識別**: 沒有明顯的 Logo 或品牌色
- **導航結構不清**: TopbarLevel1 與 TopbarLevel2 的分層意圖不明確
- **缺少搜尋功能**: 沒有全站搜尋框
- **通知系統缺失**: 沒有鈴鐺圖示 + 下拉通知面板

#### 1.1.3 內容布局問題
- **Banner 高度不穩定**: `--banner-aspect: 1440/300` 可能導致過高 banner
- **三欄式布局複雜**: 對於遊戲平台，可能不需要 RightRail
- **首頁內容未定義**: `Views/Home/Index.cshtml` 功能不明確

#### 1.1.4 卡片系統問題
- **無統一卡片組件**: 缺少 `.card` 統一樣式
- **陰影系統缺失**: 沒有定義陰影層級
- **Hover 效果缺失**: 卡片無互動反饋

#### 1.1.5 表單系統問題
- **驗證樣式不完整**: 缺少 `.is-invalid`, `.invalid-feedback` 統一樣式
- **無 Loading 狀態**: 表單送出時缺少視覺反饋
- **錯誤訊息不一致**: 沒有統一的錯誤提示模式

#### 1.1.6 Modal 系統問題
- **Modal 未自訂**: 使用 Bootstrap 預設樣式
- **缺少常用 Modal 模板**: 沒有確認、成功、警告等預設 Modal
- **Modal 動畫未優化**: 使用預設動畫

#### 1.1.7 社交功能問題
- **FloatingDock 程式碼過長**: 557 行 embedded JS，難以維護
- **應分離邏輯**: JS 應獨立為 `.js` 檔案
- **缺少多聊天視窗**: 目前僅支援單一聊天視窗

### 1.2 與巴哈姆特的差距分析

| 面向 | 巴哈姆特 | GamiPort 現況 | 差距等級 |
|------|---------|--------------|---------|
| 品牌色彩識別 | 強 (青綠色 #17a2b8) | 弱 (無主色) | ⚠️⚠️⚠️ 高 |
| 頂部導航設計 | 完整 (Logo + 搜尋 + 通知) | 基礎 (結構不清) | ⚠️⚠️⚠️ 高 |
| Modal 系統 | 完善 (多類型) | 基礎 (預設) | ⚠️⚠️ 中 |
| 卡片布局 | 統一 (陰影 + Hover) | 未定義 | ⚠️⚠️⚠️ 高 |
| 表單驗證 | 即時 + 清晰 | 不完整 | ⚠️⚠️ 中 |
| 通知系統 | 完整 (鈴鐺 + 面板) | 缺失 | ⚠️⚠️⚠️ 高 |
| 響應式設計 | 完善 (三斷點) | 基礎 (側邊欄) | ⚠️ 低 |
| 動畫效果 | 流暢自然 | 預設 | ⚠️ 低 |

**結論**: GamiPort 具備良好的架構基礎，但在**視覺識別、組件完整性、互動反饋**方面需要大幅強化。

---

## 2. 整體改造策略

### 2.1 改造三原則

**原則 1: 漸進式改造 (Progressive Enhancement)**
- ✅ 保留現有良好架構 (Layout, ViewComponent)
- ✅ 逐步引入新樣式與組件
- ✅ 避免破壞現有功能

**原則 2: 組件化思維 (Component-Based)**
- ✅ 建立可重用組件 (`.card`, `.btn-primary`, `.modal-confirm`)
- ✅ 統一命名規範 (`gp-` prefix)
- ✅ 文件化組件使用方式

**原則 3: 效能優先 (Performance First)**
- ✅ 最小化 CSS/JS 檔案大小
- ✅ 使用 CSS 變數減少重複
- ✅ 按需載入非關鍵資源

### 2.2 改造四階段

**Phase 1: 基礎設施改造 (Week 1-2)**
- 建立色彩系統 CSS 變數
- 改造頂部導航列
- 建立統一卡片系統
- 重構按鈕樣式

**Phase 2: 核心組件開發 (Week 3-4)**
- 開發 Modal 系統 (確認、成功、警告)
- 建立表單驗證系統
- 實作通知下拉面板
- 優化側邊欄

**Phase 3: 首頁與內容頁面 (Week 5-6)**
- 設計首頁布局
- 建立內容卡片網格
- 實作空狀態設計
- 優化響應式

**Phase 4: 進階功能與優化 (Week 7-8)**
- 實作 Loading 骨架屏
- 加入動畫與過渡
- 輔助功能優化
- 效能調校

### 2.3 技術棧確認

**保持不變**:
- ASP.NET Core 8.0 MVC
- Bootstrap 5.3.3
- Bootstrap Icons 1.11.3
- SignalR (for chat)

**新增**:
- Custom CSS variables (色彩、陰影、間距系統)
- Modular JavaScript (分離 embedded JS)
- Toast notification library (考慮 Notyf 或自行實作)

---

## 3. 色彩系統重構

### 3.1 建立品牌色彩 CSS 變數

**新建檔案**: `wwwroot/css/variables.css`

```css
/**
 * GamiPort Color System
 * 參考巴哈姆特配色，採用青綠色 (Teal) 作為主色
 */
:root {
  /* === Primary Colors === */
  --gp-primary: #17a2b8;
  --gp-primary-hover: #138496;
  --gp-primary-active: #117a8b;
  --gp-primary-light: #e7f5ff;
  --gp-primary-dark: #0d6475;

  /* === Neutral Colors === */
  --gp-white: #ffffff;
  --gp-gray-50: #f8f9fa;
  --gp-gray-100: #f1f3f5;
  --gp-gray-200: #e9ecef;
  --gp-gray-300: #dee2e6;
  --gp-gray-400: #ced4da;
  --gp-gray-500: #adb5bd;
  --gp-gray-600: #6c757d;
  --gp-gray-700: #495057;
  --gp-gray-800: #343a40;
  --gp-gray-900: #212529;
  --gp-black: #000000;

  /* === Semantic Colors === */
  --gp-success: #28a745;
  --gp-success-bg: #d4edda;
  --gp-success-border: #c3e6cb;

  --gp-danger: #dc3545;
  --gp-danger-bg: #f8d7da;
  --gp-danger-border: #f5c6cb;

  --gp-warning: #ffc107;
  --gp-warning-bg: #fff3cd;
  --gp-warning-border: #ffeaa7;

  --gp-info: #17a2b8;
  --gp-info-bg: #d1ecf1;
  --gp-info-border: #bee5eb;

  /* === Text Colors === */
  --gp-text-primary: #212529;
  --gp-text-secondary: #495057;
  --gp-text-muted: #6c757d;
  --gp-text-disabled: #adb5bd;
  --gp-text-link: #17a2b8;

  /* === Background Colors === */
  --gp-bg-body: #ffffff;
  --gp-bg-secondary: #f8f9fa;
  --gp-bg-tertiary: #f1f3f5;

  /* === Border Colors === */
  --gp-border-color: #dee2e6;
  --gp-border-light: #e9ecef;
  --gp-border-focus: #80bdff;

  /* === Shadow System === */
  --gp-shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --gp-shadow: 0 2px 4px rgba(0, 0, 0, 0.08);
  --gp-shadow-md: 0 4px 8px rgba(0, 0, 0, 0.12);
  --gp-shadow-lg: 0 8px 16px rgba(0, 0, 0, 0.15);
  --gp-shadow-xl: 0 16px 32px rgba(0, 0, 0, 0.2);

  /* === Spacing System (8px base) === */
  --gp-space-1: 4px;
  --gp-space-2: 8px;
  --gp-space-3: 12px;
  --gp-space-4: 16px;
  --gp-space-5: 24px;
  --gp-space-6: 32px;
  --gp-space-7: 40px;
  --gp-space-8: 48px;

  /* === Border Radius === */
  --gp-radius-sm: 4px;
  --gp-radius: 6px;
  --gp-radius-md: 8px;
  --gp-radius-lg: 12px;
  --gp-radius-pill: 24px;
  --gp-radius-circle: 50%;

  /* === Transitions === */
  --gp-transition-fast: 150ms;
  --gp-transition-base: 200ms;
  --gp-transition-medium: 250ms;
  --gp-transition-slow: 300ms;

  /* === Z-index System === */
  --gp-z-base: 1;
  --gp-z-dropdown: 1000;
  --gp-z-sticky: 1020;
  --gp-z-fixed: 1030;
  --gp-z-modal-backdrop: 1040;
  --gp-z-modal: 1050;
  --gp-z-popover: 1060;
  --gp-z-tooltip: 1070;
  --gp-z-toast: 1090;

  /* === Layout Dimensions (保留現有，並擴充) === */
  --gp-topbar-height: 56px;
  --gp-topbar-gap: 8px;
  --gp-sidebar-width-open: 260px;
  --gp-sidebar-width-collapsed: 72px;
  --gp-sidebar-width: var(--gp-sidebar-width-open);
  --gp-container-max-width: 1200px;
  --gp-content-max-width: 720px;
  --gp-form-max-width: 560px;
}
```

### 3.2 整合到 _Layout.cshtml

**修改**: `Views/Shared/_Layout.cshtml`

```html
<head>
    <!-- 現有內容 -->

    <!-- GamiPort Custom Styles - 按順序載入 -->
    <link rel="stylesheet" href="~/css/variables.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/layout-shell.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/buttons.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/cards.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/modals.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/forms.css" asp-append-version="true" />
    <!-- 更多組件 CSS... -->
</head>
```

### 3.3 更新 layout-shell.css

**修改**: `wwwroot/css/layout-shell.css`

```css
/* 移除舊的 :root 變數定義，改為使用 variables.css */

/* === Topbar Level 1 (主導航) === */
.topbar-level1 {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: var(--gp-topbar-height);
  background-color: var(--gp-primary); /* 青綠色! */
  color: var(--gp-white);
  box-shadow: var(--gp-shadow-md);
  z-index: var(--gp-z-fixed);
}

/* === Topbar Level 2 (麵包屑/操作列) === */
.topbar-level2 {
  height: 48px;
  background-color: var(--gp-bg-secondary);
  border-bottom: 1px solid var(--gp-border-color);
  padding: 0 var(--gp-space-4);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

/* === Main Container === */
.main-container {
  margin-top: var(--gp-topbar-height);
  min-height: calc(100vh - var(--gp-topbar-height));
}

/* === Sidebar === */
.sidebar {
  position: fixed;
  top: var(--gp-topbar-height);
  left: 0;
  width: var(--gp-sidebar-width);
  height: calc(100vh - var(--gp-topbar-height));
  background-color: var(--gp-white);
  border-right: 1px solid var(--gp-border-color);
  transition: width var(--gp-transition-medium) ease;
  z-index: var(--gp-z-sticky);
  overflow-x: hidden;
  overflow-y: auto;
}

.sidebar.collapsed {
  --gp-sidebar-width: var(--gp-sidebar-width-collapsed);
}

/* === Content Area === */
.content-area {
  margin-left: var(--gp-sidebar-width);
  padding: var(--gp-space-5);
  transition: margin-left var(--gp-transition-medium) ease;
}

.sidebar.collapsed + .content-area {
  margin-left: var(--gp-sidebar-width-collapsed);
}

/* === Footer === */
.footer {
  background-color: var(--gp-gray-800);
  color: var(--gp-gray-300);
  padding: var(--gp-space-6) 0;
  margin-top: var(--gp-space-8);
}
```

### 3.4 Bootstrap 主題覆寫

**新建**: `wwwroot/css/bootstrap-overrides.css`

```css
/**
 * Bootstrap Theme Overrides for GamiPort
 * 覆寫 Bootstrap 預設色為 GamiPort 品牌色
 */

/* Primary Color Override */
.btn-primary {
  --bs-btn-bg: var(--gp-primary);
  --bs-btn-border-color: var(--gp-primary);
  --bs-btn-hover-bg: var(--gp-primary-hover);
  --bs-btn-hover-border-color: var(--gp-primary-hover);
  --bs-btn-active-bg: var(--gp-primary-active);
  --bs-btn-active-border-color: var(--gp-primary-active);
}

/* Link Color Override */
a {
  color: var(--gp-text-link);
  text-decoration: none;
}

a:hover {
  color: var(--gp-primary-hover);
  text-decoration: underline;
}

/* Form Control Focus */
.form-control:focus,
.form-select:focus {
  border-color: var(--gp-border-focus);
  box-shadow: 0 0 0 0.2rem rgba(23, 162, 184, 0.25);
}

/* Badge Primary */
.badge.bg-primary {
  background-color: var(--gp-primary) !important;
}
```

---

## 4. 頂部導航列改造

### 4.1 TopbarLevel1 重新設計

**目標**: 參考巴哈姆特，建立包含 Logo、導航項目、搜尋、通知、用戶的完整頂部導航。

**修改**: `Views/Shared/Components/TopbarLevel1/Default.cshtml`

```html
<nav class="topbar-level1">
    <div class="container-fluid">
        <div class="topbar-content">
            <!-- 左側: Logo + 主要導航 -->
            <div class="topbar-left">
                <!-- Logo -->
                <a href="/" class="topbar-brand">
                    <img src="~/images/logo-white.svg" alt="GamiPort" class="topbar-logo" />
                    <span class="topbar-brand-text">GamiPort</span>
                </a>

                <!-- 主要導航項目 (桌面可見) -->
                <ul class="topbar-nav d-none d-lg-flex">
                    <li class="topbar-nav-item">
                        <a href="/" class="topbar-nav-link active">
                            <i class="bi bi-house-door"></i>
                            <span>首頁</span>
                        </a>
                    </li>
                    <li class="topbar-nav-item">
                        <a href="/MiniGame" class="topbar-nav-link">
                            <i class="bi bi-controller"></i>
                            <span>小遊戲</span>
                        </a>
                    </li>
                    <li class="topbar-nav-item">
                        <a href="/Store" class="topbar-nav-link">
                            <i class="bi bi-shop"></i>
                            <span>商城</span>
                        </a>
                    </li>
                    <li class="topbar-nav-item">
                        <a href="/Community" class="topbar-nav-link">
                            <i class="bi bi-people"></i>
                            <span>社群</span>
                        </a>
                    </li>
                </ul>
            </div>

            <!-- 右側: 搜尋 + 通知 + 用戶 -->
            <div class="topbar-right">
                <!-- 搜尋框 -->
                <div class="topbar-search d-none d-md-block">
                    <div class="search-input-wrapper">
                        <i class="bi bi-search search-icon"></i>
                        <input type="text"
                               class="search-input"
                               placeholder="搜尋遊戲或內容..."
                               id="topbarSearchInput" />
                        <button class="search-clear d-none" id="searchClearBtn">
                            <i class="bi bi-x"></i>
                        </button>
                    </div>
                </div>

                <!-- 通知鈴鐺 -->
                <div class="topbar-notification" id="notificationDropdown">
                    <button class="topbar-icon-btn"
                            data-bs-toggle="dropdown"
                            aria-expanded="false">
                        <i class="bi bi-bell"></i>
                        <span class="notification-badge" id="notificationBadge">3</span>
                    </button>
                    <div class="dropdown-menu dropdown-menu-end notification-dropdown">
                        <!-- 通知內容將在後面章節詳述 -->
                        <div class="notification-header">
                            <h6>通知</h6>
                            <button class="btn-text-sm">全部標為已讀</button>
                        </div>
                        <div class="notification-list" id="notificationList">
                            <!-- 動態載入通知項目 -->
                        </div>
                        <div class="notification-footer">
                            <a href="/Notifications" class="btn-text">查看全部</a>
                        </div>
                    </div>
                </div>

                <!-- 用戶下拉選單 -->
                <div class="topbar-user dropdown">
                    @if (User.Identity.IsAuthenticated)
                    {
                        <button class="topbar-user-btn"
                                data-bs-toggle="dropdown"
                                aria-expanded="false">
                            <img src="@(ViewBag.UserAvatar ?? "/images/default-avatar.png")"
                                 alt="@User.Identity.Name"
                                 class="topbar-avatar" />
                            <span class="topbar-username d-none d-lg-inline">@User.Identity.Name</span>
                            <i class="bi bi-chevron-down"></i>
                        </button>
                        <ul class="dropdown-menu dropdown-menu-end user-dropdown">
                            <li>
                                <div class="dropdown-header">
                                    <strong>@User.Identity.Name</strong>
                                    <small class="text-muted">@(ViewBag.UserEmail)</small>
                                </div>
                            </li>
                            <li><hr class="dropdown-divider"></li>
                            <li>
                                <a class="dropdown-item" href="/Profile">
                                    <i class="bi bi-person"></i> 個人資料
                                </a>
                            </li>
                            <li>
                                <a class="dropdown-item" href="/Settings">
                                    <i class="bi bi-gear"></i> 設定
                                </a>
                            </li>
                            <li>
                                <a class="dropdown-item" href="/Wallet">
                                    <i class="bi bi-wallet2"></i> 我的錢包
                                </a>
                            </li>
                            <li><hr class="dropdown-divider"></li>
                            <li>
                                <form method="post" action="/Account/Logout">
                                    @Html.AntiForgeryToken()
                                    <button type="submit" class="dropdown-item text-danger">
                                        <i class="bi bi-box-arrow-right"></i> 登出
                                    </button>
                                </form>
                            </li>
                        </ul>
                    }
                    else
                    {
                        <a href="/Account/Login" class="btn btn-sm btn-light">登入</a>
                    }
                </div>

                <!-- 漢堡選單 (手機) -->
                <button class="topbar-hamburger d-lg-none"
                        data-bs-toggle="offcanvas"
                        data-bs-target="#mobileNav">
                    <i class="bi bi-list"></i>
                </button>
            </div>
        </div>
    </div>
</nav>
```

### 4.2 Topbar 樣式

**新建**: `wwwroot/css/components/topbar.css`

```css
/**
 * Topbar Level 1 Styles
 * 參考巴哈姆特設計
 */

.topbar-level1 {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: var(--gp-topbar-height);
  background: linear-gradient(135deg, var(--gp-primary) 0%, var(--gp-primary-dark) 100%);
  color: var(--gp-white);
  box-shadow: var(--gp-shadow-md);
  z-index: var(--gp-z-fixed);
}

.topbar-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 100%;
  padding: 0 var(--gp-space-4);
}

/* === 左側區域 === */
.topbar-left {
  display: flex;
  align-items: center;
  gap: var(--gp-space-6);
}

/* Logo */
.topbar-brand {
  display: flex;
  align-items: center;
  gap: var(--gp-space-2);
  color: var(--gp-white);
  text-decoration: none;
  font-weight: 600;
  font-size: 20px;
  transition: opacity var(--gp-transition-fast);
}

.topbar-brand:hover {
  opacity: 0.9;
  color: var(--gp-white);
  text-decoration: none;
}

.topbar-logo {
  height: 32px;
  width: auto;
}

/* 導航項目 */
.topbar-nav {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
  gap: var(--gp-space-2);
}

.topbar-nav-link {
  display: flex;
  align-items: center;
  gap: var(--gp-space-2);
  padding: var(--gp-space-2) var(--gp-space-3);
  color: rgba(255, 255, 255, 0.9);
  text-decoration: none;
  border-radius: var(--gp-radius-sm);
  font-size: 14px;
  transition: all var(--gp-transition-fast);
  position: relative;
}

.topbar-nav-link:hover {
  background-color: rgba(255, 255, 255, 0.1);
  color: var(--gp-white);
}

.topbar-nav-link.active {
  color: var(--gp-white);
  font-weight: 500;
}

.topbar-nav-link.active::after {
  content: '';
  position: absolute;
  bottom: -12px;
  left: 50%;
  transform: translateX(-50%);
  width: 60%;
  height: 3px;
  background-color: var(--gp-white);
  border-radius: 2px;
}

/* === 右側區域 === */
.topbar-right {
  display: flex;
  align-items: center;
  gap: var(--gp-space-3);
}

/* 搜尋框 */
.topbar-search {
  position: relative;
}

.search-input-wrapper {
  position: relative;
  width: 240px;
  transition: width var(--gp-transition-medium);
}

.search-input-wrapper:focus-within {
  width: 320px;
}

.search-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--gp-gray-600);
  font-size: 16px;
  pointer-events: none;
}

.search-input {
  width: 100%;
  height: 36px;
  padding: 0 36px 0 36px;
  background-color: rgba(255, 255, 255, 0.2);
  border: 1px solid transparent;
  border-radius: var(--gp-radius-pill);
  color: var(--gp-white);
  font-size: 14px;
  transition: all var(--gp-transition-base);
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.7);
}

.search-input:focus {
  background-color: var(--gp-white);
  color: var(--gp-gray-900);
  outline: none;
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.2);
}

.search-input:focus::placeholder {
  color: var(--gp-gray-500);
}

.search-clear {
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  height: 20px;
  padding: 0;
  background: none;
  border: none;
  color: var(--gp-gray-600);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background-color var(--gp-transition-fast);
}

.search-clear:hover {
  background-color: var(--gp-gray-200);
}

/* 圖示按鈕 (通知、用戶) */
.topbar-icon-btn {
  position: relative;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
  border: none;
  color: var(--gp-white);
  font-size: 20px;
  border-radius: var(--gp-radius-circle);
  cursor: pointer;
  transition: background-color var(--gp-transition-fast);
}

.topbar-icon-btn:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

/* 通知徽章 */
.notification-badge {
  position: absolute;
  top: 6px;
  right: 6px;
  min-width: 18px;
  height: 18px;
  padding: 0 4px;
  background-color: var(--gp-danger);
  color: var(--gp-white);
  font-size: 11px;
  font-weight: 600;
  line-height: 18px;
  text-align: center;
  border-radius: var(--gp-radius-pill);
  box-shadow: 0 0 0 2px var(--gp-primary);
}

/* 用戶按鈕 */
.topbar-user-btn {
  display: flex;
  align-items: center;
  gap: var(--gp-space-2);
  padding: var(--gp-space-1) var(--gp-space-2);
  background: none;
  border: none;
  color: var(--gp-white);
  border-radius: var(--gp-radius-pill);
  cursor: pointer;
  transition: background-color var(--gp-transition-fast);
}

.topbar-user-btn:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.topbar-avatar {
  width: 32px;
  height: 32px;
  border-radius: var(--gp-radius-circle);
  object-fit: cover;
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.topbar-username {
  font-size: 14px;
  font-weight: 500;
  max-width: 120px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 漢堡選單 (手機) */
.topbar-hamburger {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
  border: none;
  color: var(--gp-white);
  font-size: 24px;
  border-radius: var(--gp-radius-sm);
  cursor: pointer;
  transition: background-color var(--gp-transition-fast);
}

.topbar-hamburger:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

/* === 響應式 === */
@media (max-width: 991px) {
  .topbar-nav {
    display: none;
  }

  .search-input-wrapper {
    width: 180px;
  }

  .search-input-wrapper:focus-within {
    width: 240px;
  }
}

@media (max-width: 767px) {
  .topbar-search {
    display: none; /* 手機搜尋移至展開選單 */
  }

  .topbar-brand-text {
    display: none; /* 僅顯示 Logo */
  }
}
```

### 4.3 通知下拉面板

**新建**: `wwwroot/css/components/notification-dropdown.css`

```css
/**
 * Notification Dropdown Styles
 */

.notification-dropdown {
  width: 400px;
  max-height: 600px;
  overflow: hidden;
  border: none;
  border-radius: var(--gp-radius-md);
  box-shadow: var(--gp-shadow-xl);
  padding: 0;
}

/* Header */
.notification-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--gp-space-4);
  border-bottom: 1px solid var(--gp-border-color);
  background-color: var(--gp-bg-secondary);
}

.notification-header h6 {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
}

.btn-text-sm {
  background: none;
  border: none;
  color: var(--gp-primary);
  font-size: 13px;
  cursor: pointer;
  padding: 0;
}

.btn-text-sm:hover {
  text-decoration: underline;
}

/* List */
.notification-list {
  max-height: 480px;
  overflow-y: auto;
}

.notification-item {
  padding: var(--gp-space-4);
  border-bottom: 1px solid var(--gp-border-light);
  cursor: pointer;
  transition: background-color var(--gp-transition-fast);
  position: relative;
}

.notification-item:hover {
  background-color: var(--gp-bg-secondary);
}

.notification-item.unread {
  background-color: var(--gp-primary-light);
}

.notification-item.unread::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 4px;
  background-color: var(--gp-primary);
}

.notification-icon {
  font-size: 20px;
  margin-right: var(--gp-space-3);
}

.notification-content {
  flex: 1;
}

.notification-title {
  font-size: 14px;
  font-weight: 600;
  margin-bottom: var(--gp-space-1);
}

.notification-message {
  font-size: 13px;
  color: var(--gp-text-secondary);
  margin-bottom: var(--gp-space-1);
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.notification-time {
  font-size: 12px;
  color: var(--gp-text-muted);
}

/* Footer */
.notification-footer {
  padding: var(--gp-space-3);
  text-align: center;
  border-top: 1px solid var(--gp-border-color);
  background-color: var(--gp-bg-secondary);
}

.notification-footer .btn-text {
  color: var(--gp-primary);
  font-size: 14px;
  text-decoration: none;
}

.notification-footer .btn-text:hover {
  text-decoration: underline;
}

/* Empty State */
.notification-empty {
  padding: var(--gp-space-8) var(--gp-space-4);
  text-align: center;
  color: var(--gp-text-muted);
}

.notification-empty-icon {
  font-size: 48px;
  margin-bottom: var(--gp-space-3);
  opacity: 0.5;
}

/* === 響應式 === */
@media (max-width: 767px) {
  .notification-dropdown {
    width: calc(100vw - 32px);
    max-width: 400px;
  }
}
```

---

## 5. 側邊欄優化

### 5.1 側邊欄結構優化

**目標**: 保留現有折疊功能，優化樣式，添加主動狀態高亮。

**修改**: `Views/Shared/_Sidebar.cshtml`

```html
<aside class="sidebar" id="mainSidebar">
    <div class="sidebar-content">
        <!-- 側邊欄 Header (可選) -->
        <div class="sidebar-header">
            <span class="sidebar-header-text">選單</span>
            <button class="sidebar-toggle-btn" id="sidebarToggleBtn" title="展開/收合 (快捷鍵: [)">
                <i class="bi bi-chevron-left" id="sidebarToggleIcon"></i>
            </button>
        </div>

        <!-- 側邊欄導航 -->
        <nav class="sidebar-nav">
            <ul class="sidebar-nav-list">
                <!-- 首頁 -->
                <li class="sidebar-nav-item">
                    <a href="/" class="sidebar-nav-link active" data-bs-toggle="tooltip" title="首頁">
                        <i class="bi bi-house-door sidebar-nav-icon"></i>
                        <span class="sidebar-nav-text">首頁</span>
                    </a>
                </li>

                <!-- 小遊戲 -->
                <li class="sidebar-nav-item">
                    <a href="/MiniGame" class="sidebar-nav-link" data-bs-toggle="tooltip" title="小遊戲">
                        <i class="bi bi-controller sidebar-nav-icon"></i>
                        <span class="sidebar-nav-text">小遊戲</span>
                    </a>
                </li>

                <!-- 我的錢包 -->
                <li class="sidebar-nav-item">
                    <a href="/Wallet" class="sidebar-nav-link" data-bs-toggle="tooltip" title="我的錢包">
                        <i class="bi bi-wallet2 sidebar-nav-icon"></i>
                        <span class="sidebar-nav-text">我的錢包</span>
                    </a>
                </li>

                <!-- 序號兌換 -->
                <li class="sidebar-nav-item">
                    <a href="/SerialCode" class="sidebar-nav-link" data-bs-toggle="tooltip" title="序號兌換">
                        <i class="bi bi-gift sidebar-nav-icon"></i>
                        <span class="sidebar-nav-text">序號兌換</span>
                    </a>
                </li>

                <!-- 分隔線 -->
                <li class="sidebar-divider"></li>

                <!-- 社群 -->
                <li class="sidebar-nav-item">
                    <a href="/Community" class="sidebar-nav-link" data-bs-toggle="tooltip" title="社群">
                        <i class="bi bi-people sidebar-nav-icon"></i>
                        <span class="sidebar-nav-text">社群</span>
                    </a>
                </li>

                <!-- 好友 -->
                <li class="sidebar-nav-item">
                    <a href="/Friends" class="sidebar-nav-link" data-bs-toggle="tooltip" title="好友">
                        <i class="bi bi-person-check sidebar-nav-icon"></i>
                        <span class="sidebar-nav-text">好友</span>
                    </a>
                </li>

                <!-- 分隔線 -->
                <li class="sidebar-divider"></li>

                <!-- 設定 -->
                <li class="sidebar-nav-item">
                    <a href="/Settings" class="sidebar-nav-link" data-bs-toggle="tooltip" title="設定">
                        <i class="bi bi-gear sidebar-nav-icon"></i>
                        <span class="sidebar-nav-text">設定</span>
                    </a>
                </li>
            </ul>
        </nav>
    </div>
</aside>
```

### 5.2 側邊欄樣式

**修改**: `wwwroot/css/components/sidebar.css` (或整合至 layout-shell.css)

```css
/**
 * Sidebar Styles
 * 參考巴哈姆特，簡潔垂直導航
 */

.sidebar {
  position: fixed;
  top: var(--gp-topbar-height);
  left: 0;
  width: var(--gp-sidebar-width);
  height: calc(100vh - var(--gp-topbar-height));
  background-color: var(--gp-white);
  border-right: 1px solid var(--gp-border-color);
  transition: width var(--gp-transition-medium) ease;
  z-index: var(--gp-z-sticky);
  overflow-x: hidden;
  overflow-y: auto;
  box-shadow: var(--gp-shadow-sm);
}

.sidebar.collapsed {
  width: var(--gp-sidebar-width-collapsed);
}

/* Scrollbar 樣式 (Webkit) */
.sidebar::-webkit-scrollbar {
  width: 6px;
}

.sidebar::-webkit-scrollbar-track {
  background: transparent;
}

.sidebar::-webkit-scrollbar-thumb {
  background: var(--gp-gray-300);
  border-radius: 3px;
}

.sidebar::-webkit-scrollbar-thumb:hover {
  background: var(--gp-gray-400);
}

/* === Sidebar Header === */
.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 56px;
  padding: 0 var(--gp-space-4);
  border-bottom: 1px solid var(--gp-border-color);
}

.sidebar-header-text {
  font-size: 14px;
  font-weight: 600;
  color: var(--gp-text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  transition: opacity var(--gp-transition-fast);
}

.sidebar.collapsed .sidebar-header-text {
  opacity: 0;
  width: 0;
}

.sidebar-toggle-btn {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
  border: none;
  color: var(--gp-text-secondary);
  border-radius: var(--gp-radius-sm);
  cursor: pointer;
  transition: all var(--gp-transition-fast);
}

.sidebar-toggle-btn:hover {
  background-color: var(--gp-bg-secondary);
  color: var(--gp-primary);
}

.sidebar.collapsed #sidebarToggleIcon {
  transform: rotate(180deg);
}

/* === Sidebar Navigation === */
.sidebar-nav {
  padding: var(--gp-space-4) 0;
}

.sidebar-nav-list {
  list-style: none;
  margin: 0;
  padding: 0;
}

.sidebar-nav-item {
  margin-bottom: var(--gp-space-1);
}

.sidebar-nav-link {
  display: flex;
  align-items: center;
  gap: var(--gp-space-3);
  padding: var(--gp-space-3) var(--gp-space-4);
  color: var(--gp-text-secondary);
  text-decoration: none;
  transition: all var(--gp-transition-fast);
  position: relative;
  border-radius: 0;
}

.sidebar-nav-link:hover {
  background-color: var(--gp-bg-secondary);
  color: var(--gp-primary);
}

.sidebar-nav-link.active {
  background-color: var(--gp-primary-light);
  color: var(--gp-primary);
  font-weight: 500;
}

.sidebar-nav-link.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 4px;
  background-color: var(--gp-primary);
}

.sidebar-nav-icon {
  flex-shrink: 0;
  font-size: 20px;
  width: 20px;
  text-align: center;
}

.sidebar-nav-text {
  font-size: 14px;
  white-space: nowrap;
  transition: opacity var(--gp-transition-fast);
}

.sidebar.collapsed .sidebar-nav-text {
  opacity: 0;
  width: 0;
  overflow: hidden;
}

.sidebar.collapsed .sidebar-nav-link {
  justify-content: center;
  padding: var(--gp-space-3) 0;
}

/* 分隔線 */
.sidebar-divider {
  height: 1px;
  background-color: var(--gp-border-color);
  margin: var(--gp-space-4) var(--gp-space-4);
}

/* === 響應式 === */
@media (max-width: 991px) {
  .sidebar {
    transform: translateX(-100%);
    transition: transform var(--gp-transition-medium) ease;
  }

  .sidebar.show {
    transform: translateX(0);
  }

  /* 或使用 Bootstrap Offcanvas */
}
```

---

## 6. Layout Shell 架構調整

### 6.1 簡化三欄式布局

**問題**: 當前 `TopbarLevel2` + `MainContent` + `RightRail` 三欄過於複雜。

**建議**:
- 保留 `TopbarLevel2` 作為**麵包屑 / 頁面操作列** (contextual)
- 移除 `RightRail` (或僅在特定頁面使用)
- `MainContent` 為主要內容區

**修改**: `Views/Shared/_Layout.cshtml`

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - GamiPort</title>

    <!-- Stylesheets -->
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css" />
    <link rel="stylesheet" href="~/css/variables.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/bootstrap-overrides.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/layout-shell.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/topbar.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/sidebar.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/buttons.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/cards.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/modals.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/forms.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/components/notification-dropdown.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body>
    <!-- Topbar Level 1: 主導航 -->
    @await Component.InvokeAsync("TopbarLevel1")

    <!-- Main Container -->
    <div class="main-container">
        <!-- Sidebar -->
        @await Html.PartialAsync("_Sidebar")

        <!-- Content Wrapper -->
        <div class="content-wrapper">
            <!-- Topbar Level 2: 麵包屑/操作列 (可選) -->
            @if (ViewData["ShowTopbarLevel2"] as bool? ?? false)
            {
                <div class="topbar-level2">
                    <nav aria-label="breadcrumb">
                        <ol class="breadcrumb mb-0">
                            @RenderSection("Breadcrumb", required: false)
                        </ol>
                    </nav>
                    <div class="topbar-actions">
                        @RenderSection("TopbarActions", required: false)
                    </div>
                </div>
            }

            <!-- Main Content Area -->
            <main class="content-area" role="main">
                @RenderBody()
            </main>

            <!-- Footer -->
            <footer class="footer">
                <div class="container">
                    <div class="row">
                        <div class="col-md-4">
                            <h6>GamiPort</h6>
                            <p class="text-muted small">您的遊戲娛樂平台</p>
                        </div>
                        <div class="col-md-4">
                            <h6>快速連結</h6>
                            <ul class="list-unstyled small">
                                <li><a href="/About" class="text-muted">關於我們</a></li>
                                <li><a href="/Terms" class="text-muted">服務條款</a></li>
                                <li><a href="/Privacy" class="text-muted">隱私權政策</a></li>
                            </ul>
                        </div>
                        <div class="col-md-4">
                            <h6>聯絡我們</h6>
                            <p class="text-muted small">
                                Email: support@gamiport.com<br>
                                客服時間: 10:00 - 22:00
                            </p>
                        </div>
                    </div>
                    <hr class="my-4" />
                    <p class="text-center text-muted small mb-0">
                        &copy; 2025 GamiPort. All rights reserved.
                    </p>
                </div>
            </footer>
        </div>
    </div>

    <!-- Floating Components -->
    @if (User.Identity.IsAuthenticated)
    {
        @await Html.PartialAsync("_FloatingDock")
    }

    <!-- Scripts -->
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/sidebar.js" asp-append-version="true"></script>
    <script src="~/js/topbar-search.js" asp-append-version="true"></script>
    <script src="~/js/notifications.js" asp-append-version="true"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

---

## 7. 首頁 (Home/Index) 重新設計

### 7.1 首頁布局規劃

**目標**: 參考巴哈姆特，建立清晰的首頁布局，展示重點內容。

**內容區塊**:
1. **Banner 輪播** - 置頂大圖輪播 (活動、新遊戲)
2. **快速操作卡片** - 4 個主要功能入口
3. **熱門小遊戲** - 橫向卡片展示
4. **最新消息** - 列表式展示
5. **社群動態** - 用戶互動內容

**新建**: `Views/Home/Index.cshtml`

```html
@{
    ViewData["Title"] = "首頁";
    ViewData["ShowTopbarLevel2"] = false; // 首頁不顯示麵包屑
}

<!-- Banner 輪播區 -->
<section class="hero-banner">
    <div id="heroBannerCarousel" class="carousel slide" data-bs-ride="carousel">
        <div class="carousel-indicators">
            <button type="button" data-bs-target="#heroBannerCarousel" data-bs-slide-to="0" class="active"></button>
            <button type="button" data-bs-target="#heroBannerCarousel" data-bs-slide-to="1"></button>
            <button type="button" data-bs-target="#heroBannerCarousel" data-bs-slide-to="2"></button>
        </div>
        <div class="carousel-inner">
            <div class="carousel-item active">
                <img src="~/images/banners/banner1.jpg" class="d-block w-100" alt="Banner 1">
                <div class="carousel-caption">
                    <h2>歡迎來到 GamiPort</h2>
                    <p>探索精彩的遊戲世界</p>
                    <a href="/MiniGame" class="btn btn-light btn-lg">立即開始</a>
                </div>
            </div>
            <div class="carousel-item">
                <img src="~/images/banners/banner2.jpg" class="d-block w-100" alt="Banner 2">
                <div class="carousel-caption">
                    <h2>新遊戲上線</h2>
                    <p>快來體驗最新小遊戲</p>
                    <a href="/MiniGame/Latest" class="btn btn-light btn-lg">查看詳情</a>
                </div>
            </div>
            <div class="carousel-item">
                <img src="~/images/banners/banner3.jpg" class="d-block w-100" alt="Banner 3">
                <div class="carousel-caption">
                    <h2>限時優惠活動</h2>
                    <p>儲值加碼送好禮</p>
                    <a href="/Promotions" class="btn btn-light btn-lg">了解更多</a>
                </div>
            </div>
        </div>
        <button class="carousel-control-prev" type="button" data-bs-target="#heroBannerCarousel" data-bs-slide="prev">
            <span class="carousel-control-prev-icon"></span>
        </button>
        <button class="carousel-control-next" type="button" data-bs-target="#heroBannerCarousel" data-bs-slide="next">
            <span class="carousel-control-next-icon"></span>
        </button>
    </div>
</section>

<!-- 快速操作區 -->
<section class="quick-actions-section">
    <div class="container-content">
        <div class="row g-4">
            <div class="col-md-3 col-sm-6">
                <a href="/MiniGame" class="quick-action-card">
                    <div class="quick-action-icon">
                        <i class="bi bi-controller"></i>
                    </div>
                    <h5>小遊戲</h5>
                    <p class="text-muted">探索有趣的遊戲</p>
                </a>
            </div>
            <div class="col-md-3 col-sm-6">
                <a href="/Wallet" class="quick-action-card">
                    <div class="quick-action-icon">
                        <i class="bi bi-wallet2"></i>
                    </div>
                    <h5>我的錢包</h5>
                    <p class="text-muted">查看餘額與交易</p>
                </a>
            </div>
            <div class="col-md-3 col-sm-6">
                <a href="/SerialCode" class="quick-action-card">
                    <div class="quick-action-icon">
                        <i class="bi bi-gift"></i>
                    </div>
                    <h5>序號兌換</h5>
                    <p class="text-muted">輸入序號領獎勵</p>
                </a>
            </div>
            <div class="col-md-3 col-sm-6">
                <a href="/Community" class="quick-action-card">
                    <div class="quick-action-icon">
                        <i class="bi bi-people"></i>
                    </div>
                    <h5>社群</h5>
                    <p class="text-muted">與玩家互動交流</p>
                </a>
            </div>
        </div>
    </div>
</section>

<!-- 熱門小遊戲 -->
<section class="popular-games-section">
    <div class="container-content">
        <div class="section-header">
            <h3 class="section-title">熱門小遊戲</h3>
            <a href="/MiniGame" class="section-link">查看全部 <i class="bi bi-arrow-right"></i></a>
        </div>
        <div class="row g-4">
            @* 動態載入遊戲卡片 *@
            @for (int i = 0; i < 4; i++)
            {
                <div class="col-lg-3 col-md-4 col-sm-6">
                    <div class="game-card">
                        <div class="game-card-image">
                            <img src="~/images/games/game@(i+1).jpg" alt="Game @(i+1)" />
                            <div class="game-card-overlay">
                                <a href="/MiniGame/Play/@(i+1)" class="btn btn-light btn-sm">立即遊玩</a>
                            </div>
                        </div>
                        <div class="game-card-body">
                            <h5 class="game-card-title">遊戲名稱 @(i+1)</h5>
                            <div class="game-card-meta">
                                <span class="game-rating">
                                    <i class="bi bi-star-fill text-warning"></i> 4.5
                                </span>
                                <span class="game-plays">
                                    <i class="bi bi-play-fill"></i> 1.2K
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
            }
        </div>
    </div>
</section>

<!-- 最新消息 -->
<section class="news-section">
    <div class="container-content">
        <div class="section-header">
            <h3 class="section-title">最新消息</h3>
            <a href="/News" class="section-link">查看全部 <i class="bi bi-arrow-right"></i></a>
        </div>
        <div class="news-list">
            @for (int i = 0; i < 5; i++)
            {
                <a href="/News/@(i+1)" class="news-item">
                    <div class="news-badge">公告</div>
                    <div class="news-title">系統維護通知 - 2025/10/@(29+i)</div>
                    <div class="news-time">@(i+1) 小時前</div>
                </a>
            }
        </div>
    </div>
</section>

@section Scripts {
    <script src="~/js/home.js" asp-append-version="true"></script>
}
```

### 7.2 首頁樣式

**新建**: `wwwroot/css/pages/home.css`

```css
/**
 * Home Page Styles
 */

/* === Banner 輪播 === */
.hero-banner {
  margin-bottom: var(--gp-space-8);
}

.hero-banner .carousel-item {
  height: 400px;
  position: relative;
}

.hero-banner .carousel-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: brightness(0.8);
}

.hero-banner .carousel-caption {
  bottom: 50%;
  transform: translateY(50%);
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
}

.hero-banner .carousel-caption h2 {
  font-size: 42px;
  font-weight: 700;
  margin-bottom: var(--gp-space-3);
}

.hero-banner .carousel-caption p {
  font-size: 18px;
  margin-bottom: var(--gp-space-5);
}

/* === 快速操作卡片 === */
.quick-actions-section {
  margin-bottom: var(--gp-space-8);
}

.quick-action-card {
  display: block;
  padding: var(--gp-space-6);
  background-color: var(--gp-white);
  border: 1px solid var(--gp-border-color);
  border-radius: var(--gp-radius-md);
  text-align: center;
  text-decoration: none;
  color: inherit;
  transition: all var(--gp-transition-base);
  box-shadow: var(--gp-shadow);
}

.quick-action-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--gp-shadow-lg);
  border-color: var(--gp-primary);
}

.quick-action-icon {
  width: 64px;
  height: 64px;
  margin: 0 auto var(--gp-space-4);
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, var(--gp-primary), var(--gp-primary-dark));
  color: var(--gp-white);
  border-radius: var(--gp-radius-circle);
  font-size: 32px;
}

.quick-action-card h5 {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: var(--gp-space-2);
}

.quick-action-card p {
  font-size: 14px;
  margin: 0;
}

/* === Section Header === */
.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: var(--gp-space-5);
}

.section-title {
  font-size: 24px;
  font-weight: 600;
  margin: 0;
}

.section-link {
  color: var(--gp-primary);
  text-decoration: none;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: var(--gp-space-1);
  transition: color var(--gp-transition-fast);
}

.section-link:hover {
  color: var(--gp-primary-hover);
}

/* === 遊戲卡片 === */
.popular-games-section {
  margin-bottom: var(--gp-space-8);
}

.game-card {
  background-color: var(--gp-white);
  border: 1px solid var(--gp-border-color);
  border-radius: var(--gp-radius-md);
  overflow: hidden;
  transition: all var(--gp-transition-base);
  box-shadow: var(--gp-shadow);
}

.game-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--gp-shadow-lg);
}

.game-card-image {
  position: relative;
  padding-top: 133.33%; /* 3:4 aspect ratio */
  overflow: hidden;
}

.game-card-image img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.game-card-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity var(--gp-transition-base);
}

.game-card:hover .game-card-overlay {
  opacity: 1;
}

.game-card-body {
  padding: var(--gp-space-4);
}

.game-card-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: var(--gp-space-2);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.game-card-meta {
  display: flex;
  align-items: center;
  gap: var(--gp-space-4);
  font-size: 13px;
  color: var(--gp-text-muted);
}

/* === 最新消息 === */
.news-section {
  margin-bottom: var(--gp-space-8);
}

.news-list {
  background-color: var(--gp-white);
  border: 1px solid var(--gp-border-color);
  border-radius: var(--gp-radius-md);
  overflow: hidden;
  box-shadow: var(--gp-shadow);
}

.news-item {
  display: flex;
  align-items: center;
  gap: var(--gp-space-4);
  padding: var(--gp-space-4);
  border-bottom: 1px solid var(--gp-border-light);
  text-decoration: none;
  color: inherit;
  transition: background-color var(--gp-transition-fast);
}

.news-item:last-child {
  border-bottom: none;
}

.news-item:hover {
  background-color: var(--gp-bg-secondary);
}

.news-badge {
  flex-shrink: 0;
  padding: 4px 12px;
  background-color: var(--gp-primary-light);
  color: var(--gp-primary);
  border-radius: var(--gp-radius-pill);
  font-size: 12px;
  font-weight: 600;
}

.news-title {
  flex: 1;
  font-size: 14px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.news-time {
  flex-shrink: 0;
  font-size: 13px;
  color: var(--gp-text-muted);
}

/* === 響應式 === */
@media (max-width: 767px) {
  .hero-banner .carousel-item {
    height: 300px;
  }

  .hero-banner .carousel-caption h2 {
    font-size: 28px;
  }

  .hero-banner .carousel-caption p {
    font-size: 14px;
  }

  .quick-action-icon {
    width: 48px;
    height: 48px;
    font-size: 24px;
  }

  .section-title {
    font-size: 20px;
  }
}
```

---

## 8. 卡片系統建立

**新建**: `wwwroot/css/components/cards.css`

```css
/**
 * Card Component Styles
 * 統一卡片系統，參考巴哈姆特設計
 */

/* === 基礎卡片 === */
.card {
  background-color: var(--gp-white);
  border: 1px solid var(--gp-border-color);
  border-radius: var(--gp-radius-md);
  box-shadow: var(--gp-shadow);
  overflow: hidden;
  transition: all var(--gp-transition-base);
}

.card:hover {
  box-shadow: var(--gp-shadow-md);
}

.card-clickable {
  cursor: pointer;
  text-decoration: none;
  color: inherit;
  display: block;
}

.card-clickable:hover {
  transform: translateY(-2px);
  box-shadow: var(--gp-shadow-lg);
}

/* === 卡片 Header === */
.card-header {
  padding: var(--gp-space-4) var(--gp-space-5);
  background-color: var(--gp-bg-secondary);
  border-bottom: 1px solid var(--gp-border-color);
  font-weight: 600;
}

/* === 卡片 Body === */
.card-body {
  padding: var(--gp-space-5);
}

.card-body-sm {
  padding: var(--gp-space-4);
}

.card-body-lg {
  padding: var(--gp-space-6);
}

/* === 卡片 Footer === */
.card-footer {
  padding: var(--gp-space-3) var(--gp-space-5);
  background-color: var(--gp-bg-secondary);
  border-top: 1px solid var(--gp-border-color);
}

/* === 卡片標題 === */
.card-title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: var(--gp-space-3);
}

.card-subtitle {
  font-size: 14px;
  color: var(--gp-text-muted);
  margin-bottom: var(--gp-space-3);
}

/* === 卡片文字 === */
.card-text {
  font-size: 14px;
  line-height: 1.6;
  color: var(--gp-text-secondary);
}

/* === 卡片圖片 === */
.card-img-top {
  width: 100%;
  height: auto;
  object-fit: cover;
}

/* === 水平卡片 === */
.card-horizontal {
  display: flex;
  flex-direction: row;
}

.card-horizontal .card-img-left {
  width: 200px;
  flex-shrink: 0;
  object-fit: cover;
}

.card-horizontal .card-body {
  flex: 1;
}

/* === 統計卡片 === */
.card-stat {
  text-align: center;
  padding: var(--gp-space-6);
}

.card-stat-value {
  font-size: 32px;
  font-weight: 700;
  color: var(--gp-primary);
  margin-bottom: var(--gp-space-2);
}

.card-stat-label {
  font-size: 14px;
  color: var(--gp-text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

/* === 響應式 === */
@media (max-width: 767px) {
  .card-body {
    padding: var(--gp-space-4);
  }

  .card-horizontal {
    flex-direction: column;
  }

  .card-horizontal .card-img-left {
    width: 100%;
    height: 200px;
  }
}
```

---

由於篇幅限制，剩餘章節 (9-20) 概要如下（部分功能已實作）：

## 9-20 章節概要與實作狀態

**9. 表單系統改造**: `.is-invalid`, `.invalid-feedback`, Loading 狀態, 驗證規則

**10. Modal 系統建立**: 確認 Modal, 成功 Modal, 警告 Modal, 自訂 Modal 模板

**11. 按鈕系統標準化**: `.btn-primary`, `.btn-outline`, `.btn-text`, `.btn-danger`, 尺寸變體

**12. 通知系統實作**: Toast 通知, 通知中心頁面, 即時推播

**13. 響應式設計強化**: 斷點調整, 手機導航 Offcanvas, 觸控優化

**14. 社交功能優化** ✅ **已實作**:
- FloatingDock (613行) - 單窗一對一聊天
- FriendDock - 好友清單 + 未讀徽章
- ProfanityFilter (104行) - 前後端穢語遮蔽
- supportHub.js (125行) - Support Hub 前端單例
- QuickFab (244行) - 浮動按鈕統一系統
- QuickFabSupport (205行) - 客訴 FAB

**15. 載入與錯誤狀態**: Skeleton Loading, 空狀態設計, 404/500 頁面

**16. 動畫與過渡效果** ✅ **部分實作**:
- QuickFab 入場動畫（交錯延遲 0/35/70ms）
- Ripple 效果
- Modal fade-in/out

**17. 輔助功能改善**: 鍵盤導航, ARIA 屬性, Focus 樣式

**18. 效能優化建議** ✅ **已實作**:
- Singleton 服務（ProfanityFilter、ChatNotifier）
- HttpContext.Items 快取（AppCurrentUser）
- AsNoTracking 查詢優化
- SignalR KeepAlive 設定

**19. 實作優先級與時程**: Phase 1-4 詳細分工與里程碑

**20. 驗收標準**: 視覺一致性, 響應式完整性, 效能指標

---

## 補充：已實作功能詳細說明

### OnlineStore 電商模組（完整實作）

**購物車系統** (SqlCartService.cs - 470行):
- 匿名/登入用戶支援（Cookie/Session）
- 即時運費計算與優惠套用
- AJAX 無刷新更新
- Navbar 徽章即時同步

**結帳流程** (CheckoutController.cs - 409行):
- 三步驟: Step1 (配送) → Step2 (付款) → Review → PlaceOrder
- TempData 串接防竄改
- 呼叫 SP `usp_Order_CreateFromCart` 原子性建單
- Success 頁面支援 orderCode/orderId 雙查詢

**ECPay 金流整合** (EcpayController.cs - 416行):
- Return (前台回傳) - GET/POST 雙向
- OrderResult (橘色測試鍵) - GET/POST 雙向
- Notify (伺服器回呼) - 唯一可信來源
- HMAC-SHA256 簽章驗證
- 冪等處理（provider + provider_txn 唯一索引）
- SO_PaymentAudit 完整稽核記錄

### Support 客訴系統（完整實作）

**SignalR Hub 跨站整合** (SupportHub.cs - 167行):
- 雙路線加入機制:
  1. Join(ticketId) - 使用者路線（Cookie 驗證）
  2. JoinAsManager(ticketId, managerId, expires, sig) - 管理員路線（簽章驗證）
- HMAC-SHA256 簽章 + 時間窗保護（120秒）
- 授權檢查：目前指派者/歷史指派者/主管權限
- CORS 支援後台連前台 (localhost:7160)

**前端單例模式** (supportHub.js - 125行):
- 自動重連 + re-join 機制
- 記住已加入的 ticket 群組
- API: `GP.support.joinTicket()` / `joinTicketAsManager()` / `on()` / `off()`

**QuickFab 整合** (QuickFabSupport.cshtml - 205行):
- 救生圈圖示按鈕
- 子選單：我要客訴/我的客訴
- 建立工單 Modal + 自動開新 tab

### Chat 聊天系統（完整實作）

**即時聊天** (FloatingDock.cshtml - 613行):
- 拖移浮動視窗（360x68vh）
- LINE 風格聊天介面
- 日期分隔線自動插入（chat-date-divider.js）
- 已讀回執與 "viewed" 時間戳

**穢語遮蔽** (ProfanityFilter.cs - 104行):
- Singleton 生命週期 + IServiceScopeFactory
- 前端自動更新：
  1. 開窗時呼叫 `/profanity/list?nocache=1`
  2. 監聽 Hub 的 `ProfanityUpdated` 事件
  3. 監聽全域 `gp-profanity-updated` 事件
- DB 存原文，輸出前遮蔽
- 即時重套遮蔽（保存原文在 `data-raw`）

**好友系統** (FriendDock.cshtml):
- 右側抽屜
- 未讀徽章顯示
- 點擊開啟 FloatingDock

### 統一認證介面（完整實作）

**IAppCurrentUser** (36行):
- Claims 優先讀取（AppUserId → NameIdentifier）
- ILoginIdentity 備援
- HttpContext.Items 同請求快取
- 使用於 ChatHub、SupportHub、Controllers

---

**文件版本**: v2.0 (更新版)
**最後更新**: 2025-10-29
**維護者**: Claude Code Analysis Team
**狀態**: 整合最新功能實作狀態
