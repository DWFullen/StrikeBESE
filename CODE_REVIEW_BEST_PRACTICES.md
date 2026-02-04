# Code Review Best Practices

## Introduction
Code reviews are a critical part of software development that improve code quality, share knowledge, and maintain team standards. This guide covers best practices for both reviewers and authors.

## Goals of Code Review

1. **Catch bugs and errors** before they reach production
2. **Ensure code quality** and maintainability
3. **Share knowledge** across the team
4. **Maintain consistency** in coding standards
5. **Improve security** by identifying vulnerabilities
6. **Foster collaboration** and team communication

## For Code Authors

### Before Submitting for Review

#### 1. Self-Review Your Code
```csharp
// ❌ Before self-review - debugging code left in
public async Task<Wallet> CreateWallet()
{
    Console.WriteLine("Debug: Creating wallet");
    var wallet = new Wallet();
    Console.WriteLine($"Debug: Wallet ID is {wallet.Id}");
    return wallet;
}

// ✅ After self-review - clean production code
public async Task<Wallet> CreateWallet()
{
    _logger.LogInformation("Creating new wallet");
    var wallet = new Wallet();
    return wallet;
}
```

#### 2. Write a Clear Pull Request Description
```markdown
## Summary
Implement Bitcoin wallet balance retrieval API endpoint

## Changes Made
- Added `GetBalance` endpoint to `BitcoinWalletsController`
- Implemented `GetBalanceAsync` in `BitcoinWalletService`
- Added integration with Bitcoin RPC client
- Created unit tests with 95% coverage

## Testing
- Unit tests: All passing (47/47)
- Manual testing with testnet wallet
- Verified correct balance calculation

## Related Issues
Fixes #123 - Add wallet balance endpoint
```

#### 3. Keep PRs Small and Focused
```
✅ Good PR Size:
- Single feature or bug fix
- 200-400 lines of changes
- Can be reviewed in 30 minutes

❌ Too Large:
- Multiple unrelated features
- 2000+ lines of changes
- Takes hours to review
```

#### 4. Run All Checks Before Submitting
```bash
# Format code
dotnet format

# Run linter
dotnet build

# Run all tests
dotnet test

# Check code coverage
dotnet test /p:CollectCoverage=true

# Check for security vulnerabilities
dotnet list package --vulnerable
```

### Responding to Review Comments

#### Be Open to Feedback
```csharp
// Reviewer comment: "This method is doing too much. Consider splitting it."

// ❌ Defensive response
// "This is fine as is. It's not that complex."

// ✅ Constructive response
// "Good catch! I'll split this into GetBalance and FormatResponse methods."
```

#### Ask for Clarification
```markdown
// If a comment is unclear:
"Could you elaborate on what you mean by 'improve error handling here'? 
Are you suggesting we catch specific exceptions or add more context to the error message?"
```

#### Mark Resolved Issues
```markdown
✅ "Fixed in commit abc123"
✅ "Created issue #456 to track this for later"
✅ "Updated based on your suggestion - see latest commit"
```

## For Code Reviewers

### Review Checklist

#### 1. Functionality
- [ ] Does the code do what it's supposed to do?
- [ ] Are there any obvious bugs or logic errors?
- [ ] Are edge cases handled properly?
- [ ] Is error handling appropriate?

#### 2. Code Quality
- [ ] Is the code easy to understand?
- [ ] Are variable and method names clear and descriptive?
- [ ] Is the code properly structured and organized?
- [ ] Are there code duplications that could be refactored?

#### 3. Testing
- [ ] Are there adequate unit tests?
- [ ] Do tests cover edge cases?
- [ ] Are tests clear and well-named?
- [ ] Do all tests pass?

#### 4. Security
- [ ] Are there any security vulnerabilities?
- [ ] Is input validated properly?
- [ ] Are credentials or secrets properly handled?
- [ ] Is authentication/authorization implemented correctly?

#### 5. Performance
- [ ] Are there any obvious performance issues?
- [ ] Are database queries optimized?
- [ ] Is caching used appropriately?
- [ ] Are resources properly disposed?

#### 6. Documentation
- [ ] Are public APIs documented?
- [ ] Are complex algorithms explained?
- [ ] Is the README updated if needed?
- [ ] Are breaking changes documented?

### How to Give Feedback

