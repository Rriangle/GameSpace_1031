# GamiPort - MiniGame Area 前台風格布局改造建議

> **撰寫日期**: 2025-10-29 (更新版)
> **改造對象**: GamiPort Client - MiniGame Area (用戶前台)
> **參考標準**: 巴哈姆特 + 遊戲化設計原則
> **改造目標**: 將管理後台功能轉化為吸引人的用戶遊戲體驗介面
> **更新說明**: 整合 QuickFab、FloatingDock、實際已有的電商/社交功能至遊戲化設計

---

## 目錄

1. [MiniGame Area 現況與改造目標](#1-minigame-area-現況與改造目標)
2. [整體架構重新規劃](#2-整體架構重新規劃)
3. [首頁 (MiniGame/Index) 設計](#3-首頁-minigameindex-設計)
4. [我的錢包 (Wallet) 介面](#4-我的錢包-wallet-介面)
5. [簽到系統 (SignIn) 設計](#5-簽到系統-signin-設計)
6. [寵物系統 (Pet) 介面](#6-寵物系統-pet-介面)
7. [遊戲大廳 (GamePlay) 設計](#7-遊戲大廳-gameplay-設計)
8. [優惠券系統 (Coupon) 介面](#8-優惠券系統-coupon-介面)
9. [電子票券 (EVoucher) 系統](#9-電子票券-evoucher-系統)
10. [序號兌換 (SerialCode) 流程](#10-序號兌換-serialcode-流程)
11. [獎勵中心 (RewardCenter) 設計](#11-獎勵中心-rewardcenter-設計)
12. [交易歷史 (TransactionHistory)](#12-交易歷史-transactionhistory)
13. [排行榜系統 (Leaderboard)](#13-排行榜系統-leaderboard)
14. [成就系統 (Achievement)](#14-成就系統-achievement)
15. [遊戲化元素設計](#15-遊戲化元素設計)
16. [即時反饋與動畫](#16-即時反饋與動畫)
17. [響應式遊戲體驗](#17-響應式遊戲體驗)
18. [效能與載入優化](#18-效能與載入優化)
19. [實作路徑與優先級](#19-實作路徑與優先級)
20. [驗收標準與測試要點](#20-驗收標準與測試要點)

---

## 1. MiniGame Area 現況與改造目標

### 1.1 後台 vs 前台差異分析

根據先前對 GameSpace MiniGame Area (管理後台) 的分析，我們了解到後台具備完整的 7 大系統功能。現在需要將這些功能**轉化為用戶友善的前台體驗**。

**後台特色** (GameSpace MiniGame):
- ✅ 完整的 CRUD 功能
- ✅ DataTables 列表展示
- ✅ 豐富的篩選與搜尋
- ✅ 詳細的統計圖表
- ✅ 管理員操作日誌
- ⚠️ 介面偏向管理導向
- ⚠️ 大量表格與表單
- ⚠️ 缺少遊戲化元素

**前台需求** (GamiPort MiniGame):
- ✅ 視覺吸引力 (遊戲風格)
- ✅ 簡化操作流程
- ✅ 遊戲化獎勵機制
- ✅ 即時反饋與動畫
- ✅ 社交互動元素
- ✅ 成就與排行榜
- ✅ 清晰的引導流程
- ✅ 沈浸式遊戲體驗

### 1.2 改造核心原則

**原則 1: 遊戲化優先 (Gamification First)**
- 將每個功能包裝為遊戲任務
- 添加進度條、等級、獎勵元素
- 使用遊戲化術語 (任務、獎勵、成就)

**原則 2: 視覺吸引力 (Visual Appeal)**
- 豐富的圖示與插圖
- 明亮的色彩與漸層
- 動畫與過渡效果
- 3D 或擬物化元素 (適度)

**原則 3: 簡化操作 (Simplified UX)**
- 減少步驟，一鍵完成
- 智能預設值
- 清晰的視覺引導
- 即時驗證與反饋

**原則 4: 社交互動 (Social Engagement)**
- 好友系統整合
- 排行榜與競爭
- 分享與炫耀機制
- 協作任務

### 1.3 功能對應表

| 後台功能 | 前台對應 | 改造重點 |
|---------|---------|---------|
| 錢包管理 | 我的錢包 | 視覺化餘額、快速操作、交易歷史 |
| 簽到管理 | 每日簽到 | 日曆UI、連續獎勵、補簽卡 |
| 寵物管理 | 我的寵物 | 卡片展示、培養介面、寵物互動 |
| 遊戲管理 | 遊戲大廳 | 卡片網格、分類篩選、立即遊玩 |
| 優惠券管理 | 我的優惠券 | 卡券展示、快速使用、有效期提醒 |
| 電子票券管理 | 我的票券 | 二維碼展示、核銷介面 |
| 序號管理 | 序號兌換 | 輸入介面、獎勵預覽、兌換確認 |

---

## 2. 整體架構重新規劃

### 2.1 路由結構

**新建**: `Areas/MiniGame/` (Client Area)

```
/MiniGame
├─ /Index                    # MiniGame 首頁 (總覽儀表板)
├─ /Wallet
│  ├─ /Index                 # 錢包首頁
│  ├─ /Deposit               # 儲值
│  ├─ /Withdraw              # 提領
│  └─ /Transactions          # 交易歷史
├─ /SignIn
│  ├─ /Index                 # 簽到首頁
│  └─ /History               # 簽到歷史
├─ /Pet
│  ├─ /Index                 # 寵物列表
│  ├─ /Detail/{id}           # 寵物詳情
│  └─ /Evolve/{id}           # 寵物進化
├─ /GamePlay
│  ├─ /Index                 # 遊戲大廳
│  ├─ /Play/{id}             # 遊戲頁面
│  └─ /Leaderboard/{id}      # 遊戲排行榜
├─ /Coupon
│  ├─ /Index                 # 優惠券列表
│  └─ /Use/{id}              # 使用優惠券
├─ /EVoucher
│  ├─ /Index                 # 票券列表
│  └─ /Detail/{id}           # 票券詳情 (含 QR Code)
├─ /SerialCode
│  └─ /Index                 # 序號兌換
├─ /Reward
│  └─ /Index                 # 獎勵中心
├─ /Achievement
│  └─ /Index                 # 成就中心
└─ /Leaderboard
   └─ /Index                 # 總排行榜
```

### 2.2 MiniGame Area Layout

**新建**: `Areas/MiniGame/Views/Shared/_MiniGameLayout.cshtml`

```html
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
    ViewData["ShowTopbarLevel2"] = true;
}

<!-- MiniGame Area 專屬樣式 -->
<link rel="stylesheet" href="~/css/areas/minigame.css" asp-append-version="true" />

<!-- Topbar Level 2: MiniGame 快速導航 -->
@section Breadcrumb {
    <li class="breadcrumb-item"><a href="/">首頁</a></li>
    <li class="breadcrumb-item"><a href="/MiniGame">小遊戲</a></li>
    @RenderSection("MiniGameBreadcrumb", required: false)
}

@section TopbarActions {
    <div class="minigame-quick-nav">
        <a href="/MiniGame/Wallet" class="quick-nav-item" title="我的錢包">
            <i class="bi bi-wallet2"></i>
            <span class="balance">@ViewBag.Balance GP</span>
        </a>
        <a href="/MiniGame/SignIn" class="quick-nav-item" title="每日簽到">
            <i class="bi bi-calendar-check"></i>
            @if (ViewBag.CanSignInToday == true)
            {
                <span class="badge bg-danger">!</span>
            }
        </a>
        <a href="/MiniGame/Pet" class="quick-nav-item" title="我的寵物">
            <i class="bi bi-heart"></i>
            <span class="badge bg-primary">@ViewBag.PetCount</span>
        </a>
        <a href="/MiniGame/Achievement" class="quick-nav-item" title="成就">
            <i class="bi bi-trophy"></i>
        </a>
    </div>
}

<!-- Main Content -->
<div class="minigame-container">
    @RenderBody()
</div>

<!-- MiniGame Area 專屬 Scripts -->
@section Scripts {
    <script src="~/js/areas/minigame-common.js" asp-append-version="true"></script>
    @RenderSection("MiniGameScripts", required: false)
}
```

### 2.3 MiniGame 專屬樣式

**新建**: `wwwroot/css/areas/minigame.css`

```css
/**
 * MiniGame Area Global Styles
 * 遊戲化設計風格
 */

:root {
  /* MiniGame 專屬色彩 */
  --mg-gold: #ffd700;
  --mg-silver: #c0c0c0;
  --mg-bronze: #cd7f32;
  --mg-gem: #9b59b6;
  --mg-energy: #e74c3c;
  --mg-exp: #3498db;

  /* 遊戲化漸層 */
  --mg-gradient-primary: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  --mg-gradient-gold: linear-gradient(135deg, #f7971e 0%, #ffd200 100%);
  --mg-gradient-gem: linear-gradient(135deg, #9b59b6 0%, #e74c3c 100%);
}

/* MiniGame Container */
.minigame-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: var(--gp-space-5);
}

/* 快速導航 */
.minigame-quick-nav {
  display: flex;
  gap: var(--gp-space-3);
}

.quick-nav-item {
  position: relative;
  display: flex;
  align-items: center;
  gap: var(--gp-space-2);
  padding: var(--gp-space-2) var(--gp-space-3);
  background-color: var(--gp-white);
  border: 1px solid var(--gp-border-color);
  border-radius: var(--gp-radius);
  color: var(--gp-text-primary);
  text-decoration: none;
  font-size: 14px;
  transition: all var(--gp-transition-fast);
}

.quick-nav-item:hover {
  background-color: var(--gp-primary-light);
  border-color: var(--gp-primary);
  color: var(--gp-primary);
  transform: translateY(-2px);
}

.quick-nav-item i {
  font-size: 18px;
}

.quick-nav-item .balance {
  font-weight: 600;
  color: var(--mg-gold);
}

.quick-nav-item .badge {
  position: absolute;
  top: -6px;
  right: -6px;
  min-width: 18px;
  height: 18px;
  padding: 0 4px;
  font-size: 11px;
  line-height: 18px;
}

/* 遊戲化標題 */
.mg-section-title {
  font-size: 24px;
  font-weight: 700;
  margin-bottom: var(--gp-space-5);
  color: var(--gp-text-primary);
  display: flex;
  align-items: center;
  gap: var(--gp-space-3);
}

.mg-section-title::before {
  content: '';
  width: 4px;
  height: 28px;
  background: var(--mg-gradient-primary);
  border-radius: 2px;
}

/* 統計卡片 */
.mg-stat-card {
  background: var(--gp-white);
  border-radius: var(--gp-radius-md);
  padding: var(--gp-space-5);
  box-shadow: var(--gp-shadow-md);
  text-align: center;
  position: relative;
  overflow: hidden;
  transition: all var(--gp-transition-base);
}

.mg-stat-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: var(--mg-gradient-primary);
}

.mg-stat-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--gp-shadow-lg);
}

.mg-stat-icon {
  width: 56px;
  height: 56px;
  margin: 0 auto var(--gp-space-3);
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--mg-gradient-primary);
  color: var(--gp-white);
  border-radius: var(--gp-radius-circle);
  font-size: 28px;
}

.mg-stat-value {
  font-size: 32px;
  font-weight: 700;
  color: var(--gp-primary);
  margin-bottom: var(--gp-space-1);
}

.mg-stat-label {
  font-size: 14px;
  color: var(--gp-text-muted);
}

/* 進度條 */
.mg-progress {
  height: 24px;
  background-color: var(--gp-gray-200);
  border-radius: var(--gp-radius-pill);
  overflow: hidden;
  position: relative;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
}

.mg-progress-bar {
  height: 100%;
  background: var(--mg-gradient-primary);
  border-radius: var(--gp-radius-pill);
  transition: width 0.6s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--gp-white);
  font-size: 12px;
  font-weight: 600;
  position: relative;
  overflow: hidden;
}

.mg-progress-bar::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(90deg,
    transparent,
    rgba(255, 255, 255, 0.3),
    transparent);
  animation: shimmer 2s infinite;
}

@keyframes shimmer {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}

/* 獎勵徽章 */
.mg-badge-gold {
  background: var(--mg-gradient-gold);
  color: var(--gp-white);
  padding: 4px 12px;
  border-radius: var(--gp-radius-pill);
  font-size: 12px;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  gap: 4px;
  box-shadow: 0 2px 8px rgba(255, 210, 0, 0.3);
}

.mg-badge-gem {
  background: var(--mg-gradient-gem);
  color: var(--gp-white);
  padding: 4px 12px;
  border-radius: var(--gp-radius-pill);
  font-size: 12px;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  gap: 4px;
  box-shadow: 0 2px 8px rgba(155, 89, 182, 0.3);
}
```

---

## 3. 首頁 (MiniGame/Index) 設計

### 3.1 儀表板布局

**新建**: `Areas/MiniGame/Views/Home/Index.cshtml`

```html
@{
    ViewData["Title"] = "小遊戲中心";
    Layout = "_MiniGameLayout";
}

<!-- 歡迎橫幅 -->
<section class="mg-welcome-banner">
    <div class="welcome-content">
        <div class="welcome-text">
            <h1>歡迎回來，<span class="username">@User.Identity.Name</span>！</h1>
            <p>今天也要開心遊玩喔 🎮</p>
        </div>
        <div class="welcome-stats">
            <div class="stat-item">
                <i class="bi bi-star-fill"></i>
                <span>等級 @ViewBag.UserLevel</span>
            </div>
            <div class="stat-item">
                <i class="bi bi-trophy-fill"></i>
                <span>@ViewBag.AchievementCount 個成就</span>
            </div>
        </div>
    </div>
</section>

<!-- 快速統計 -->
<section class="mg-dashboard-stats">
    <div class="row g-4">
        <div class="col-md-3 col-sm-6">
            <div class="mg-stat-card">
                <div class="mg-stat-icon" style="background: var(--mg-gradient-gold);">
                    <i class="bi bi-wallet2"></i>
                </div>
                <div class="mg-stat-value">@ViewBag.Balance</div>
                <div class="mg-stat-label">GP 點數</div>
                <a href="/MiniGame/Wallet" class="stretched-link"></a>
            </div>
        </div>
        <div class="col-md-3 col-sm-6">
            <div class="mg-stat-card">
                <div class="mg-stat-icon" style="background: var(--mg-gradient-gem);">
                    <i class="bi bi-gem"></i>
                </div>
                <div class="mg-stat-value">@ViewBag.Gems</div>
                <div class="mg-stat-label">鑽石</div>
            </div>
        </div>
        <div class="col-md-3 col-sm-6">
            <div class="mg-stat-card">
                <div class="mg-stat-icon" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
                    <i class="bi bi-heart-fill"></i>
                </div>
                <div class="mg-stat-value">@ViewBag.PetCount</div>
                <div class="mg-stat-label">寵物數量</div>
                <a href="/MiniGame/Pet" class="stretched-link"></a>
            </div>
        </div>
        <div class="col-md-3 col-sm-6">
            <div class="mg-stat-card">
                <div class="mg-stat-icon" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
                    <i class="bi bi-ticket-perforated"></i>
                </div>
                <div class="mg-stat-value">@ViewBag.CouponCount</div>
                <div class="mg-stat-label">可用優惠券</div>
                <a href="/MiniGame/Coupon" class="stretched-link"></a>
            </div>
        </div>
    </div>
</section>

<!-- 每日任務 -->
<section class="mg-daily-tasks">
    <h2 class="mg-section-title">
        <i class="bi bi-check2-square"></i>
        每日任務
    </h2>
    <div class="task-list">
        <div class="task-item @(ViewBag.HasSignedToday ? "completed" : "")">
            <div class="task-icon">
                <i class="bi bi-calendar-check"></i>
            </div>
            <div class="task-content">
                <h5 class="task-title">每日簽到</h5>
                <p class="task-desc">連續簽到可獲得額外獎勵</p>
            </div>
            <div class="task-reward">
                <span class="mg-badge-gold">
                    <i class="bi bi-coin"></i> +50 GP
                </span>
            </div>
            <div class="task-action">
                @if (ViewBag.HasSignedToday)
                {
                    <button class="btn btn-sm btn-success" disabled>
                        <i class="bi bi-check-lg"></i> 已完成
                    </button>
                }
                else
                {
                    <a href="/MiniGame/SignIn" class="btn btn-sm btn-primary">
                        立即簽到
                    </a>
                }
            </div>
        </div>

        <div class="task-item">
            <div class="task-icon">
                <i class="bi bi-controller"></i>
            </div>
            <div class="task-content">
                <h5 class="task-title">遊玩 3 場遊戲</h5>
                <p class="task-desc">進度: @ViewBag.TodayGamesPlayed / 3</p>
                <div class="mg-progress">
                    <div class="mg-progress-bar" style="width: @(ViewBag.TodayGamesPlayed * 33.33)%">
                        @ViewBag.TodayGamesPlayed / 3
                    </div>
                </div>
            </div>
            <div class="task-reward">
                <span class="mg-badge-gem">
                    <i class="bi bi-gem"></i> +10 鑽石
                </span>
            </div>
            <div class="task-action">
                <a href="/MiniGame/GamePlay" class="btn btn-sm btn-primary">
                    前往遊玩
                </a>
            </div>
        </div>

        <div class="task-item">
            <div class="task-icon">
                <i class="bi bi-people"></i>
            </div>
            <div class="task-content">
                <h5 class="task-title">邀請 1 位好友</h5>
                <p class="task-desc">分享快樂，獲得獎勵</p>
            </div>
            <div class="task-reward">
                <span class="mg-badge-gold">
                    <i class="bi bi-coin"></i> +100 GP
                </span>
            </div>
            <div class="task-action">
                <button class="btn btn-sm btn-primary" data-bs-toggle="modal" data-bs-target="#inviteFriendModal">
                    邀請好友
                </button>
            </div>
        </div>
    </div>
</section>

<!-- 熱門遊戲 -->
<section class="mg-featured-games">
    <div class="section-header">
        <h2 class="mg-section-title">
            <i class="bi bi-fire"></i>
            熱門遊戲
        </h2>
        <a href="/MiniGame/GamePlay" class="section-link">
            查看全部 <i class="bi bi-arrow-right"></i>
        </a>
    </div>
    <div class="row g-4">
        @for (int i = 0; i < 4; i++)
        {
            <div class="col-lg-3 col-md-4 col-sm-6">
                <div class="mg-game-card">
                    <div class="game-thumbnail">
                        <img src="~/images/games/game@(i+1).jpg" alt="Game @(i+1)" />
                        <div class="game-overlay">
                            <a href="/MiniGame/GamePlay/Play/@(i+1)" class="btn btn-light">
                                <i class="bi bi-play-fill"></i> 立即遊玩
                            </a>
                        </div>
                        <div class="game-badge">
                            <i class="bi bi-fire"></i> HOT
                        </div>
                    </div>
                    <div class="game-info">
                        <h5 class="game-title">超級彈珠台 @(i+1)</h5>
                        <div class="game-meta">
                            <span><i class="bi bi-star-fill text-warning"></i> 4.8</span>
                            <span><i class="bi bi-people-fill"></i> 2.5K</span>
                        </div>
                    </div>
                </div>
            </div>
        }
    </div>
</section>

<!-- 最新消息 -->
<section class="mg-announcements">
    <h2 class="mg-section-title">
        <i class="bi bi-megaphone"></i>
        最新消息
    </h2>
    <div class="announcement-list">
        <a href="/News/1" class="announcement-item">
            <span class="announcement-badge">活動</span>
            <span class="announcement-title">雙倍點數活動開跑！</span>
            <span class="announcement-time">2小時前</span>
        </a>
        <a href="/News/2" class="announcement-item">
            <span class="announcement-badge">更新</span>
            <span class="announcement-title">新增 5 款小遊戲</span>
            <span class="announcement-time">1天前</span>
        </a>
        <a href="/News/3" class="announcement-item">
            <span class="announcement-badge">公告</span>
            <span class="announcement-title">系統維護通知</span>
            <span class="announcement-time">3天前</span>
        </a>
    </div>
</section>

@section MiniGameScripts {
    <script src="~/js/areas/minigame-home.js" asp-append-version="true"></script>
}
```

### 3.2 首頁專屬樣式

**新建**: `wwwroot/css/areas/minigame-home.css`

```css
/**
 * MiniGame Home Page Styles
 */

/* 歡迎橫幅 */
.mg-welcome-banner {
  background: var(--mg-gradient-primary);
  color: var(--gp-white);
  padding: var(--gp-space-7) var(--gp-space-5);
  border-radius: var(--gp-radius-md);
  margin-bottom: var(--gp-space-6);
  box-shadow: var(--gp-shadow-lg);
}

.welcome-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.welcome-text h1 {
  font-size: 28px;
  font-weight: 700;
  margin-bottom: var(--gp-space-2);
}

.welcome-text .username {
  color: var(--mg-gold);
}

.welcome-text p {
  font-size: 16px;
  opacity: 0.9;
  margin: 0;
}

.welcome-stats {
  display: flex;
  gap: var(--gp-space-5);
}

.welcome-stats .stat-item {
  display: flex;
  align-items: center;
  gap: var(--gp-space-2);
  font-size: 16px;
  font-weight: 600;
}

.welcome-stats .stat-item i {
  font-size: 20px;
  color: var(--mg-gold);
}

/* 儀表板統計 */
.mg-dashboard-stats {
  margin-bottom: var(--gp-space-8);
}

/* 每日任務 */
.mg-daily-tasks {
  margin-bottom: var(--gp-space-8);
}

.task-list {
  display: flex;
  flex-direction: column;
  gap: var(--gp-space-4);
}

.task-item {
  display: flex;
  align-items: center;
  gap: var(--gp-space-4);
  padding: var(--gp-space-4);
  background-color: var(--gp-white);
  border: 2px solid var(--gp-border-color);
  border-radius: var(--gp-radius-md);
  transition: all var(--gp-transition-base);
  position: relative;
}

.task-item:hover {
  border-color: var(--gp-primary);
  box-shadow: var(--gp-shadow-md);
}

.task-item.completed {
  border-color: var(--gp-success);
  background-color: rgba(40, 167, 69, 0.05);
}

.task-item.completed::after {
  content: '✓';
  position: absolute;
  top: 8px;
  right: 8px;
  width: 24px;
  height: 24px;
  background-color: var(--gp-success);
  color: var(--gp-white);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 14px;
}

.task-icon {
  width: 56px;
  height: 56px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--mg-gradient-primary);
  color: var(--gp-white);
  border-radius: var(--gp-radius-md);
  font-size: 28px;
}

.task-content {
  flex: 1;
}

.task-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: var(--gp-space-1);
}

.task-desc {
  font-size: 13px;
  color: var(--gp-text-muted);
  margin-bottom: var(--gp-space-2);
}

.task-reward {
  flex-shrink: 0;
}

.task-action {
  flex-shrink: 0;
}

/* 遊戲卡片 */
.mg-featured-games {
  margin-bottom: var(--gp-space-8);
}

.mg-game-card {
  background-color: var(--gp-white);
  border-radius: var(--gp-radius-md);
  overflow: hidden;
  box-shadow: var(--gp-shadow);
  transition: all var(--gp-transition-base);
}

.mg-game-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--gp-shadow-lg);
}

.game-thumbnail {
  position: relative;
  padding-top: 133.33%; /* 3:4 */
  overflow: hidden;
}

.game-thumbnail img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.game-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity var(--gp-transition-base);
}

.mg-game-card:hover .game-overlay {
  opacity: 1;
}

.game-badge {
  position: absolute;
  top: 8px;
  right: 8px;
  padding: 4px 12px;
  background: linear-gradient(135deg, #f7971e 0%, #ffd200 100%);
  color: var(--gp-white);
  border-radius: var(--gp-radius-pill);
  font-size: 12px;
  font-weight: 700;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

.game-info {
  padding: var(--gp-space-4);
}

.game-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: var(--gp-space-2);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.game-meta {
  display: flex;
  gap: var(--gp-space-3);
  font-size: 13px;
  color: var(--gp-text-muted);
}

/* 公告列表 */
.mg-announcements {
  margin-bottom: var(--gp-space-8);
}

.announcement-list {
  background-color: var(--gp-white);
  border: 1px solid var(--gp-border-color);
  border-radius: var(--gp-radius-md);
  overflow: hidden;
  box-shadow: var(--gp-shadow);
}

.announcement-item {
  display: flex;
  align-items: center;
  gap: var(--gp-space-3);
  padding: var(--gp-space-4);
  border-bottom: 1px solid var(--gp-border-light);
  text-decoration: none;
  color: inherit;
  transition: background-color var(--gp-transition-fast);
}

.announcement-item:last-child {
  border-bottom: none;
}

.announcement-item:hover {
  background-color: var(--gp-bg-secondary);
}

.announcement-badge {
  flex-shrink: 0;
  padding: 4px 12px;
  background-color: var(--gp-primary-light);
  color: var(--gp-primary);
  border-radius: var(--gp-radius-pill);
  font-size: 12px;
  font-weight: 600;
}

.announcement-title {
  flex: 1;
  font-size: 14px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.announcement-time {
  flex-shrink: 0;
  font-size: 13px;
  color: var(--gp-text-muted);
}

/* 響應式 */
@media (max-width: 991px) {
  .welcome-content {
    flex-direction: column;
    align-items: flex-start;
    gap: var(--gp-space-4);
  }

  .task-item {
    flex-wrap: wrap;
  }

  .task-action {
    width: 100%;
    margin-top: var(--gp-space-3);
  }

  .task-action .btn {
    width: 100%;
  }
}
```

---

## 4. 我的錢包 (Wallet) 介面

### 4.1 錢包首頁設計

**新建**: `Areas/MiniGame/Views/Wallet/Index.cshtml`

```html
@{
    ViewData["Title"] = "我的錢包";
    Layout = "_MiniGameLayout";
}

@section MiniGameBreadcrumb {
    <li class="breadcrumb-item active">我的錢包</li>
}

<!-- 錢包總覽卡片 -->
<div class="wallet-overview-card">
    <div class="wallet-balance-section">
        <div class="balance-icon">
            <i class="bi bi-wallet2"></i>
        </div>
        <div class="balance-info">
            <p class="balance-label">我的餘額</p>
            <h2 class="balance-amount">@ViewBag.Balance <span class="currency">GP</span></h2>
            <p class="balance-update">更新時間: @DateTime.Now.ToString("yyyy/MM/dd HH:mm")</p>
        </div>
    </div>
    <div class="wallet-actions">
        <button class="btn btn-primary btn-lg" data-bs-toggle="modal" data-bs-target="#depositModal">
            <i class="bi bi-plus-circle"></i> 儲值
        </button>
        <button class="btn btn-outline-primary btn-lg" data-bs-toggle="modal" data-bs-target="#withdrawModal">
            <i class="bi bi-arrow-down-circle"></i> 提領
        </button>
    </div>
</div>

<!-- 快速統計 -->
<div class="row g-4 mb-5">
    <div class="col-md-4">
        <div class="wallet-stat-card">
            <div class="stat-icon" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
                <i class="bi bi-arrow-up-circle"></i>
            </div>
            <div class="stat-content">
                <p class="stat-label">本月儲值</p>
                <p class="stat-value">@ViewBag.MonthlyDeposit <small>GP</small></p>
            </div>
        </div>
    </div>
    <div class="col-md-4">
        <div class="wallet-stat-card">
            <div class="stat-icon" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
                <i class="bi bi-arrow-down-circle"></i>
            </div>
            <div class="stat-content">
                <p class="stat-label">本月消費</p>
                <p class="stat-value">@ViewBag.MonthlySpending <small>GP</small></p>
            </div>
        </div>
    </div>
    <div class="col-md-4">
        <div class="wallet-stat-card">
            <div class="stat-icon" style="background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);">
                <i class="bi bi-gift"></i>
            </div>
            <div class="stat-content">
                <p class="stat-label">累積獎勵</p>
                <p class="stat-value">@ViewBag.TotalRewards <small>GP</small></p>
            </div>
        </div>
    </div>
</div>

<!-- 交易歷史 -->
<div class="transaction-history-section">
    <div class="section-header">
        <h3 class="mg-section-title">
            <i class="bi bi-clock-history"></i>
            交易歷史
        </h3>
        <div class="section-actions">
            <select class="form-select form-select-sm" id="transactionFilter">
                <option value="all">全部交易</option>
                <option value="deposit">儲值</option>
                <option value="withdraw">提領</option>
                <option value="spending">消費</option>
                <option value="reward">獎勵</option>
            </select>
        </div>
    </div>

    <div class="transaction-list">
        @for (int i = 0; i < 10; i++)
        {
            var types = new[] { "deposit", "spending", "reward", "withdraw" };
            var type = types[i % 4];
            var icons = new Dictionary<string, string> {
                { "deposit", "bi-arrow-down-circle" },
                { "withdraw", "bi-arrow-up-circle" },
                { "spending", "bi-cart" },
                { "reward", "bi-gift" }
            };
            var colors = new Dictionary<string, string> {
                { "deposit", "success" },
                { "withdraw", "danger" },
                { "spending", "warning" },
                { "reward", "primary" }
            };
            var labels = new Dictionary<string, string> {
                { "deposit", "儲值" },
                { "withdraw", "提領" },
                { "spending", "消費" },
                { "reward", "獎勵" }
            };

            <div class="transaction-item">
                <div class="transaction-icon bg-@colors[type]">
                    <i class="bi @icons[type]"></i>
                </div>
                <div class="transaction-content">
                    <p class="transaction-title">@labels[type] - @(type == "spending" ? "購買優惠券" : "每日簽到獎勵")</p>
                    <p class="transaction-time">@DateTime.Now.AddHours(-i).ToString("yyyy/MM/dd HH:mm")</p>
                </div>
                <div class="transaction-amount @(type == "deposit" || type == "reward" ? "positive" : "negative")">
                    @(type == "deposit" || type == "reward" ? "+" : "-")@((i + 1) * 10) GP
                </div>
            </div>
        }
    </div>

    <div class="text-center mt-4">
        <a href="/MiniGame/Wallet/Transactions" class="btn btn-outline-primary">
            查看完整記錄
        </a>
    </div>
</div>

<!-- 儲值 Modal -->
<div class="modal fade" id="depositModal" tabindex="-1">
    <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title">
                    <i class="bi bi-plus-circle"></i> 儲值 GP 點數
                </h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
            </div>
            <div class="modal-body">
                <form id="depositForm">
                    <div class="mb-4">
                        <label class="form-label">選擇儲值方案</label>
                        <div class="deposit-plans">
                            <input type="radio" class="btn-check" name="depositPlan" id="plan1" value="100" checked />
                            <label class="btn btn-outline-primary deposit-plan-btn" for="plan1">
                                <div class="plan-amount">100 GP</div>
                                <div class="plan-price">NT$ 100</div>
                            </label>

                            <input type="radio" class="btn-check" name="depositPlan" id="plan2" value="500" />
                            <label class="btn btn-outline-primary deposit-plan-btn" for="plan2">
                                <div class="plan-amount">500 GP</div>
                                <div class="plan-price">NT$ 450</div>
                                <div class="plan-badge">省 10%</div>
                            </label>

                            <input type="radio" class="btn-check" name="depositPlan" id="plan3" value="1000" />
                            <label class="btn btn-outline-primary deposit-plan-btn" for="plan3">
                                <div class="plan-amount">1000 GP</div>
                                <div class="plan-price">NT$ 800</div>
                                <div class="plan-badge">省 20%</div>
                            </label>

                            <input type="radio" class="btn-check" name="depositPlan" id="plan4" value="5000" />
                            <label class="btn btn-outline-primary deposit-plan-btn" for="plan4">
                                <div class="plan-amount">5000 GP</div>
                                <div class="plan-price">NT$ 3500</div>
                                <div class="plan-badge">省 30%</div>
                            </label>
                        </div>
                    </div>

                    <div class="mb-4">
                        <label class="form-label">支付方式</label>
                        <select class="form-select" required>
                            <option value="">請選擇支付方式</option>
                            <option value="credit">信用卡</option>
                            <option value="atm">ATM 轉帳</option>
                            <option value="cvs">超商代碼</option>
                            <option value="line">LINE Pay</option>
                        </select>
                    </div>

                    <div class="alert alert-info">
                        <i class="bi bi-info-circle"></i>
                        儲值後點數將立即到帳，請確認金額無誤後再進行支付
                    </div>
                </form>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">取消</button>
                <button type="submit" form="depositForm" class="btn btn-primary">
                    確認儲值
                </button>
            </div>
        </div>
    </div>
</div>

@section MiniGameScripts {
    <script src="~/js/areas/minigame-wallet.js" asp-append-version="true"></script>
}
```

### 4.2 錢包樣式

**新建**: `wwwroot/css/areas/minigame-wallet.css`

```css
/**
 * Wallet Page Styles
 */

/* 錢包總覽卡片 */
.wallet-overview-card {
  background: var(--mg-gradient-gold);
  color: var(--gp-white);
  padding: var(--gp-space-7);
  border-radius: var(--gp-radius-md);
  box-shadow: var(--gp-shadow-xl);
  margin-bottom: var(--gp-space-6);
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: relative;
  overflow: hidden;
}

.wallet-overview-card::before {
  content: '';
  position: absolute;
  top: -50%;
  right: -10%;
  width: 300px;
  height: 300px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 50%;
}

.wallet-balance-section {
  display: flex;
  align-items: center;
  gap: var(--gp-space-5);
  position: relative;
  z-index: 1;
}

.balance-icon {
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.2);
  border-radius: var(--gp-radius-circle);
  font-size: 40px;
}

.balance-label {
  font-size: 14px;
  opacity: 0.9;
  margin-bottom: var(--gp-space-1);
}

.balance-amount {
  font-size: 48px;
  font-weight: 700;
  margin-bottom: var(--gp-space-1);
  line-height: 1;
}

.balance-amount .currency {
  font-size: 24px;
  font-weight: 600;
  opacity: 0.9;
}

.balance-update {
  font-size: 12px;
  opacity: 0.8;
  margin: 0;
}

.wallet-actions {
  display: flex;
  gap: var(--gp-space-3);
  position: relative;
  z-index: 1;
}

/* 統計卡片 */
.wallet-stat-card {
  background: var(--gp-white);
  border-radius: var(--gp-radius-md);
  padding: var(--gp-space-5);
  box-shadow: var(--gp-shadow);
  display: flex;
  align-items: center;
  gap: var(--gp-space-4);
  transition: all var(--gp-transition-base);
}

.wallet-stat-card:hover {
  transform: translateY(-2px);
  box-shadow: var(--gp-shadow-md);
}

.wallet-stat-card .stat-icon {
  width: 56px;
  height: 56px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--gp-white);
  border-radius: var(--gp-radius-md);
  font-size: 28px;
}

.wallet-stat-card .stat-content {
  flex: 1;
}

.wallet-stat-card .stat-label {
  font-size: 14px;
  color: var(--gp-text-muted);
  margin-bottom: var(--gp-space-1);
}

.wallet-stat-card .stat-value {
  font-size: 24px;
  font-weight: 700;
  color: var(--gp-text-primary);
  margin: 0;
}

.wallet-stat-card .stat-value small {
  font-size: 14px;
  font-weight: 500;
  color: var(--gp-text-muted);
}

/* 交易歷史 */
.transaction-history-section {
  background: var(--gp-white);
  border-radius: var(--gp-radius-md);
  padding: var(--gp-space-5);
  box-shadow: var(--gp-shadow);
}

.transaction-list {
  display: flex;
  flex-direction: column;
  gap: var(--gp-space-3);
}

.transaction-item {
  display: flex;
  align-items: center;
  gap: var(--gp-space-4);
  padding: var(--gp-space-4);
  background: var(--gp-bg-secondary);
  border-radius: var(--gp-radius);
  transition: all var(--gp-transition-fast);
}

.transaction-item:hover {
  background: var(--gp-bg-tertiary);
}

.transaction-icon {
  width: 48px;
  height: 48px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--gp-white);
  border-radius: var(--gp-radius-circle);
  font-size: 20px;
}

.transaction-icon.bg-success {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.transaction-icon.bg-danger {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.transaction-icon.bg-warning {
  background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);
}

.transaction-icon.bg-primary {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
}

.transaction-content {
  flex: 1;
}

.transaction-title {
  font-size: 14px;
  font-weight: 600;
  margin-bottom: 4px;
}

.transaction-time {
  font-size: 12px;
  color: var(--gp-text-muted);
  margin: 0;
}

.transaction-amount {
  font-size: 18px;
  font-weight: 700;
}

.transaction-amount.positive {
  color: var(--gp-success);
}

.transaction-amount.negative {
  color: var(--gp-danger);
}

/* 儲值方案 */
.deposit-plans {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: var(--gp-space-3);
}

.deposit-plan-btn {
  padding: var(--gp-space-4);
  text-align: center;
  position: relative;
  height: 100%;
}

.deposit-plan-btn .plan-amount {
  font-size: 20px;
  font-weight: 700;
  margin-bottom: var(--gp-space-2);
}

.deposit-plan-btn .plan-price {
  font-size: 16px;
  color: var(--gp-text-muted);
}

.deposit-plan-btn .plan-badge {
  position: absolute;
  top: -8px;
  right: -8px;
  padding: 2px 8px;
  background: var(--gp-danger);
  color: var(--gp-white);
  border-radius: var(--gp-radius-pill);
  font-size: 11px;
  font-weight: 700;
}

.btn-check:checked + .deposit-plan-btn {
  background: var(--mg-gradient-gold);
  color: var(--gp-white);
  border-color: transparent;
}

.btn-check:checked + .deposit-plan-btn .plan-price {
  color: rgba(255, 255, 255, 0.9);
}

/* 響應式 */
@media (max-width: 991px) {
  .wallet-overview-card {
    flex-direction: column;
    align-items: flex-start;
    gap: var(--gp-space-5);
  }

  .balance-amount {
    font-size: 36px;
  }

  .wallet-actions {
    width: 100%;
  }

  .wallet-actions .btn {
    flex: 1;
  }
}
```

---

由於篇幅和token限制，我將概述剩餘章節內容：

## 5-20 章節概要

**5. 簽到系統**: 月曆UI、連續簽到進度、補簽卡、獎勵預覽

**6. 寵物系統**: 寵物卡片展示、餵食/培養介面、進化動畫、寵物圖鑑

**7. 遊戲大廳**: 遊戲分類、搜尋篩選、遊戲卡片、立即遊玩按鈕、收藏功能

**8. 優惠券系統**: 卡券視覺設計、使用限制顯示、有效期倒數、快速使用

**9. 電子票券**: 票券卡片、QR Code 展示、核銷狀態、分享功能

**10. 序號兌換**: 輸入介面設計、格式驗證、獎勵預覽 Modal、兌換成功動畫

**11. 獎勵中心**: 待領取獎勵、一鍵領取、獎勵歷史、過期提醒

**12. 交易歷史**: 篩選與搜尋、分頁載入、詳細資訊 Modal

**13. 排行榜**: 全域/遊戲排行、好友排行、時間範圍篩選、我的排名高亮

**14. 成就系統**: 成就卡片、解鎖動畫、進度追蹤、稀有度分級

**15. 遊戲化元素**: 等級系統、經驗值、每日任務、限時活動

**16. 即時反饋**: Toast 通知、Success 動畫、Confetti 效果、聲音反饋

**17. 響應式遊戲體驗**: 橫屏遊戲支援、觸控優化、手勢操作

**18. 效能與載入**: Skeleton Loading、圖片懶載入、分頁與虛擬滾動

**19. 實作路徑**: Phase 1-4 詳細規劃 (20 工作天)

**20. 驗收標準**: 功能完整性、視覺一致性、效能指標、用戶體驗評分

---

**文件版本**: v1.0
**最後更新**: 2025-10-29
**維護者**: Claude Code Analysis Team
**狀態**: MiniGame Area 專項建議書 (篇幅限制，詳細實作可分階段展開)
