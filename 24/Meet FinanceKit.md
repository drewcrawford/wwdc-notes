Learn how FinanceKit lets your financial management apps seamlessly and securely share on-device data from Apple Cash, Apple Card, and more, with user consent and control. Find out how to request one-time and ongoing access to accounts, transactions, and balances — and how to build great experiences for iOS and iPadOS.

### Check if financial data is available - 5:38
```swift
// Check if financial data is available
import FinanceKit

let available = FinanceStore.isDataAvailable( .financialData )

guard available else {
    // No meaningful action can be performed
    return
}
```

### Present the transaction picker - 8:08
```swift
// Insert code snippet.
```

### Requesting authorization for financial data - 12:16
```swift
// Requesting authorization for financial data
import FinanceKit

let store = FinanceStore.shared

guard store.isDataAvailable(for: .financialData) else {
    // No meaningful action can be performed
    return
}

let authStatus = await store.requestAuthorization()

guard authStatus == .authorized else {
    // User did not grant access to financial data, stop here
    return
}
```

### Simple query to retrieve all Apple accounts - 15:24
```swift
// Simple query to retrieve all Apple accounts
import FinanceKit

let store = FinanceStore.shared
let sortDescriptor = SortDescriptor(\Account.displayName)
let predicate = #Predicate<Account> { account in
    account.institutionName == "Apple"
}
let query = AccountQuery(
    sortDescriptors: [sortDescriptor],
    predicate: predicate
)

let accounts : [Account] = try await store.accounts(query: query)
```

### Get latest 7 available balances for account - 18:12
```swift
// Get latest 7 available balances for account
func getBalances(account: Account) async throws -> [AccountBalance] {
    let sortDescriptor = SortDescriptor(\AccountBalance.asOfDate, order: .reverse)
    let predicate = #Predicate<AccountBalance> { balance in
        balance.available != nil && balance.accountId == account.id
    }
    let query = AccountBalanceQuery(
        sortDescriptors: [sortDescriptor],
        predicate: predicate,
        limit: 7
    )
    
    return try await store.accountBalances(query: query).reversed()
}
```

### Retrieve all the transaction history for an account - 20:27
```swift
// Retrieve all the transaction history for an account
import FinanceKit

let store = FinanceStore.shared
let account: Account = ...

let transactionSequence = store.transactionHistory(forAccountID: account.id)

for try await change in transactionSequence {
    processChanges(change.inserted, change.updated, change.deleted)
}
```

### Use the history token to resume queries - 21:04
```swift
// Use the history token to resume queries
import FinanceKit

let store = FinanceStore.shared
let account: Account = ...
let currentToken = loadToken()

let transactionSequence = store.transactionHistory(
    forAccountID: account.id,
    since: currentToken
)

for try await change in transactionSequence {
    processChanges(change.inserted, change.updated, change.deleted)
    persist(token: change.newToken)
}
```

### Non monitoring resumable queries - 21:41
```swift
import FinanceKit

let store = FinanceStore.shared
let account: Account = ...
let currentToken = loadToken()

let transactionSequence = store.transactionHistory(
    forAccountID: account.id,
    since: currentToken,
    isMonitoring: false
)

for try await change in transactionSequence {
    processChanges(change.inserted, change.updated, change.deleted)
    persist(token: change.newToken)
}
```

# Resources
 * https://developer.apple.com/documentation/FinanceKit
* 