# Bitcoin Hot-Wallet Deployment Plan
## Custodial Wallet System using Blazor and Google Cloud Platform

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Project Overview](#project-overview)
3. [Architecture Design](#architecture-design)
4. [Technology Stack](#technology-stack)
5. [Phase 1: Environment Setup](#phase-1-environment-setup)
6. [Phase 2: Backend API Development](#phase-2-backend-api-development)
7. [Phase 3: Blazor Frontend Development](#phase-3-blazor-frontend-development)
8. [Phase 4: Google Cloud Platform Infrastructure](#phase-4-google-cloud-platform-infrastructure)
9. [Phase 5: Security Implementation](#phase-5-security-implementation)
10. [Phase 6: Testing Strategy](#phase-6-testing-strategy)
11. [Phase 7: Deployment](#phase-7-deployment)
12. [Phase 8: Monitoring & Maintenance](#phase-8-monitoring--maintenance)
13. [Success Metrics](#success-metrics)
14. [Risk Management](#risk-management)
15. [Timeline & Resources](#timeline--resources)

---

## Executive Summary

This deployment plan outlines the complete process for building and deploying a production-ready custodial Bitcoin hot-wallet system. The application will be built using **Blazor Server** for the frontend and **ASP.NET Core Web API** for the backend, hosted on **Google Cloud Platform (GCP)**. This plan leverages all the guidance documents in this repository to showcase comprehensive backend engineering competency.

### Key Objectives
- Build a secure, scalable custodial Bitcoin wallet system
- Demonstrate proficiency in API development, Bitcoin integration, and cloud deployment
- Implement industry best practices for security, testing, and performance
- Deploy on Google Cloud Platform with production-grade infrastructure
- Create a maintainable system with comprehensive monitoring

---

## Project Overview

### What is a Custodial Hot-Wallet?
A **custodial hot-wallet** is a Bitcoin wallet system where:
- **Custodial**: The service provider (not the end user) controls the private keys
- **Hot-wallet**: Private keys are stored online for quick transaction processing
- **Use Case**: Exchanges, payment processors, and services that need instant Bitcoin transactions

### Project Scope
This project will deliver:
1. **User Management**: Registration, authentication, and authorization
2. **Wallet Management**: Create and manage Bitcoin wallets for users
3. **Transaction Processing**: Send and receive Bitcoin transactions
4. **Balance Tracking**: Real-time balance updates and transaction history
5. **Admin Dashboard**: System monitoring and management interface
6. **API Integration**: RESTful APIs for all operations
7. **Security Features**: Encryption, rate limiting, audit logging
8. **Cloud Infrastructure**: Scalable GCP deployment

---

## Architecture Design

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Google Cloud Platform                     │
│                                                              │
│  ┌─────────────────────┐        ┌────────────────────────┐ │
│  │   Cloud Load        │        │    Cloud Armor         │ │
│  │   Balancer          │───────▶│    (DDoS Protection)   │ │
│  └──────────┬──────────┘        └────────────────────────┘ │
│             │                                                │
│  ┌──────────▼───────────────────────────────────────────┐  │
│  │              Blazor Server Application                │  │
│  │          (App Engine or Cloud Run)                    │  │
│  │  ┌──────────────────────────────────────────────┐   │  │
│  │  │  - User Interface                            │   │  │
│  │  │  - Real-time SignalR Updates                 │   │  │
│  │  │  - Authentication UI                         │   │  │
│  │  └──────────────────────────────────────────────┘   │  │
│  └──────────┬───────────────────────────────────────────┘  │
│             │                                                │
│  ┌──────────▼───────────────────────────────────────────┐  │
│  │           ASP.NET Core Web API                        │  │
│  │          (Cloud Run Containers)                       │  │
│  │  ┌──────────────────────────────────────────────┐   │  │
│  │  │  Controllers:                                 │   │  │
│  │  │  - WalletsController                          │   │  │
│  │  │  - TransactionsController                     │   │  │
│  │  │  - UsersController                            │   │  │
│  │  │  - AdminController                            │   │  │
│  │  └──────────────────────────────────────────────┘   │  │
│  │  ┌──────────────────────────────────────────────┐   │  │
│  │  │  Services:                                    │   │  │
│  │  │  - BitcoinWalletService                       │   │  │
│  │  │  - TransactionService                         │   │  │
│  │  │  - UserService                                │   │  │
│  │  │  - SecurityService                            │   │  │
│  │  └──────────────────────────────────────────────┘   │  │
│  └──────────┬───────────────────────────────────────────┘  │
│             │                                                │
│  ┌──────────▼───────────────────────────────────────────┐  │
│  │           Cloud SQL (PostgreSQL)                      │  │
│  │  - User accounts                                      │  │
│  │  - Encrypted wallet data                              │  │
│  │  - Transaction records                                │  │
│  │  - Audit logs                                         │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           Cloud Memorystore (Redis)                   │  │
│  │  - Session storage                                    │  │
│  │  - Rate limiting data                                 │  │
│  │  - Cached Bitcoin prices                             │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           Secret Manager                              │  │
│  │  - Database credentials                               │  │
│  │  - API keys                                           │  │
│  │  - Encryption keys                                    │  │
│  │  - Bitcoin node credentials                           │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │           Cloud Monitoring & Logging                  │  │
│  │  - Application logs                                   │  │
│  │  - Performance metrics                                │  │
│  │  - Security alerts                                    │  │
│  │  - Audit trail                                        │  │
│  └───────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
                           │
                           │ External Connection
                           │
              ┌────────────▼──────────────┐
              │   Bitcoin Network Node    │
              │   (Self-hosted or         │
              │    3rd-party service)     │
              └───────────────────────────┘
```

### Component Responsibilities

#### Blazor Server Frontend
- User authentication and authorization
- Dashboard for wallet management
- Transaction history viewing
- Send/receive Bitcoin interface
- Real-time balance updates via SignalR
- Admin panel for system management

#### ASP.NET Core API Backend
- RESTful API endpoints
- Business logic implementation
- Bitcoin wallet operations using NBitcoin
- Transaction processing and validation
- Integration with Bitcoin network
- Security and encryption services

#### Database Layer (Cloud SQL)
- User account management
- Wallet metadata storage
- Transaction records
- Audit logging
- Encrypted private key storage

#### Caching Layer (Redis)
- Session management
- Rate limiting counters
- Frequently accessed data (Bitcoin prices, network stats)
- Temporary transaction data

---

## Technology Stack

### Frontend
- **Framework**: Blazor Server (.NET 8)
- **UI Components**: MudBlazor or Radzen Blazor Components
- **Real-time Communication**: SignalR
- **State Management**: Fluxor (Flux/Redux pattern for Blazor)

### Backend
- **Framework**: ASP.NET Core 8 Web API
- **Bitcoin Library**: NBitcoin
- **ORM**: Entity Framework Core
- **Authentication**: ASP.NET Core Identity + JWT
- **API Documentation**: Swagger/OpenAPI

### Database & Storage
- **Primary Database**: Cloud SQL (PostgreSQL)
- **Cache**: Cloud Memorystore (Redis)
- **Secret Storage**: Google Secret Manager
- **File Storage**: Cloud Storage (for logs/backups)

### Testing
- **Unit Testing**: xUnit
- **Mocking**: Moq
- **Integration Testing**: WebApplicationFactory
- **Load Testing**: k6 or Apache JMeter

### Infrastructure (GCP)
- **Compute**: Cloud Run (containerized apps)
- **Load Balancing**: Cloud Load Balancer
- **Security**: Cloud Armor, Identity-Aware Proxy
- **Networking**: VPC, Cloud NAT
- **Monitoring**: Cloud Monitoring, Cloud Logging
- **CI/CD**: Cloud Build, Artifact Registry

### DevOps
- **Containerization**: Docker
- **Orchestration**: Cloud Run (serverless containers)
- **Source Control**: GitHub
- **CI/CD**: GitHub Actions + Google Cloud Build

---

## Phase 1: Environment Setup

### Step 1.1: Development Environment
**Duration**: 1-2 days

**Tasks**:
1. Install required software:
   ```bash
   # .NET 8 SDK
   wget https://dot.net/v1/dotnet-install.sh
   bash dotnet-install.sh --channel 8.0
   
   # Docker Desktop
   # Download from docker.com
   
   # Google Cloud SDK
   curl https://sdk.cloud.google.com | bash
   exec -l $SHELL
   gcloud init
   ```

2. Set up IDE:
   - Visual Studio 2022 or VS Code with C# Dev Kit
   - Install extensions: C#, Docker, Google Cloud Code

3. Create project structure:
   ```bash
   mkdir BitcoinHotWallet
   cd BitcoinHotWallet
   
   # Create solution
   dotnet new sln -n BitcoinHotWallet
   
   # Create projects
   dotnet new webapi -n BitcoinHotWallet.Api
   dotnet new blazorserver -n BitcoinHotWallet.Web
   dotnet new classlib -n BitcoinHotWallet.Core
   dotnet new classlib -n BitcoinHotWallet.Infrastructure
   dotnet new xunit -n BitcoinHotWallet.Tests
   
   # Add projects to solution
   dotnet sln add **/*.csproj
   ```

4. Install NuGet packages:
   ```bash
   # API project
   cd BitcoinHotWallet.Api
   dotnet add package NBitcoin
   dotnet add package NBitcoin.RPC
   dotnet add package Microsoft.EntityFrameworkCore
   dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
   dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
   dotnet add package Swashbuckle.AspNetCore
   dotnet add package StackExchange.Redis
   dotnet add package Serilog.AspNetCore
   
   # Core library
   cd ../BitcoinHotWallet.Core
   dotnet add package NBitcoin
   
   # Infrastructure library
   cd ../BitcoinHotWallet.Infrastructure
   dotnet add package Microsoft.EntityFrameworkCore
   dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
   
   # Web project
   cd ../BitcoinHotWallet.Web
   dotnet add package MudBlazor
   
   # Test project
   cd ../BitcoinHotWallet.Tests
   dotnet add package Moq
   dotnet add package FluentAssertions
   dotnet add package Microsoft.AspNetCore.Mvc.Testing
   ```

**Reference**: See [API_DEVELOPMENT_GUIDE.md](API_DEVELOPMENT_GUIDE.md) for project structure patterns

### Step 1.2: Google Cloud Project Setup
**Duration**: 1 day

**Tasks**:
1. Create GCP project:
   ```bash
   gcloud projects create bitcoin-hotwallet-prod --name="Bitcoin Hot Wallet"
   gcloud config set project bitcoin-hotwallet-prod
   
   # Enable billing
   gcloud beta billing accounts list
   gcloud beta billing projects link bitcoin-hotwallet-prod \
     --billing-account=YOUR_BILLING_ACCOUNT_ID
   ```

2. Enable required APIs:
   ```bash
   gcloud services enable \
     run.googleapis.com \
     sqladmin.googleapis.com \
     redis.googleapis.com \
     secretmanager.googleapis.com \
     cloudresourcemanager.googleapis.com \
     compute.googleapis.com \
     vpcaccess.googleapis.com \
     cloudbuild.googleapis.com \
     artifactregistry.googleapis.com \
     monitoring.googleapis.com \
     logging.googleapis.com
   ```

3. Set up service accounts:
   ```bash
   # Create service account for the application
   gcloud iam service-accounts create bitcoin-wallet-app \
     --display-name="Bitcoin Wallet Application"
   
   # Grant necessary permissions
   gcloud projects add-iam-policy-binding bitcoin-hotwallet-prod \
     --member="serviceAccount:bitcoin-wallet-app@bitcoin-hotwallet-prod.iam.gserviceaccount.com" \
     --role="roles/cloudsql.client"
   
   gcloud projects add-iam-policy-binding bitcoin-hotwallet-prod \
     --member="serviceAccount:bitcoin-wallet-app@bitcoin-hotwallet-prod.iam.gserviceaccount.com" \
     --role="roles/secretmanager.secretAccessor"
   ```

### Step 1.3: Bitcoin Development Environment
**Duration**: 1 day

**Tasks**:
1. Set up Bitcoin testnet node (Option A - Local):
   ```bash
   # Download Bitcoin Core
   wget https://bitcoin.org/bin/bitcoin-core-26.0/bitcoin-26.0-x86_64-linux-gnu.tar.gz
   tar xzf bitcoin-26.0-x86_64-linux-gnu.tar.gz
   
   # Create bitcoin.conf for testnet
   mkdir -p ~/.bitcoin
   cat > ~/.bitcoin/bitcoin.conf << EOF
   testnet=1
   server=1
   rpcuser=bitcoinrpc
   rpcpassword=your_secure_password
   rpcallowip=127.0.0.1
   EOF
   
   # Start Bitcoin daemon
   ./bitcoin-26.0/bin/bitcoind -daemon
   ```

2. Or use a Bitcoin testnet API service (Option B - Recommended for development):
   - BlockCypher API: https://www.blockcypher.com/
   - Blockchain.com API: https://www.blockchain.com/api
   - BTCPay Server: Self-hosted Bitcoin payment processor

3. Get testnet Bitcoin:
   - Faucet: https://testnet-faucet.mempool.co/
   - Faucet: https://bitcoinfaucet.uo1.net/

**Reference**: See [BITCOIN_INTEGRATION_GUIDE.md](BITCOIN_INTEGRATION_GUIDE.md) for detailed Bitcoin setup

---

## Phase 2: Backend API Development

### Step 2.1: Database Schema Design
**Duration**: 2-3 days

**Tasks**:
1. Design database schema:
   ```csharp
   // BitcoinHotWallet.Core/Entities/User.cs
   public class User
   {
       public Guid Id { get; set; }
       public string Email { get; set; }
       public string PasswordHash { get; set; }
       public string FirstName { get; set; }
       public string LastName { get; set; }
       public bool EmailVerified { get; set; }
       public bool TwoFactorEnabled { get; set; }
       public DateTime CreatedAt { get; set; }
       public DateTime? LastLoginAt { get; set; }
       public bool IsActive { get; set; }
       
       // Navigation
       public ICollection<Wallet> Wallets { get; set; }
   }
   
   // BitcoinHotWallet.Core/Entities/Wallet.cs
   public class Wallet
   {
       public Guid Id { get; set; }
       public Guid UserId { get; set; }
       public string Address { get; set; }
       public string EncryptedPrivateKey { get; set; }
       public string PublicKey { get; set; }
       public string Label { get; set; }
       public decimal Balance { get; set; }
       public DateTime CreatedAt { get; set; }
       public bool IsActive { get; set; }
       
       // Navigation
       public User User { get; set; }
       public ICollection<Transaction> Transactions { get; set; }
   }
   
   // BitcoinHotWallet.Core/Entities/Transaction.cs
   public class Transaction
   {
       public Guid Id { get; set; }
       public Guid WalletId { get; set; }
       public string TransactionHash { get; set; }
       public string FromAddress { get; set; }
       public string ToAddress { get; set; }
       public decimal Amount { get; set; }
       public decimal Fee { get; set; }
       public TransactionType Type { get; set; } // Send/Receive
       public TransactionStatus Status { get; set; } // Pending/Confirmed/Failed
       public int Confirmations { get; set; }
       public DateTime CreatedAt { get; set; }
       public DateTime? ConfirmedAt { get; set; }
       public string Note { get; set; }
       
       // Navigation
       public Wallet Wallet { get; set; }
   }
   
   // BitcoinHotWallet.Core/Entities/AuditLog.cs
   public class AuditLog
   {
       public Guid Id { get; set; }
       public Guid? UserId { get; set; }
       public string Action { get; set; }
       public string Entity { get; set; }
       public string EntityId { get; set; }
       public string OldValue { get; set; }
       public string NewValue { get; set; }
       public string IpAddress { get; set; }
       public string UserAgent { get; set; }
       public DateTime Timestamp { get; set; }
   }
   ```

2. Create DbContext:
   ```csharp
   // BitcoinHotWallet.Infrastructure/Data/ApplicationDbContext.cs
   public class ApplicationDbContext : DbContext
   {
       public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
           : base(options)
       {
       }
       
       public DbSet<User> Users { get; set; }
       public DbSet<Wallet> Wallets { get; set; }
       public DbSet<Transaction> Transactions { get; set; }
       public DbSet<AuditLog> AuditLogs { get; set; }
       
       protected override void OnModelCreating(ModelBuilder modelBuilder)
       {
           base.OnModelCreating(modelBuilder);
           
           // User configuration
           modelBuilder.Entity<User>(entity =>
           {
               entity.HasKey(e => e.Id);
               entity.HasIndex(e => e.Email).IsUnique();
               entity.Property(e => e.Email).IsRequired().HasMaxLength(256);
           });
           
           // Wallet configuration
           modelBuilder.Entity<Wallet>(entity =>
           {
               entity.HasKey(e => e.Id);
               entity.HasIndex(e => e.Address).IsUnique();
               entity.HasIndex(e => e.UserId);
               entity.Property(e => e.Address).IsRequired().HasMaxLength(100);
               entity.Property(e => e.Balance).HasPrecision(18, 8);
               
               entity.HasOne(w => w.User)
                   .WithMany(u => u.Wallets)
                   .HasForeignKey(w => w.UserId)
                   .OnDelete(DeleteBehavior.Restrict);
           });
           
           // Transaction configuration
           modelBuilder.Entity<Transaction>(entity =>
           {
               entity.HasKey(e => e.Id);
               entity.HasIndex(e => e.TransactionHash);
               entity.HasIndex(e => new { e.WalletId, e.CreatedAt });
               entity.Property(e => e.Amount).HasPrecision(18, 8);
               entity.Property(e => e.Fee).HasPrecision(18, 8);
               
               entity.HasOne(t => t.Wallet)
                   .WithMany(w => w.Transactions)
                   .HasForeignKey(t => t.WalletId)
                   .OnDelete(DeleteBehavior.Restrict);
           });
           
           // Audit log configuration
           modelBuilder.Entity<AuditLog>(entity =>
           {
               entity.HasKey(e => e.Id);
               entity.HasIndex(e => new { e.UserId, e.Timestamp });
               entity.HasIndex(e => e.Timestamp);
           });
       }
   }
   ```

3. Create and run migrations:
   ```bash
   cd BitcoinHotWallet.Infrastructure
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

**Reference**: See [API_DEVELOPMENT_GUIDE.md](API_DEVELOPMENT_GUIDE.md) for service layer patterns

### Step 2.2: Implement Core Services
**Duration**: 5-7 days

**Tasks**:
1. **Wallet Service** - Create, manage, and retrieve wallet information
2. **Transaction Service** - Send, receive, and track transactions
3. **Bitcoin Service** - Interface with Bitcoin network using NBitcoin
4. **Security Service** - Encryption, key management, validation
5. **User Service** - User management and authentication

Example implementation:
```csharp
// BitcoinHotWallet.Core/Services/IWalletService.cs
public interface IWalletService
{
    Task<WalletDto> CreateWalletAsync(Guid userId, string label);
    Task<WalletDto> GetWalletAsync(Guid walletId);
    Task<List<WalletDto>> GetUserWalletsAsync(Guid userId);
    Task<decimal> GetBalanceAsync(Guid walletId);
    Task<bool> UpdateBalanceAsync(Guid walletId);
}

// BitcoinHotWallet.Infrastructure/Services/WalletService.cs
public class WalletService : IWalletService
{
    private readonly ApplicationDbContext _context;
    private readonly IBitcoinService _bitcoinService;
    private readonly ISecurityService _securityService;
    private readonly ILogger<WalletService> _logger;
    
    public WalletService(
        ApplicationDbContext context,
        IBitcoinService bitcoinService,
        ISecurityService securityService,
        ILogger<WalletService> logger)
    {
        _context = context;
        _bitcoinService = bitcoinService;
        _securityService = securityService;
        _logger = logger;
    }
    
    public async Task<WalletDto> CreateWalletAsync(Guid userId, string label)
    {
        _logger.LogInformation("Creating wallet for user {UserId}", userId);
        
        // Generate new Bitcoin wallet
        var walletInfo = _bitcoinService.GenerateWallet();
        
        // Encrypt private key
        var encryptedPrivateKey = _securityService.EncryptPrivateKey(walletInfo.PrivateKey);
        
        // Create wallet entity
        var wallet = new Wallet
        {
            Id = Guid.NewGuid(),
            UserId = userId,
            Address = walletInfo.Address,
            EncryptedPrivateKey = encryptedPrivateKey,
            PublicKey = walletInfo.PublicKey,
            Label = label,
            Balance = 0,
            CreatedAt = DateTime.UtcNow,
            IsActive = true
        };
        
        _context.Wallets.Add(wallet);
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Wallet {WalletId} created for user {UserId}", wallet.Id, userId);
        
        return MapToDto(wallet);
    }
    
    // ... other methods
}
```

**Reference**: 
- [BITCOIN_INTEGRATION_GUIDE.md](BITCOIN_INTEGRATION_GUIDE.md) for Bitcoin operations
- [API_DEVELOPMENT_GUIDE.md](API_DEVELOPMENT_GUIDE.md) for service patterns

### Step 2.3: Implement API Controllers
**Duration**: 3-4 days

**Tasks**:
1. Create API controllers with proper error handling
2. Implement authentication and authorization
3. Add request/response DTOs
4. Configure Swagger documentation

Example controller:
```csharp
// BitcoinHotWallet.Api/Controllers/WalletsController.cs
[ApiController]
[Route("api/v1/[controller]")]
[Authorize]
public class WalletsController : ControllerBase
{
    private readonly IWalletService _walletService;
    private readonly ILogger<WalletsController> _logger;
    
    public WalletsController(IWalletService walletService, ILogger<WalletsController> logger)
    {
        _walletService = walletService;
        _logger = logger;
    }
    
    /// <summary>
    /// Create a new wallet for the authenticated user
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(WalletDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> CreateWallet([FromBody] CreateWalletRequest request)
    {
        var userId = GetAuthenticatedUserId();
        var wallet = await _walletService.CreateWalletAsync(userId, request.Label);
        
        return CreatedAtAction(nameof(GetWallet), new { id = wallet.Id }, wallet);
    }
    
    /// <summary>
    /// Get wallet by ID
    /// </summary>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(WalletDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetWallet(Guid id)
    {
        var wallet = await _walletService.GetWalletAsync(id);
        if (wallet == null)
            return NotFound();
            
        return Ok(wallet);
    }
    
    /// <summary>
    /// Get all wallets for the authenticated user
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(List<WalletDto>), StatusCodes.Status200OK)]
    public async Task<IActionResult> GetUserWallets()
    {
        var userId = GetAuthenticatedUserId();
        var wallets = await _walletService.GetUserWalletsAsync(userId);
        
        return Ok(wallets);
    }
    
    private Guid GetAuthenticatedUserId()
    {
        var userIdClaim = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        return Guid.Parse(userIdClaim);
    }
}
```

**Reference**: See [API_DEVELOPMENT_GUIDE.md](API_DEVELOPMENT_GUIDE.md) for controller patterns

### Step 2.4: Implement Security Features
**Duration**: 3-4 days

**Tasks**:
1. Private key encryption/decryption
2. JWT authentication
3. Rate limiting
4. Input validation
5. Audit logging

**Reference**: See [BITCOIN_INTEGRATION_GUIDE.md](BITCOIN_INTEGRATION_GUIDE.md) for security best practices

---

## Phase 3: Blazor Frontend Development

### Step 3.1: Application Layout and Navigation
**Duration**: 2-3 days

**Tasks**:
1. Set up MudBlazor theme and layout
2. Create navigation structure
3. Implement authentication UI
4. Create responsive layout

Example layout:
```razor
@* BitcoinHotWallet.Web/Shared/MainLayout.razor *@
@inherits LayoutComponentBase

<MudThemeProvider Theme="_theme" />
<MudDialogProvider />
<MudSnackbarProvider />

<MudLayout>
    <MudAppBar Elevation="1">
        <MudIconButton Icon="@Icons.Material.Filled.Menu" Color="Color.Inherit" Edge="Edge.Start" OnClick="@ToggleDrawer" />
        <MudText Typo="Typo.h5" Class="ml-3">Bitcoin Hot Wallet</MudText>
        <MudSpacer />
        <AuthorizeView>
            <Authorized>
                <MudText Typo="Typo.body1" Class="mr-3">@context.User.Identity.Name</MudText>
                <MudButton Color="Color.Inherit" OnClick="Logout">Logout</MudButton>
            </Authorized>
            <NotAuthorized>
                <MudButton Href="/login" Color="Color.Inherit">Login</MudButton>
            </NotAuthorized>
        </AuthorizeView>
    </MudAppBar>
    
    <MudDrawer @bind-Open="_drawerOpen" ClipMode="DrawerClipMode.Always" Elevation="2">
        <NavMenu />
    </MudDrawer>
    
    <MudMainContent>
        <MudContainer MaxWidth="MaxWidth.ExtraLarge" Class="mt-4">
            @Body
        </MudContainer>
    </MudMainContent>
</MudLayout>

@code {
    private bool _drawerOpen = true;
    private MudTheme _theme = new MudTheme();
    
    private void ToggleDrawer()
    {
        _drawerOpen = !_drawerOpen;
    }
    
    private void Logout()
    {
        // Implement logout logic
    }
}
```

### Step 3.2: Dashboard and Wallet Management Pages
**Duration**: 4-5 days

**Tasks**:
1. **Dashboard**: Overview of all wallets and recent transactions
2. **Wallet List**: Display user's wallets with balances
3. **Wallet Details**: Detailed view with transaction history
4. **Create Wallet**: Form to create new wallet
5. **Send Bitcoin**: Transaction form with validation
6. **Receive Bitcoin**: Display QR code and address

Example dashboard:
```razor
@* BitcoinHotWallet.Web/Pages/Dashboard.razor *@
@page "/dashboard"
@attribute [Authorize]
@inject IWalletService WalletService
@inject ITransactionService TransactionService

<PageTitle>Dashboard</PageTitle>

<MudGrid>
    <MudItem xs="12" md="8">
        <MudPaper Class="pa-4" Elevation="2">
            <MudText Typo="Typo.h5" Class="mb-4">My Wallets</MudText>
            
            @if (_wallets == null)
            {
                <MudProgressCircular Indeterminate="true" />
            }
            else if (!_wallets.Any())
            {
                <MudText>No wallets found. Create your first wallet to get started.</MudText>
                <MudButton Color="Color.Primary" Href="/wallets/create" Class="mt-2">
                    Create Wallet
                </MudButton>
            }
            else
            {
                <MudTable Items="_wallets" Hover="true" Breakpoint="Breakpoint.Sm">
                    <HeaderContent>
                        <MudTh>Label</MudTh>
                        <MudTh>Address</MudTh>
                        <MudTh>Balance</MudTh>
                        <MudTh>Actions</MudTh>
                    </HeaderContent>
                    <RowTemplate>
                        <MudTd DataLabel="Label">@context.Label</MudTd>
                        <MudTd DataLabel="Address">
                            <code>@context.Address.Substring(0, 10)...</code>
                        </MudTd>
                        <MudTd DataLabel="Balance">
                            <strong>@context.Balance.ToString("F8") BTC</strong>
                        </MudTd>
                        <MudTd DataLabel="Actions">
                            <MudButton Size="Size.Small" Href="@($"/wallets/{context.Id}")">
                                View
                            </MudButton>
                        </MudTd>
                    </RowTemplate>
                </MudTable>
            }
        </MudPaper>
    </MudItem>
    
    <MudItem xs="12" md="4">
        <MudPaper Class="pa-4" Elevation="2">
            <MudText Typo="Typo.h6" Class="mb-2">Total Balance</MudText>
            <MudText Typo="Typo.h4" Color="Color.Primary">
                @_totalBalance.ToString("F8") BTC
            </MudText>
            <MudText Typo="Typo.body2" Class="mt-2">
                ≈ $@_usdValue.ToString("N2") USD
            </MudText>
        </MudPaper>
    </MudItem>
    
    <MudItem xs="12">
        <MudPaper Class="pa-4" Elevation="2">
            <MudText Typo="Typo.h6" Class="mb-4">Recent Transactions</MudText>
            <TransactionList Transactions="_recentTransactions" />
        </MudPaper>
    </MudItem>
</MudGrid>

@code {
    private List<WalletDto> _wallets;
    private List<TransactionDto> _recentTransactions;
    private decimal _totalBalance;
    private decimal _usdValue;
    
    protected override async Task OnInitializedAsync()
    {
        await LoadData();
    }
    
    private async Task LoadData()
    {
        _wallets = await WalletService.GetUserWalletsAsync();
        _totalBalance = _wallets.Sum(w => w.Balance);
        _recentTransactions = await TransactionService.GetRecentTransactionsAsync(10);
        _usdValue = await CalculateUsdValue(_totalBalance);
    }
    
    private async Task<decimal> CalculateUsdValue(decimal btcAmount)
    {
        // Get Bitcoin price from API
        return btcAmount * 43000; // Placeholder
    }
}
```

### Step 3.3: Real-time Updates with SignalR
**Duration**: 2-3 days

**Tasks**:
1. Configure SignalR hub
2. Implement real-time balance updates
3. Push transaction notifications
4. Add connection status indicator

**Reference**: Blazor Server uses SignalR by default for component updates

---

## Phase 4: Google Cloud Platform Infrastructure

### Step 4.1: Database Setup (Cloud SQL)
**Duration**: 1-2 days

**Tasks**:
1. Create Cloud SQL instance:
   ```bash
   gcloud sql instances create bitcoin-wallet-db \
     --database-version=POSTGRES_15 \
     --tier=db-f1-micro \
     --region=us-central1 \
     --backup \
     --backup-start-time=03:00 \
     --maintenance-window-day=SUN \
     --maintenance-window-hour=04 \
     --database-flags=max_connections=100
   ```

2. Create database and user:
   ```bash
   gcloud sql databases create bitcoinwallet --instance=bitcoin-wallet-db
   
   gcloud sql users create appuser \
     --instance=bitcoin-wallet-db \
     --password=YOUR_SECURE_PASSWORD
   ```

3. Store credentials in Secret Manager:
   ```bash
   echo -n "Host=INSTANCE_IP;Database=bitcoinwallet;Username=appuser;Password=YOUR_SECURE_PASSWORD" | \
     gcloud secrets create db-connection-string --data-file=-
   ```

### Step 4.2: Redis Cache Setup
**Duration**: 1 day

**Tasks**:
1. Create Memorystore instance:
   ```bash
   gcloud redis instances create bitcoin-wallet-cache \
     --size=1 \
     --region=us-central1 \
     --redis-version=redis_7_0
   ```

2. Store Redis connection in Secret Manager:
   ```bash
   REDIS_HOST=$(gcloud redis instances describe bitcoin-wallet-cache \
     --region=us-central1 --format="value(host)")
   REDIS_PORT=$(gcloud redis instances describe bitcoin-wallet-cache \
     --region=us-central1 --format="value(port)")
   
   echo -n "$REDIS_HOST:$REDIS_PORT" | \
     gcloud secrets create redis-connection-string --data-file=-
   ```

### Step 4.3: Container Registry Setup
**Duration**: 1 day

**Tasks**:
1. Create Artifact Registry repository:
   ```bash
   gcloud artifacts repositories create bitcoin-wallet-images \
     --repository-format=docker \
     --location=us-central1 \
     --description="Bitcoin wallet Docker images"
   ```

2. Configure Docker authentication:
   ```bash
   gcloud auth configure-docker us-central1-docker.pkg.dev
   ```

### Step 4.4: Networking Configuration
**Duration**: 1-2 days

**Tasks**:
1. Create VPC network:
   ```bash
   gcloud compute networks create bitcoin-wallet-vpc \
     --subnet-mode=custom
   
   gcloud compute networks subnets create bitcoin-wallet-subnet \
     --network=bitcoin-wallet-vpc \
     --region=us-central1 \
     --range=10.0.0.0/24
   ```

2. Create VPC connector for Cloud Run:
   ```bash
   gcloud compute networks vpc-access connectors create bitcoin-wallet-connector \
     --region=us-central1 \
     --network=bitcoin-wallet-vpc \
     --range=10.8.0.0/28
   ```

### Step 4.5: Load Balancer and SSL
**Duration**: 1-2 days

**Tasks**:
1. Reserve static IP:
   ```bash
   gcloud compute addresses create bitcoin-wallet-ip \
     --global
   ```

2. Set up Cloud Load Balancer with SSL certificate
3. Configure Cloud Armor for DDoS protection

---

## Phase 5: Security Implementation

### Step 5.1: Private Key Management
**Duration**: 2-3 days

**Tasks**:
1. Implement encryption service using Google Cloud KMS:
   ```csharp
   public class KmsEncryptionService : ISecurityService
   {
       private readonly KeyManagementServiceClient _kmsClient;
       private readonly string _keyName;
       
       public string EncryptPrivateKey(string privateKey)
       {
           var request = new EncryptRequest
           {
               Name = _keyName,
               Plaintext = ByteString.CopyFromUtf8(privateKey)
           };
           
           var response = _kmsClient.Encrypt(request);
           return Convert.ToBase64String(response.Ciphertext.ToByteArray());
       }
       
       public string DecryptPrivateKey(string encryptedKey)
       {
           var request = new DecryptRequest
           {
               Name = _keyName,
               Ciphertext = ByteString.CopyFrom(Convert.FromBase64String(encryptedKey))
           };
           
           var response = _kmsClient.Decrypt(request);
           return response.Plaintext.ToStringUtf8();
       }
   }
   ```

2. Never store private keys in plain text
3. Implement key rotation strategy
4. Add audit logging for key access

**Reference**: See [BITCOIN_INTEGRATION_GUIDE.md](BITCOIN_INTEGRATION_GUIDE.md) Security Best Practices section

### Step 5.2: Authentication & Authorization
**Duration**: 2-3 days

**Tasks**:
1. Implement JWT authentication
2. Add role-based authorization
3. Implement 2FA (Two-Factor Authentication)
4. Add API key management for service-to-service communication

### Step 5.3: Rate Limiting and DDoS Protection
**Duration**: 1-2 days

**Tasks**:
1. Implement API rate limiting using Redis
2. Configure Cloud Armor rules
3. Add IP allowlisting/blocklisting
4. Implement request throttling

### Step 5.4: Security Monitoring
**Duration**: 1-2 days

**Tasks**:
1. Set up security event logging
2. Configure alerts for suspicious activity
3. Implement audit trail
4. Add intrusion detection

**Reference**: See [CODE_REVIEW_BEST_PRACTICES.md](CODE_REVIEW_BEST_PRACTICES.md) for security review checklist

---

## Phase 6: Testing Strategy

### Step 6.1: Unit Tests
**Duration**: 3-4 days

**Tasks**:
1. Write unit tests for all services
2. Test wallet creation and management
3. Test transaction processing
4. Test Bitcoin integration
5. Aim for >80% code coverage

Example test:
```csharp
// BitcoinHotWallet.Tests/Services/WalletServiceTests.cs
public class WalletServiceTests
{
    private readonly Mock<ApplicationDbContext> _mockContext;
    private readonly Mock<IBitcoinService> _mockBitcoinService;
    private readonly Mock<ISecurityService> _mockSecurityService;
    private readonly WalletService _walletService;
    
    public WalletServiceTests()
    {
        _mockContext = new Mock<ApplicationDbContext>();
        _mockBitcoinService = new Mock<IBitcoinService>();
        _mockSecurityService = new Mock<ISecurityService>();
        
        _walletService = new WalletService(
            _mockContext.Object,
            _mockBitcoinService.Object,
            _mockSecurityService.Object,
            Mock.Of<ILogger<WalletService>>()
        );
    }
    
    [Fact]
    public async Task CreateWalletAsync_ValidInput_CreatesWallet()
    {
        // Arrange
        var userId = Guid.NewGuid();
        var label = "My Wallet";
        
        var walletInfo = new WalletInfo
        {
            Address = "tb1q...",
            PrivateKey = "cV...",
            PublicKey = "03..."
        };
        
        _mockBitcoinService
            .Setup(x => x.GenerateWallet())
            .Returns(walletInfo);
            
        _mockSecurityService
            .Setup(x => x.EncryptPrivateKey(It.IsAny<string>()))
            .Returns("encrypted_key");
        
        // Act
        var result = await _walletService.CreateWalletAsync(userId, label);
        
        // Assert
        result.Should().NotBeNull();
        result.Address.Should().Be(walletInfo.Address);
        result.Label.Should().Be(label);
        
        _mockBitcoinService.Verify(x => x.GenerateWallet(), Times.Once);
        _mockSecurityService.Verify(x => x.EncryptPrivateKey(walletInfo.PrivateKey), Times.Once);
    }
    
    [Fact]
    public async Task CreateWalletAsync_BitcoinServiceFails_ThrowsException()
    {
        // Arrange
        _mockBitcoinService
            .Setup(x => x.GenerateWallet())
            .Throws<Exception>();
        
        // Act & Assert
        await Assert.ThrowsAsync<Exception>(
            () => _walletService.CreateWalletAsync(Guid.NewGuid(), "Test")
        );
    }
}
```

**Reference**: See [TESTING_GUIDE.md](TESTING_GUIDE.md) for comprehensive testing patterns

### Step 6.2: Integration Tests
**Duration**: 2-3 days

**Tasks**:
1. Test API endpoints end-to-end
2. Test database operations
3. Test Bitcoin network integration (testnet)
4. Test authentication flows

### Step 6.3: Performance Tests
**Duration**: 2 days

**Tasks**:
1. Load testing with k6 or JMeter
2. Test API response times
3. Test database query performance
4. Identify and fix bottlenecks

**Reference**: See [PERFORMANCE_OPTIMIZATION_GUIDE.md](PERFORMANCE_OPTIMIZATION_GUIDE.md)

### Step 6.4: Security Tests
**Duration**: 2 days

**Tasks**:
1. Penetration testing
2. Vulnerability scanning
3. Test authentication/authorization
4. Test encryption/decryption

---

## Phase 7: Deployment

### Step 7.1: Containerization
**Duration**: 2 days

**Tasks**:
1. Create Dockerfiles:
   ```dockerfile
   # BitcoinHotWallet.Api/Dockerfile
   FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
   WORKDIR /app
   EXPOSE 8080
   
   FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
   WORKDIR /src
   COPY ["BitcoinHotWallet.Api/BitcoinHotWallet.Api.csproj", "BitcoinHotWallet.Api/"]
   COPY ["BitcoinHotWallet.Core/BitcoinHotWallet.Core.csproj", "BitcoinHotWallet.Core/"]
   COPY ["BitcoinHotWallet.Infrastructure/BitcoinHotWallet.Infrastructure.csproj", "BitcoinHotWallet.Infrastructure/"]
   RUN dotnet restore "BitcoinHotWallet.Api/BitcoinHotWallet.Api.csproj"
   COPY . .
   WORKDIR "/src/BitcoinHotWallet.Api"
   RUN dotnet build "BitcoinHotWallet.Api.csproj" -c Release -o /app/build
   
   FROM build AS publish
   RUN dotnet publish "BitcoinHotWallet.Api.csproj" -c Release -o /app/publish
   
   FROM base AS final
   WORKDIR /app
   COPY --from=publish /app/publish .
   ENTRYPOINT ["dotnet", "BitcoinHotWallet.Api.dll"]
   ```

2. Build and test Docker images locally
3. Push images to Artifact Registry

### Step 7.2: Deploy to Cloud Run
**Duration**: 2-3 days

**Tasks**:
1. Deploy API service:
   ```bash
   gcloud run deploy bitcoin-wallet-api \
     --image=us-central1-docker.pkg.dev/bitcoin-hotwallet-prod/bitcoin-wallet-images/api:latest \
     --platform=managed \
     --region=us-central1 \
     --allow-unauthenticated \
     --vpc-connector=bitcoin-wallet-connector \
     --set-secrets=DB_CONNECTION_STRING=db-connection-string:latest,REDIS_CONNECTION=redis-connection-string:latest \
     --set-env-vars=ASPNETCORE_ENVIRONMENT=Production \
     --min-instances=1 \
     --max-instances=10 \
     --memory=512Mi \
     --cpu=1
   ```

2. Deploy Web application:
   ```bash
   gcloud run deploy bitcoin-wallet-web \
     --image=us-central1-docker.pkg.dev/bitcoin-hotwallet-prod/bitcoin-wallet-images/web:latest \
     --platform=managed \
     --region=us-central1 \
     --allow-unauthenticated \
     --vpc-connector=bitcoin-wallet-connector \
     --set-env-vars=API_URL=https://bitcoin-wallet-api-xxxxx-uc.a.run.app \
     --min-instances=1 \
     --max-instances=10 \
     --memory=1Gi \
     --cpu=1
   ```

### Step 7.3: CI/CD Pipeline
**Duration**: 2-3 days

**Tasks**:
1. Create GitHub Actions workflow:
   ```yaml
   # .github/workflows/deploy.yml
   name: Build and Deploy
   
   on:
     push:
       branches: [ main ]
     pull_request:
       branches: [ main ]
   
   env:
     PROJECT_ID: bitcoin-hotwallet-prod
     REGION: us-central1
     REGISTRY: us-central1-docker.pkg.dev
   
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Setup .NET
           uses: actions/setup-dotnet@v3
           with:
             dotnet-version: '8.0.x'
         - name: Restore dependencies
           run: dotnet restore
         - name: Build
           run: dotnet build --no-restore
         - name: Test
           run: dotnet test --no-build --verbosity normal
     
     build-and-push:
       needs: test
       runs-on: ubuntu-latest
       if: github.event_name == 'push' && github.ref == 'refs/heads/main'
       steps:
         - uses: actions/checkout@v3
         
         - name: Authenticate to Google Cloud
           uses: google-github-actions/auth@v1
           with:
             credentials_json: ${{ secrets.GCP_SA_KEY }}
         
         - name: Set up Cloud SDK
           uses: google-github-actions/setup-gcloud@v1
         
         - name: Configure Docker
           run: gcloud auth configure-docker ${{ env.REGISTRY }}
         
         - name: Build and Push API Image
           run: |
             docker build -t ${{ env.REGISTRY }}/${{ env.PROJECT_ID }}/bitcoin-wallet-images/api:${{ github.sha }} \
               -t ${{ env.REGISTRY }}/${{ env.PROJECT_ID }}/bitcoin-wallet-images/api:latest \
               -f BitcoinHotWallet.Api/Dockerfile .
             docker push ${{ env.REGISTRY }}/${{ env.PROJECT_ID }}/bitcoin-wallet-images/api:${{ github.sha }}
             docker push ${{ env.REGISTRY }}/${{ env.PROJECT_ID }}/bitcoin-wallet-images/api:latest
         
         - name: Deploy to Cloud Run
           run: |
             gcloud run deploy bitcoin-wallet-api \
               --image=${{ env.REGISTRY }}/${{ env.PROJECT_ID }}/bitcoin-wallet-images/api:${{ github.sha }} \
               --platform=managed \
               --region=${{ env.REGION }} \
               --project=${{ env.PROJECT_ID }}
   ```

2. Set up automated testing in pipeline
3. Implement blue-green deployment

### Step 7.4: Domain and SSL Configuration
**Duration**: 1 day

**Tasks**:
1. Register domain or use existing
2. Configure Cloud DNS
3. Set up SSL certificate with Let's Encrypt or Google-managed certificate
4. Configure domain mapping for Cloud Run services

---

## Phase 8: Monitoring & Maintenance

### Step 8.1: Logging and Monitoring
**Duration**: 2-3 days

**Tasks**:
1. Configure structured logging:
   ```csharp
   // Program.cs
   builder.Services.AddLogging(logging =>
   {
       logging.AddConsole();
       logging.AddGoogle(builder.Configuration);
   });
   
   builder.Services.AddSingleton<ILoggerProvider, GoogleLoggerProvider>();
   ```

2. Set up Cloud Monitoring dashboards:
   - API response times
   - Error rates
   - Transaction volumes
   - Wallet creation rates
   - Database performance

3. Configure log-based metrics
4. Set up log sinks for long-term storage

### Step 8.2: Alerting
**Duration**: 1-2 days

**Tasks**:
1. Create alert policies:
   - High error rate (>5%)
   - Slow API responses (>2 seconds)
   - Database connection failures
   - High memory/CPU usage
   - Failed Bitcoin transactions
   - Suspicious activity (multiple failed login attempts)

2. Configure notification channels:
   - Email
   - SMS
   - Slack/Discord
   - PagerDuty (for critical alerts)

### Step 8.3: Backup and Disaster Recovery
**Duration**: 1-2 days

**Tasks**:
1. Configure automated database backups
2. Set up backup retention policies
3. Test backup restoration process
4. Document disaster recovery procedures
5. Create runbooks for common incidents

### Step 8.4: Maintenance Procedures
**Duration**: 1 day

**Tasks**:
1. Document deployment procedures
2. Create database migration process
3. Establish update schedule
4. Plan for dependency updates
5. Schedule security patches

**Reference**: See [PERFORMANCE_OPTIMIZATION_GUIDE.md](PERFORMANCE_OPTIMIZATION_GUIDE.md) for monitoring tools

---

## Success Metrics

### Technical Metrics
- **API Response Time**: < 200ms for 95th percentile
- **Uptime**: 99.9% availability
- **Error Rate**: < 0.1% of requests
- **Test Coverage**: > 80% code coverage
- **Security**: Zero critical vulnerabilities
- **Transaction Success Rate**: > 99%

### Business Metrics
- **User Registration**: Track new user signups
- **Wallet Creation**: Monitor wallet creation rate
- **Transaction Volume**: Track BTC transaction volume
- **Active Users**: Daily/Monthly active users
- **User Retention**: 7-day and 30-day retention rates

### Performance Benchmarks
- **Wallet Creation**: < 1 second
- **Balance Lookup**: < 100ms
- **Transaction Submission**: < 2 seconds
- **Database Queries**: < 50ms average
- **Page Load Time**: < 2 seconds

---

## Risk Management

### Technical Risks

| Risk | Impact | Probability | Mitigation Strategy |
|------|--------|------------|---------------------|
| Private key compromise | Critical | Low | Use Google KMS, encryption at rest, strict access controls, audit logging |
| Database failure | High | Low | Automated backups, multi-zone deployment, point-in-time recovery |
| Bitcoin network issues | Medium | Medium | Implement retry logic, use multiple nodes, graceful degradation |
| DDoS attack | High | Medium | Cloud Armor, rate limiting, auto-scaling |
| API vulnerabilities | High | Low | Regular security audits, automated scanning, code reviews |
| Data loss | Critical | Very Low | Regular backups, replication, disaster recovery plan |

### Operational Risks

| Risk | Impact | Probability | Mitigation Strategy |
|------|--------|------------|---------------------|
| Insufficient capacity | Medium | Low | Auto-scaling, performance monitoring, load testing |
| Configuration errors | Medium | Medium | Infrastructure as Code, automated testing, code reviews |
| Third-party service failure | Medium | Low | Fallback options, circuit breakers, monitoring |
| Regulatory changes | High | Medium | Stay informed, flexible architecture, compliance reviews |

### Financial Risks

| Risk | Impact | Probability | Mitigation Strategy |
|------|--------|------------|---------------------|
| High GCP costs | Medium | Medium | Cost monitoring, budget alerts, resource optimization |
| Bitcoin volatility | Low | High | Display USD equivalent, transaction limits |
| Fraudulent transactions | High | Low | Transaction validation, user verification, rate limiting |

---

## Timeline & Resources

### Estimated Timeline

| Phase | Duration | Team Size |
|-------|----------|-----------|
| Phase 1: Environment Setup | 1 week | 1 developer |
| Phase 2: Backend API Development | 3 weeks | 2 developers |
| Phase 3: Blazor Frontend Development | 2 weeks | 1 frontend developer |
| Phase 4: GCP Infrastructure | 1 week | 1 DevOps engineer |
| Phase 5: Security Implementation | 2 weeks | 1 security-focused developer |
| Phase 6: Testing Strategy | 2 weeks | 2 developers + 1 QA |
| Phase 7: Deployment | 1 week | 1 DevOps engineer |
| Phase 8: Monitoring & Maintenance | 1 week | 1 DevOps engineer |
| **Total** | **13 weeks** | **3-4 people** |

### Recommended Team Composition

1. **Backend Developer** (Senior)
   - API development
   - Bitcoin integration
   - Database design
   - Security implementation

2. **Frontend Developer** (Mid-level)
   - Blazor application development
   - UI/UX implementation
   - Real-time features

3. **DevOps Engineer** (Mid-level)
   - GCP infrastructure
   - CI/CD pipeline
   - Monitoring and alerting
   - Security configuration

4. **QA Engineer** (Optional, can be handled by developers)
   - Test planning
   - Test automation
   - Performance testing
   - Security testing

### Cost Estimate (GCP)

**Monthly Estimated Costs** (Low traffic - first 6 months):
- Cloud Run (API + Web): $20-50
- Cloud SQL (db-f1-micro): $7-15
- Cloud Memorystore (1GB Redis): $36
- Cloud Load Balancer: $18
- Cloud Monitoring & Logging: $10-20
- Cloud Storage (backups): $5
- **Total**: ~$100-150/month

**Monthly Estimated Costs** (Medium traffic - production):
- Cloud Run (scaled): $100-300
- Cloud SQL (db-custom-2-8192): $80-150
- Cloud Memorystore (5GB Redis): $180
- Cloud Load Balancer: $18
- Cloud Monitoring & Logging: $50-100
- Cloud Storage: $10-20
- **Total**: ~$450-800/month

---

## Appendix: Quick Reference

### Key Technologies
- **.NET 8**: Application framework
- **Blazor Server**: Frontend framework
- **NBitcoin**: Bitcoin library for .NET
- **Entity Framework Core**: ORM
- **PostgreSQL**: Primary database
- **Redis**: Caching and session storage
- **Docker**: Containerization
- **Google Cloud Run**: Container hosting
- **Cloud SQL**: Managed PostgreSQL
- **Cloud Memorystore**: Managed Redis

### Essential Commands

```bash
# Local Development
dotnet build
dotnet test
dotnet run --project BitcoinHotWallet.Api
dotnet run --project BitcoinHotWallet.Web

# Docker
docker build -t bitcoin-wallet-api -f BitcoinHotWallet.Api/Dockerfile .
docker run -p 8080:8080 bitcoin-wallet-api

# GCP Deployment
gcloud run deploy bitcoin-wallet-api --image=... --platform=managed
gcloud sql instances describe bitcoin-wallet-db
gcloud redis instances describe bitcoin-wallet-cache

# Database Migrations
dotnet ef migrations add MigrationName
dotnet ef database update
```

### Important Documentation Links
- [API Development Guide](API_DEVELOPMENT_GUIDE.md)
- [Bitcoin Integration Guide](BITCOIN_INTEGRATION_GUIDE.md)
- [Testing Guide](TESTING_GUIDE.md)
- [Performance Optimization Guide](PERFORMANCE_OPTIMIZATION_GUIDE.md)
- [Code Review Best Practices](CODE_REVIEW_BEST_PRACTICES.md)

### Security Checklist
- [ ] Private keys encrypted with Google KMS
- [ ] HTTPS enforced everywhere
- [ ] JWT authentication implemented
- [ ] Rate limiting configured
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS protection enabled
- [ ] CSRF protection enabled
- [ ] Security headers configured
- [ ] Audit logging implemented
- [ ] Regular security scans scheduled
- [ ] Dependency vulnerability scanning
- [ ] Secrets stored in Secret Manager
- [ ] Database encrypted at rest
- [ ] Backup encryption enabled

---

## Conclusion

This deployment plan provides a comprehensive roadmap for building and deploying a production-ready custodial Bitcoin hot-wallet system using Blazor and Google Cloud Platform. By following this plan and leveraging the technical guides in this repository, you will demonstrate:

✅ **Backend Software Engineering Competency**
- API design and development
- Bitcoin integration expertise
- Security best practices
- Performance optimization
- Testing and code quality
- Cloud deployment skills

✅ **Technology Stack Proficiency**
- C#/.NET development
- Blazor framework
- NBitcoin library
- xUnit and Moq testing
- Google Cloud Platform
- Docker and containerization

✅ **Software Development Best Practices**
- Clean architecture
- Test-driven development
- Security-first approach
- Comprehensive documentation
- Code review practices
- Performance monitoring

This project showcases the complete software development lifecycle from design to deployment, demonstrating the skills required for a Backend Software Engineer role focused on Bitcoin and API development.

**Good luck with your deployment!** 🚀🔐💰

---

## After: Rootstock (RSK) Integration Considerations

### What is Rootstock?

**Rootstock (RSK)** is a smart contract platform that is connected to the Bitcoin blockchain through a two-way peg mechanism, also known as a sidechain. It brings Ethereum-compatible smart contract functionality to the Bitcoin ecosystem, allowing developers to build decentralized applications (dApps) while leveraging Bitcoin's security.

### Key Features

#### 1. **Bitcoin-Backed Smart Contracts**
- RSK enables Ethereum-compatible smart contracts secured by Bitcoin's proof-of-work
- Compatible with Ethereum Virtual Machine (EVM) and Solidity programming language
- Allows developers to use familiar Ethereum tools and frameworks

#### 2. **Two-Way Peg (2WP)**
- BTC can be converted to RBTC (Rootstock's native token) and back
- Maintains a 1:1 peg with Bitcoin
- Enables Bitcoin holders to interact with smart contracts without selling their BTC

#### 3. **Merge Mining**
- Secured by Bitcoin miners through merge mining
- Provides substantial security without requiring separate mining infrastructure
- Currently secured by a significant portion of Bitcoin's hash rate

#### 4. **Fast Block Times**
- Average block time of 30 seconds (compared to Bitcoin's 10 minutes)
- Enables faster transaction confirmations for smart contract interactions
- Better user experience for dApp users

### Integration Opportunities

#### Extending the Hot-Wallet System for RSK

If you want to extend this Bitcoin hot-wallet deployment to support Rootstock, consider the following integration points:

##### 1. **RBTC Wallet Support**
```csharp
// Example: Extend wallet service to support RBTC
public interface IRootstock WalletService
{
    Task<string> ConvertBtcToRbtc(decimal amount, string btcAddress);
    Task<string> ConvertRbtcToBtc(decimal amount, string rbtcAddress);
    Task<decimal> GetRbtcBalance(string address);
    Task<RskTransaction> SendRbtcTransaction(string from, string to, decimal amount);
}
```

##### 2. **Smart Contract Integration**
- Deploy and interact with smart contracts on RSK
- Manage token transfers (ERC-20 compatible tokens on RSK)
- Handle DeFi protocols built on Rootstock

##### 3. **Cross-Chain Functionality**
- Implement BTC ↔ RBTC conversion workflows
- Monitor peg transactions
- Handle both Bitcoin and RSK transactions within the same wallet interface

### Technical Resources

#### Development Documentation
- **Developer Portal**: https://dev.rootstock.io/
  - API documentation
  - Smart contract guides
  - Integration tutorials
  - SDK and library references

#### Main Resources
- **Official Website**: https://rootstock.io/
  - Overview and ecosystem information
  - Use cases and success stories
  - Community resources

#### Key Libraries and Tools
- **RSKj**: Full node implementation (Java-based)
- **Web3.js/Ethers.js**: Compatible JavaScript libraries for RSK interaction
- **Remix IDE**: Smart contract development environment
- **Hardhat/Truffle**: Development frameworks compatible with RSK

### Deployment Considerations

#### Infrastructure Requirements
1. **RSK Node**
   - Run an RSK node for direct blockchain interaction
   - Alternative: Use RPC providers like RSK Public Nodes or Infrastructure providers

2. **Smart Contract Deployment**
   - Deploy contracts using standard Ethereum tooling
   - Gas costs paid in RBTC (typically lower than Ethereum)

3. **Monitoring and Indexing**
   - RSK Block Explorer integration
   - Custom indexers for wallet-specific data
   - Event monitoring for smart contract interactions

#### Security Considerations
1. **Key Management**
   - Separate key management for BTC and RBTC/RSK accounts
   - Consider using the same KMS approach (Google Cloud KMS)
   - Implement multi-signature wallets for enhanced security

2. **Peg Monitoring**
   - Monitor two-way peg transactions
   - Implement safety checks for peg-in/peg-out operations
   - Track confirmation times and transaction status

3. **Smart Contract Audits**
   - Any custom smart contracts should be professionally audited
   - Use established, audited contracts when possible
   - Implement upgrade mechanisms for contracts

### Use Cases for RSK Integration

1. **DeFi Services**
   - Lending and borrowing platforms
   - Decentralized exchanges (DEXs)
   - Yield farming opportunities

2. **Tokenization**
   - Issue custom tokens on RSK
   - Create Bitcoin-backed synthetic assets
   - NFT minting and trading

3. **Payment Solutions**
   - Fast, programmable Bitcoin payments
   - Conditional payment smart contracts
   - Automated payment splitting and distribution

4. **Cross-Chain Services**
   - Bridge between Bitcoin and other blockchain ecosystems
   - Liquidity provision across chains
   - Arbitrage opportunities

### Migration Path

If you decide to extend this hot-wallet system to support Rootstock:

#### Phase 1: Research & Planning (1-2 weeks)
- [ ] Deep dive into RSK documentation and architecture
- [ ] Identify specific RSK features needed for your use case
- [ ] Design wallet data model extensions for RBTC support
- [ ] Plan smart contract requirements (if any)

#### Phase 2: Development Environment (1 week)
- [ ] Set up RSK testnet/regtest environment
- [ ] Configure RSK node or RPC endpoint access
- [ ] Install and configure development tools (Hardhat, Web3 libraries)
- [ ] Create test accounts and acquire testnet RBTC

#### Phase 3: Core Integration (2-3 weeks)
- [ ] Implement RBTC wallet generation and management
- [ ] Add RBTC transaction creation and signing
- [ ] Implement balance checking and transaction history
- [ ] Build two-way peg integration (BTC ↔ RBTC)

#### Phase 4: Smart Contract Layer (2-4 weeks, if applicable)
- [ ] Develop or integrate with existing smart contracts
- [ ] Implement contract interaction methods
- [ ] Add event listening and processing
- [ ] Create admin functions for contract management

#### Phase 5: Testing & Security (2 weeks)
- [ ] Unit tests for all RSK-related functionality
- [ ] Integration tests with RSK testnet
- [ ] Security audit of new code
- [ ] Penetration testing for RSK components

#### Phase 6: Deployment (1 week)
- [ ] Deploy RSK node or configure mainnet RPC access
- [ ] Deploy smart contracts to RSK mainnet (if applicable)
- [ ] Migrate existing infrastructure to support RSK
- [ ] Monitor initial transactions closely

### Cost Implications

#### Additional Infrastructure Costs
- **RSK Node Hosting**: $100-300/month (if self-hosting)
- **RPC Provider Services**: $50-200/month (alternative to self-hosting)
- **Additional Storage**: $20-50/month for RSK blockchain data
- **Smart Contract Deployment**: One-time gas costs (typically low on RSK)

#### Development Effort
- **Estimated Development Time**: 8-12 weeks for full integration
- **Maintenance Overhead**: +20% ongoing maintenance compared to Bitcoin-only

### Limitations and Challenges

1. **Ecosystem Maturity**
   - Smaller ecosystem compared to Ethereum
   - Fewer available dApps and integrations
   - Limited tooling in some areas

2. **Peg Transaction Times**
   - Two-way peg operations can take time (multiple Bitcoin confirmations)
   - Not suitable for instant conversions
   - Requires careful UX design to manage user expectations

3. **Additional Complexity**
   - Need to manage both Bitcoin and RSK transaction logic
   - Smart contract upgrades and maintenance
   - More complex error handling and recovery scenarios

4. **Documentation Gaps**
   - Some advanced features may have limited documentation
   - Smaller community compared to Bitcoin or Ethereum mainnet
   - May require more independent research and experimentation

### Conclusion

Rootstock represents an exciting opportunity to extend Bitcoin's functionality with smart contracts while maintaining Bitcoin's security guarantees. For a hot-wallet system, RSK integration can unlock DeFi capabilities, faster transactions, and programmable money features that aren't possible with Bitcoin alone.

However, this comes with added complexity and should only be pursued if there's a clear business case for smart contract functionality. Start with a solid Bitcoin-only implementation (as outlined in this deployment plan), then consider RSK as a valuable extension once the core system is stable and proven.

**Key Takeaway**: Rootstock bridges Bitcoin's security with Ethereum's programmability, offering a powerful platform for developers who want to build on Bitcoin without leaving its ecosystem.

### Further Reading
- Explore the Rootstock documentation at https://dev.rootstock.io/
- Join the RSK community forums and Discord channels
- Review example projects and dApps built on Rootstock
- Study the two-way peg mechanism in detail
- Investigate RIF (Rootstock Infrastructure Framework) services

**Good luck exploring Rootstock!** ⛓️🔗💻