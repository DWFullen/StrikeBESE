---
title: StrikeBESE Bitcoin Hotwallet — System Specification
version: 1.0
date_created: 2026-03-12
last_updated: 2026-03-12
owner: DWFullen
tags: architecture, design, infrastructure, app, bitcoin, dotnet, gcp
---

# Introduction

StrikeBESE is a learning-focused Bitcoin testnet hotwallet application built with Blazor (.NET) and hosted on Google Cloud Platform (GCP). The project provides hands-on experience with HD wallet management, raw Bitcoin transaction construction and broadcasting, cloud-native deployment, and production-grade secret management — all against the Bitcoin testnet, where no real funds are at risk.

This specification translates the product requirements captured in `documents/prd.md` into a structured, machine-readable set of requirements, interfaces, data contracts, and acceptance criteria that can be consumed by developers, code reviewers, and AI code-generation tooling.

## 1. Purpose & Scope

### 1.1 Purpose

Define the technical requirements, constraints, interfaces, and acceptance criteria for the StrikeBESE application so that any engineer (or AI assistant) can implement, extend, or review the system with a complete and unambiguous understanding of what must be built.

### 1.2 Scope

This specification covers:

- Blazor (.NET) front-end and back-end service layer
- Bitcoin testnet integration (wallet management, balance/history queries, transaction construction and broadcast, fee estimation)
- Google Cloud Platform hosting (Cloud Run, Secret Manager, Artifact Registry)
- CI/CD pipeline (GitHub Actions)
- Security controls for private key and mnemonic handling
- Application health and observability

### 1.3 Out of Scope

- Bitcoin mainnet transactions
- Lightning Network payment channels
- Multi-signature wallets
- Fiat currency conversion or exchange rates
- Public-facing production service with multiple users

### 1.4 Intended Audience

- Solo developer (learner) building and operating the application
- Code reviewers or hiring engineers assessing technical quality
- AI code-generation agents implementing features described herein

### 1.5 Assumptions

- The Bitcoin testnet (testnet3 or testnet4) is used exclusively.
- Only one wallet is active at a time per application instance.
- The application is single-user; no authentication or authorisation layer is required beyond GCP IAM for infrastructure.
- NBitcoin is the canonical .NET library for all Bitcoin cryptographic operations.

---

## 2. Definitions

| Term | Definition |
|---|---|
| **tBTC** | Testnet Bitcoin; the native currency of the Bitcoin testnet. Has no real monetary value. |
| **UTXO** | Unspent Transaction Output. The fundamental unit of Bitcoin's accounting model. |
| **TXID** | Transaction Identifier. A 32-byte hash uniquely identifying a Bitcoin transaction. |
| **sat / satoshi** | The smallest unit of Bitcoin (1 BTC = 100,000,000 satoshis). |
| **sat/vByte** | Fee rate unit: satoshis per virtual byte of transaction size. |
| **BIP-32** | Bitcoin Improvement Proposal 32 — Hierarchical Deterministic (HD) wallet derivation standard. |
| **BIP-39** | Bitcoin Improvement Proposal 39 — Mnemonic seed phrase standard (12 or 24 words). |
| **BIP-44** | Bitcoin Improvement Proposal 44 — Multi-account HD wallet derivation path standard. |
| **P2PKH** | Pay-to-Public-Key-Hash. Legacy address type; testnet addresses begin with `m` or `n`. |
| **P2WPKH** | Pay-to-Witness-Public-Key-Hash. Native SegWit address type; testnet addresses begin with `tb1`. |
| **HD Wallet** | Hierarchical Deterministic wallet derived from a single master seed using BIP-32. |
| **Mnemonic** | A human-readable BIP-39 seed phrase (12 or 24 English words) that encodes the wallet master key. |
| **WIF** | Wallet Import Format. A Base58Check-encoded representation of a Bitcoin private key. |
| **Esplora** | Blockstream's open-source Bitcoin block explorer and REST API server. |
| **BlockCypher** | Commercial Bitcoin API provider offering testnet endpoints. |
| **GCP** | Google Cloud Platform. The cloud infrastructure provider used by this project. |
| **Cloud Run** | GCP's managed serverless container execution environment. |
| **Secret Manager** | GCP service for storing, managing, and accessing secrets at runtime. |
| **Artifact Registry** | GCP service for storing Docker container images. |
| **CI/CD** | Continuous Integration / Continuous Delivery. Automated pipeline for building, testing, and deploying code. |
| **Blazor** | Microsoft's .NET framework for building web UIs with C#. Supports both Server and WebAssembly hosting models. |
| **RBF** | Replace-by-Fee. A Bitcoin protocol feature allowing an unconfirmed transaction to be replaced with a higher-fee version. |
| **CPFP** | Child-pays-for-parent. A technique to accelerate confirmation of an unconfirmed transaction by spending one of its outputs with a high-fee transaction. |
| **Dust limit** | The minimum amount (in satoshis) that a Bitcoin output must carry to be considered economically viable (typically 546 sat for P2PKH). |

