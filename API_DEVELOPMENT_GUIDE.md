# API Development Guide

## Introduction
This guide covers best practices and patterns for building robust, scalable APIs in C#/.NET.

## API Design Principles

### RESTful Design
1. **Resource-Based URLs**: Use nouns, not verbs
   - ✅ `GET /api/users/{id}`
   - ❌ `GET /api/getUser/{id}`

2. **HTTP Methods**: Use appropriate HTTP verbs
   - `GET`: Retrieve resources
   - `POST`: Create new resources
   - `PUT`: Update entire resources
   - `PATCH`: Partial updates
   - `DELETE`: Remove resources

3. **Status Codes**: Return appropriate HTTP status codes
   - `200 OK`: Successful GET, PUT, PATCH
   - `201 Created`: Successful POST
   - `204 No Content`: Successful DELETE
   - `400 Bad Request`: Invalid input
   - `401 Unauthorized`: Authentication required
   - `403 Forbidden`: Insufficient permissions
   - `404 Not Found`: Resource doesn't exist
   - `500 Internal Server Error`: Server-side error

### API Versioning
```csharp
// URL versioning
[Route("api/v1/users")]
public class UsersV1Controller : ControllerBase { }

// Header versioning
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("api/users")]
public class UsersController : ControllerBase
{
    [MapToApiVersion("1.0")]
    public IActionResult GetV1() { }
    
    [MapToApiVersion("2.0")]
    public IActionResult GetV2() { }
}
```

## Example API Structure

### Controller Example
```csharp
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;

namespace StrikeBESE.Api.Controllers
{
    [ApiController]
    [Route("api/v1/[controller]")]
    public class BitcoinWalletsController : ControllerBase
    {
        private readonly IBitcoinWalletService _walletService;
        private readonly ILogger<BitcoinWalletsController> _logger;

        public BitcoinWalletsController(
            IBitcoinWalletService walletService,
            ILogger<BitcoinWalletsController> logger)
        {
            _walletService = walletService;
            _logger = logger;
        }

        /// <summary>
        /// Get wallet balance
        /// </summary>
        /// <param name="walletId">The wallet identifier</param>
        /// <returns>Wallet balance information</returns>
        [HttpGet("{walletId}/balance")]
        [ProducesResponseType(typeof(WalletBalanceResponse), StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status404NotFound)]
        public async Task<IActionResult> GetBalance(string walletId)
        {
            try
            {
                var balance = await _walletService.GetBalanceAsync(walletId);
                if (balance == null)
                {
                    return NotFound(new { message = $"Wallet {walletId} not found" });
                }
                return Ok(balance);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error retrieving balance for wallet {WalletId}", walletId);
                return StatusCode(500, new { message = "An error occurred while processing your request" });
            }
        }

        /// <summary>
        /// Create a new transaction
        /// </summary>
        [HttpPost("{walletId}/transactions")]
        [ProducesResponseType(typeof(TransactionResponse), StatusCodes.Status201Created)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        public async Task<IActionResult> CreateTransaction(
            string walletId,
            [FromBody] CreateTransactionRequest request)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            try
            {
                var transaction = await _walletService.CreateTransactionAsync(walletId, request);
                return CreatedAtAction(
                    nameof(GetTransaction),
                    new { walletId, transactionId = transaction.Id },
                    transaction);
            }
            catch (InsufficientFundsException ex)
            {
                return BadRequest(new { message = ex.Message });
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error creating transaction for wallet {WalletId}", walletId);
                return StatusCode(500, new { message = "An error occurred while processing your request" });
            }
        }

        [HttpGet("{walletId}/transactions/{transactionId}")]
        public async Task<IActionResult> GetTransaction(string walletId, string transactionId)
        {
            var transaction = await _walletService.GetTransactionAsync(walletId, transactionId);
            if (transaction == null)
            {
                return NotFound();
            }
            return Ok(transaction);
        }
    }
}
```

### Request/Response Models
```csharp
namespace StrikeBESE.Api.Models
{
    public class CreateTransactionRequest
    {
        [Required]
        public string ToAddress { get; set; }
        
        [Required]
        [Range(0.00000001, double.MaxValue)]
        public decimal Amount { get; set; }
        
        public decimal? Fee { get; set; }
        
        public string Note { get; set; }
    }

    public class TransactionResponse
    {
        public string Id { get; set; }
        public string FromWallet { get; set; }
        public string ToAddress { get; set; }
        public decimal Amount { get; set; }
        public decimal Fee { get; set; }
        public string Status { get; set; }
        public DateTime CreatedAt { get; set; }
        public string TransactionHash { get; set; }
    }

    public class WalletBalanceResponse
    {
        public string WalletId { get; set; }
        public decimal Balance { get; set; }
        public decimal PendingBalance { get; set; }
        public string Currency { get; set; }
        public DateTime LastUpdated { get; set; }
    }
}
```

