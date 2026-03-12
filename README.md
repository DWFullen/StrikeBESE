# StrikeBESE

> **Note:** The Product Requirements Document (PRD) for this project is located at [`documents/prd.md`](documents/prd.md).
> The full PRD content is reproduced below for quick reference. Once you have access to a shell, run:
> ```bash
> mkdir -p documents && mv README.md documents/prd.md && echo "# StrikeBESE" > README.md
> ```
> Or simply copy the PRD section below into `documents/prd.md`.

---

# PRD: StrikeBESE Bitcoin Hotwallet

## 1. Product overview

### 1.1 Document title and version

- PRD: StrikeBESE Bitcoin Hotwallet
- Version: 1.0

### 1.2 Product summary

StrikeBESE is a learning-focused Bitcoin hotwallet application built to develop backend software engineering skills aligned with the practices used at Strike. The project targets the Bitcoin testnet, providing a safe and cost-free environment for experimentation with real Bitcoin transaction flows, wallet management, and cryptographic key handling — all without risking real funds.

The application is built on the Blazor (.NET) framework and hosted on Google Cloud Platform (GCP). This combination mirrors modern, production-grade cloud-native development and gives the developer hands-on experience with .NET server-side rendering, RESTful or gRPC service design, secret management, and cloud infrastructure provisioning.

By completing this project, the developer will have demonstrated end-to-end ownership of a financially sensitive backend system: from wallet generation and testnet transaction broadcasting to secure key storage and cloud deployment — the core competency stack for a backend engineer at a Bitcoin fintech company like Strike.

## 2. Goals

### 2.1 Business goals

- Build a functional Bitcoin testnet hotwallet that demonstrates production-level engineering practices.
- Serve as a portfolio piece showcasing backend, cryptography, and cloud skills relevant to Strike's engineering stack.
- Establish a reusable codebase that can be extended into more advanced features (Lightning Network, mainnet migration, etc.).
- Document architectural decisions to support knowledge transfer and future learning.

### 2.2 User goals

- Generate and manage Bitcoin testnet wallets (key pairs and addresses) securely.
- Check wallet balances and transaction history against the Bitcoin testnet.
- Send and receive Bitcoin testnet transactions (tBTC) through the application UI.
- Monitor transaction status (pending, confirmed, failed) in real time.
- Learn how Bitcoin primitives (UTXOs, signing, broadcasting) work through hands-on development.

### 2.3 Non-goals

- This project will **not** support Bitcoin mainnet transactions in its initial version.
- This project will **not** implement a Lightning Network payment channel in its initial version.
- This project will **not** be exposed as a public-facing production service.
- This project will **not** implement multi-signature wallets in its initial version.
- This project will **not** handle fiat currency conversion or exchange rates.

## 3. User personas

### 3.1 Key user types

- Developer/learner operating and testing the wallet locally or in GCP.
- Potential code reviewer or hiring engineer evaluating the codebase.

### 3.2 Basic persona details

- **Developer (Danny)**: A software engineer learning backend development patterns used at Strike. Danny uses the application to experiment with Bitcoin testnet wallet creation, sending transactions, and cloud deployment. He cares about code quality, security best practices, and understanding the full lifecycle of a Bitcoin transaction.
- **Code Reviewer**: A senior engineer or hiring manager reviewing the project to assess Danny's technical depth, code organization, API design, and cloud infrastructure choices.

### 3.3 Role-based access

- **Admin/Owner**: Full access to wallet management, transaction broadcasting, key management, and application configuration. This is the only role for this learning project.

## 4. Functional requirements

- **Wallet generation** (Priority: High)

  - Generate a new Bitcoin testnet wallet consisting of a private key, public key, and corresponding P2PKH or P2WPKH (native SegWit) address.
  - Derive wallets using BIP-32 HD wallet standards with a BIP-39 mnemonic seed phrase.
  - Display the generated mnemonic, public address, and masked private key in the UI.
  - Allow the user to import an existing wallet by entering a mnemonic seed phrase.