---

## 3. Requirements, Constraints & Guidelines

### Functional Requirements

- **REQ-001**: The application **must** generate a new BIP-39 mnemonic (12 or 24 English words) and derive a BIP-32 HD wallet (private key, public key, testnet address) using the BIP-44 derivation path `m/44'/1'/0'/0/0`.
- **REQ-002**: The application **must** support importing an existing wallet by accepting a valid BIP-39 mnemonic and re-deriving the corresponding key pair and testnet address.
- **REQ-003**: The application **must** display the active wallet's confirmed and unconfirmed balance in both satoshis and tBTC.
- **REQ-004**: The application **must** retrieve and display transaction history for the active wallet address, showing TXID, direction, amount (tBTC), confirmation count, and timestamp.
- **REQ-005**: The application **must** allow the user to send tBTC by providing a recipient testnet address and an amount, then constructing, signing, and broadcasting a raw Bitcoin testnet transaction.
- **REQ-006**: The application **must** display a QR code and copyable text of the active wallet's receiving address on a dedicated Receive page.
- **REQ-007**: The application **must** fetch fee rate recommendations (sat/vByte) and compute an estimated total fee before the user confirms a send.
- **REQ-008**: The application **must** expose a `GET /health` endpoint that returns the application's status and testnet API connectivity.
- **REQ-009**: The application **must** support a CI/CD pipeline that builds, tests, containerises, and deploys the application to GCP Cloud Run on every push to the `main` branch.

### Security Requirements

- **SEC-001**: Private keys and mnemonic seed phrases **must never** be written to application logs, the browser console, network traffic, or any plaintext storage medium.
- **SEC-002**: Private keys and mnemonic seed phrases **must** be stored exclusively in GCP Secret Manager; they **must not** be stored in `appsettings.json`, environment variables that are logged, or a database.
- **SEC-003**: The GCP service account used by the application **must** be granted only the `roles/secretmanager.secretAccessor` role and no broader IAM permissions.
- **SEC-004**: The application **must** validate that a recipient address is a valid Bitcoin **testnet** address before constructing a transaction (prefix check: `m`, `n`, or `tb1`); mainnet addresses **must** be rejected.
- **SEC-005**: The application **must not** bake secrets into the container image; all secrets **must** be injected at runtime via Secret Manager or environment variable references.
- **SEC-006**: Sensitive UI elements (private key, mnemonic) **must** be hidden by default and only revealed on explicit user interaction ("click to reveal"). The mnemonic **must** be displayed only once after wallet creation, with a backup warning.

### Constraints

- **CON-001**: The application **must** target the Bitcoin testnet only; mainnet is out of scope for this version.
- **CON-002**: The Blazor front-end and service layer **must** be implemented in C# on .NET 8 or later.
- **CON-003**: All Bitcoin cryptographic operations (key generation, address derivation, transaction construction and signing) **must** use the NBitcoin library; custom cryptographic implementations are prohibited.
- **CON-004**: The application is single-user; no multi-tenancy, user registration, or session-based authentication is required.
- **CON-005**: The application **must** respect rate limits of the chosen public testnet API (e.g., BlockCypher free tier); caching **must** be applied to balance and transaction history responses.
- **CON-006**: The CI/CD pipeline end-to-end execution time **must not** exceed 5 minutes.
- **CON-007**: Automated test coverage of business logic **must** be ≥ 80%.

### Guidelines