#### Be Specific and Actionable
```csharp
// ❌ Vague feedback
// "This is confusing"

// ✅ Specific and actionable
// "Consider renaming this method from 'Process' to 'ValidateAndSaveTransaction' 
// to better describe what it does"

// Original code
public async Task<bool> Process(Transaction tx)
{
    if (!Validate(tx)) return false;
    await Save(tx);
    return true;
}

// Suggested improvement
public async Task<bool> ValidateAndSaveTransaction(Transaction transaction)
{
    if (!IsValidTransaction(transaction)) 
    {
        return false;
    }
    
    await SaveTransactionAsync(transaction);
    return true;
}
```

#### Use Questions to Start Discussions
```csharp
// Instead of: "This is wrong"
// Try: "Have you considered using a different approach here? 
// What if we use a dictionary for O(1) lookups instead of a list?"

// Current implementation
public Wallet FindWallet(string id)
{
    return _wallets.FirstOrDefault(w => w.Id == id); // O(n)
}

// Suggested
private Dictionary<string, Wallet> _walletsById;

public Wallet FindWallet(string id)
{
    return _walletsById.TryGetValue(id, out var wallet) ? wallet : null; // O(1)
}
```

#### Categorize Your Comments
```markdown
**Critical**: Must be fixed before merge
🔴 Security vulnerability - private key is logged in plain text

**Important**: Should be fixed, but not blocking
🟡 Consider extracting this 50-line method into smaller methods

**Suggestion**: Nice to have
🟢 You might want to add a comment explaining this complex regex

**Question**: Seeking clarification
💬 Why did we choose this approach over using the existing utility method?

**Praise**: Positive feedback
👍 Great job handling all the edge cases here!
```

#### Provide Examples
```csharp
// Comment: "This could be simplified using LINQ"

// Current code
var confirmedTransactions = new List<Transaction>();
foreach (var tx in transactions)
{
    if (tx.Confirmations >= 6)
    {
        confirmedTransactions.Add(tx);
    }
}

// Suggested
var confirmedTransactions = transactions
    .Where(tx => tx.Confirmations >= 6)
    .ToList();
```

### Review Patterns and Anti-Patterns

#### ✅ Good Review Practices

1. **Start with the Big Picture**
```markdown
First review:
- Architecture and design decisions
- Overall approach
- Major patterns used

Then dive into:
- Implementation details
- Code style
- Minor issues
```

2. **Balance Criticism with Praise**
```markdown
👍 Great error handling here!
🟡 Consider caching this result to avoid repeated API calls
👍 Excellent test coverage
🔴 Security issue: Need to validate this input
👍 Clear and descriptive method names throughout
```

3. **Focus on Important Issues**
```csharp
// Don't nitpick formatting if tools can handle it
// Focus on:
// - Logic errors
// - Security issues
// - Performance problems
// - Maintainability concerns
```

#### ❌ Poor Review Practices

1. **Being Too Strict on Style**
```markdown
❌ "You should always use 'var' instead of explicit types"
✅ "Our team standard is to use 'var' for better readability. 
    Could you update this to match our style guide?"
```

2. **Not Explaining WHY**
```markdown
❌ "Don't do it this way"
✅ "This approach could cause a memory leak because the event handler 
    isn't being unsubscribed. Consider using a WeakReference or 
    unsubscribe in Dispose()"
```

3. **Reviewing Line by Line**
```markdown
❌ Commenting on every small issue
✅ Grouping similar issues: "There are several places where we should 
    validate input before processing (lines 23, 45, 67)"
```

## Review Process Workflow

### 1. Initial Review (15-30 minutes)
```markdown
1. Read the PR description
2. Understand the changes at a high level
3. Check if tests pass
4. Review architecture/design decisions
5. Leave high-level comments
```

### 2. Detailed Review (30-60 minutes)
```markdown
1. Review implementation details
2. Check for security issues
3. Verify test coverage
4. Look for performance concerns
5. Leave specific comments
```

### 3. Follow-up Review (10-15 minutes)
```markdown
1. Verify author addressed comments
2. Check new changes introduced
3. Approve or request additional changes
```

## Common Code Issues to Watch For

### 1. Bitcoin-Specific Issues
```csharp
// ❌ Not validating addresses
public void SendBitcoin(string address, decimal amount)
{
    _client.Send(address, amount);
}

// ✅ Validating addresses
public void SendBitcoin(string address, decimal amount)
{
    if (!BitcoinAddress.IsValid(address))
    {
        throw new ArgumentException("Invalid Bitcoin address");
    }
    
    _client.Send(address, amount);
}

// ❌ Storing private keys in plain text
public class Wallet
{
    public string PrivateKey { get; set; }
}

// ✅ Encrypting private keys
public class Wallet
{
    private string _encryptedPrivateKey;
    
    public void SetPrivateKey(string privateKey, IDataProtector protector)
    {
        _encryptedPrivateKey = protector.Protect(privateKey);
    }
}
```

