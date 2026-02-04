# Performance Optimization Guide

## Introduction
This guide covers performance optimization techniques for .NET applications, with a focus on API development and Bitcoin-related operations.

## Performance Principles

### 1. Measure Before Optimizing
> "Premature optimization is the root of all evil" - Donald Knuth

Always profile and measure before making performance changes:
```bash
# Use BenchmarkDotNet for micro-benchmarks
dotnet add package BenchmarkDotNet

# Use Application Insights for production monitoring
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

### 2. Identify Bottlenecks
Common bottlenecks in backend systems:
- **Database queries**: Slow or N+1 queries
- **External API calls**: Network latency
- **Memory allocation**: Excessive GC pressure
- **Synchronous operations**: Blocking threads
- **Inefficient algorithms**: O(n²) when O(n) possible

## Database Optimization

### 1. Query Optimization
```csharp
// ❌ N+1 Query Problem
public async Task<List<WalletDto>> GetWalletsWithTransactions()
{
    var wallets = await _context.Wallets.ToListAsync();
    
    foreach (var wallet in wallets)
    {
        // This executes a query for EACH wallet!
        wallet.Transactions = await _context.Transactions
            .Where(t => t.WalletId == wallet.Id)
            .ToListAsync();
    }
    
    return wallets;
}

// ✅ Use Eager Loading
public async Task<List<WalletDto>> GetWalletsWithTransactions()
{
    return await _context.Wallets
        .Include(w => w.Transactions)  // Single query with JOIN
        .ToListAsync();
}

// ✅ Or use explicit loading with one additional query
public async Task<List<WalletDto>> GetWalletsWithTransactions()
{
    var wallets = await _context.Wallets.ToListAsync();
    
    var walletIds = wallets.Select(w => w.Id).ToList();
    var transactions = await _context.Transactions
        .Where(t => walletIds.Contains(t.WalletId))
        .ToListAsync();
    
    // Map transactions to wallets in memory
    return MapWalletsWithTransactions(wallets, transactions);
}
```

### 2. Use Pagination
```csharp
// ❌ Loading all records
public async Task<List<Transaction>> GetTransactions()
{
    return await _context.Transactions.ToListAsync(); // Could be millions!
}

// ✅ Paginated results
public async Task<PagedResult<Transaction>> GetTransactions(int page = 1, int pageSize = 50)
{
    var query = _context.Transactions.OrderByDescending(t => t.CreatedAt);
    
    var total = await query.CountAsync();
    var items = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();
    
    return new PagedResult<Transaction>
    {
        Items = items,
        TotalCount = total,
        Page = page,
        PageSize = pageSize
    };
}
```

### 3. Use Indexes
```csharp
// Add indexes to frequently queried columns
public class WalletConfiguration : IEntityTypeConfiguration<Wallet>
{
    public void Configure(EntityTypeBuilder<Wallet> builder)
    {
        // Index on frequently searched columns
        builder.HasIndex(w => w.Address);
        builder.HasIndex(w => w.UserId);
        
        // Composite index for common query patterns
        builder.HasIndex(w => new { w.UserId, w.CreatedAt });
    }
}
```

### 4. Use AsNoTracking for Read-Only Queries
```csharp
// ❌ Change tracking adds overhead for read-only operations
public async Task<List<Transaction>> GetTransactionHistory(string walletId)
{
    return await _context.Transactions
        .Where(t => t.WalletId == walletId)
        .ToListAsync();
}

// ✅ Disable change tracking when not needed
public async Task<List<Transaction>> GetTransactionHistory(string walletId)
{
    return await _context.Transactions
        .AsNoTracking()
        .Where(t => t.WalletId == walletId)
        .ToListAsync();
}
```

## Caching Strategies

### 1. In-Memory Cache
```csharp
public class BitcoinPriceService
{
    private readonly IMemoryCache _cache;
    private readonly IBitcoinApiClient _apiClient;

    public BitcoinPriceService(IMemoryCache cache, IBitcoinApiClient apiClient)
    {
        _cache = cache;
        _apiClient = apiClient;
    }

