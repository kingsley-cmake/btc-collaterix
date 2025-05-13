
# BTC-Collaterix — Bitcoin-Backed Lending Protocol on Stacks

BTC-Collaterix is a decentralized lending protocol built on the [Stacks blockchain](https://www.stacks.co/), enabling users to securely lock Bitcoin (BTC) as collateral and borrow STX. The protocol is designed with robust collateral management, dynamic interest calculations, and automated liquidation to ensure transparency, trust, and solvency.

## Key Features

* **Bitcoin-Backed Lending**: Use BTC as collateral to borrow STX in a trust-minimized, smart contract–governed environment.
* **Over-Collateralized Loans**: Protects protocol health with a configurable collateral ratio (default 150%).
* **Automatic Liquidations**: If the collateral value drops below the liquidation threshold (default 120%), positions are automatically liquidated to maintain solvency.
* **Dynamic Interest Calculation**: Interest accrues per block based on a configurable rate.
* **Governance Controls**: The contract owner can update key parameters like collateral ratio, liquidation threshold, and price feeds.
* **Transparent Accounting**: Publicly accessible stats on loan and platform data.

## Smart Contract Overview

* **Language**: Clarity (Stacks smart contract language)
* **Contract Name**: `btc-collaterix`

## Components

### Constants & Configuration

* `minimum-collateral-ratio`: Default `u150` (150%)
* `liquidation-threshold`: Default `u120` (120%)
* `platform-fee-rate`: Default `u1` (1%)
* `VALID-ASSETS`: Supported collateral assets (currently BTC & STX)

### Data Storage

* `loans`: Mapping of loan ID to loan data
* `user-loans`: Tracks user’s active loan IDs
* `collateral-prices`: Maintains real-time asset prices
* `platform-initialized`: One-time setup flag
* `total-btc-locked`: Tracks total BTC collateral
* `total-loans-issued`: Running count of loans

### Security Checks

* Access control via `tx-sender`
* Error codes for all failure conditions
* Value bounds for safe asset pricing

## Workflow

### Platform Initialization

```clojure
(initialize-platform)
```

Only the contract owner can call this. Must be called before any lending operations.

### Depositing Collateral

```clojure
(deposit-collateral (amount uint))
```

Deposits BTC (off-chain lock required) into the platform's accounting system.

### Requesting a Loan

```clojure
(request-loan (collateral uint) (loan-amount uint))
```

Locks BTC as collateral and issues an STX loan if the required collateral ratio is met.

### Repaying a Loan

```clojure
(repay-loan (loan-id uint) (amount uint))
```

Allows the borrower to repay the loan plus accrued interest. Releases BTC collateral upon repayment.

### 🛠 Governance

* `update-collateral-ratio`
* `update-liquidation-threshold`
* `update-price-feed`

These functions allow the contract owner to dynamically manage the platform's risk parameters.

## Read-Only Functions

* `get-loan-details (loan-id uint)`: Fetch details of a specific loan.
* `get-user-loans (user principal)`: Retrieve a list of a user’s active loans.
* `get-platform-stats`: Overview of the platform’s operational metrics.
* `get-valid-assets`: View all accepted collateral assets.

## Liquidation Process

A background process or off-chain bot should call `check-liquidation(loan-id)` periodically to evaluate whether a loan has fallen below the liquidation threshold. If so, `liquidate-position` is triggered internally.

## Technical Notes

* Interest is calculated using:

  ```
  interest = (principal * rate / (100 * blocksPerDay)) * elapsedBlocks
  ```
* All monetary values are assumed to be in smallest units (e.g., satoshis).
* Collateral and loans must be settled with off-chain integrations for BTC movement.

## Error Codes Reference

| Code   | Description             |
| ------ | ----------------------- |
| `u100` | Unauthorized            |
| `u101` | Insufficient collateral |
| `u102` | Below minimum value     |
| `u103` | Invalid amount          |
| `u104` | Already initialized     |
| `u105` | Not initialized         |
| `u106` | Invalid liquidation     |
| `u107` | Loan not found          |
| `u108` | Loan not active         |
| `u109` | Invalid loan ID         |
| `u110` | Invalid price           |
| `u111` | Invalid asset           |

## Example Use Case

1. **User A** deposits 1 BTC at \$50,000.
2. User A requests a loan for 20,000 STX (assuming 150% collateralization).
3. Interest accrues every block at 5% annualized.
4. If BTC price drops and the loan’s ratio hits 120%, it's auto-liquidated.
5. Once repaid, User A’s BTC is released.

## Contributing

Feel free to fork and improve this contract. Suggestions for better liquidation mechanics, improved interest models, or price feed integrations are welcome!
