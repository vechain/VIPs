---
Title: VTHO Burn mechanism
Description: Introduce a mechanism to allow for the burning of VTHO tokens to reduce total supply and enhance tokenomics.
Author: Sebastian Debaillie <vecorgi@gmail.com>
Discussions: <URL>
Category: Core
Status: Idea
CreatedAt: 2023-12-09

---

## Overview

The introduction of two functions, burn and burnFrom, to the VTHO contract. These functions will enable users and smart contracts to destroy a specified amount of VTHO tokens, effectively reducing the total supply. This mechanism aims to provide a way to manage the VTHO supply dynamically, offering a new tool for vechain's tokenomics.


## Motivation

To provide a controlled mechanism for reducing the total supply of VTHO tokens. In blockchain ecosystems, the ability to burn tokens can be a vital tool for managing inflation, encouraging stability, and potentially increasing the value of remaining tokens. By introducing a standardized and secure method for burning VTHO, this aims to enhance the overall economic model of the vechain ecosystem.

## Rationale

The burn functions are based on OpenZeppelin's ERC20Burnable scheme but adapted to fit the built-in energy contract. 
  
## Specification

### Functons
```solidity
burn(uint256 amount)
```
Destroys `amount` tokens from `account`, reducing the total supply.
Emits a transfer event with to set `to` the zero address.

  - #### Requirements

    - `account` cannot be the zero address.

    - `account` must have at least `amount` tokens.

```solidity
burnFrom(address account, uint256 amount)
```
Destroys `amount` tokens from `account.amount` is then deducted from the caller’s allowance.

## Test Cases

### `burn(uint256 amount)`

#### 1. Normal Operation
- **Purpose**: Verify that tokens are successfully burned from an account.
- **Input**: Valid `amount` of tokens to burn from the caller's account.
- **Expected Output**: Successful burning of the specified amount, reduction in total supply, and emission of a transfer event with the `to` address set to the zero address.

#### 2. Insufficient Balance
- **Purpose**: Ensure function fails if the account does not have enough tokens.
- **Input**: `amount` greater than the account's balance.
- **Expected Output**: Function fails/reverts due to insufficient balance.

#### 3. Zero Address
- **Purpose**: Ensure function fails if called by the zero address.
- **Input**: Zero address calls the function with a valid `amount`.
- **Expected Output**: Function fails/reverts due to call from zero address.

#### 4. Zero Amount
- **Purpose**: Validate behavior when the burn amount is zero.
- **Input**: `amount` is set to zero.
- **Expected Output**: No change in total supply or account balance.

### `burnFrom(address account, uint256 amount)`

#### 1. Normal Operation
- **Purpose**: Verify tokens are burned from a specified account.
- **Input**: Valid `account` and `amount`, with sufficient allowance for the caller.
- **Expected Output**: Tokens are burned from the specified account, allowance is reduced accordingly, and total supply is decreased.

#### 2. Insufficient Allowance
- **Purpose**: Ensure function fails if the caller does not have enough allowance from the specified account.
- **Input**: `amount` exceeds the caller's allowance from `account`.
- **Expected Output**: Function fails/reverts due to insufficient allowance.

#### 3. Insufficient Balance
- **Purpose**: Ensure function fails if the specified account does not have enough tokens.
- **Input**: `amount` greater than the balance of the specified `account`.
- **Expected Output**: Function fails/reverts due to insufficient balance in the specified account.

#### 4. Zero Address
- **Purpose**: Validate the function's response to the zero address as the `account`.
- **Input**: Zero address specified for `account` with a valid `amount`.
- **Expected Output**: Function fails/reverts due to the zero address being used.

#### 5. Zero Amount
- **Purpose**: Check behavior when the burn amount is zero.
- **Input**: `amount` is set to zero with a valid `account`.
- **Expected Output**: No change in the account balance, total supply, or allowance.
  
## Reference Implementation

Modification to `energy.sol`
```solidity
function burn(uint256 amount) public returns(bool) {
    _burn(msg.sender, amount)
    return true;
}

function burnFrom(address account, uint256 amount) public returns(bool) {
    require(allowed[account][msg.sender] >= amount, "builtin: insufficient allowance");
    allowed[account][msg.sender] -= amount;
    _burn(account, amount)
    return true;
}

function _burn(address account, uint256 amount) internal {
    if (amount > 0) {
        require(EnergyNative(this).native_sub(account, amount), "builtin: insufficient balance");
    }
    emit Transfer(account, address(0), amount);
}
```

## Security Considerations

### Approval Frontrunning

This issue arises when a user submits multiple `approve` calls, creating a vulnerability that can be exploited by malicious actors. The scenario unfolds as follows:
1. **Initial Approval**: A user utilizes the `approve` function to grant a smart contract system permission to transfer `x` number of their ERC20 tokens (in this case, VTHO tokens).
2. **Change in Allowance**: Subsequently, the user decides to alter this allowance to a different amount `y`. To do this, they issue another `approve` request.
3. **Frontrunning Opportunity**: In the interim, before the user's transaction is confirmed and included in a block, an attacker can observe this pending transaction and exploit the situation. The attacker initiates the `transferFrom` function to withdraw `x` tokens from the user’s account.
4. **Execution Sequence**: If the attacker's transaction is processed and included in the blockchain before the user's new `approve` transaction, the attacker has effectively transferred `x` tokens. Once the user's `approve` transaction (with the updated allowance `y`) is also processed, the attacker can potentially exploit this to transfer an additional `y` tokens.
5. **Total Unauthorized Transfer**: The sum total of unauthorized transfer in such a scenario could be as high as `x + y` tokens, where `x` is the initial approved amount and `y` is the newly approved amount.

Copyright and related rights waived via [CC0](./LICENSE.md).