- **Balance inquiry** (Priority: High)

  - Query the Bitcoin testnet for the current confirmed and unconfirmed balance of the active wallet address.
  - Display balance in satoshis and tBTC.
  - Refresh balance on demand via a UI button.

- **Transaction history** (Priority: High)

  - Retrieve and display a list of past transactions (inbound and outbound) for the active wallet address from the testnet.
  - Show transaction ID (TXID), amount, direction, confirmation count, and timestamp for each transaction.
  - Link each transaction to a testnet block explorer (e.g., mempool.space testnet).

- **Send transaction** (Priority: High)

  - Allow the user to enter a recipient testnet Bitcoin address and an amount in satoshis or tBTC.
  - Construct, sign, and broadcast a raw Bitcoin testnet transaction using the active wallet's private key.
  - Display the resulting TXID and a link to the testnet block explorer upon successful broadcast.
  - Validate inputs: address format, sufficient balance, valid amount (> dust limit).

- **Receive / address display** (Priority: Medium)

  - Display the active wallet's receiving address prominently.
  - Render a QR code for the receiving address to facilitate testnet faucet funding.

- **Transaction fee estimation** (Priority: Medium)

  - Fetch current testnet fee rate recommendations (sat/vByte) from a testnet fee estimation API or node.
  - Compute and display estimated transaction fee before the user confirms a send.
  - Allow the user to choose between slow, medium, and fast fee tiers.

- **Secure key storage** (Priority: High)

  - Private keys and mnemonic seed phrases must never be logged or stored in plaintext in a database or configuration file.
  - Use Google Cloud Secret Manager or equivalent GCP service to store sensitive key material at rest.
  - Implement encryption of the private key in memory and at rest when persisted.

- **Application health & status** (Priority: Low)

  - Expose a `/health` endpoint returning the application's status (healthy/unhealthy) and testnet node connectivity status.
  - Display a connection status indicator in the UI (connected/disconnected to testnet).

## 5. User experience

### 5.1 Entry points & first-time user flow

- User navigates to the hosted Blazor application URL on GCP (Cloud Run or App Engine).
- On first visit, the user is presented with a "Create Wallet" or "Import Wallet" option.
- After wallet creation or import, the user lands on the main wallet dashboard.

### 5.2 Core experience

- **Dashboard**: The main page displays the active wallet address, current balance (confirmed and unconfirmed), and a summary of recent transactions.

  - A clear, uncluttered layout ensures the developer can quickly verify wallet state.

- **Send page**: A simple form with fields for recipient address, amount, and fee tier selection.

  - Real-time validation feedback prevents invalid submissions and guides the user toward correct inputs.

- **Transaction history page**: A sortable, paginated list of all past transactions.

  - Each row links out to the testnet block explorer so the user can drill into raw transaction data and reinforce Bitcoin concepts.

- **Receive page**: Prominently displays the wallet address as both text and a QR code.

  - Simplifies the process of funding the wallet via a testnet faucet.

### 5.3 Advanced features & edge cases

- Handling of unconfirmed UTXOs when constructing new transactions (RBF / CPFP awareness).
- Graceful degradation when the testnet API or block explorer is unreachable (cached last-known balance with a stale indicator).
- Detection and display of double-spend or dropped transactions.
- Input sanitization for Bitcoin address validation (mainnet vs. testnet prefix check to prevent accidental mainnet address usage).

### 5.4 UI/UX highlights

- Responsive Blazor Server or WebAssembly UI with clear typographic hierarchy.
- Sensitive data (private key, mnemonic) displayed only on explicit user action (click to reveal), never auto-displayed.
- Color-coded transaction direction (green for received, red for sent).
- Confirmation dialogs before broadcasting any transaction.
- Accessible design: sufficient color contrast, labeled form controls, keyboard-navigable interface.

## 6. Narrative

