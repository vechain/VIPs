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

We will accomplish this by optionally expanding the `signature` field to contain an additional `gasPayerSignature` concatenated with the original signature, and by including the `origin` in an index of `reserved` for replay protection.

To construct a valid VIP-191 transaction, the sender must include `origin` and `gasPayerSignature`, however they are both otherwise optional to preserve backwards compatibility.

#### `signature` field update
| Bytes | Attribute | Type | Optional
| --- | --- | --- | --- |
| 0-64 | `originatorSignature` |  byte[65] | no |
| 65-129 | `gasPayerSignature` |  byte[65] | yes |

The `gasPayer` can then be recovered from this additional `signature` data, similarly to the originator signature.

#### `reserved` field update
| Index | Attribute | Type | Optional |
| --- | --- | --- | --- |
| 0 | `origin` |  address | yes |

Adding `origin` as a `reserved` field allows us to verify that the `gasPayer` is aware of the sender at time of signature, and that only a single sender may send the eventual transaction. It also allows the `gasPayer` to verify that an attacker has not tricked it into signing a transaction from itself. In order for the VIP-191 transaction to be valid, the `origin` must be present and match the originator signature. This replay attack is discussed further in [#Replay Protection Discussion](#Replay-Protection-Discussion).

#### Transaction hash
The transaction hash calculation may be unchanged since only the `tx.Signer()` is incorporated rather than the full signature.

Since the `gasPayer` information is not part of the payload signed by the sender, we should not incorporate any of this information in the transaction hash. Doing so could arbitrarily allow an attacker to resend a signed transaction by changing the `gasPayer`.

#### Interaction with MPP

MPP should be respected above a `gasPayerSignature`. If both exist, MPP should cover the gas fee. Only when neither a `gasPayerSignature` nor an MPP credit is available should the user pay for the gas.

## Replay Protection Discussion

Consider the attack where a user submits a transaction to a gas relayer for signature. The relayer verifies and signs this single transaction as acceptable. The end user then replicates and signs this transaction with multiple addresses. All replications (one per address) will be able to be successfully submitted even though the gas relayer may have only intended a single transaction to be placed.

### Chosen Option: Add `origin` as transaction field

Generally speaking including `origin` in the transaction message is considered redundant since it can be recovered from the `signature`, however including it for VIP-191 transactions will ensure that the gas relayer knows and agrees to a specific address sending the signed transaction.

#### `reserved` field update
| Index | Field Name | Type |
| --- | --- | --- |
| 0 | origin |  address |

From a node implementation perspective, this method is also much simpler as it only requires adding a check to verify that `origin` matches the transaction `signature`.

A drawback of this option is that the relayer then knows who the sender is which could be used as information to censor a transaction. This issue is relatively minor because a sender should be able to find a willing relayer in a sufficiently large pool, but still worth considering.

### Alternative Discussed: Add `gasPayerNonce`

Alternatively, we may introduce a `gasPayerNonce` which will ensure that a `gasPayer`-signed transaction may only be used once by a single address.

#### `reserved` field update
| Index | Field Name | Type |
| --- | --- | --- |
| 0 | gasPayerNonce |  uint256 |

This method will require more significant updates to the node codebase as we will need to keep track of the used `gasPayerNonce` set for each `gasPayer`. This can be accomplished similarly to how we store the commited `transactionId` map. This would result in a slightly larger transaction payload than Option (1), while also using up more memory in the node to store the `gasPayerNonce` map.

Since `origin` is not part of the transaction, we get the benefit of more censor-resistant transactions.

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
The main issue with gas relayers is censorship: relayers may choose not to relay your transaction. By flipping this model we are able to collect quotes from many relayers, and only then choose to transmit the transaction ourselves. With this, censorship is only possible if none of the relayers sign the transaction, which with a sufficiently large and decentralized pool is increasingly unlikely.

# Implementation

#### Thor Logic
```
gasPayer = user

if tx.hasGasPayerSignature():
    if tx.Signer() != tx.Origin():
        revert
    gasPayer = recover(tx, tx.gasPayerSignature())

if isMPPed(tx):
    gasPayer = tx.Sponsor()
```

#### Thorify (web3) update
We'll need a way to pass in a `gasPayerSignature` optionally into a web3 transaction send call to support the application-specific relayer case.

We can accomplish this by allowing `gasPayerSignature` to be passed in as an optional parameter similar to `from`.

#### Connex
Similarly Connex will need to support this optional parameter in its transaction send calls.

Exact implementation TBD

### Full implementation

TBD