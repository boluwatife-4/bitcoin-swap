# Bitcoin-Swap: Decentralized AMM Protocol for Stacks Layer 2

A cutting-edge Automated Market Maker protocol natively integrated with Bitcoin via Stacks L2, enabling decentralized trading and liquidity provision with Bitcoin-compliant security.

## Key Features

- **Bitcoin-Native Design**  
  Built for Stacks Layer 2 with native Bitcoin settlement compatibility
- **Dynamic Fee Structure**  
  Configurable protocol fees (0-100%) with 0.3% default swap fee
- **Advanced Pool Protection**
  - Emergency pool pause/resume functionality
  - Slippage protection on all transactions
  - Precision-optimized calculations (6 decimals)
- **Liquidity Incentives**
  - Fair LP token minting algorithm
  - Proportional share-based rewards
  - Minimum return guarantees
- **Enterprise-Grade Security**
  - Principal-based access controls
  - Reserve validation checks
  - Atomic transaction handling

## Smart Contract Architecture

### Core Components

```clarity
(define-map pools {...})          ;; Registry of all liquidity pools
(define-map liquidity-providers {pool-id, provider} shares)  ;; LP positions
(define-data-var protocol-fee-rate u3000)  ;; 0.3% in basis points
```

### System Flow

1. **Pool Creation**  
   `create-pool` initializes new trading pairs (X/Y)
2. **Liquidity Management**
   - `add-liquidity`: Deposit assets & mint LP tokens
   - `remove-liquidity`: Burn LP tokens & withdraw assets
3. **Swapping Mechanism**  
   `swap-exact-tokens` executes trades using CPMM formula
4. **Fee Accrual**  
   Protocol fees accumulated in native token reserves

## Core Functions

### 1. Pool Management

**Create New Pool**

```clarity
(create-pool (token-x <ft-trait>) (token-y <ft-trait>))
```

- Initializes new trading pair
- Restricted to contract owner
- Prevents duplicate pools

**Emergency Controls**

```clarity
(pause-pool (pool-id uint))  ;; Freeze pool operations
(resume-pool (pool-id uint)) ;; Reactivate paused pool
```

### 2. Liquidity Operations

**Add Liquidity**

```clarity
(add-liquidity
  pool-id
  token-x
  token-y
  amount-x
  amount-y
  min-shares
)
```

- Requires proportional deposits
- Mints LP shares using geometric mean:
  ```math
  shares = min(Δx * totalShares / xReserve, Δy * totalShares / yReserve)
  ```

**Remove Liquidity**

```clarity
(remove-liquidity
  pool-id
  token-x
  token-y
  shares
  min-amount-x
  min-amount-y
)
```

- Proportional asset withdrawal:
  ```math
  amountX = (shares * xReserve) / totalShares
  amountY = (shares * yReserve) / totalShares
  ```

### 3. Swap Mechanism

**Execute Trade**

```clarity
(swap-exact-tokens
  pool-id
  token-in
  token-out
  amount-in
  min-amount-out
  x-to-y
)
```

- Implements CPMM formula:
  ```math
  output = (input * feeAdjusted * outputReserve) / (inputReserve * PRECISION + input * feeAdjusted)
  ```
- Slippage protection via `min-amount-out`

### 4. Fee Management

**Protocol Fee Adjustment**

```clarity
(set-protocol-fee (new-fee uint))  ;; Owner-restricted
```

- Fee range: 0% (0) to 100% (1000000)
- Fees collected in trade token

## Security Model

### Access Controls

- Critical functions restricted to `CONTRACT-OWNER`
- Multi-layered validation:
  ```clarity
  (asserts! (is-eq tx-sender CONTRACT-OWNER) ERR-NOT-AUTHORIZED)
  (asserts! (> amount-x u0) ERR-INVALID-AMOUNT)
  ```

### Protection Mechanisms

- Reentrancy protection via atomic transfers
- Reserve validation on liquidity removal
- Precision loss prevention through 6-decimal math
- Pool activity state checks

## Error Reference

| Error Code | Description                 |
| ---------- | --------------------------- |
| ERR-100    | Unauthorized access attempt |
| ERR-101    | Invalid parameter value     |
| ERR-102    | Insufficient balance        |
| ERR-103    | Nonexistent pool access     |
| ERR-104    | Token pair mismatch         |
| ERR-105    | Slippage beyond tolerance   |
| ERR-106    | Zero liquidity operation    |

## Usage Examples

**Create BTC/USDA Pool**

```clarity
(create-pool .bitcoin-token .usda-token)
```

**Add Liquidity**

```clarity
(add-liquidity
  0
  .bitcoin-token
  .usda-token
  u1000000   ;; 1.0 BTC
  u50000000  ;; 50.0 USDA
  u950000    ;; Minimum 950k shares
)
```

**Execute Swap**

```clarity
(swap-exact-tokens
  0
  .bitcoin-token
  .usda-token
  u100000    ;; 0.1 BTC
  u4800      ;; Min 4800 USDA
  true       ;; X->Y direction
)
```