Danny, a backend developer learning the ropes at a Bitcoin fintech company, opens the StrikeBESE application deployed on GCP. He creates a new HD wallet, copies the testnet address, and pastes it into a Bitcoin testnet faucet to receive some tBTC. Within moments, he sees the incoming transaction appear in his transaction history with zero confirmations. Once confirmed, he navigates to the Send page, enters a recipient address, selects the medium fee tier, reviews the fee estimate, and broadcasts the transaction. The resulting TXID appears with a direct link to mempool.space testnet, where he can trace every input and output of his transaction. Through this end-to-end flow, Danny has internalized UTXO selection, transaction signing, fee markets, and cloud secret management — exactly the skills required to contribute as a backend engineer at Strike.

## 7. Success metrics

### 7.1 User-centric metrics

- Successful wallet generation and import flow completes without errors.
- Send transaction flow completes end-to-end (input → sign → broadcast → TXID displayed) in under 10 seconds on a standard connection.
- Balance and transaction history load within 3 seconds of page navigation.
- Zero plaintext private key exposures in application logs, browser console, or network traffic.

### 7.2 Business metrics

- Project is deployable to GCP from scratch via documented steps in under 30 minutes.
- Codebase passes code review by a senior engineer with no critical findings.
- All functional requirements are covered by automated tests with ≥ 80% code coverage on business logic.

### 7.3 Technical metrics

- Application health endpoint returns 200 OK with testnet connectivity confirmed.
- Testnet transaction broadcast success rate ≥ 95% under normal testnet conditions.
- Build and deploy CI/CD pipeline completes in under 5 minutes.
- No high-severity security findings from a static analysis scan (e.g., SonarQube, Semgrep).

## 8. Technical considerations

### 8.1 Integration points

- **Bitcoin testnet node / API**: Integration with a public testnet API such as BlockCypher testnet, Blockstream Esplora testnet (`https://blockstream.info/testnet/api`), or a self-hosted Bitcoin Core testnet node for UTXO queries, transaction broadcasting, and fee estimation.
- **Google Cloud Platform**: Cloud Run or App Engine for hosting the Blazor application; Cloud Secret Manager for private key/mnemonic storage; Cloud Build or GitHub Actions for CI/CD; Artifact Registry for container images.
- **BIP-32/39/44 Libraries**: Use an established .NET Bitcoin library (e.g., NBitcoin) for HD wallet derivation, transaction construction, and signing.
- **QR Code generation**: A .NET-compatible QR code library (e.g., QRCoder) for rendering the receiving address QR code in Blazor.

### 8.2 Data storage & privacy

- Private keys and mnemonic phrases are stored exclusively in Google Cloud Secret Manager and never written to a relational database or configuration file.
- Application logs must redact or omit any sensitive key material.
- No personally identifiable information (PII) is collected; the application is single-user by design.
- Wallet state (address, balance cache) may be stored in a lightweight in-memory or file-based store; no production database is required for the initial version.

### 8.3 Scalability & performance

- The application is single-user and learning-focused; horizontal scaling is not required in the initial version.
- API rate limits from public testnet services (e.g., BlockCypher free tier) must be respected; implement caching with a configurable TTL for balance and transaction history responses.
- Use asynchronous .NET patterns (`async`/`await`) throughout to keep the Blazor UI responsive during network calls.

### 8.4 Potential challenges

- Testnet instability: Bitcoin testnet can be unreliable (chain resets, slow block times). The application must handle API timeouts and partial data gracefully.
- UTXO management: Correctly selecting UTXOs and constructing valid raw transactions requires precise implementation; use a well-tested library (NBitcoin) rather than custom serialization.
- Key security in a learning environment: Even on testnet, practicing proper key hygiene (Secret Manager, never log keys) is essential for building good habits.
- GCP IAM configuration: Setting up the correct service account roles for Secret Manager access from Cloud Run requires careful configuration to avoid both over-permissioning and deployment failures.
- Cold start latency: Blazor Server or WASM on Cloud Run may experience cold starts; configure minimum instance count or use Cloud Run's always-on option for a smoother demo experience.

