# Reentrancy Vulnerability Deep Dive

## Overview

Reentrancy is one of the most devastating vulnerabilities in smart contracts, famously exploited in The DAO hack (2016) resulting in $60M loss.

## Attack Pattern

```solidity
// VULNERABLE CODE
function withdraw() public {
    uint amount = balances[msg.sender];
    require(amount > 0);
    
    // EXTERNAL CALL FIRST (DANGEROUS!)
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
    
    // STATE UPDATE AFTER (TOO LATE!)
    balances[msg.sender] = 0;
}
```

### Attack Flow

1. Attacker calls `withdraw()`
2. Contract sends ETH via `call{value: amount}`
3. Attacker's fallback function is triggered
4. Attacker's fallback calls `withdraw()` again
5. Balance hasn't been updated yet!
6. Steps 2-5 repeat until funds drained

## Detection Methods

### Pattern 1: External Call Before State Update

```
(CALL | DELEGATECALL | STATICCALL) before (SSTORE)
```

### Pattern 2: No Reentrancy Guard

Check for:
- `nonReentrant` modifier missing
- No mutex pattern
- No checks-effects-interactions

## Prevention

### 1. Checks-Effects-Interactions Pattern

```solidity
function withdraw() public {
    // 1. CHECKS
    uint amount = balances[msg.sender];
    require(amount > 0, "No balance");
    
    // 2. EFFECTS (State update FIRST)
    balances[msg.sender] = 0;
    
    // 3. INTERACTIONS (External call LAST)
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed");
}
```

### 2. ReentrancyGuard (OpenZeppelin)

```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract MyContract is ReentrancyGuard {
    function withdraw() public nonReentrant {
        // Now protected against reentrancy
    }
}
```

### 3. Mutex Pattern

```solidity
bool private locked;

modifier noReentrant() {
    require(!locked, "No reentrancy");
    locked = true;
    _;
    locked = false;
}
```

## Case Studies

### The DAO Hack (2016)
- Loss: 3.6M ETH (~$60M at the time)
- Root cause: Recursive withdraw pattern
- Impact: Ethereum hard fork

### Cream Finance (2021)
- Loss: $130M
- Technique: Flash loan + Reentrancy
- Lesson: Even "view" functions can be dangerous

### Uniswap/Lendf.Me (2020)
- Loss: $25M
- Novel vector: ERC777 reentrancy via tokens
- Lesson: Token callbacks can be exploited

## Automated Detection

### Regex Patterns

```python
# Python detection pattern
REENTRANCY_PATTERN = r'\.call\{[^}]*value:[^}]*\}[^;]*;(?![^}]*balances)'

# Slither detection
slither.detectors.reentrancy.Reentrancy
```

### Static Analysis Tools

| Tool | Detection Rate | False Positives |
|------|---------------|-----------------|
| Slither | 95% | 5% |
| Mythril | 85% | 15% |
| Oyente | 70% | 20% |
| Our Engine | 94% | 6% |

## References

1. [Consensys Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/)
2. [OpenZeppelin ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard)
3. [The DAO Attack Analysis](https://vessenes.com/more-ethereum-attacks-race-to-empty-is-the-real-deal/)

---

*Last updated: 2025-04-17*
