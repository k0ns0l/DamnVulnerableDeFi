# [Challenge #1: Unstoppable](https://www.damnvulnerabledefi.xyz/challenges/unstoppable/)
---
## Objective
The goal is to break the flash loan feature so that the monitor contract will pause the vault and transfer ownership back to the deployer.

## Analysis
The `UnstoppableVault` contract is an ERC4626-compliant vault that allows flash loans and can be paused by its owner (the `UnstoppableMonitor` contract). The monitor contract checks the health of the vault's flash loan feature and pauses the vault if a flash loan fails.

**Vulnerability:** 
The vault assumes its internal accounting (`convertToShares(totalSupply)`) always matches its actual token balance (`totalAssets()`). Though anyone can transfer tokens directly to the vault contract, bypassing the intended deposit logic. 
This causes the vault's balance to become inconsistent with its accounting, triggering the `InvalidBalance()` revert in the `flashLoan` function. As a result, the monitor contract detects the failure, pauses the vault, and returns ownership to the deployer.
[ERC4626 accounting mismatch]

## Solution
High-level approach to exploit the vulnerability.

### Attack Steps
- By transferring *at least* `1 WEI` directly to the vault, you bypass the vault's deposit logic
- This causes the vault's internal accounting (`totalAssets()`) to become inconsistent with its actual token balance
- When the monitor tries to perform a flash loan, the vault's checks fail (because of the accounting mismatch), causing the flash loan to revert
- The monitor contract then:
    - Emits `FlashLoanStatus(false)`
    - Pauses the vault
    - Transfers ownership to the deployer

### Implementation
```Unstoppable.t.sol
function test_unstoppable() public checkSolvedByPlayer {
        token.transfer(address(vault), 1 wei);
}
```

## Takeaway
Direct token transfers to vault contracts can often times break internal accounting assumptions, leading to critical issues. It is thus advised, to ensure vault logic accounts for unexpected token movements to maintain protocol integrity

---