## 9. Milestones & sequencing

### 9.1 Project estimate

- Medium: 4–6 weeks (part-time learning pace)

### 9.2 Team size & composition

- Solo developer: 1 backend/full-stack developer (learner)

### 9.3 Suggested phases

- **Phase 1**: Project scaffolding and local wallet primitives (1 week)

  - Initialize Blazor (.NET) project structure.
  - Integrate NBitcoin library.
  - Implement BIP-39 mnemonic generation and BIP-32 HD wallet derivation.
  - Unit tests for key generation and address derivation.

- **Phase 2**: Testnet connectivity and balance/history (1–2 weeks)

  - Integrate with a public testnet API (Blockstream Esplora or BlockCypher).
  - Implement balance inquiry and transaction history retrieval.
  - Display data in Blazor UI components.
  - Handle API errors and timeouts gracefully.

- **Phase 3**: Send transaction flow (1–2 weeks)

  - Implement UTXO selection, transaction construction, signing, and broadcasting.
  - Implement fee estimation integration.
  - Build send form with validation and confirmation dialog.
  - Integration tests for transaction construction (using testnet).

- **Phase 4**: GCP deployment and secret management (1 week)

  - Containerize the application (Dockerfile).
  - Configure Google Cloud Secret Manager for key storage.
  - Deploy to Cloud Run with appropriate IAM roles.
  - Set up Cloud Build or GitHub Actions CI/CD pipeline.
  - Document deployment steps in README.

- **Phase 5**: Polish, security hardening, and documentation (0.5–1 week)

  - Add application health endpoint.
  - Perform static analysis scan and resolve findings.
  - Complete inline code documentation and README.
  - Final end-to-end test on deployed GCP environment.

## 10. User stories

### 10.1. Create a new wallet

- **ID**: GH-001
- **Description**: As a developer, I want to generate a new Bitcoin testnet HD wallet so that I have a unique address and key pair to use for testing transactions.
- **Acceptance criteria**:
  - A BIP-39 mnemonic seed phrase (12 or 24 words) is generated and displayed to the user.
  - A BIP-32 derived private key, public key, and testnet address are produced from the mnemonic.
  - The generated address is a valid Bitcoin testnet address (starts with `m`, `n`, or `tb1`).
  - The private key is never displayed in plaintext without an explicit "reveal" action by the user.
  - The mnemonic is shown only once, with a warning to back it up before proceeding.

### 10.2. Import an existing wallet

- **ID**: GH-002
- **Description**: As a developer, I want to import an existing wallet using a BIP-39 mnemonic so that I can restore access to a previously created testnet wallet.
- **Acceptance criteria**:
  - The user can enter a 12- or 24-word BIP-39 mnemonic in an input field.
  - The application validates the mnemonic (checksum check) and displays an error if it is invalid.
  - Upon successful import, the wallet dashboard loads with the correct address, balance, and transaction history for the imported wallet.
  - Invalid or empty mnemonic inputs produce a descriptive validation error message.

### 10.3. View wallet balance

- **ID**: GH-003
- **Description**: As a developer, I want to view the current confirmed and unconfirmed balance of my testnet wallet so that I know how much tBTC is available to spend.
- **Acceptance criteria**:
  - The dashboard displays the confirmed balance and unconfirmed (pending) balance separately.
  - Balances are shown in both satoshis and tBTC.
  - A "Refresh" button fetches the latest balance from the testnet API.
  - If the testnet API is unreachable, the last-known balance is shown with a "stale data" indicator and timestamp.

### 10.4. View transaction history

- **ID**: GH-004
- **Description**: As a developer, I want to see a list of past transactions for my wallet so that I can track funds sent and received during testing.
- **Acceptance criteria**:
  - The transaction history page lists all transactions associated with the active wallet address.
  - Each transaction row displays: TXID (truncated with full value on hover/copy), direction (sent/received), amount in tBTC, confirmation count, and timestamp.
  - Each TXID links to the corresponding transaction on a testnet block explorer (e.g., mempool.space testnet).
  - Sent transactions are visually distinguished from received transactions (e.g., color coding).
  - The list is paginated or scrollable if there are more than 20 transactions.

