# Testing Guide: xUnit and Moq

## Introduction
This guide covers testing best practices for .NET applications using xUnit and Moq, with a focus on testing Bitcoin APIs and services.

## Getting Started

### Installation
```bash
dotnet add package xUnit
dotnet add package xUnit.runner.visualstudio
dotnet add package Moq
dotnet add package Microsoft.NET.Test.Sdk
dotnet add package FluentAssertions  # Optional but recommended
```

### Test Project Structure
```
StrikeBESE.Tests/
├── Unit/
│   ├── Controllers/
│   │   └── BitcoinWalletsControllerTests.cs
│   ├── Services/
│   │   ├── BitcoinWalletServiceTests.cs
│   │   └── TransactionServiceTests.cs
│   └── Validators/
│       └── TransactionValidatorTests.cs
├── Integration/
│   ├── Api/
│   │   └── BitcoinApiIntegrationTests.cs
│   └── Database/
│       └── WalletRepositoryTests.cs
└── Fixtures/
    └── TestDataFixture.cs
```

## xUnit Basics

### Basic Test Structure
```csharp
using Xunit;

namespace StrikeBESE.Tests.Unit
{
    public class BitcoinAddressValidatorTests
    {
        [Fact]
        public void ValidateAddress_WithValidBitcoinAddress_ReturnsTrue()
        {
            // Arrange
            var validator = new BitcoinAddressValidator();
            var validAddress = "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa";

            // Act
            var result = validator.IsValid(validAddress);

            // Assert
            Assert.True(result);
        }

        [Fact]
        public void ValidateAddress_WithInvalidAddress_ReturnsFalse()
        {
            // Arrange
            var validator = new BitcoinAddressValidator();
            var invalidAddress = "invalid-address";

            // Act
            var result = validator.IsValid(invalidAddress);

            // Assert
            Assert.False(result);
        }
    }
}
```

### Theory Tests (Parameterized Tests)
```csharp
public class BitcoinAmountConverterTests
{
    [Theory]
    [InlineData(1.0, 100000000)]  // 1 BTC = 100,000,000 satoshis
    [InlineData(0.5, 50000000)]
    [InlineData(0.00000001, 1)]   // 1 satoshi
    public void ConvertBtcToSatoshis_WithValidAmount_ReturnsCorrectValue(
        double btc, long expectedSatoshis)
    {
        // Arrange
        var converter = new BitcoinAmountConverter();

        // Act
        var result = converter.BtcToSatoshis(btc);

        // Assert
        Assert.Equal(expectedSatoshis, result);
    }

    [Theory]
    [MemberData(nameof(GetInvalidAmounts))]
    public void ConvertBtcToSatoshis_WithInvalidAmount_ThrowsException(double invalidAmount)
    {
        // Arrange
        var converter = new BitcoinAmountConverter();

        // Act & Assert
        Assert.Throws<ArgumentException>(() => converter.BtcToSatoshis(invalidAmount));
    }

    public static IEnumerable<object[]> GetInvalidAmounts()
    {
        yield return new object[] { -1.0 };
        yield return new object[] { -0.5 };
        yield return new object[] { double.NaN };
    }
}
```

## Moq: Mocking Dependencies