- **GUD-001**: Use `async`/`await` throughout all I/O-bound operations to keep the Blazor UI responsive.
- **GUD-002**: Implement graceful degradation for all external API dependencies (testnet API, fee API): display last-known cached data with a "stale" indicator and timestamp when the API is unreachable.
- **GUD-003**: Use structured logging (e.g., `ILogger<T>` with JSON output) and ensure log fields never contain key material.
- **GUD-004**: Apply a configurable TTL-based in-memory cache for balance and transaction history responses to reduce API call frequency.
- **GUD-005**: Use the `QRCoder` (or equivalent) .NET library for QR code generation; do not shell out or call an external QR service.
- **GUD-006**: Provide a confirmation dialog before broadcasting any transaction that summarises the recipient address, amount, and estimated fee.
- **GUD-007**: Use SOLID principles, dependency injection (via `Microsoft.Extensions.DependencyInjection`), and interface-based design for all service classes to facilitate unit testing.
- **GUD-008**: Expose all configuration values (testnet API base URL, cache TTL, fee tier targets) via `IOptions<T>`-bound configuration classes rather than magic strings.

### Patterns

- **PAT-001**: Repository / Service pattern — abstract all Bitcoin API interactions behind an `IBitcoinApiService` interface; inject via DI.
- **PAT-002**: Options pattern — bind all external configuration to strongly typed POCO classes registered with `IOptions<T>`.
- **PAT-003**: Result / error type — use a discriminated union or `Result<T, TError>` type for all operations that can fail (broadcast, import, API calls) rather than throwing exceptions for expected error conditions.
- **PAT-004**: Feature flags — decouple code deployment from feature visibility where feasible (e.g., an "advanced fee" toggle).

---

## 4. Interfaces & Data Contracts

### 4.1 Application Pages / Routes

| Route | Page | Description |
|---|---|---|
| `/` | Dashboard | Active wallet address, confirmed/unconfirmed balance, recent transactions summary |
| `/create` | Create Wallet | Generate or import a BIP-39 mnemonic and derive a wallet |
| `/receive` | Receive | Display receiving address as text and QR code |
| `/send` | Send | Form to send tBTC; fee selection; confirmation dialog |
| `/history` | Transaction History | Paginated list of all past transactions |
| `/health` | Health (HTTP API) | JSON health status endpoint |

### 4.2 `GET /health` Response Contract

```json
{
  "status": "healthy | unhealthy",
  "testnet": "connected | disconnected",
  "timestamp": "2026-03-12T21:38:17Z"
}
```

- Returns HTTP **200** when `status` is `"healthy"`.
- Returns HTTP **503** when `status` is `"unhealthy"` or `testnet` is `"disconnected"`.

### 4.3 Wallet Data Model

```csharp
public record WalletInfo
{
    public string Mnemonic { get; init; }       // BIP-39 mnemonic (sensitive)
    public string PrivateKeyWif { get; init; }  // WIF-encoded private key (sensitive)
    public string PublicKeyHex { get; init; }   // Compressed public key (hex)
    public string Address { get; init; }        // Testnet Bitcoin address
    public string Network { get; init; }        // "TestNet" | "TestNet4"
    public string DerivationPath { get; init; } // e.g., "m/44'/1'/0'/0/0"
}
```

### 4.4 Balance Model

```csharp
public record WalletBalance
{
    public long ConfirmedSatoshis { get; init; }
    public long UnconfirmedSatoshis { get; init; }
    public decimal ConfirmedTbtc => ConfirmedSatoshis / 100_000_000m;
    public decimal UnconfirmedTbtc => UnconfirmedSatoshis / 100_000_000m;
    public DateTimeOffset FetchedAt { get; init; }
    public bool IsStale { get; init; }
}
```

### 4.5 Transaction History Item Model

```csharp
public record TransactionSummary
{
    public string Txid { get; init; }
    public TransactionDirection Direction { get; init; } // Sent | Received
    public long AmountSatoshis { get; init; }
    public int Confirmations { get; init; }
    public DateTimeOffset Timestamp { get; init; }
    public string ExplorerUrl { get; init; } // Link to mempool.space testnet
}

public enum TransactionDirection { Sent, Received }
```

### 4.6 Fee Estimate Model

```csharp
public record FeeEstimate
{
    public int SlowSatPerVbyte { get; init; }
    public int MediumSatPerVbyte { get; init; }
    public int FastSatPerVbyte { get; init; }
    public DateTimeOffset FetchedAt { get; init; }
}
```

### 4.7 Send Transaction Request / Response