### 10.5. Receive Bitcoin (display address and QR code)

- **ID**: GH-005
- **Description**: As a developer, I want to see my wallet's receiving address and a QR code so that I can easily fund the wallet from a testnet faucet.
- **Acceptance criteria**:
  - The Receive page displays the full wallet testnet address as copyable text.
  - A QR code encoding the testnet address is rendered on the page.
  - A "Copy Address" button copies the address to the clipboard.
  - The address displayed matches the address derived from the active wallet's key.

### 10.6. Send a Bitcoin testnet transaction

- **ID**: GH-006
- **Description**: As a developer, I want to send tBTC to another testnet address so that I can practice constructing, signing, and broadcasting real Bitcoin transactions.
- **Acceptance criteria**:
  - The Send page provides input fields for recipient address and amount (in satoshis or tBTC).
  - The recipient address is validated as a valid Bitcoin testnet address before submission.
  - The amount is validated to be greater than the dust limit and not exceed the available confirmed balance minus the estimated fee.
  - The user can select a fee tier (slow / medium / fast) with the estimated fee displayed for each tier.
  - A confirmation dialog summarizes the recipient, amount, and fee before broadcasting.
  - Upon successful broadcast, the TXID is displayed with a link to the testnet block explorer.
  - If the broadcast fails (e.g., insufficient funds, rejected by node), a descriptive error message is displayed and no TXID is shown.

### 10.7. Estimate transaction fees

- **ID**: GH-007
- **Description**: As a developer, I want to see a fee estimate before sending a transaction so that I understand how Bitcoin fee markets work and can choose an appropriate fee rate.
- **Acceptance criteria**:
  - Fee rates (in sat/vByte) for slow, medium, and fast confirmation targets are fetched from a testnet fee API.
  - The estimated total fee in satoshis is computed and displayed for the current transaction size before broadcast.
  - The fee estimate updates automatically when the recipient address or amount changes.
  - If fee estimation data is unavailable, a sensible default fee rate is used and the user is notified.

### 10.8. Secure key storage in GCP Secret Manager

- **ID**: GH-008
- **Description**: As a developer, I want private keys and mnemonic phrases to be stored in Google Cloud Secret Manager so that I practice production-grade secret management and never expose key material in plaintext.
- **Acceptance criteria**:
  - The application writes the wallet mnemonic or private key to GCP Secret Manager on wallet creation/import.
  - The application reads the secret from GCP Secret Manager at runtime; it is never stored in `appsettings.json`, environment variables exposed in logs, or the database.
  - Application logs do not contain any private key or mnemonic data (verified by log inspection during testing).
  - The GCP service account used by the application has only the `secretmanager.secretAccessor` role and no broader permissions.
  - If Secret Manager is unreachable, the application fails with a clear error rather than falling back to an insecure storage mechanism.

### 10.9. Deploy to Google Cloud Platform

- **ID**: GH-009
- **Description**: As a developer, I want to deploy the application to GCP Cloud Run so that I gain hands-on experience with cloud-native deployment of a .NET application.
- **Acceptance criteria**:
  - The application is containerized with a Dockerfile that produces a working image.
  - The container image is pushed to GCP Artifact Registry via a CI/CD pipeline (Cloud Build or GitHub Actions).
  - The application is deployed to Cloud Run and is reachable at a public HTTPS URL.
  - The deployment uses environment variables or Secret Manager references for all configuration; no secrets are baked into the container image.
  - The README documents the complete deployment steps (GCP project setup, IAM configuration, first deploy, re-deploy).

### 10.10. View application health status

