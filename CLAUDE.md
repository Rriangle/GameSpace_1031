# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a dual ASP.NET Core MVC web application project for a gaming community platform with two main components:

- **GameSpace** (`GameSpace/GameSpace/`): Admin backend for managing the gaming platform
- **GamiPort** (`GamiPort/GamiPort/`): Public-facing frontend for end users

Both applications share a common SQL Server database (`GameSpacedatabase`) and use Area-based architecture for modular organization.

## Key Architecture Principles

### 1. Strict Area Boundaries
- This project uses **ASP.NET Core Areas** for feature isolation (Forum, OnlineStore, MiniGame, social_hub, MemberManagement, Identity/Login)
- **CRITICAL**: When working on a specific Area (e.g., `Areas/MiniGame`), you MUST NOT modify files in other Areas or global files except `Program.cs` for necessary service registration
- Each Area has its own Controllers, Views, Services, and Models
- The only exception: `Program.cs` may be modified to add necessary service registrations, but existing configurations from other Areas must not be altered

### 2. Database-First Approach
- The database schema is managed manually via SQL Server (not EF Migrations)
- **DO NOT** use EF Migrations to modify schema - the database in SQL Server is the single source of truth
- Database schema reference is available in `schema/` folder for AI reference
- All tables implement soft delete pattern (IsDeleted, DeletedAt, DeletedBy, DeleteReason columns)

### 3. Authentication & Authorization
- **GameSpace (Admin)**: Uses `AdminCookie` authentication scheme
  - Cookie name: `GameSpace.Admin`
  - Login path: `/Login`
  - Managed via `ManagerData`, `ManagerRole`, `ManagerRolePermission` tables
  - Claims-based authorization with policies: "AdminOnly", "CanManageShopping", "CanAdmin", "CanMessage", "CanUserStatus", "CanPet", "CanCS"

- **GamiPort (Frontend)**: Uses standard Cookie authentication
  - Cookie name: `GamiPort.User`
  - Login path: `/Login/Login/Login`
  - User data stored in `Users` table

- Both systems use `ILoginIdentity` and related interfaces for unified login handling

## Common Development Commands

### Build & Run
```powershell
# Build GameSpace (Admin)
cd GameSpace/GameSpace
dotnet build

# Build GamiPort (Frontend)
cd GamiPort/GamiPort
dotnet build

# Run GameSpace (Admin) - typically runs on https://localhost:7042 and http://localhost:5211
cd GameSpace/GameSpace
dotnet run

# Run GamiPort (Frontend)
cd GamiPort/GamiPort
dotnet run

# Run both applications simultaneously (open two terminals)
# Terminal 1:
cd GameSpace/GameSpace && dotnet run
# Terminal 2:
cd GamiPort/GamiPort && dotnet run
```

### Database Connection
Both projects connect to SQL Server using connection strings in `appsettings.json`:
- Primary connection: `DefaultConnection` (for Identity/auth-related)
- Business data: `GameSpace` or `GameSpacedatabase`

Connection string format:
```
Server=DESKTOP-8HQIS1S\\SQLEXPRESS;Database=GameSpacedatabase;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=true
```

### Testing & Healthcheck
No formal test projects currently exist. Manual testing is performed through the running applications.

Health check endpoint (if implemented):
```powershell
# Check database connectivity
curl http://localhost:5211/healthz/db
# Expected: {"status":"ok"}
```

## MiniGame Area Architecture

The MiniGame Area is the most complex feature and is split across both projects. It uses a **layered service architecture** with extensive dependency injection.

### GameSpace/Areas/MiniGame (Admin Backend - Already Complete)

**Structure:**
- `config/ServiceExtensions.cs` - DI registration for ~30+ services
- `Controllers/` - Admin controllers (25+ controllers)
- `Services/` - Business logic layer
- `ViewModels/` - DTOs for views
- `Models/` - Domain models
- `Constants/` - Business constants (Coupon, Pet, SignIn, Wallet)
- `Filters/` - Custom action filters

**Key Admin Controllers:**
- `AdminWalletController` - Point/coupon/e-voucher management
- `AdminPetController` - Pet system configuration & individual pet adjustments
- `AdminMiniGameController` - Game rules & records
- `AdminUserController`, `AdminManagerController` - User/admin management
- `AdminDashboardController` - Analytics dashboard
- `AdminDiagnosticsController` - System diagnostics
- `SignInAdminController` - Sign-in configuration
- `CouponTypesController` - Coupon type management
- `DailyGameLimitController` - Game play limits