    public async Task<decimal> GetCurrentPriceAsync()
    {
        const string cacheKey = "bitcoin_price";
        
        if (_cache.TryGetValue(cacheKey, out decimal cachedPrice))
        {
            return cachedPrice;
        }

        var price = await _apiClient.GetPriceAsync();
        
        var cacheOptions = new MemoryCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5),
            SlidingExpiration = TimeSpan.FromMinutes(2)
        };
        
        _cache.Set(cacheKey, price, cacheOptions);
        
        return price;
    }
}
```

### 2. Distributed Cache (Redis)
```csharp
public class TransactionCacheService
{
    private readonly IDistributedCache _cache;
    private readonly ITransactionRepository _repository;

    public TransactionCacheService(
        IDistributedCache cache, 
        ITransactionRepository repository)
    {
        _cache = cache;
        _repository = repository;
    }

    public async Task<Transaction> GetTransactionAsync(string txId)
    {
        var cacheKey = $"transaction:{txId}";
        
        var cachedData = await _cache.GetStringAsync(cacheKey);
        if (cachedData != null)
        {
            return JsonSerializer.Deserialize<Transaction>(cachedData);
        }

        var transaction = await _repository.GetByIdAsync(txId);
        if (transaction != null)
        {
            var serialized = JsonSerializer.Serialize(transaction);
            await _cache.SetStringAsync(
                cacheKey, 
                serialized,
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1)
                });
        }

        return transaction;
    }
}
```

### 3. Response Caching
```csharp
[ApiController]
[Route("api/[controller]")]
public class BitcoinStatsController : ControllerBase
{
    // Cache response for 5 minutes
    [HttpGet("network-info")]
    [ResponseCache(Duration = 300, Location = ResponseCacheLocation.Any)]
    public async Task<IActionResult> GetNetworkInfo()
    {
        var info = await _bitcoinService.GetNetworkInfoAsync();
        return Ok(info);
    }

    // Cache per user
    [HttpGet("portfolio")]
    [ResponseCache(Duration = 60, Location = ResponseCacheLocation.Client, VaryByQueryKeys = new[] { "userId" })]
    public async Task<IActionResult> GetPortfolio([FromQuery] string userId)
    {
        var portfolio = await _portfolioService.GetAsync(userId);
        return Ok(portfolio);
    }
}
```

## Async/Await Optimization

### 1. Use Async All the Way
```csharp
// ❌ Blocking async code
public decimal GetBalance(string walletId)
{
    return _repository.GetBalanceAsync(walletId).Result; // Deadlock risk!
}

// ✅ Async all the way
public async Task<decimal> GetBalanceAsync(string walletId)
{
    return await _repository.GetBalanceAsync(walletId);
}
```

### 2. Parallel Operations
```csharp
// ❌ Sequential operations
public async Task<DashboardData> GetDashboardAsync(string userId)
{
    var wallets = await _walletService.GetWalletsAsync(userId);
    var transactions = await _transactionService.GetRecentAsync(userId);
    var balance = await _balanceService.GetTotalAsync(userId);
    
    return new DashboardData { Wallets = wallets, Transactions = transactions, Balance = balance };
}

// ✅ Parallel operations (if independent)
public async Task<DashboardData> GetDashboardAsync(string userId)
{
    var walletsTask = _walletService.GetWalletsAsync(userId);
    var transactionsTask = _transactionService.GetRecentAsync(userId);
    var balanceTask = _balanceService.GetTotalAsync(userId);
    
    await Task.WhenAll(walletsTask, transactionsTask, balanceTask);
    
    return new DashboardData 
    { 
        Wallets = walletsTask.Result, 
        Transactions = transactionsTask.Result, 
        Balance = balanceTask.Result 
    };
}
```

### 3. ValueTask for Hot Paths
```csharp
// For frequently synchronous paths, use ValueTask to avoid allocation
public ValueTask<Wallet> GetWalletAsync(string id)
{
    // Check cache first (often hits)
    if (_cache.TryGetValue(id, out Wallet wallet))
    {
        return new ValueTask<Wallet>(wallet); // No allocation
    }
    
    // Fallback to async database call
    return new ValueTask<Wallet>(GetWalletFromDatabaseAsync(id));
}

