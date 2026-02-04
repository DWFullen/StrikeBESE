# Bitcoin Integration Guide

## Introduction
This guide covers Bitcoin integration fundamentals for backend developers, including wallet management, transaction handling, and best practices for working with Bitcoin in .NET applications.

## Bitcoin Basics

### Key Concepts

1. **Blockchain**: A distributed ledger of all Bitcoin transactions
2. **Address**: A unique identifier (public key) where Bitcoin can be sent
3. **Private Key**: Secret key used to sign transactions and prove ownership
4. **Transaction**: A transfer of Bitcoin from one address to another
5. **Confirmation**: Number of blocks mined after a transaction
6. **Satoshi**: The smallest unit of Bitcoin (0.00000001 BTC)

### Address Types

- **P2PKH (Legacy)**: Starts with `1` (e.g., `1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa`)
- **P2SH (Script Hash)**: Starts with `3` (e.g., `3J98t1WpEZ73CNmYviecrnyiWrnqRhWNLy`)
- **Bech32 (SegWit)**: Starts with `bc1` (e.g., `bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh`)

## Bitcoin Libraries for .NET

### NBitcoin
The most popular Bitcoin library for .NET:

```bash
dotnet add package NBitcoin
```

### Basic Usage

```csharp
using NBitcoin;

namespace StrikeBESE.Bitcoin
{
    public class BitcoinWalletManager
    {
        private readonly Network _network;

        public BitcoinWalletManager(bool useMainnet = false)
        {
            // Use testnet for development, mainnet for production
            _network = useMainnet ? Network.Main : Network.TestNet;
        }

        /// <summary>
        /// Generate a new Bitcoin wallet
        /// </summary>
        public WalletInfo GenerateWallet()
        {
            // Generate a new private key
            var privateKey = new Key();
            
            // Derive the public key
            var publicKey = privateKey.PubKey;
            
            // Get the Bitcoin address
            var address = publicKey.GetAddress(ScriptPubKeyType.Legacy, _network);

            return new WalletInfo
            {
                PrivateKey = privateKey.GetWif(_network).ToString(),
                PublicKey = publicKey.ToHex(),
                Address = address.ToString(),
                Network = _network.Name
            };
        }

        /// <summary>
        /// Generate a wallet from a mnemonic phrase (BIP39)
        /// </summary>
        public WalletInfo GenerateWalletFromMnemonic(string mnemonic = null)
        {
            // Generate or use provided mnemonic
            var mnemonicObj = mnemonic == null 
                ? new Mnemonic(Wordlist.English, WordCount.Twelve)
                : new Mnemonic(mnemonic, Wordlist.English);

            // Derive the master key
            var hdRoot = mnemonicObj.DeriveExtKey();

            // Use BIP44 path for Bitcoin: m/44'/0'/0'/0/0
            var keyPath = new KeyPath("m/44'/0'/0'/0/0");
            var derivedKey = hdRoot.Derive(keyPath).PrivateKey;

            var address = derivedKey.PubKey.GetAddress(ScriptPubKeyType.Legacy, _network);

            return new WalletInfo
            {
                Mnemonic = mnemonicObj.ToString(),
                PrivateKey = derivedKey.GetWif(_network).ToString(),
                PublicKey = derivedKey.PubKey.ToHex(),
                Address = address.ToString(),
                Network = _network.Name
            };
        }

        /// <summary>
        /// Validate a Bitcoin address
        /// </summary>
        public bool IsValidAddress(string address)
        {
            try
            {
                BitcoinAddress.Create(address, _network);
                return true;
            }
            catch
            {
                return false;
            }
        }

        /// <summary>
        /// Import wallet from private key
        /// </summary>
        public WalletInfo ImportWallet(string privateKeyWif)
        {
            try
            {
                var privateKey = Key.Parse(privateKeyWif, _network);
                var address = privateKey.PubKey.GetAddress(ScriptPubKeyType.Legacy, _network);

                return new WalletInfo
                {
                    PrivateKey = privateKeyWif,
                    PublicKey = privateKey.PubKey.ToHex(),
                    Address = address.ToString(),
                    Network = _network.Name
                };
            }
            catch (Exception ex)
            {
                throw new ArgumentException("Invalid private key", ex);
            }
        }
    }

    public class WalletInfo
    {
        public string Mnemonic { get; set; }
        public string PrivateKey { get; set; }
        public string PublicKey { get; set; }
        public string Address { get; set; }
        public string Network { get; set; }
    }
}
```

## Creating Transactions

