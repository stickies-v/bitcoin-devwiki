# Improving Large Wallet Performance

Users who have large wallets often experience some performance issues. These are largely apparent when loading the wallet, fetching the wallet balances, sending Bitcoin, and looking at the transactions list. The root cause of these issues is typically the number of transactions that are present in the wallet.

The number of transactions can have a significant effect on the wallet's performance because the wallet will load all transactions into memory and it will iterate all of the transactions to perform certain tasks. Generally, solving this will require not loading all transactions into memory and having a way to calculate balances without iterating all transactions.

## Goals

* Load transactions only when they are needed (i.e. when being inspected)
* Avoid iterating all records in the database
* Store UTXOs in the wallet and use them instead of figuring them out by iterating all transactions

## The Plan

1. In the GUI, change the transactions list to a paginated one so that not all transactions need to be in memory.
2. Store UTXOs in the wallet
3. Use stored UTXOs for coin selection and balance calculation
4. Do not store all transactions in memory. Read them from disk when needed. During loading, their records can still be iterated
5. Change loading to be looking for specific records rather than iterating all of them.