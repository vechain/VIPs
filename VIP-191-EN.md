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
With VIP-191 we expect at least two types of Gas Relayers to emerge: `Generic Relayers` and `Application-Specific Relayers`. Both types of relayers will consist of a centralized service that takes in unsign transactions via an API and returns `gasPayer`-signed transactions per its internal instruction set.

### Generic Relayer
We expect a generic relayer to accept any transaction, while adding an additional clause to charge the user in VET or a VIP180 token to cover the cost of gas (as well as a fee for the service).

In this case wallets may plug in to a pool of gas relayers to provide lowest-cost service to their users such that users can send transactions without necessarily holding VTHO.

We expect a fee market to develop ensuring that the users recieve at-market price for fee delegation.

#### Experience
- User constructs unsigned transaction
- User sends unsigned tx to one or many Gas Relayers
- Gas Relayer optionally attaches a fee via an additional transfer clause
- If acceptable, Gas Relayer signs the unsigned transaction (sans `gasPayerSignature` field) to create the `gasPayerSignature`
- Gas Relayer returns the `gasPayerSignature` and optional fee transfer clause to the end user
- User verifies the the added fee is acceptable, or chooses the lowest fee from multiple relayers
- User constructs final transaction including `gasPayerSignature`, signs, and then transmits to the node

**Note:** This process would be handled automatically by the wallet, dApp, or Connex/web3 provider.

### Application Specific Relayer
In addition to generic relayers this also gives dApps more flexibility in paying for their own transactions.

Consider the case where Wrapped VET must be used within a dApp's flow. The dApp does not own the contracts for Wrapped VET, however would like to pay for the wrapping and approving process in relation to usage with its platform. This cannot be done with pure MPP, however with this proposal the dApp would be able to create a relayer that covers the transaction fees in specifically those scenarios for free.


#### Consorship
The main issue with gas relayers is censorship: relayers may choose not to relay your transaction. By flipping this model we are able to collect quotes from many relayers, and only then choose to transmit the transaction ourselves. With this, censorship is only possible if none of the relayers sign the transaction, which with a sufficiently large and decentralized pool is increasingly unlikely. Further since the transactions are still unsigned, there is no way to deduce the sender, further obfuscating potential information that could be used to censor.

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