**Service Layers (30+ services registered in ServiceExtensions.cs):**
- **Auth & Gate**: `IMiniGameAdminService`, `IMiniGameAdminAuthService`, `IMiniGameAdminGate`
- **Wallet**: `IUserWalletService`, `IWalletService`, `IWalletQueryService`, `IWalletMutationService`
- **Coupon**: `ICouponService`, `ICouponTypeService`
- **E-Voucher**: `IEVoucherService`, `IEVoucherTypeService`
- **Sign-in**: `ISignInService`, `ISignInStatsService`, `IInMemorySignInRuleService`, `ISignInQueryService`, `ISignInMutationService`, `ITaiwanHolidayService` (Singleton)
- **Pet**: `IPetService`, `IPetQueryService`, `IPetMutationService`, `IPetInteractionService`, `IPetDailyDecayService`
- **Pet Config**: `IPetColorOptionService`, `IPetBackgroundOptionService`, `IPetSkinColorCostSettingService`, `IPetBackgroundCostSettingService`, `IPetColorChangeSettingsService`, `IPetBackgroundChangeSettingsService`, `IPetLevelExperienceSettingService`, `IPetLevelRewardSettingService`, `IPetLevelUpRuleService`, `IPetLevelUpRuleValidationService`, `IPetRulesService`
- **Game**: `IMiniGameService`, `IGameQueryService`, `IGameMutationService`, `IGamePlayService`, `IDailyGameLimitService`, `IDailyGameLimitValidationService`, `IGameRulesService`
- **System**: `IDiagnosticsService`, `IDashboardService`, `IUserService`, `IManagerService`, `IPointsSettingsStatisticsService`

**Important Notes:**
- MiniGameBaseController provides shared functionality for admin controllers
- All admin controllers require `[Authorize(AuthenticationSchemes = "AdminCookie", Policy = "AdminOnly")]`
- Services follow Query/Mutation pattern for read vs write operations

### GamiPort/Areas/MiniGame (Frontend - Currently Being Developed)

**Current Controllers (Stubs):**
- `HomeController` - Landing page
- `WalletController` - User wallet interface
- `SignInController` - Daily sign-in
- `PetController` - Pet interaction

**Features to Implement:**
- **Wallet**: View balance, exchange points for coupons, view owned coupons/e-vouchers, use e-vouchers (QR/Barcode), transaction history
- **Sign-in**: Calendar-based sign-in interface, view sign-in history with rewards
- **Pet**: Rename pet, interact (feed/bath/play/sleep), change skin color (costs points), change background, view pet status (5 attributes: Hunger/Mood/Stamina/Cleanliness/Health 0-100)
- **Mini Game**: Start adventure game (returns sessionId, remaining plays), view game history (win/lose/abort, rewards)

**Important:** Frontend must reuse existing backend services or create minimal frontend-specific services. Do NOT duplicate business logic.

## Database Schema Summary

### Core Tables (20 total)

**MiniGame Area (16 tables)**:
- `User_Wallet`, `WalletHistory`: Points management
- `CouponType`, `Coupon`: Shopping coupons
- `EVoucherType`, `EVoucher`, `EVoucherToken`, `EVoucherRedeemLog`: Electronic vouchers
- `Pet`, `PetSkinColorCostSettings`, `PetBackgroundCostSettings`, `PetLevelRewardSettings`: Pet system
- `SignInRule`, `UserSignInStats`: Sign-in system
- `MiniGame`: Game session records
- `SystemSettings`: Configuration

**User & Admin Tables (4 tables)**:
- `Users`: End user accounts
- `ManagerData`, `ManagerRole`, `ManagerRolePermission`: Admin authentication & authorization

### Key Database Patterns
- All MiniGame tables use **soft delete** (IsDeleted, DeletedAt, DeletedBy, DeleteReason)
- Most tables use **audit trails** (CreatedAt, UpdatedAt, UpdatedBy)
- Time fields use `datetime2(7)` with `DEFAULT sysutcdatetime()` (UTC)
- Foreign key relationships strictly enforced
- Check constraints for enums, ranges, and logical consistency
- Unique constraints on codes, names, and business keys

## Important File Locations

