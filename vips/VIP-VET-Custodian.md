---
VIP: 240
Title: VET Custodian Contract Standard
Description: A standard for smart contracts to store VET and distribute VTHO back to users fairly and automatically.
Author: Xiqing Chu <xiqing.chu@vechain.org>
Discussion: https://vechain.discourse.group/t/add-vip-vet-custodian-contract-standard/44
Category: Application
Status: Draft
CreatedAt: 2023-07-08
---

## 1. Abstract

This document outlines the standardized process for implementing a `StakingModel` in smart contracts and dapps, allowing them to accept VET and distribute VTHO back to depositors in a proportional manner.

This approach was initially introduced as part of **"wrapped VET"** VIP-180 tokens, which are tokens pegged 1:1 to VET. Holders of wrapped VET continue to receive benefits from VTHO generation, just as if they were holding VET.

The guidelines presented in this expository material aim to assist developers in comprehending this complex topic by providing a well-structured explanation.


## 2. Rationale

### Due-Token Model

Since the release of the VeChain blockchain's v1.0 whitepaper, two tokens have been in existence: the native token VET and its derivative VTHO. Unlike Ethereum, where transaction gas is paid with ETH, VTHO is utilized for this purpose in VeChain. 

VTHO is generated automatically for every account with a positive VET balance. The incremental of VTHO is solely depending on two facts: the amount of VET in the account and the duration of time.

However, different from the story of Ethereum, if one deposits VET to a smart contract, it loses the entitlement to VTHO because of the transfer of the ownership of his VET to the smart contract.

This can be solved by a smart contract design to monitor and credit VTHO to users in a timely manner. Therefor the users can deposit VET into such smart contract while keeping their VTHO interest.

### Use Cases

Most projects that aim to provide VTHO rebates to end users fall into this category.