```csharp
public record SendTransactionRequest
{
    public string RecipientAddress { get; init; }
    public long AmountSatoshis { get; init; }
    public FeeTier FeeTier { get; init; } // Slow | Medium | Fast
}

public enum FeeTier { Slow, Medium, Fast }

public record SendTransactionResult
{
    public bool Success { get; init; }
    public string Txid { get; init; }         // null on failure
    public string ExplorerUrl { get; init; }  // null on failure
    public string ErrorMessage { get; init; } // null on success
}
```

### 4.8 IBitcoinApiService Interface

```csharp
public interface IBitcoinApiService
{
    Task<WalletBalance> GetBalanceAsync(string address, CancellationToken ct = default);
    Task<IReadOnlyList<TransactionSummary>> GetTransactionHistoryAsync(string address, CancellationToken ct = default);
    Task<FeeEstimate> GetFeeEstimateAsync(CancellationToken ct = default);
    Task<SendTransactionResult> BroadcastTransactionAsync(string rawTxHex, CancellationToken ct = default);
    Task<bool> CheckConnectivityAsync(CancellationToken ct = default);
}
```

### 4.9 IWalletService Interface

```csharp
public interface IWalletService
{
    WalletInfo GenerateWallet();
    WalletInfo ImportWallet(string mnemonic);
    bool ValidateMnemonic(string mnemonic);
    bool ValidateTestnetAddress(string address);
    string BuildSignedRawTransaction(
        WalletInfo wallet,
        IEnumerable<Utxo> utxos,
        string recipientAddress,
        long amountSatoshis,
        long feeSatoshis);
}
```

### 4.10 ISecretStorageService Interface

```csharp
public interface ISecretStorageService
{
    Task StoreWalletSecretAsync(string secretName, string secretValue, CancellationToken ct = default);
    Task<string> RetrieveWalletSecretAsync(string secretName, CancellationToken ct = default);
}
```

### 4.11 Testnet API Integration

The application **must** integrate with one of the following public testnet APIs (configurable via `appsettings.json`):

| Provider | Base URL | Notes |
|---|---|---|
| Blockstream Esplora | `https://blockstream.info/testnet/api` | Preferred; open-source; no API key required |
| BlockCypher | `https://api.blockcypher.com/v1/btc/test3` | Rate-limited on free tier; API key optional |

---

## 5. Acceptance Criteria

- **AC-001** (REQ-001 / GH-001): Given the user visits `/create` and clicks "Create Wallet", when the wallet is generated, then a 12- or 24-word BIP-39 mnemonic is displayed once with a backup warning, and a valid testnet address (starting with `m`, `n`, or `tb1`) is shown. The private key is not visible without an explicit "reveal" action.
- **AC-002** (REQ-002 / GH-002): Given the user enters a valid 12- or 24-word BIP-39 mnemonic and clicks "Import", when the import completes, then the wallet dashboard loads showing the correct address and balance. Given the user enters an invalid mnemonic, when they submit, then a descriptive validation error is displayed.
- **AC-003** (REQ-003 / GH-003): Given the dashboard is loaded, when balance data is available, then confirmed and unconfirmed balances are shown in both satoshis and tBTC. A "Refresh" button triggers an API call. When the API is unreachable, the last-known balance is shown with a stale indicator and the time of the last successful fetch.
- **AC-004** (REQ-004 / GH-004): Given the user navigates to `/history`, when transactions exist, then a paginated list displays TXID (truncated; full value on hover/copy), direction, amount in tBTC, confirmation count, and timestamp. Each TXID links to the testnet block explorer. Sent and received transactions are visually distinct.
- **AC-005** (REQ-005 / GH-005): Given the Receive page is loaded, then the full testnet address is displayed as copyable text and as a QR code. A "Copy Address" button copies the address to the clipboard.
- **AC-006** (REQ-005 / GH-006): Given the user completes the send form with a valid recipient testnet address and amount, when they confirm the transaction, then the application constructs, signs, and broadcasts the raw transaction and displays the resulting TXID with a block explorer link. Given the broadcast fails, then a descriptive error is shown and no TXID is displayed.
- **AC-007** (SEC-004): Given the user enters a Bitcoin mainnet address (starts with `1`, `3`, or `bc1`) in the send form, when they submit, then a validation error is displayed and the transaction is not constructed or broadcast.
- **AC-008** (REQ-007 / GH-007): Given the send form is displayed, then fee rates for slow, medium, and fast tiers are fetched and the estimated total fee in satoshis is shown for each tier before broadcast. When fee data is unavailable, a sensible default is applied and the user is notified.
- **AC-009** (SEC-001 / SEC-002 / GH-008): Given the application is running, when inspecting application logs, then no private key or mnemonic data appears. Wallet secrets are read from and written to GCP Secret Manager only.
- **AC-010** (REQ-008 / GH-010): Given a `GET /health` request is made, when the testnet API is reachable, then HTTP 200 is returned with `{ "status": "healthy", "testnet": "connected" }`. When the testnet API is unreachable, then HTTP 503 is returned with `"testnet": "disconnected"`, and the UI shows a disconnected status indicator.
- **AC-011** (REQ-009 / GH-011): Given a push to the `main` branch, when the CI/CD pipeline runs, then `dotnet build` and `dotnet test` execute; a failing test blocks the deploy. On success, a Docker image is pushed to GCP Artifact Registry and deployed to Cloud Run. The full pipeline completes in under 5 minutes.
- **AC-012** (CON-007): Given the test suite runs, then code coverage on business logic classes is ≥ 80%.
- **AC-013** (GH-009): Given a clean GCP project, when the documented deployment steps are followed, then the application is reachable at a public HTTPS Cloud Run URL in under 30 minutes.
- **AC-014** (SEC-005): Given the container image is inspected, then no private keys, mnemonic phrases, or GCP credentials are embedded in the image.

