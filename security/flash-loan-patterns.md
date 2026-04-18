# Flash Loan Attack Patterns

## What are Flash Loans?

Flash loans allow borrowing any amount of assets without collateral, as long as the loan is repaid within the same transaction.

```solidity
// Flash loan lifecycle
function flashLoan(
    address receiver,
    uint256 amount,
    bytes calldata data
) external {
    // 1. Send loan
    transfer(receiver, amount);
    
    // 2. Execute callback
    IFlashLoanReceiver(receiver).execute(data);
    
    // 3. Verify repayment
    require(balanceOf(address(this)) >= amount + fee);
}
```

## Common Attack Vectors

### 1. Price Oracle Manipulation

Attack flow:
1. Borrow flash loan (e.g., 10,000 ETH)
2. Dump ETH on DEX to crash price
3. Exploit undervalued collateral on lending protocol
4. Buy back ETH at lower price
5. Repay flash loan
6. Profit = stolen collateral - price manipulation cost

### 2. Governance Attacks

1. Flash borrow governance tokens
2. Vote on critical proposal
3. Repay loan
4. Proposal passes with temporary voting power

### 3. Liquidation Cascades

1. Manipulate collateral price down
2. Trigger mass liquidations
3. Buy liquidated assets at discount
4. Manipulate price back up
5. Profit from arbitrage

## Detection Indicators

### On-Chain Patterns

```
[FLASH_LOAN_BORROW]
  ↓
[PRICE_IMPACTING_SWAP]
  ↓
[PROFIT_EXTRACTION]
  ↓
[FLASH_LOAN_REPAY]
```

### Red Flags

1. Large borrowed amount relative to protocol TVL
2. Multiple DEX swaps in single transaction
3. Liquidation events during transaction
4. Unusual gas usage (>500k)
5. Profit extraction to fresh address

## Historical Attacks

| Date | Protocol | Loss | Technique |
|------|----------|------|-----------|
| 2021-10 | Cream Finance | $130M | Price manipulation |
| 2021-11 | Indexed Finance | $16M | Flash loan + reentrancy |
| 2022-02 | Wormhole | $320M | Signature verification |
| 2022-03 | Ronin | $625M | Bridge compromise |
| 2022-04 | Beanstalk | $182M | Governance attack |

## Prevention Strategies

### 1. Time-Weighted Average Price (TWAP)

```solidity
function getPrice() public view returns (uint256) {
    // Use 30-minute TWAP instead of spot price
    return twapOracle.consult(token, amount);
}
```

### 2. Circuit Breakers

```solidity
modifier checkPriceImpact() {
    uint256 priceBefore = getPrice();
    _;
    uint256 priceAfter = getPrice();
    require(
        priceAfter > priceBefore * 95 / 100, 
        "Price impact too high"
    );
}
```

### 3. Withdrawal Delays

- Implement time delay for large withdrawals
- Force multi-block arbitrage

## References

1. [Flash Loans in Aave](https://docs.aave.com/faq/flash-loans)
2. [Flash Boys 2.0](https://arxiv.org/abs/1904.05234)
3. [Flash Loan Security](https://medium.com/cream-finance/flash-loan-security-8d8f5f1b8e89)

---

*Research by: @Ajatfnr21*
*Last updated: 2025-04-17*
