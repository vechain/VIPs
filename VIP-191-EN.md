---
VIP: 191
Title: Native Gas Delegation
Category: Core
Author: Kevin Britz <kevin@totientlabs.com>, Arjun Rao <arjun@totientlabs.com>
Status: Draft
CreatedAt: 2019-02-16
---

# Overview

The below VIP-191 proposal outlines an upgrade to the core transaction format of the Vechain blockchain to enable native gas delegation.

# Motivation

MPP covers a large subset of fee delegation use cases, however there exists some that cannot be accomplished through pure MPP such as:

- Multi-clause transactions where each clause is to a different contract
- Sponsoring a transaction for a specific operation where the sponsor is not the master of the contract

We would like we propose a complimentary protocol to generically allow a **Gas Relayer** to pay transaction fees for specific transactions.


# Specification

We will accomplish this by adding a new `gasPayerSignature` field encoded as an index of the `reserved` field.

The `gasPayer` can then be recovered from this field, similarly to the transaction signature as a whole.

MPP should be respected above a `gasPayerSignature`. If both exist, MPP should cover the gas fee. Only when neither a `gasPayerSignature` nor an MPP credit is available should the user pay for the gas.

## Replay Protection

Consider the attack where a user submits a transaction to a gas relayer for signature. The relayer verifies and signs this single transaction as acceptable. The end user then replicates and signs this transaction with multiple addresses. All replications (one per address) will be able to be successfully submitted even though the gas relayer may have only intended a single transaction to be placed.

To solve this issue we propose two solutions, each with their own tradeoffs.

### Option 1: Add `origin` as transaction field

Generally speaking including `origin` in the transaction message is considered redundant since it can be recovered from the `signature`, however including it for VIP-191 transactions will ensure that the gas relayer knows and agrees to a specific address sending the signed transaction.

#### `reserved` field update
| Index | Field Name | Type |
| --- | --- | --- |
| 0 | origin |  address |
| 1 | gasPayerSignature |  byte[65] |

From a node implementation perspective, this method is also much simpler as it only requires adding a check to verify that `origin` matches the transaction `signature`.

A drawback of this option is that the relayer then knows who the sender is which could be used as information to censor a transaction. This issue is relatively minor because a sender should be able to find a willing relayer in a sufficiently large pool, but still worth considering.

### Option 2: Add `gasPayerNonce`

Alternatively, we may introduce a `gasPayerNonce` which will ensure that a `gasPayer`-signed transaction may only be used once by a single address.

#### `reserved` field update
| Index | Field Name | Type |
| --- | --- | --- |
| 0 | gasPayerNonce |  uint256 |
| 1 | gasPayerSignature |  byte[65] |

This method will require more significant updates to the node codebase as we will need to keep track of the used `gasPayerNonce` set for each `gasPayer`. This can be accomplished similarly to how we store the commited `transactionId` map. This would result in a slightly larger transaction payload than Option (1), while also using up more memory in the node to store the `gasPayerNonce` map.

As discussed later, since `origin` is not part of the transaction, we get the benefit of more censor-resistant transactions.

# Expected Usage
With VIP-191 we expect at least two types of Gas Relayers to emerge: `Generic Relayers` and `Application-Specific Relayers`. Both types of relayers will consist of a centralized service that takes in unsign transactions via an API and returns `gasPayer`-signed transactions per their internal instruction sets.

### Generic Relayer
A `Generic Relayer` is a Gas Relayer that accepts all transactions for delegation and in exchange attaches an additional transfer clause to the transaction which charges a fee in whichever token the end user prefers. This allows an end user to still pay for their transaction fee, but to pay in another currency such as `VET` or a stablecoin. 

In this case wallets may plug in to a pool of Generic Relayers (ideally with a standardized API) to provide lowest-cost service to their users such that users may send transactions using whichever currency they may hold. Following this we expect a fee market to develop ensuring that the users receive at-market price for fee delegation.

#### Experience
- Wallet receives unsigned transaction for signature from dApp
- Wallet checks its VTHO balance (or MPP credit), sees it doesn't not have enough VTHO but does have enough of X token
- Wallet sends unsigned transaction to one or many Generic Relayers
- Each Generic Relayer attaches what it deems a reasonable fee in X token via an additional transfer clause and signs
- Relayer returns the `gasPayer`-signed transaction to the Wallet
- Wallet selects the `gasPayer`-signed transaction that is both correct and has the lowest fee
- Wallet signs and broadcasts selected transaction

**Note:** In this process the Wallet (such as Sync or Comet) will handle this process for each transaction as a generic feature

### Application Specific Relayer
In addition to Generic Relayers we also forsee a case where dApps themselves would like to operate a Gas Relayer. This would grant the ability for dApps to pay fees on behalf of their users for specific transactions (similar but more extensible than MPP).

Consider the case where Wrapped VET must be used within a dApp's flow. The dApp does not own the contracts for Wrapped VET, however would like to pay for the wrapping and approving process in relation to usage with its platform. This cannot be done with pure MPP, however with this proposal the dApp would be able to create a relayer that covers the transaction fees in specifically those scenarios for free.

#### Experience
- dApp frontend messages dApp-backend to say that it would like to send a specified transaction that is `gasPayer`-signed by the dApp
- dApp-backend authenticates request and applies any other risk checks before `gasPayer`-signing the transaction (without adding a fee) and returning to the dApp-frontend
- dApp-frontend sends the `gasPayer`-signed transaction to the Wallet
- Wallet sees that the transaction is already `gasPayer`-signed and skips all internal gas checks (balance, MPP, or Generic Relayer)
- Wallet signs and broadcasts transaction

**Note:** In this process the dApp (such as CometVerse) will handle this process for each `gasPayer`-signed transaction, only needing the wallet to handle recognizing the present signature for proper UX.

### Consorship
The main issue with gas relayers is censorship: relayers may choose not to relay your transaction. By flipping this model we are able to collect quotes from many relayers, and only then choose to transmit the transaction ourselves. With this, censorship is only possible if none of the relayers sign the transaction, which with a sufficiently large and decentralized pool is increasingly unlikely. Further, (only if we select Option 2 for Replay Protection) since the transactions are still unsigned, there is no way to deduce the sender, further obfuscating potential information that could be used to censor.

# Implementation

Pseudocode:
```
gasPayer = user

if exists(tx.gasPayerSignature()):
    gasPayer = recover(tx, tx.gasPayerSignature())

if isMPPed(tx):
    gasPayer = sponsor
```

Full implementation: TBD