### Basic Mocking
```csharp
using Moq;
using Xunit;

namespace StrikeBESE.Tests.Unit.Services
{
    public class BitcoinWalletServiceTests
    {
        [Fact]
        public async Task GetBalance_WithValidWalletId_ReturnsBalance()
        {
            // Arrange
            var mockRepository = new Mock<IWalletRepository>();
            var mockBitcoinClient = new Mock<IBitcoinClient>();
            var mockLogger = new Mock<ILogger<BitcoinWalletService>>();

            var walletId = "wallet-123";
            var wallet = new Wallet
            {
                Id = walletId,
                Address = "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"
            };

            var balance = new BitcoinBalance
            {
                Confirmed = 1.5m,
                Unconfirmed = 0.2m
            };

            // Setup mocks
            mockRepository
                .Setup(r => r.GetByIdAsync(walletId))
                .ReturnsAsync(wallet);

            mockBitcoinClient
                .Setup(c => c.GetBalanceAsync(wallet.Address))
                .ReturnsAsync(balance);

            var service = new BitcoinWalletService(
                mockRepository.Object,
                mockBitcoinClient.Object,
                mockLogger.Object);

            // Act
            var result = await service.GetBalanceAsync(walletId);

            // Assert
            Assert.NotNull(result);
            Assert.Equal(1.5m, result.Balance);
            Assert.Equal(0.2m, result.PendingBalance);
            Assert.Equal("BTC", result.Currency);

            // Verify that methods were called
            mockRepository.Verify(r => r.GetByIdAsync(walletId), Times.Once);
            mockBitcoinClient.Verify(c => c.GetBalanceAsync(wallet.Address), Times.Once);
        }

        [Fact]
        public async Task GetBalance_WithNonExistentWallet_ReturnsNull()
        {
            // Arrange
            var mockRepository = new Mock<IWalletRepository>();
            var mockBitcoinClient = new Mock<IBitcoinClient>();
            var mockLogger = new Mock<ILogger<BitcoinWalletService>>();

            mockRepository
                .Setup(r => r.GetByIdAsync(It.IsAny<string>()))
                .ReturnsAsync((Wallet)null);

            var service = new BitcoinWalletService(
                mockRepository.Object,
                mockBitcoinClient.Object,
                mockLogger.Object);

            // Act
            var result = await service.GetBalanceAsync("non-existent");

            // Assert
            Assert.Null(result);
            
            // Verify Bitcoin client was never called
            mockBitcoinClient.Verify(
                c => c.GetBalanceAsync(It.IsAny<string>()), 
                Times.Never);
        }
    }
}
```

### Testing Exception Handling
```csharp
public class TransactionServiceTests
{
    [Fact]
    public async Task CreateTransaction_WithInsufficientFunds_ThrowsException()
    {
        // Arrange
        var mockRepository = new Mock<IWalletRepository>();
        var mockBitcoinClient = new Mock<IBitcoinClient>();
        var mockLogger = new Mock<ILogger<BitcoinWalletService>>();

        var walletId = "wallet-123";
        var wallet = new Wallet { Id = walletId, Address = "address-123" };
        var balance = new BitcoinBalance { Confirmed = 0.5m, Unconfirmed = 0 };

        mockRepository.Setup(r => r.GetByIdAsync(walletId)).ReturnsAsync(wallet);
        mockBitcoinClient.Setup(c => c.GetBalanceAsync(wallet.Address)).ReturnsAsync(balance);

        var service = new BitcoinWalletService(
            mockRepository.Object,
            mockBitcoinClient.Object,
            mockLogger.Object);

        var request = new CreateTransactionRequest
        {
            ToAddress = "destination-address",
            Amount = 1.0m,  // More than available balance
            Fee = 0.0001m
        };

        // Act & Assert
        var exception = await Assert.ThrowsAsync<InsufficientFundsException>(
            () => service.CreateTransactionAsync(walletId, request));

        Assert.Equal("Insufficient funds for transaction", exception.Message);
    }
}
```

### Mocking with Callbacks
```csharp
public class WalletServiceTests
{
    [Fact]
    public async Task CreateWallet_SavesWalletToRepository()
    {
        // Arrange
        var mockRepository = new Mock<IWalletRepository>();
        Wallet savedWallet = null;

        mockRepository
            .Setup(r => r.SaveAsync(It.IsAny<Wallet>()))
            .Callback<Wallet>(w => savedWallet = w)
            .ReturnsAsync((Wallet w) => w);

        var service = new WalletService(mockRepository.Object);

        // Act
        var result = await service.CreateWalletAsync("user-123");

        // Assert
        Assert.NotNull(savedWallet);
        Assert.Equal("user-123", savedWallet.UserId);
        Assert.NotNull(savedWallet.Address);
        mockRepository.Verify(r => r.SaveAsync(It.IsAny<Wallet>()), Times.Once);
    }
}
```