---

## 6. Test Automation Strategy

- **Test Levels**:
  - **Unit**: Test all `IWalletService`, `IBitcoinApiService` implementations, fee calculation logic, UTXO selection, address validation, and mnemonic validation in isolation using mocked dependencies.
  - **Integration**: Test GCP Secret Manager integration against a local emulator or a dedicated test GCP project. Test HTTP clients against Blockstream/BlockCypher testnet APIs.
  - **End-to-End**: Manual or automated E2E test of the full wallet creation → fund → send flow using the Bitcoin testnet faucet.

- **Frameworks**: xUnit (test runner), Moq (mocking), FluentAssertions (assertions), NBitcoin (Bitcoin primitives in tests).

- **Test Data Management**: Use NBitcoin's `Network.TestNet` in tests; generate deterministic test wallets from known mnemonics; never use real funds or mainnet keys in tests.

- **CI/CD Integration**: `dotnet test` runs in GitHub Actions on every push and pull request to `main`. A failing test blocks Docker build and deployment.

- **Coverage Requirements**: ≥ 80% line coverage on all classes in the `Services` and `Bitcoin` namespaces, enforced via `dotnet-coverage` or Coverlet with a minimum threshold gate in CI.

- **Performance Testing**: Manual verification that balance and transaction history load within 3 seconds on a standard broadband connection. Send transaction flow completes end-to-end in under 10 seconds.

- **Security Testing**: Static analysis scan (SonarQube or Semgrep) run in CI; no high-severity findings permitted to pass the gate. Log inspection test verifies no key material appears in output.

---

## 7. Rationale & Context

StrikeBESE is designed as a skill-building portfolio project for a backend software engineer learning the practices used at Strike (a Bitcoin fintech company). Every technical decision is made to mirror real production practices:

- **NBitcoin over custom crypto**: Using a battle-tested library eliminates the risk of cryptographic implementation errors, which are a leading cause of wallet vulnerabilities.
- **GCP Secret Manager**: Even on testnet, practicing proper secret hygiene builds the habits required for handling real funds. The principle of never logging key material is non-negotiable.
- **BIP-32/39/44**: Industry-standard derivation paths ensure interoperability with hardware wallets and other software wallets. Testnet uses coin type `1` (vs. `0` for mainnet) per BIP-44.
- **Blazor (.NET)**: Mirrors Strike's backend technology stack, providing relevant C#/.NET experience.
- **Cloud Run**: A serverless container platform that provides hands-on experience with GCP infrastructure, IAM, and container-based deployment without requiring Kubernetes.
- **Testnet-only**: Eliminates financial risk while providing a functionally identical development environment to mainnet.
- **Graceful degradation**: Testnet infrastructure is less reliable than mainnet; the application must tolerate API instability without crashing or silently returning incorrect data.

---

## 8. Dependencies & External Integrations

### External Systems

- **EXT-001**: Bitcoin Testnet — The Bitcoin testnet blockchain; accessed via a public REST API (Esplora or BlockCypher).
- **EXT-002**: Testnet Block Explorer — `mempool.space/testnet` (or equivalent); linked to from transaction history and send result pages.
- **EXT-003**: Bitcoin Testnet Faucet — External faucet service (e.g., `https://coinfaucet.eu/en/btc-testnet/`) used manually to fund the test wallet during development.

