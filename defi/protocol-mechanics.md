# DeFi Protocol Mechanics

## Lending Protocols (Aave, Compound)

### Core Mechanism

1. **Supply Assets** → Receive cTokens/aTokens
2. **Borrow Assets** → Must supply collateral
3. **Interest Rates** → Algorithmic based on utilization
4. **Liquidation** → When health factor < 1

### Health Factor Calculation

```
Health Factor = (Collateral Value × Liquidation Threshold) / Borrow Value

Liquidate when: HF < 1.0
```

### Interest Rate Model

```
Borrow Rate = Base Rate + (Utilization × Slope)

Where:
  Utilization = Total Borrows / Total Liquidity
```

## AMM Mechanics (Uniswap, Curve)

### Constant Product (x × y = k)

```solidity
// Uniswap V2 core formula
function getAmountOut(
    uint amountIn,
    uint reserveIn,
    uint reserveOut
) public pure returns (uint amountOut) {
    uint amountInWithFee = amountIn * 997;
    uint numerator = amountInWithFee * reserveOut;
    uint denominator = reserveIn * 1000 + amountInWithFee;
    amountOut = numerator / denominator;
}
```

### Impermanent Loss

```
IL = 2√(price_ratio) / (1 + price_ratio) - 1

Example:
  Price doubles (2x) → IL = 5.7%
  Price 4x → IL = 20%
  Price 10x → IL = 42%
```

## Yield Aggregators (Yearn, Convex)

### Strategy Lifecycle

1. **Deposit** → Funds allocated to strategy
2. **Harvest** → Yield claimed and reinvested
3. **Withdraw** → Funds + yield returned

### Key Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| APY | Annual yield | >10% |
| TVL | Total value locked | Growing |
| Utilization | Active vs idle | >70% |

---

*Part of Web3 Research Notes collection*