A notable example is **Wrapped VET** (see [[VVET]](https://explore.vechain.org/accounts/0x45429a2255e7248e57fce99e7239aed3f84b7a53/transfer)), which is a VIP-180 token. Similar to its counterpart WETH (*Appendix<sup>(I)</sup>*), it accepts VET and creates a VVET token with a 1:1 peg. Currently it has guarded up to 10 million VET and tracks over 1.5 million VTHO. It has safely distributed 886k VTHO to end users, with over 80k transactions made to this smart contract.

Users of VVET can claim VTHO based on the duration of time and the amount of VET they have deposited into this smart contract.

Projects such as NFT market place, token exchange, cross-chain bridge can adopt a similar scheme to automatically manage VTHO tracking, similar to what is done by VVET.

## 3. Staking Model

A smart contract that aims to manage VET deposit from users and distribute VTHO is akin to the checking account service offered by banks. In this analogy, VET is the *currency*, and VTHO is the *interest*. The logic an be encapsulated within one library, `StakingModel.sol`. 

```
                               ┌──────────────────────────────┐
┌────────────┐                 │                              │
│            │                 │            dApp              │
│    User    │                 │                              │
│            │                 │      ┌─────────────────────┐ │
└─────┬──────┘                 │      │                     │ │
      │                        │      │   StakingModel.sol  │ │
      │          VET           │      │                     │ │
      ├───────────────────────►├──────┼────►                │ │
      │                        │      │                     │ │
      │                        │      │        VTHO         │ │
      │                        │      │      { Growth }     │ │
      │                        │      │        Logic        │ │
      │                        │      │                     │ │
      │      VTHO + VET        │      │                     │ │
      │◄───────────────────────┤◄─────┼─────                │ │
      │                        │      │                     │ │
      │                        │      └─────────────────────┘ │
      │                        │                              │
      │                        │                              │
                               └──────────────────────────────┘
```

Formula for a user's VTHO interest is:
```
VTHO = VET x GrowthRate x Time
```
Above calculation complicates when users dynamically add and remove VET over time.

The core concept of `StakingModel.sol` is that it accepts VET and tracks VTHO derivative over time. And correctly credit VTHO fairly to each user by contribution (amount of VET and the duration of time).

The library MUST address several key problems:

1. Bookkeeping user accounts.
2. Calculating VTHO generated on each account.
3. Functions to read the balance of users' VET and VTHO.
4. Functions to update/remove/add users' VET and VTHO.


### Data Structure

The basic data container is `User` structure which is stored in a mapping from user address to `User`. `User` is designed with a max length of `uint256`. It is composed of `balance`, `energy` and `lastUpdatedTime` three fields.

To prevent variables overload, following data types are used:
- `uint104` for `balance`, enough to store ALL VET in wei in VeChain.
- `uint104` for `energy`, enough to store VTHO in wei for the next 100+ years.
- `uint48` for `lastUpdatedTime`, sufficient to store 30,000+ years.


```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.0;

contract StakingModel {

    struct User {
        uint104 balance; // vet in wei
        uint104 energy;  // vtho in wei
        uint48 lastUpdatedTime; // timestamp
    }

    mapping(address => User) private users; // Main ledger

    modifier restrict(uint256 amount) { // Restriction of number input
        require(amount <= type(uint104).max, "value should <= type(uint104).max");
        _;
    }
}
```

### VTHO Calculation

Two supporting functions are needed to calculate and update VTHO balance for each user account. 

The `calculateVTHO` tells the exact amount of VTHO generated by a given amount of VET . It keeps the same generation rate specified by the VeChain blockchain `((vetAmount * 5) * (t2 - t1)) / (10**9)`. 

The `calculateVTHO` is a `public pure` function. It doesn't modify state.

The `_update` function adjusts the VTHO balance (energy) for a user from the last updated time until the present moment. It must be called whenever a user adds or removes VET/VTHO.

The `_update` is a helper function that modifies state. It should NOT be called randomly by external users. So it is marked as `internal`.

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.0;

contract StakingModel {

    // Calculate vtho generated between time t1 and t2
    // @param t1 Time in seconds
    // @param t2 Time in seconds
    // @param vetAmount VET in wei
    // @return vtho generated in wei
    function calculateVTHO(
        uint48 t1,
        uint48 t2,
        uint104 vetAmount
    ) public pure returns (uint104 vtho) {
        require(t1 <= t2, "t1 should be <= t2");
        return ((vetAmount * 5) * (t2 - t1)) / (10**9);
    }

    // Sync the vtho balance that the address has
    // up till current block (timestamp)
    function _update(address addr) internal {
        uint48 currentTime = uint48(block.timestamp);
        if (users[addr].lastUpdatedTime > 0) {
            assert(users[addr].lastUpdatedTime <= currentTime);
            users[addr].energy += calculateVTHO(
                users[addr].lastUpdatedTime,
                currentTime,
                users[addr].balance
            );
        }

        users[addr].lastUpdatedTime = currentTime;
    }
}
```
### Main Interfaces

`StakingModel.sol` deals with (tracks) the in and out of VET and VTHO. This section defines **six** fundamental interface functions.

Once user adds/removes VET, this model updates the VTHO balance by invoking `_update()` function. The add and remove functions are meant to be called by periphery Solidity code, so they are marked as `internal`.

```solidity
contract StakingModel {

    // Add VET
    function addVET(address addr, uint256 amount) restrict(amount) internal {
        _update(addr);
        users[addr].balance += uint104(amount);
    }

    // Remove VET
    function removeVET(address addr, uint256 amount) restrict(amount) internal {
        _update(addr);
        require(users[addr].balance >= uint104(amount), "insufficient vet");
        users[addr].balance -= uint104(amount);
    }

    // Read VET balance
    function vetBalance(address addr) public view returns (uint256 amount) {
        return users[addr].balance;
    }
}
```

Same as VET operations, users have the freedom to add/remove VTHO as desired. Although real-life programs may restrict VTHO addition as necessary. Both the add and remove operations are inteded to be called from other parts of periphery Solidity code, so they are marked as `internal` as well.

One trick is the `vthoBalance` function. We simply avoid using `_update()` inside but instead calculate the VTHO balance on the fly. By this we  avoid gas consumption to keep the function `view` only.

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.0;

contract StakingModel {

    // Add VTHO
    function addVTHO(address addr, uint256 amount) restrict(amount) internal {
        _update(addr);
        users[addr].energy += uint104(amount);
    }

    // Remove VTHO
    function removeVTHO(address addr, uint256 amount) restrict(amount) internal {
        _update(addr);
        require(users[addr].energy >= uint104(amount), "insufficient vtho");
        users[addr].energy -= uint104(amount);
    }

    // Read VTHO balance (Here is a trick to make it view only)
    function vthoBalance(address addr) public view returns (uint256 amount) {
        User memory user = users[addr];
        if (user.lastUpdatedTime == 0) {
            return 0;
        }
        return user.energy + calculateVTHO(user.lastUpdatedTime, uint48(block.timestamp), user.balance);
    }
}
```

## Integration with Your dApp

The final step is to integrate `StakingModel` to your dApp. One way to achieve this is through inheritance. To enable users to withdraw VTHO, we provide an example function `claimVTHO()`. The remaining of the code can be found in VVET example project (*Appendix<sup>(II)</sup>*).

**Claimable**
```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.0;

interface IVthoClaimable { // To withdraw VTHO
    function claimVTHO(address to, uint256 amount) external returns (bool);
}
```

**Energy**

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.0;

interface IEnergy { // Control interface of VTHO VIP-180 token.
	function balanceOf(address _owner) external view returns(uint256);
	function transfer(address _to, uint256 _amount) external returns(bool);
}
```

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity ^0.8.0;

contract YourApp is StakingModel, IVthoClaimable { // Important: inheritance
    address constant energyContractAddress = 0x0000000000000000000000000000456E65726779;
    
    // 'msg.sender' withdraw vtho, send to the receiver
    function claimVTHO(address to, uint256 amount) public override returns (bool) {
        removeVTHO(msg.sender, amount); // StakingModel
        IEnergy(energyContractAddress).transfer(to, amount); // VTHO
        emit ClaimVTHO(msg.sender, to, amount); // Event
        return true;
    }
}

```


## Security Considerations

### Audit
The old saying goes "*better safe than sorry*". There isn't such thing as being too careful. 

Developers can copy the audited `StakingModel.sol` from VVET project and include it to their projects. However, it is strongly recommended to conduct a thorough audit of the remaining code that is newly created.

### Autonomous
By default, the deployer of a smart contract assumes the role of the *master*, granting them the ability to extract VTHO from it (like a *superuser*). If the deployer wish to minimize the attack surface, it is essential to safeguard the deployer's private key.

Alternatively, to wavier the burden of a lost key, set master to `0x00...00` address. With `StakingModel.sol`, the entire operation can be automated without an admin.


## Appendix 
### (I) WETH on Ethereum

Long exists the Ether wrapper (see [[WETH9]](https://github.com/gnosis/canonical-weth/blob/0dd1ea3e295eef916d0c6223ec63141137d22d67/contracts/WETH9.sol)) that allows users to create a ERC-20 token named **WETH** that 1:1 pegs ETH. WETH is fully decentralized and can be redeemed back to ETH anytime by anyone. 

Modern fintech infrastructure, such as UniSwap, heavily relies on WETH as a fundamental building block to facilitate trades with other ERC-20 tokens. This approach greatly simplifies the logic in smart contracts by shifting away from dealing with both ETH and ERC-20 tokens to focusing solely on ERC-20 tokens. WETH acts as the fuel of the industry and flows seamlessly throughout the entire system.

### (II) VVET Example Project

The audited and running dapp of `StakingModel` is included in `VVET` project created by core VeChain developers. It ensures a fair distribution of VTHO. The source code is at [[github]](https://github.com/vechainlabs/vvet), and the deployed address on-chan is at [[0x45429a2255e7248e57fce99e7239aed3f84b7a53]](https://explore.vechain.org/accounts/0x45429a2255e7248e57fce99e7239aed3f84b7a53/transfer)