### Third-Party Services

- **SVC-001**: Blockstream Esplora Testnet API (`https://blockstream.info/testnet/api`) — Used for balance queries, transaction history retrieval, transaction broadcast, and fee estimation. No API key required; public endpoint.
- **SVC-002**: BlockCypher Testnet API (`https://api.blockcypher.com/v1/btc/test3`) — Alternative to SVC-001; subject to free-tier rate limits (3 requests/second, 200/hour unauthenticated).

### Infrastructure Dependencies

- **INF-001**: GCP Cloud Run — Managed container runtime for the Blazor application. Requires a GCP project with the Cloud Run API enabled.
- **INF-002**: GCP Secret Manager — Stores wallet private keys and mnemonic phrases at rest. Requires the Secret Manager API to be enabled and the application's service account to have `roles/secretmanager.secretAccessor`.
- **INF-003**: GCP Artifact Registry — Docker image registry for the application container. Requires the Artifact Registry API to be enabled.
- **INF-004**: GitHub Actions — CI/CD pipeline execution environment. Requires a GCP Workload Identity Federation or service account key for authentication to GCP.

### Technology Platform Dependencies

- **PLT-001**: .NET 8 (or later) runtime — Required for the Blazor application host.
- **PLT-002**: Docker — Required for building and pushing the container image.
- **PLT-003**: NBitcoin — .NET Bitcoin library for all cryptographic operations (key generation, address derivation, transaction construction and signing).
- **PLT-004**: QRCoder (or equivalent) — .NET library for generating QR code images from the wallet address string.
- **PLT-005**: Google.Cloud.SecretManager.V1 — GCP Secret Manager .NET client library.

### Compliance Dependencies

- **COM-001**: No personally identifiable information (PII) is collected; no GDPR/CCPA controls are required in the initial version.
- **COM-002**: Financial regulations do not apply as the application operates exclusively on Bitcoin testnet with no real monetary value.

---

## 9. Examples & Edge Cases

### 9.1 HD Wallet Derivation (BIP-44, Testnet)

```csharp
// BIP-44 coin type for testnet is 1 (not 0 as used on mainnet)
// Derivation path: m/44'/1'/0'/0/0
var mnemonic = new Mnemonic("abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about", Wordlist.English);
var hdRoot = mnemonic.DeriveExtKey();
var keyPath = new KeyPath("m/44'/1'/0'/0/0");
var derivedKey = hdRoot.Derive(keyPath).PrivateKey;
var address = derivedKey.PubKey.GetAddress(ScriptPubKeyType.Legacy, Network.TestNet);
// Expected: mzqiDnETWkunRDZxjUQ34JzN1LDevh5DpU (deterministic for this mnemonic)
```

### 9.2 Address Validation — Testnet vs Mainnet

```csharp
// Valid testnet addresses (must accept):
// P2PKH:  starts with 'm' or 'n'
// P2WPKH: starts with 'tb1'

// Invalid (must reject with validation error):
// P2PKH mainnet:  starts with '1'
// P2SH mainnet:   starts with '3'
// P2WPKH mainnet: starts with 'bc1'

bool IsValidTestnetAddress(string address)
{
    try
    {
        BitcoinAddress.Create(address, Network.TestNet);
        return true;
    }
    catch
    {
        return false;
    }
}
```

### 9.3 Edge Cases

| Scenario | Expected Behaviour |
|---|---|
| Send amount equals confirmed balance (no fee buffer) | Validation error: "Insufficient funds to cover the transaction fee." |
| Send amount below dust limit (< 546 sat) | Validation error: "Amount is below the dust limit of 546 satoshis." |
| Testnet API returns HTTP 429 (rate limit) | Retry with exponential back-off (max 3 attempts); on final failure, serve stale cached data with a stale indicator. |
| Wallet mnemonic checksum failure on import | Display error: "The mnemonic phrase is invalid. Please check for typos and try again." |
| Secret Manager unreachable on wallet read | Application fails fast with a logged (non-sensitive) error and displays a user-facing error page; does not fall back to any local storage. |
| Unconfirmed UTXOs present when constructing a new send | Select only confirmed UTXOs by default; optionally note to the user that pending UTXOs are excluded. |
| User enters mainnet address in send form | Validation error: "This appears to be a Bitcoin mainnet address. Please enter a testnet address." |
| Testnet chain reset | Balance and history return empty or unexpected data; the application displays what the API returns without crashing; a stale indicator is shown if data is older than the configured TTL. |