## Testing Controllers

```csharp
using Microsoft.AspNetCore.Mvc;
using Moq;
using Xunit;

namespace StrikeBESE.Tests.Unit.Controllers
{
    public class BitcoinWalletsControllerTests
    {
        [Fact]
        public async Task GetBalance_WithValidWalletId_ReturnsOkResult()
        {
            // Arrange
            var mockService = new Mock<IBitcoinWalletService>();
            var mockLogger = new Mock<ILogger<BitcoinWalletsController>>();

            var walletId = "wallet-123";
            var expectedBalance = new WalletBalanceResponse
            {
                WalletId = walletId,
                Balance = 1.5m,
                PendingBalance = 0.2m,
                Currency = "BTC"
            };

            mockService
                .Setup(s => s.GetBalanceAsync(walletId))
                .ReturnsAsync(expectedBalance);

            var controller = new BitcoinWalletsController(mockService.Object, mockLogger.Object);

            // Act
            var result = await controller.GetBalance(walletId);

            // Assert
            var okResult = Assert.IsType<OkObjectResult>(result);
            var balance = Assert.IsType<WalletBalanceResponse>(okResult.Value);
            Assert.Equal(1.5m, balance.Balance);
            Assert.Equal(0.2m, balance.PendingBalance);
        }

        [Fact]
        public async Task GetBalance_WithNonExistentWallet_ReturnsNotFound()
        {
            // Arrange
            var mockService = new Mock<IBitcoinWalletService>();
            var mockLogger = new Mock<ILogger<BitcoinWalletsController>>();

            mockService
                .Setup(s => s.GetBalanceAsync(It.IsAny<string>()))
                .ReturnsAsync((WalletBalanceResponse)null);

            var controller = new BitcoinWalletsController(mockService.Object, mockLogger.Object);

            // Act
            var result = await controller.GetBalance("non-existent");

            // Assert
            var notFoundResult = Assert.IsType<NotFoundObjectResult>(result);
            Assert.NotNull(notFoundResult.Value);
        }

        [Fact]
        public async Task CreateTransaction_WithValidRequest_ReturnsCreatedResult()
        {
            // Arrange
            var mockService = new Mock<IBitcoinWalletService>();
            var mockLogger = new Mock<ILogger<BitcoinWalletsController>>();

            var walletId = "wallet-123";
            var request = new CreateTransactionRequest
            {
                ToAddress = "destination-address",
                Amount = 0.5m,
                Fee = 0.0001m
            };

            var expectedTransaction = new TransactionResponse
            {
                Id = "tx-123",
                FromWallet = walletId,
                ToAddress = request.ToAddress,
                Amount = request.Amount,
                Status = "Pending"
            };

            mockService
                .Setup(s => s.CreateTransactionAsync(walletId, request))
                .ReturnsAsync(expectedTransaction);

            var controller = new BitcoinWalletsController(mockService.Object, mockLogger.Object);

            // Act
            var result = await controller.CreateTransaction(walletId, request);

            // Assert
            var createdResult = Assert.IsType<CreatedAtActionResult>(result);
            var transaction = Assert.IsType<TransactionResponse>(createdResult.Value);
            Assert.Equal("tx-123", transaction.Id);
            Assert.Equal("Pending", transaction.Status);
        }

        [Fact]
        public async Task CreateTransaction_WithInsufficientFunds_ReturnsBadRequest()
        {
            // Arrange
            var mockService = new Mock<IBitcoinWalletService>();
            var mockLogger = new Mock<ILogger<BitcoinWalletsController>>();

            mockService
                .Setup(s => s.CreateTransactionAsync(It.IsAny<string>(), It.IsAny<CreateTransactionRequest>()))
                .ThrowsAsync(new InsufficientFundsException("Insufficient funds"));

            var controller = new BitcoinWalletsController(mockService.Object, mockLogger.Object);
            var request = new CreateTransactionRequest
            {
                ToAddress = "address",
                Amount = 100m
            };

            // Act
            var result = await controller.CreateTransaction("wallet-123", request);

            // Assert
            var badRequestResult = Assert.IsType<BadRequestObjectResult>(result);
            Assert.NotNull(badRequestResult.Value);
        }
    }
}
```