private async Task<Wallet> GetWalletFromDatabaseAsync(string id)
{
    return await _context.Wallets.FindAsync(id);
}
```

## Algorithm Optimization

### 1. Choose Right Data Structure
```csharp
// ❌ Using List for lookups - O(n)
public class WalletManager
{
    private List<Wallet> _wallets = new();
    
    public Wallet Find(string id)
    {
        return _wallets.FirstOrDefault(w => w.Id == id); // O(n)
    }
}

// ✅ Using Dictionary for lookups - O(1)
public class WalletManager
{
    private Dictionary<string, Wallet> _wallets = new();
    
    public Wallet Find(string id)
    {
        return _wallets.TryGetValue(id, out var wallet) ? wallet : null; // O(1)
    }
}
```

### 2. Avoid LINQ in Hot Paths
```csharp
// ❌ LINQ creates overhead in tight loops
public decimal CalculateTotalBalance(List<Wallet> wallets)
{
    return wallets.Sum(w => w.Balance); // Allocates iterator
}

// ✅ Use for loop for better performance
public decimal CalculateTotalBalance(List<Wallet> wallets)
{
    decimal total = 0;
    for (int i = 0; i < wallets.Count; i++)
    {
        total += wallets[i].Balance;
    }
    return total;
}

// ✅ Or use Span for even better performance with arrays
public decimal CalculateTotalBalance(Span<Wallet> wallets)
{
    decimal total = 0;
    foreach (var wallet in wallets)
    {
        total += wallet.Balance;
    }
    return total;
}
```

## Memory Optimization

### 1. Object Pooling
```csharp
public class TransactionProcessor
{
    private static readonly ArrayPool<byte> _bytePool = ArrayPool<byte>.Shared;

    public async Task ProcessTransactionAsync(Transaction tx)
    {
        // Rent from pool instead of allocating
        var buffer = _bytePool.Rent(4096);
        try
        {
            // Use buffer
            await ProcessWithBufferAsync(tx, buffer);
        }
        finally
        {
            // Return to pool
            _bytePool.Return(buffer);
        }
    }
}
```

### 2. String Concatenation
```csharp
// ❌ String concatenation in loops
public string BuildTransactionSummary(List<Transaction> transactions)
{
    string summary = "";
    foreach (var tx in transactions)
    {
        summary += $"{tx.Id},{tx.Amount}\n"; // Creates new string each time
    }
    return summary;
}

// ✅ Use StringBuilder
public string BuildTransactionSummary(List<Transaction> transactions)
{
    var sb = new StringBuilder(transactions.Count * 50); // Pre-size if possible
    foreach (var tx in transactions)
    {
        sb.Append(tx.Id).Append(',').Append(tx.Amount).Append('\n');
    }
    return sb.ToString();
}
```

### 3. Reduce Allocations with Span<T>
```csharp
// ❌ Creates substring allocations
public string ExtractAddressPrefix(string address)
{
    return address.Substring(0, 4); // Allocates new string
}

// ✅ Use Span to avoid allocation
public ReadOnlySpan<char> ExtractAddressPrefix(ReadOnlySpan<char> address)
{
    return address.Slice(0, 4); // No allocation
}
```

## API Performance

### 1. Response Compression
```csharp
// In Program.cs or Startup.cs
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<GzipCompressionProvider>();
    options.Providers.Add<BrotliCompressionProvider>();
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = System.IO.Compression.CompressionLevel.Fastest;
});
```

### 2. Minimize Payload Size
```csharp
// ❌ Returning entire entities
[HttpGet]
public async Task<List<Transaction>> GetTransactions()
{
    return await _context.Transactions.ToListAsync(); // Returns all columns
}

// ✅ Return only needed data with DTOs
[HttpGet]
public async Task<List<TransactionSummaryDto>> GetTransactions()
{
    return await _context.Transactions
        .Select(t => new TransactionSummaryDto
        {
            Id = t.Id,
            Amount = t.Amount,
            Date = t.CreatedAt
            // Only fields client needs
        })
        .ToListAsync();
}
```

### 3. Use HTTP/2
```csharp
// Enable HTTP/2 in Kestrel configuration
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(5001, listenOptions =>
    {
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
        listenOptions.UseHttps();
    });
});
```

## Monitoring and Profiling

### 1. Application Insights
```csharp
public class BitcoinWalletService
{
    private readonly TelemetryClient _telemetry;