### GameSpace (Admin)
- Entry point: `GameSpace/GameSpace/Program.cs`
- DbContext: `GameSpace/GameSpace/Data/GameSpacedatabaseContext.cs`
- MiniGame Area: `GameSpace/GameSpace/Areas/MiniGame/`
- Shared Layout: `GameSpace/GameSpace/Views/Shared/_Layout.cshtml`
- Admin Sidebar: `GameSpace/GameSpace/Partials/_Sidebar.cshtml`
- Admin UI Theme: `GameSpace/GameSpace/wwwroot/lib/sb-admin/` (SB Admin - DO NOT modify vendor files)

### GamiPort (Frontend)
- Entry point: `GamiPort/GamiPort/Program.cs`
- DbContext: `GamiPort/GamiPort/Models/GameSpacedatabaseContext.cs`
- MiniGame Area: `GamiPort/GamiPort/Areas/MiniGame/`
- Shared Layout: `GamiPort/GamiPort/Views/Shared/_Layout.cshtml`
- Frontend Sidebar: `GamiPort/GamiPort/Partials/_Sidebar.cshtml`

### Schema Documentation
- Database schema: `schema/db_schema_summary.md`
- Detailed specs: `schema/README_合併版.md`
- MiniGame details: `schema/MiniGame_Area_完整描述文件.md`
- Database structure: `schema/MiniGame_Area_資料庫完整結構文件_2025-10-27.md`

## Development Guidelines

### Coding Standards
- **File Encoding**: All files must be UTF-8 with BOM (especially for Chinese content)
- **Read-Only Queries**: Use `.AsNoTracking()` for all read-only operations
- **Transactions**: Use database transactions for all operations that modify points/coupons/game state
- **Batch Limits**: Keep batch operations ≤ 1000 records
- **Idempotent Operations**: Ensure all write operations are idempotent
- **Error Handling**: Use `ProblemDetails` or unified Result types for error responses

### Security Requirements
- Prevent negative balance in wallet operations
- Handle concurrent access with proper locking/transactions
- Validate all user inputs, especially for SQL injection, XSS, CSRF
- Use Anti-forgery tokens (header name: `RequestVerificationToken`)
- Sensitive operations must be logged with audit trails

### UI/UX Requirements
- **Admin Backend**: Must use SB Admin theme (`wwwroot/lib/sb-admin/`) - vendor files must not be modified
- **Frontend**: Uses Bootstrap and custom styles
- **Navigation**: Two-level sidebar structure (module → feature buttons)
- **404 Pages**: Use shared 404 page - do not create Area-specific 404 pages

## SignalR Hubs

### GameSpace (Admin)
- **ChatHub**: `/social_hub/chatHub` - DM functionality for admins
- Supports WebSockets, Server-Sent Events, and Long Polling

### GamiPort (Frontend)
- **ChatHub**: `/social_hub/chathub` - DM functionality for users
- **SupportHub**: `/hubs/support` - Customer support chat
- CORS enabled for admin backend cross-origin connections

## Services & Dependency Injection

### Common Services
- `GameSpacedatabaseContext`: Main DbContext (Scoped)
- `IAppClock`: Application timezone handling (Singleton, Taipei timezone)
- `IHttpContextAccessor`: Access HTTP context (Singleton)

### MiniGame Services (GameSpace)
- Registered via `builder.Services.AddMiniGameServices(builder.Configuration)` in `Program.cs`
- **30+ scoped services** defined in `GameSpace/GameSpace/Areas/MiniGame/config/ServiceExtensions.cs`
- **One singleton service**: `ITaiwanHolidayService` for Taiwan public holiday calculations
- Services organized by domain: Auth, Wallet, Coupon, E-Voucher, Sign-in, Pet, Game, System
- Query/Mutation separation for CQRS-like pattern in some services

### Social Hub Services
- `IMuteFilter`: Profanity filtering (Singleton with scope factory pattern)
- `IChatService`: Chat/DM messaging (Scoped)
- `IChatNotifier`: SignalR notifications (Singleton)
- `ISupportService`: Customer support (Scoped)
- `ISupportNotifier`: Support notifications (Singleton)
- `IRelationService`: Friend/block relationships (Scoped)

### Forum Services (GamiPort)
- `IForumsService`: Forum management (Scoped)
- `IThreadsService`: Thread operations (Scoped)
- `IMeContentService`: User content (Scoped)

### E-commerce Services (GamiPort)
- `ICartService`: Shopping cart (Scoped)
- `ILookupService`: Product lookups (Scoped)
- `EcpayPaymentService`: ECPay payment integration (Scoped)