## Test Fixtures and Shared Context

```csharp
namespace StrikeBESE.Tests.Fixtures
{
    // Fixture for sharing setup across tests
    public class BitcoinTestFixture : IDisposable
    {
        public Network Network { get; }
        public List<WalletInfo> TestWallets { get; }

        public BitcoinTestFixture()
        {
            Network = Network.TestNet;
            TestWallets = new List<WalletInfo>();

            // Generate test wallets
            var walletManager = new BitcoinWalletManager(useMainnet: false);
            for (int i = 0; i < 3; i++)
            {
                TestWallets.Add(walletManager.GenerateWallet());
            }
        }

        public void Dispose()
        {
            // Cleanup if needed
        }
    }

    // Using the fixture
    public class BitcoinWalletTests : IClassFixture<BitcoinTestFixture>
    {
        private readonly BitcoinTestFixture _fixture;

        public BitcoinWalletTests(BitcoinTestFixture fixture)
        {
            _fixture = fixture;
        }

        [Fact]
        public void TestWallets_AreOnTestNet()
        {
            foreach (var wallet in _fixture.TestWallets)
            {
                Assert.Equal("TestNet", wallet.Network);
            }
        }
    }
}
```

## Integration Tests

```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using Xunit;

namespace StrikeBESE.Tests.Integration
{
    public class BitcoinApiIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
    {
        private readonly WebApplicationFactory<Program> _factory;
        private readonly HttpClient _client;

        public BitcoinApiIntegrationTests(WebApplicationFactory<Program> factory)
        {
            _factory = factory;
            _client = factory.CreateClient();
        }

        [Fact]
        public async Task GetWalletBalance_ReturnsSuccessStatusCode()
        {
            // Arrange
            var walletId = "test-wallet-123";

            // Act
            var response = await _client.GetAsync($"/api/v1/bitcoinwallets/{walletId}/balance");

            // Assert
            response.EnsureSuccessStatusCode();
            var content = await response.Content.ReadAsStringAsync();
            Assert.NotEmpty(content);
        }
    }
}
```

## FluentAssertions (Optional Enhancement)

```csharp
using FluentAssertions;
using Xunit;

public class TransactionTests
{
    [Fact]
    public void Transaction_ShouldHaveCorrectProperties()
    {
        // Arrange
        var transaction = new TransactionResponse
        {
            Id = "tx-123",
            Amount = 1.5m,
            Status = "Pending"
        };

        // Assert
        transaction.Should().NotBeNull();
        transaction.Id.Should().Be("tx-123");
        transaction.Amount.Should().BeGreaterThan(0);
        transaction.Status.Should().BeOneOf("Pending", "Confirmed", "Failed");
    }

    [Fact]
    public void WalletList_ShouldContainExpectedWallets()
    {
        // Arrange
        var wallets = new List<Wallet>
        {
            new Wallet { Id = "1", Balance = 1.0m },
            new Wallet { Id = "2", Balance = 2.0m },
            new Wallet { Id = "3", Balance = 3.0m }
        };

        // Assert
        wallets.Should().HaveCount(3);
        wallets.Should().Contain(w => w.Id == "2");
        wallets.Should().OnlyContain(w => w.Balance > 0);
        wallets.Sum(w => w.Balance).Should().Be(6.0m);
    }
}
```