    public async Task<Wallet> CreateWalletAsync()
    {
        using var operation = _telemetry.StartOperation<DependencyTelemetry>("CreateWallet");
        
        var stopwatch = Stopwatch.StartNew();
        try
        {
            var wallet = await GenerateWalletAsync();
            await _repository.SaveAsync(wallet);
            
            _telemetry.TrackMetric("WalletCreationTime", stopwatch.ElapsedMilliseconds);
            
            return wallet;
        }
        catch (Exception ex)
        {
            operation.Telemetry.Success = false;
            _telemetry.TrackException(ex);
            throw;
        }
    }
}
```

### 2. Custom Metrics
```csharp
public class PerformanceMonitor
{
    private readonly ILogger<PerformanceMonitor> _logger;

    public async Task<T> MeasureAsync<T>(string operationName, Func<Task<T>> operation)
    {
        var stopwatch = Stopwatch.StartNew();
        try
        {
            var result = await operation();
            stopwatch.Stop();
            
            _logger.LogInformation(
                "Operation {Operation} completed in {ElapsedMs}ms",
                operationName,
                stopwatch.ElapsedMilliseconds);
            
            return result;
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            _logger.LogError(
                ex,
                "Operation {Operation} failed after {ElapsedMs}ms",
                operationName,
                stopwatch.ElapsedMilliseconds);
            throw;
        }
    }
}
```

### 3. Benchmarking with BenchmarkDotNet
```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
public class BitcoinAddressValidationBenchmark
{
    private const string ValidAddress = "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa";

    [Benchmark]
    public bool RegexValidation()
    {
        return Regex.IsMatch(ValidAddress, @"^[13][a-km-zA-HJ-NP-Z1-9]{25,34}$");
    }

    [Benchmark]
    public bool NBitcoinValidation()
    {
        try
        {
            BitcoinAddress.Create(ValidAddress, Network.Main);
            return true;
        }
        catch
        {
            return false;
        }
    }
}

// Run benchmarks
// BenchmarkRunner.Run<BitcoinAddressValidationBenchmark>();
```

## Performance Checklist

### Database
- [ ] Use indexes on frequently queried columns
- [ ] Implement pagination for large result sets
- [ ] Use AsNoTracking() for read-only queries
- [ ] Avoid N+1 queries with eager loading
- [ ] Use compiled queries for repeated patterns

### Caching
- [ ] Cache expensive computations
- [ ] Cache frequently accessed data
- [ ] Use appropriate cache expiration
- [ ] Consider distributed cache for scale-out

### API
- [ ] Enable response compression
- [ ] Minimize response payload size
- [ ] Use response caching where appropriate
- [ ] Implement pagination
- [ ] Use HTTP/2

### Code
- [ ] Use async/await properly
- [ ] Choose appropriate data structures
- [ ] Avoid allocations in hot paths
- [ ] Use StringBuilder for string concatenation
- [ ] Consider object pooling for frequently allocated objects

### Monitoring
- [ ] Set up Application Insights or similar
- [ ] Track key performance metrics
- [ ] Set up alerts for performance degradation
- [ ] Regular performance testing
- [ ] Profile production issues

## Tools and Resources

- **Profilers**: dotTrace, Visual Studio Profiler, PerfView
- **Benchmarking**: BenchmarkDotNet
- **Monitoring**: Application Insights, Prometheus, Grafana
- **Load Testing**: k6, Apache JMeter, Artillery
- **Database**: SQL Server Profiler, Entity Framework logging

## Common Performance Anti-Patterns

1. **Premature Optimization**: Optimize based on metrics, not assumptions
2. **Over-caching**: Cache invalidation is hard; cache only what's needed
3. **Ignoring the Database**: Often the bottleneck in APIs
4. **Blocking on Async**: Causes thread pool starvation
5. **Not Monitoring Production**: Can't improve what you don't measure