---

## 10. Validation Criteria

| ID | Criterion | How to Verify |
|---|---|---|
| VAL-001 | BIP-39 mnemonic generation produces a valid, checksummed 12- or 24-word phrase | Unit test: parse generated mnemonic with `new Mnemonic(phrase, Wordlist.English)` — no exception thrown |
| VAL-002 | BIP-44 derivation path `m/44'/1'/0'/0/0` produces a known testnet address from a known mnemonic | Unit test: derive from "abandon × 11 + about" mnemonic; compare address to expected value |
| VAL-003 | Mainnet addresses are rejected as invalid testnet addresses | Unit test: pass `1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa` to `ValidateTestnetAddress`; expect `false` |
| VAL-004 | Private keys never appear in application logs | Integration test: run wallet creation flow; inspect captured log output for WIF-format strings |
| VAL-005 | `GET /health` returns HTTP 200 when API is reachable | Integration / E2E test: mock API returns 200; assert health endpoint returns 200 with `"status":"healthy"` |
| VAL-006 | `GET /health` returns HTTP 503 when API is unreachable | Integration test: mock API throws `HttpRequestException`; assert health endpoint returns 503 |
| VAL-007 | Transaction fee estimate is computed and displayed before broadcast | Unit test: given known UTXO set and recipient, assert estimated fee equals `txVirtualSize × feeRate` |
| VAL-008 | Send transaction rejects amount below dust limit | Unit test: call `BuildSignedRawTransaction` with `amountSatoshis = 500`; expect `ArgumentException` or `Result.Failure` |
| VAL-009 | CI/CD pipeline completes in ≤ 5 minutes | Measure GitHub Actions workflow duration over 5 consecutive runs |
| VAL-010 | Code coverage on business logic ≥ 80% | CI gate: `dotnet test --collect:"XPlat Code Coverage"` with Coverlet threshold `80` |
| VAL-011 | Container image contains no embedded secrets | Inspect image layers with `docker history` and `docker inspect`; grep for WIF-format strings and GCP service account key patterns |
| VAL-012 | Balance and transaction history load within 3 seconds | Manual E2E test on a standard broadband connection with Chrome DevTools network timing |

---

## 11. Related Specifications / Further Reading

- [`documents/prd.md`](prd.md) — Product Requirements Document; the primary source of truth for product scope and user stories.
- [`BITCOIN_INTEGRATION_GUIDE.md`](../BITCOIN_INTEGRATION_GUIDE.md) — Code-level guide for Bitcoin integration using NBitcoin in .NET.
- [`BITCOIN_HOTWALLET_DEPLOYMENT_PLAN.md`](../BITCOIN_HOTWALLET_DEPLOYMENT_PLAN.md) — GCP deployment plan and infrastructure setup steps.
- [`TESTING_GUIDE.md`](../TESTING_GUIDE.md) — Testing standards, tooling, and patterns for the project.
- [`API_DEVELOPMENT_GUIDE.md`](../API_DEVELOPMENT_GUIDE.md) — API design standards and patterns used in the project.
- [`PERFORMANCE_OPTIMIZATION_GUIDE.md`](../PERFORMANCE_OPTIMIZATION_GUIDE.md) — Caching, async patterns, and performance guidelines.
- [BIP-39 Specification](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) — Mnemonic code for generating deterministic keys.
- [BIP-32 Specification](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) — Hierarchical deterministic wallets.
- [BIP-44 Specification](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) — Multi-account hierarchy for deterministic wallets.
- [Blockstream Esplora API Docs](https://github.com/Blockstream/esplora/blob/master/API.md) — REST API reference for balance, history, broadcast, and fee queries.
- [NBitcoin Documentation](https://programmingblockchain.gitbook.io/programmingblockchain/) — Comprehensive guide to using NBitcoin for Bitcoin development in .NET.
- [GCP Secret Manager Documentation](https://cloud.google.com/secret-manager/docs) — Official GCP documentation for Secret Manager setup and .NET client usage.
- [GCP Cloud Run Documentation](https://cloud.google.com/run/docs) — Official GCP documentation for containerised application deployment.