- **ID**: GH-010
- **Description**: As a developer, I want a health endpoint and a status indicator in the UI so that I can confirm the application is running and connected to the Bitcoin testnet.
- **Acceptance criteria**:
  - A `GET /health` endpoint returns HTTP 200 with a JSON body indicating application status and testnet API connectivity (e.g., `{ "status": "healthy", "testnet": "connected" }`).
  - The UI displays a visual status indicator (e.g., colored dot or badge) showing whether the testnet API is reachable.
  - If the testnet API is unreachable, the health endpoint returns a non-200 status or `"testnet": "disconnected"` in the response body, and the UI reflects this state.

### 10.11. CI/CD pipeline for build and deploy

- **ID**: GH-011
- **Description**: As a developer, I want an automated CI/CD pipeline so that every code push is automatically built, tested, and deployable to GCP without manual steps.
- **Acceptance criteria**:
  - A GitHub Actions workflow (or Cloud Build trigger) runs on every push to the main branch.
  - The pipeline executes `dotnet build` and `dotnet test`; a failing test blocks the deployment.
  - On success, the pipeline builds a Docker image and pushes it to GCP Artifact Registry.
  - The pipeline deploys the new image to Cloud Run (or marks it ready for manual promotion).
  - Pipeline execution time is under 5 minutes end-to-end.
**Backend Software Engineer Interview Preparation Repository**

This repository contains comprehensive guides and examples to prepare for a Backend Software Engineer role focused on API development, Bitcoin integration, and .NET/C# development.

## 🚀 Featured Project: Bitcoin Hot-Wallet Deployment Plan
- **[BITCOIN_HOTWALLET_DEPLOYMENT_PLAN.md](BITCOIN_HOTWALLET_DEPLOYMENT_PLAN.md)** - Complete step-by-step plan for deploying a production-ready custodial Bitcoin hot-wallet system
  - Blazor Server frontend with real-time updates
  - ASP.NET Core Web API backend
  - Google Cloud Platform infrastructure
  - Comprehensive security implementation
  - Complete testing strategy
  - Production deployment guide
  - Monitoring and maintenance procedures

This deployment plan brings together all the technical guides in this repository to showcase a complete, real-world project that demonstrates backend engineering competency.

## 📚 Documentation Overview

### Core Role Understanding
- **[ROLE_SUMMARY.md](ROLE_SUMMARY.md)** - Comprehensive overview of the Backend Software Engineer role, responsibilities, required skills, and success metrics

### Technical Guides

#### API Development
- **[API_DEVELOPMENT_GUIDE.md](API_DEVELOPMENT_GUIDE.md)** - Complete guide to building robust RESTful APIs in C#/.NET
  - RESTful design principles
  - Controller and service layer patterns
  - Request/response models
  - Authentication and authorization
  - Error handling and validation
  - Performance considerations
  - Security best practices

#### Bitcoin Integration
- **[BITCOIN_INTEGRATION_GUIDE.md](BITCOIN_INTEGRATION_GUIDE.md)** - Bitcoin fundamentals and integration patterns
  - Bitcoin basics and key concepts
  - NBitcoin library usage
  - Wallet generation and management
  - Transaction creation and broadcasting
  - Bitcoin RPC integration
  - Security best practices for cryptocurrency handling
  - Testing strategies

#### Testing
- **[TESTING_GUIDE.md](TESTING_GUIDE.md)** - Comprehensive testing guide using xUnit and Moq
  - xUnit test structure and patterns
  - Mocking with Moq
  - Testing controllers and services
  - Integration testing
  - Test fixtures and shared context
  - Code coverage
  - Testing best practices

### Team Practices

#### Code Review
- **[CODE_REVIEW_BEST_PRACTICES.md](CODE_REVIEW_BEST_PRACTICES.md)** - Guidelines for effective code reviews
  - For code authors: preparing and responding to reviews
  - For reviewers: giving constructive feedback
  - Common code issues to watch for
  - Communication tips
  - Review workflow