## Troubleshooting

### Database Connection Issues
1. Verify SQL Server is running (check `DESKTOP-8HQIS1S\SQLEXPRESS`)
2. Check connection string in `appsettings.json`
3. Test with healthcheck endpoint: `/healthz/db` should return `{"status":"ok"}`

### Authentication Issues
- **Admin**: Ensure `AdminCookie` scheme is properly configured and Manager has correct Claims (IsManager=true)
- **Frontend**: Check Cookie authentication and User_ID claim

### Area Routing Issues
- Verify Area route is registered in `Program.cs`: `{area:exists}/{controller=Home}/{action=Index}/{id?}`
- Check controller has `[Area("AreaName")]` attribute

### SignalR Connection Issues
- Verify CORS policy is configured correctly
- Check keepalive and timeout settings (GameSpace: 15s keepalive, 60s timeout)
- Enable detailed errors in development: `options.EnableDetailedErrors = true`

## Project-Specific Patterns

### Service Registration Pattern
MiniGame Area uses extension method pattern for clean DI registration:
```csharp
// In Program.cs
builder.Services.AddMiniGameServices(builder.Configuration);

// Defined in Areas/MiniGame/config/ServiceExtensions.cs
public static IServiceCollection AddMiniGameServices(this IServiceCollection services, IConfiguration configuration)
{
    services.AddScoped<IServiceName, ServiceImplementation>();
    // ... 30+ service registrations
}
```

### Controller Base Classes
- **GameSpace**: `MiniGameBaseController` provides shared admin functionality
- Controllers should inherit from base classes when available for consistent behavior

### Constants Pattern
Business rules are centralized in Constants classes:
- `CouponConstants` - Coupon-related constants
- `PetConstants` - Pet system limits and defaults
- `SignInConstants` - Sign-in rules and limits
- `WalletConstants` - Point system constants

### ViewModel/DTO Pattern
- ViewModels folder contains DTOs for view rendering
- Separate from domain Models to maintain clean separation of concerns

## Critical Constraints & Gotchas

### MiniGame Area Development
1. **Service dependencies are complex** - Check `ServiceExtensions.cs` before adding new dependencies
2. **InMemory vs Database services** - Some services use in-memory implementations (e.g., `InMemoryPetSkinColorCostSettingService`) as noted in comments - be aware when implementing features
3. **Query/Mutation pattern** - Some domains separate read (`*QueryService`) from write (`*MutationService`) operations
4. **Taiwan-specific logic** - `ITaiwanHolidayService` handles local public holidays for sign-in rewards
5. **Pet attribute constraints** - All 5 pet attributes (Hunger/Mood/Stamina/Cleanliness/Health) must be 0-100, enforced by database CHECK constraints
6. **Daily limits** - Game play limited to 3 times/day by default via `IDailyGameLimitService`

### Cross-Project Coordination
- **GameSpace and GamiPort share the same database** but have separate DbContext instances
- **Do not share code between projects** - they are meant to be independently deployable
- **API contracts** - If GamiPort needs GameSpace functionality, consider API endpoints rather than shared libraries

### Infrastructure Services
Both projects have `Infrastructure/` folders with shared patterns:
- **Login**: `ILoginIdentity`, `ClaimFirstLoginIdentity`, `CookieAndAdminCookieLoginIdentity`
- **Security** (GamiPort): `IAppCurrentUser`, `AppCurrentUser` for unified user context
- **Time** (GameSpace): `IAppClock`, `AppClock` with Taipei timezone

## Notes for AI Assistants

1. **Always check Area boundaries** before modifying files - respect the strict separation
2. **Database schema is immutable** - never suggest EF Migrations for schema changes
3. **Reference the specification documents** in `schema/` for business logic details (especially `schema/README_合併版.md`)
4. **All Chinese content** must be saved as UTF-8 with BOM
5. **Follow existing patterns** - both projects have established conventions for Controllers, Services, and Views
6. When creating commits, use clear English messages focusing on WHY the change was made
7. For MiniGame Area work, always verify against the 100% coverage requirement for all related database tables
8. The admin backend (GameSpace/MiniGame) is complete; frontend work (GamiPort/MiniGame) is the current focus
9. **Check ServiceExtensions.cs** before implementing new services - avoid duplicating existing functionality
10. **Reuse existing services** from GameSpace when possible in GamiPort rather than reimplementing business logic