### 2. Resource Management
```csharp
// ❌ Not disposing resources
public void ProcessFile(string path)
{
    var stream = File.OpenRead(path);
    // Use stream
    // Forgot to dispose!
}

// ✅ Using 'using' statement
public void ProcessFile(string path)
{
    using var stream = File.OpenRead(path);
    // Use stream
    // Automatically disposed
}
```

### 3. Null Reference Issues
```csharp
// ❌ Not checking for null
public decimal GetBalance(string walletId)
{
    var wallet = _repository.GetWallet(walletId);
    return wallet.Balance; // NullReferenceException if wallet is null
}

// ✅ Proper null checking
public decimal GetBalance(string walletId)
{
    var wallet = _repository.GetWallet(walletId);
    if (wallet == null)
    {
        throw new NotFoundException($"Wallet {walletId} not found");
    }
    return wallet.Balance;
}
```

### 4. Async/Await Issues
```csharp
// ❌ Blocking on async code
public Wallet GetWallet(string id)
{
    return _repository.GetWalletAsync(id).Result; // Can cause deadlocks
}

// ✅ Async all the way
public async Task<Wallet> GetWalletAsync(string id)
{
    return await _repository.GetWalletAsync(id);
}
```

### 5. Exception Handling
```csharp
// ❌ Swallowing exceptions
public void ProcessTransaction(Transaction tx)
{
    try
    {
        _service.Process(tx);
    }
    catch
    {
        // Silent failure - bad!
    }
}

// ✅ Proper exception handling
public void ProcessTransaction(Transaction tx)
{
    try
    {
        _service.Process(tx);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to process transaction {TxId}", tx.Id);
        throw;
    }
}
```

## Code Review Tools

### GitHub Pull Request Features
- **Review comments**: Line-by-line feedback
- **Suggestions**: Propose code changes directly
- **Review status**: Approve, Request Changes, Comment
- **Draft PRs**: Work in progress reviews

### Automated Tools
```bash
# Static analysis
dotnet tool install --global dotnet-format
dotnet format

# Security scanning
dotnet tool install --global security-scan
security-scan analyze

# Code metrics
dotnet tool install --global dotnet-code-metrics
```

## Review Time Guidelines

- **Small PR (< 100 lines)**: 15-30 minutes
- **Medium PR (100-400 lines)**: 30-60 minutes
- **Large PR (> 400 lines)**: Should be split into smaller PRs

## Communication Tips

### For Authors
```markdown
✅ "Thanks for catching that! I've updated the code."
✅ "Good point. I created issue #123 to track this for later."
✅ "I chose this approach because... What do you think?"

❌ "This is fine as-is."
❌ "We can fix that later."
❌ "That's not important."
```

### For Reviewers
```markdown
✅ "Great work on this feature! I have a few suggestions..."
✅ "Have you considered... ?"
✅ "This could be clearer if we..."

❌ "This is wrong."
❌ "Why did you do it this way?"
❌ "This code is terrible."
```

## Escalation Process

If there's disagreement:
1. **Discuss**: Have a conversation (video call if needed)
2. **Compromise**: Find middle ground
3. **Team decision**: Bring to team meeting
4. **Tech lead decision**: Final call if consensus isn't reached

## Metrics to Track

- **Review turnaround time**: Aim for < 24 hours
- **Comments per PR**: Track for patterns
- **Revisions per PR**: Too many might indicate unclear requirements
- **Defects found in review**: Measure review effectiveness

## Resources

- **Google Engineering Practices**: https://google.github.io/eng-practices/review/
- **Microsoft Code Review Guide**: Internal MS documentation
- **"Code Review Best Practices"**: Various articles and books

## Summary Checklist

### For Authors
- [ ] Self-review your code
- [ ] Write clear PR description
- [ ] Keep PR small and focused
- [ ] Run all checks before submitting
- [ ] Respond to comments promptly
- [ ] Be open to feedback

### For Reviewers
- [ ] Review promptly (within 24 hours)
- [ ] Start with big picture, then details
- [ ] Be specific and actionable
- [ ] Balance criticism with praise
- [ ] Focus on important issues
- [ ] Explain the "why" behind suggestions
- [ ] Use appropriate tone and language