```csharp
using NBitcoin;
using NBitcoin.RPC;

namespace StrikeBESE.Bitcoin
{
    public class BitcoinTransactionBuilder
    {
        private readonly Network _network;
        private readonly RPCClient _rpcClient;

        public BitcoinTransactionBuilder(Network network, RPCClient rpcClient = null)
        {
            _network = network;
            _rpcClient = rpcClient;
        }

        /// <summary>
        /// Create a simple transaction
        /// </summary>
        public async Task<Transaction> CreateTransaction(
            string fromPrivateKeyWif,
            string toAddress,
            decimal amountBtc,
            decimal feeBtc = 0.0001m)
        {
            // Parse private key
            var privateKey = Key.Parse(fromPrivateKeyWif, _network);
            var fromAddress = privateKey.PubKey.GetAddress(ScriptPubKeyType.Legacy, _network);

            // Get unspent transaction outputs (UTXOs)
            var utxos = await GetUnspentOutputsAsync(fromAddress.ToString());

            if (utxos == null || !utxos.Any())
            {
                throw new InvalidOperationException("No unspent outputs available");
            }

            // Build transaction
            var txBuilder = _network.CreateTransactionBuilder();
            
            var tx = txBuilder
                .AddCoins(utxos)
                .AddKeys(privateKey)
                .Send(BitcoinAddress.Create(toAddress, _network), Money.Coins(amountBtc))
                .SendFees(Money.Coins(feeBtc))
                .SetChange(fromAddress)
                .BuildTransaction(sign: true);

            // Verify transaction
            if (!txBuilder.Verify(tx, out var errors))
            {
                throw new InvalidOperationException($"Transaction verification failed: {string.Join(", ", errors)}");
            }

            return tx;
        }

        /// <summary>
        /// Broadcast transaction to the network
        /// </summary>
        public async Task<string> BroadcastTransactionAsync(Transaction transaction)
        {
            if (_rpcClient == null)
            {
                throw new InvalidOperationException("RPC client not configured");
            }

            // Broadcast the transaction
            await _rpcClient.SendRawTransactionAsync(transaction);
            
            return transaction.GetHash().ToString();
        }

        /// <summary>
        /// Get unspent outputs for an address
        /// </summary>
        private async Task<ICoin[]> GetUnspentOutputsAsync(string address)
        {
            if (_rpcClient == null)
            {
                // In production, you'd use a Bitcoin node or API service
                throw new NotImplementedException("UTXO lookup requires RPC client or external API");
            }

            var unspentOutputs = await _rpcClient.ListUnspentAsync();
            
            return unspentOutputs
                .Where(u => u.Address.ToString() == address)
                .Select(u => u.AsCoin())
                .ToArray();
        }
    }
}
```

## Bitcoin RPC Integration

```csharp
using NBitcoin;
using NBitcoin.RPC;

namespace StrikeBESE.Bitcoin
{
    public class BitcoinRpcService
    {
        private readonly RPCClient _rpcClient;
        private readonly Network _network;

        public BitcoinRpcService(string rpcUrl, string rpcUser, string rpcPassword, Network network)
        {
            _network = network;
            _rpcClient = new RPCClient(
                new NetworkCredential(rpcUser, rpcPassword),
                new Uri(rpcUrl),
                network);
        }

        /// <summary>
        /// Get blockchain info
        /// </summary>
        public async Task<BlockchainInfo> GetBlockchainInfoAsync()
        {
            var info = await _rpcClient.GetBlockchainInfoAsync();
            
            return new BlockchainInfo
            {
                Chain = info.Chain.ToString(),
                Blocks = info.Blocks,
                Headers = info.Headers,
                BestBlockHash = info.BestBlockHash.ToString(),
                Difficulty = info.Difficulty,
                VerificationProgress = info.VerificationProgress
            };
        }

        /// <summary>
        /// Get balance for an address
        /// </summary>
        public async Task<decimal> GetBalanceAsync(string address)
        {
            var unspent = await _rpcClient.ListUnspentAsync(0, 999999, 
                new[] { BitcoinAddress.Create(address, _network) });
            
            return unspent.Sum(u => u.Amount.ToUnit(MoneyUnit.BTC));
        }

        /// <summary>
        /// Get transaction details
        /// </summary>
        public async Task<TransactionDetails> GetTransactionAsync(string txId)
        {
            var txHash = uint256.Parse(txId);
            var tx = await _rpcClient.GetRawTransactionAsync(txHash, true);

            return new TransactionDetails
            {
                TxId = tx.Transaction.GetHash().ToString(),
                Confirmations = tx.Confirmations,
                BlockHash = tx.BlockHash?.ToString(),
                BlockTime = tx.BlockTime,
                Inputs = tx.Transaction.Inputs.Count,
                Outputs = tx.Transaction.Outputs.Count,
                TotalOutput = tx.Transaction.TotalOut.ToUnit(MoneyUnit.BTC)
            };
        }

        /// <summary>
        /// Monitor mempool for new transactions
        /// </summary>
        public async Task<int> GetMempoolCountAsync()
        {
            var mempool = await _rpcClient.GetRawMempoolAsync();
            return mempool.Length;
        }
    }

    public class BlockchainInfo
    {
        public string Chain { get; set; }
        public int Blocks { get; set; }
        public int Headers { get; set; }
        public string BestBlockHash { get; set; }
        public double Difficulty { get; set; }
        public double VerificationProgress { get; set; }
    }

    public class TransactionDetails
    {
        public string TxId { get; set; }
        public int Confirmations { get; set; }
        public string BlockHash { get; set; }
        public DateTimeOffset? BlockTime { get; set; }
        public int Inputs { get; set; }
        public int Outputs { get; set; }
        public decimal TotalOutput { get; set; }
    }
}
```