## Service Layer Pattern
```csharp
namespace StrikeBESE.Services
{
    public interface IBitcoinWalletService
    {
        Task<WalletBalanceResponse> GetBalanceAsync(string walletId);
        Task<TransactionResponse> CreateTransactionAsync(string walletId, CreateTransactionRequest request);
        Task<TransactionResponse> GetTransactionAsync(string walletId, string transactionId);
    }

    public class BitcoinWalletService : IBitcoinWalletService
    {
        private readonly IWalletRepository _walletRepository;
        private readonly IBitcoinClient _bitcoinClient;
        private readonly ILogger<BitcoinWalletService> _logger;

        public BitcoinWalletService(
            IWalletRepository walletRepository,
            IBitcoinClient bitcoinClient,
            ILogger<BitcoinWalletService> logger)
        {
            _walletRepository = walletRepository;
            _bitcoinClient = bitcoinClient;
            _logger = logger;
        }

        public async Task<WalletBalanceResponse> GetBalanceAsync(string walletId)
        {
            var wallet = await _walletRepository.GetByIdAsync(walletId);
            if (wallet == null)
            {
                return null;
            }

            var balance = await _bitcoinClient.GetBalanceAsync(wallet.Address);
            
            return new WalletBalanceResponse
            {
                WalletId = walletId,
                Balance = balance.Confirmed,
                PendingBalance = balance.Unconfirmed,
                Currency = "BTC",
                LastUpdated = DateTime.UtcNow
            };
        }

        public async Task<TransactionResponse> CreateTransactionAsync(
            string walletId,
            CreateTransactionRequest request)
        {
            var wallet = await _walletRepository.GetByIdAsync(walletId);
            if (wallet == null)
            {
                throw new NotFoundException($"Wallet {walletId} not found");
            }

            // Validate balance
            var balance = await _bitcoinClient.GetBalanceAsync(wallet.Address);
            if (balance.Confirmed < request.Amount + (request.Fee ?? 0))
            {
                throw new InsufficientFundsException("Insufficient funds for transaction");
            }

            // Create and broadcast transaction
            var txHash = await _bitcoinClient.SendTransactionAsync(
                wallet.PrivateKey,
                request.ToAddress,
                request.Amount,
                request.Fee);

            var transaction = new TransactionResponse
            {
                Id = Guid.NewGuid().ToString(),
                FromWallet = walletId,
                ToAddress = request.ToAddress,
                Amount = request.Amount,
                Fee = request.Fee ?? 0,
                Status = "Pending",
                CreatedAt = DateTime.UtcNow,
                TransactionHash = txHash
            };

            await _walletRepository.SaveTransactionAsync(transaction);

            return transaction;
        }

        public async Task<TransactionResponse> GetTransactionAsync(string walletId, string transactionId)
        {
            return await _walletRepository.GetTransactionAsync(walletId, transactionId);
        }
    }
}
```

## Best Practices

### 1. Input Validation
- Always validate input using data annotations or FluentValidation
- Return clear error messages for validation failures
- Sanitize input to prevent injection attacks

### 2. Error Handling
- Use global exception handling middleware
- Log errors with sufficient context
- Never expose sensitive information in error messages
- Return consistent error response format

### 3. Authentication & Authorization
```csharp
[Authorize(Roles = "Admin,User")]
[HttpGet]
public IActionResult GetSecureData()
{
    // Only authenticated users with Admin or User role can access
}
```

### 4. Rate Limiting
Implement rate limiting to prevent abuse:
```csharp
[RateLimit(Requests = 100, Period = "1m")]
[HttpGet]
public IActionResult GetData()
{
    // Limited to 100 requests per minute
}
```

### 5. Response Caching
```csharp
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any)]
[HttpGet]
public IActionResult GetCachedData()
{
    // Response cached for 60 seconds
}
```

### 6. API Documentation
Use Swagger/OpenAPI for automatic API documentation:
```csharp
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "StrikeBESE API",
        Version = "v1",
        Description = "Bitcoin wallet management API"
    });
});
```

## Performance Considerations

1. **Pagination**: Always paginate large result sets
2. **Async/Await**: Use async operations for I/O-bound work
3. **Connection Pooling**: Reuse database and HTTP connections
4. **Caching**: Cache frequently accessed data
5. **Compression**: Enable response compression
6. **Minimal APIs**: Consider minimal APIs for simple endpoints

## Security Checklist

- ✅ Use HTTPS only
- ✅ Implement authentication and authorization
- ✅ Validate all input
- ✅ Use parameterized queries (prevent SQL injection)
- ✅ Implement rate limiting
- ✅ Keep dependencies updated
- ✅ Log security events
- ✅ Implement CORS properly
- ✅ Use API keys for service-to-service communication
- ✅ Never log sensitive data (passwords, private keys)