#### Performance Optimization
- **[PERFORMANCE_OPTIMIZATION_GUIDE.md](PERFORMANCE_OPTIMIZATION_GUIDE.md)** - Performance tuning strategies
  - Database optimization
  - Caching strategies (in-memory and distributed)
  - Async/await optimization
  - Algorithm and memory optimization
  - API performance
  - Monitoring and profiling tools

## 🎯 Key Responsibilities Covered

This repository addresses all key responsibilities of a Backend Software Engineer:

1. ✅ **API Design & Development** - Covered in API Development Guide
2. ✅ **Efficient, Modular Code** - Patterns and examples throughout all guides
3. ✅ **Internal Tooling** - Developer efficiency practices in Performance Guide
4. ✅ **Performance Issues** - Dedicated Performance Optimization Guide
5. ✅ **Code Reviews** - Complete Code Review Best Practices guide
6. ✅ **Best Practices Communication** - Examples and patterns in all documents

## 🛠️ Required Skills Addressed

- ✅ **Bitcoin Experience** - Comprehensive Bitcoin Integration Guide
- ✅ **xUnit Testing** - Detailed xUnit examples and patterns in Testing Guide
- ✅ **Moq Framework** - Extensive Moq usage examples in Testing Guide
- ✅ **API Development** - Complete API development guide with examples
- ✅ **C#/.NET** - All code examples use C# and .NET best practices

## 🚀 Getting Started

### For Interview Preparation
1. **Start with [ROLE_SUMMARY.md](ROLE_SUMMARY.md)** to understand the position and expectations
2. **Review [API_DEVELOPMENT_GUIDE.md](API_DEVELOPMENT_GUIDE.md)** for core API development skills
3. **Study [BITCOIN_INTEGRATION_GUIDE.md](BITCOIN_INTEGRATION_GUIDE.md)** for cryptocurrency-specific knowledge
4. **Practice with [TESTING_GUIDE.md](TESTING_GUIDE.md)** to master xUnit and Moq
5. **Understand [CODE_REVIEW_BEST_PRACTICES.md](CODE_REVIEW_BEST_PRACTICES.md)** for team collaboration
6. **Learn from [PERFORMANCE_OPTIMIZATION_GUIDE.md](PERFORMANCE_OPTIMIZATION_GUIDE.md)** to optimize code

### For Building a Portfolio Project
**Follow the [BITCOIN_HOTWALLET_DEPLOYMENT_PLAN.md](BITCOIN_HOTWALLET_DEPLOYMENT_PLAN.md)** to build a complete, production-ready Bitcoin wallet application that showcases:
- Full-stack development (Blazor + ASP.NET Core)
- Bitcoin integration and cryptocurrency handling
- Cloud deployment on Google Cloud Platform
- Security best practices
- Professional testing and monitoring
- Real-world application architecture

## 💡 Key Takeaways

### API Development Best Practices
- Use RESTful design principles
- Implement proper error handling and validation
- Secure APIs with authentication and authorization
- Document APIs with Swagger/OpenAPI
- Design for scalability and performance

### Bitcoin Integration
- Always use testnet for development
- Never store private keys in plain text
- Validate addresses before transactions
- Implement proper security measures
- Use NBitcoin library for Bitcoin operations

### Testing Approach
- Follow AAA pattern (Arrange, Act, Assert)
- Use descriptive test names
- Mock external dependencies with Moq
- Aim for high test coverage
- Write both unit and integration tests

### Code Review Philosophy
- Be constructive and specific
- Focus on important issues
- Balance criticism with praise
- Explain the "why" behind suggestions
- Keep PRs small and focused

### Performance Mindset
- Measure before optimizing
- Optimize database queries
- Implement appropriate caching
- Use async/await properly
- Monitor production performance

## 📖 Additional Resources

Each guide includes links to official documentation, tools, and best practices from the community.

## 🤝 Contributing

This is a personal preparation repository, but feedback and suggestions are welcome!

---

**Good luck with your Backend Software Engineer interview preparation!** 🚀