## Security Best Practices

### 1. Private Key Management
```csharp
// ❌ NEVER store private keys in plain text
public class BadExample
{
    public string PrivateKey { get; set; } // DON'T DO THIS
}

// ✅ Use encryption for private keys
public class SecureWalletStorage
{
    private readonly IDataProtector _protector;

    public SecureWalletStorage(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("WalletProtection");
    }

    public string EncryptPrivateKey(string privateKey)
    {
        return _protector.Protect(privateKey);
    }

    public string DecryptPrivateKey(string encryptedKey)
    {
        return _protector.Unprotect(encryptedKey);
    }
}
```

### 2. Transaction Validation
```csharp
public class TransactionValidator
{
    /// <summary>
    /// Validate transaction before broadcasting
    /// </summary>
    public ValidationResult ValidateTransaction(Transaction tx, decimal maxAmount)
    {
        var result = new ValidationResult { IsValid = true };

        // Check transaction size
        if (tx.GetSerializedSize() > 100000)
        {
            result.IsValid = false;
            result.Errors.Add("Transaction too large");
        }

        // Check output amounts
        var totalOutput = tx.TotalOut.ToUnit(MoneyUnit.BTC);
        if (totalOutput > maxAmount)
        {
            result.IsValid = false;
            result.Errors.Add($"Transaction amount exceeds maximum: {maxAmount} BTC");
        }

        // Check for dust outputs
        foreach (var output in tx.Outputs)
        {
            if (output.Value < Money.Coins(0.00001m))
            {
                result.Errors.Add("Output amount too small (dust)");
            }
        }

        return result;
    }
}

public class ValidationResult
{
    public bool IsValid { get; set; }
    public List<string> Errors { get; set; } = new List<string>();
}
```

### 3. Rate Limiting and Monitoring
```csharp
public class BitcoinApiRateLimiter
{
    private readonly Dictionary<string, RateLimitInfo> _rateLimits = new();
    private readonly int _maxRequestsPerMinute = 10;

    public bool CanMakeRequest(string userId)
    {
        if (!_rateLimits.ContainsKey(userId))
        {
            _rateLimits[userId] = new RateLimitInfo();
        }

        var info = _rateLimits[userId];
        var now = DateTime.UtcNow;

        // Reset if minute has passed
        if ((now - info.WindowStart).TotalMinutes >= 1)
        {
            info.RequestCount = 0;
            info.WindowStart = now;
        }

        if (info.RequestCount >= _maxRequestsPerMinute)
        {
            return false;
        }

        info.RequestCount++;
        return true;
    }
}

public class RateLimitInfo
{
    public int RequestCount { get; set; }
    public DateTime WindowStart { get; set; } = DateTime.UtcNow;
}
```

## Testing Bitcoin Integration

### Unit Test Example
See TESTING_GUIDE.md for comprehensive examples using xUnit and Moq.

## Best Practices Checklist

- ✅ **Never** store private keys in plain text
- ✅ Use testnet for development and testing
- ✅ Validate all addresses before sending transactions
- ✅ Implement proper error handling for network failures
- ✅ Use appropriate transaction fees based on network conditions
- ✅ Wait for sufficient confirmations (typically 6 for high-value transactions)
- ✅ Implement rate limiting for API calls
- ✅ Log all transaction attempts with sufficient detail
- ✅ Monitor for unusual activity (large transactions, multiple failed attempts)
- ✅ Keep NBitcoin and other dependencies updated
- ✅ Use multi-signature wallets for high-value storage
- ✅ Implement backup and recovery procedures
- ✅ Use hardware wallets or HSMs for production private keys

## Resources

- **NBitcoin Documentation**: https://github.com/MetacoSA/NBitcoin
- **Bitcoin Developer Guide**: https://developer.bitcoin.org/devguide/
- **Bitcoin RPC API**: https://developer.bitcoin.org/reference/rpc/
- **BIP39 (Mnemonic)**: https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki
- **BIP44 (HD Wallets)**: https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki
- **Bitcoin Testnet Faucet**: https://testnet-faucet.mempool.co/

## Common Pitfalls to Avoid

1. **Using Mainnet for Testing**: Always use testnet during development
2. **Not Checking Confirmations**: Wait for adequate confirmations before considering transactions final
3. **Ignoring Transaction Fees**: Transactions with low fees may never confirm
4. **Poor Key Management**: Compromised private keys = lost funds
5. **Not Handling Reorgs**: Be aware that recent blocks can be reorganized
6. **Insufficient Error Handling**: Network issues are common, handle them gracefully
7. **Hardcoded Fee Rates**: Fee rates change based on network congestion