## Test Organization Best Practices

### 1. AAA Pattern (Arrange, Act, Assert)
Always structure tests with clear sections:
```csharp
[Fact]
public void TestMethod()
{
    // Arrange - Setup test data and dependencies
    var service = new Service();
    
    // Act - Execute the method under test
    var result = service.DoSomething();
    
    // Assert - Verify the results
    Assert.Equal(expected, result);
}
```

### 2. Test Naming Convention
Use descriptive names that indicate:
- Method being tested
- Scenario being tested
- Expected outcome

```csharp
[Fact]
public void MethodName_WithCondition_ExpectedBehavior()
{
    // Test implementation
}
```

### 3. One Assert Per Test (Generally)
Focus each test on a single behavior:
```csharp
// Good - Single concern
[Fact]
public void CreateWallet_ReturnsWalletWithAddress()
{
    var wallet = _service.CreateWallet();
    Assert.NotNull(wallet.Address);
}

// Good - Single concern
[Fact]
public void CreateWallet_GeneratesUniqueId()
{
    var wallet = _service.CreateWallet();
    Assert.NotEqual(Guid.Empty.ToString(), wallet.Id);
}
```

## Running Tests

```bash
# Run all tests
dotnet test

# Run tests with detailed output
dotnet test --verbosity detailed

# Run specific test
dotnet test --filter "FullyQualifiedName~BitcoinWalletServiceTests"

# Run tests in a specific category
dotnet test --filter "Category=Unit"

# Generate code coverage
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
```

## Code Coverage

```bash
# Install coverage tools
dotnet tool install --global dotnet-reportgenerator-globaltool

# Run tests with coverage
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura

# Generate HTML report
reportgenerator -reports:coverage.cobertura.xml -targetdir:coveragereport

# Open report
# Open coveragereport/index.html in browser
```

## Mocking Best Practices

### ✅ Do:
- Mock external dependencies (databases, APIs, file systems)
- Use strict mocks when order matters
- Verify important interactions
- Keep mocks simple and focused

### ❌ Don't:
- Mock what you own - test concrete implementations when possible
- Create complex mock setups - may indicate design issues
- Mock everything - some dependencies are simple enough to use directly
- Forget to verify important interactions

## Common Testing Patterns

### Testing Async Methods
```csharp
[Fact]
public async Task AsyncMethod_ReturnsExpectedResult()
{
    var result = await _service.GetDataAsync();
    Assert.NotNull(result);
}
```

### Testing for Exceptions
```csharp
[Fact]
public void Method_WithInvalidInput_ThrowsException()
{
    var exception = Assert.Throws<ArgumentException>(
        () => _service.ProcessData(null));
    Assert.Equal("Data cannot be null", exception.Message);
}
```

### Testing with Multiple Scenarios
```csharp
[Theory]
[InlineData("valid-address", true)]
[InlineData("invalid", false)]
[InlineData("", false)]
[InlineData(null, false)]
public void ValidateAddress_WithVariousInputs_ReturnsExpectedResult(
    string address, bool expected)
{
    var result = _validator.IsValid(address);
    Assert.Equal(expected, result);
}
```

## Resources

- **xUnit Documentation**: https://xunit.net/
- **Moq Documentation**: https://github.com/moq/moq4
- **FluentAssertions**: https://fluentassertions.com/
- **Test Patterns**: http://xunitpatterns.com/

## Testing Checklist

- ✅ Test happy path scenarios
- ✅ Test edge cases and boundary conditions
- ✅ Test error conditions and exceptions
- ✅ Test null and empty inputs
- ✅ Verify all important interactions
- ✅ Use descriptive test names
- ✅ Follow AAA pattern
- ✅ Keep tests independent
- ✅ Mock external dependencies
- ✅ Aim for high code coverage (but focus on quality over quantity)
