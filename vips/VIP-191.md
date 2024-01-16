---
VIP: 191
Title: Designated Gas Payer
Category: Core
Author: Kevin Britz <kevin@totientlabs.com>, Arjun Rao <arjun@totientlabs.com>
Status: Final
CreatedAt: 2019-02-16
---

# Overview

The below VIP-191 proposal outlines an upgrade to the core transaction format of the Vechain blockchain to enable transactions to designate a gas payer distinct from the sender.

# Motivation

MPP covers a large subset of fee delegation use cases, however there exists some that cannot be accomplished through pure MPP such as:

- Multi-clause transactions where each clause is to a different contract
- Sponsoring a transaction for a specific operation where the sponsor is not the master of the contract

We would like we propose a complimentary protocol to generically allow a **gas payer** to cosign any specific transaction to pay for its gas fee on behalf of the sender.


# Specification

We will accomplish this by optionally expanding the `signature` field to contain an additional `gasPayerSignature` concatenated with the sender signature, and by including `isGasPaid` as an index of `reserved` for replay protection.

To construct a valid VIP-191 transaction, the sender must include `isGasPaid` and `gasPayerSignature`, however they are both otherwise optional to preserve backwards compatibility. If only `isGasPaid` or `gasPayerSignature` is included, the transaction should be reverted as it contains redundant data.

#### `signature` field update
| Bytes | Attribute | Type | Optional
| --- | --- | --- | --- |
| 0-64 | `senderSignature` |  byte[65] | no |
| 65-129 | `gasPayerSignature` |  byte[65] | yes |

The gas payer can then be recovered from this additional `signature` data, similarly to the sender signature.

#### `gasPayerSignature` Specification
```
payload = blake2b256(blake2b256(encodedUnsignedTx), origin)  // Same as txId computation
gasPayerSignature = sign(payload, gasPayer)
```

#### `reserved` field update
| Index | Attribute | Type | Optional |
| --- | --- | --- | --- |
| 0 | `isGasPaid` |  boolean | yes |

Adding `isGasPaid` as a `reserved` field allows us to signal to the VM that this transaction should be paid for by an external gas payer. Because `origin` is part of the payload that the gas payer signs, the gas payer may verify that it is not being tricked into signing a transaction from itself. This method also allows either the gas payer or the sender to sign the transaction first as the gas payer only needs to know the `origin` and not the full sender signature to sign.

Because the sender may sign first or last, both delegation and relaying patterns are possible depending on the developer's use case.

#### Transaction hash
The transaction hash calculation may be unchanged since only the `tx.Signer()` is incorporated rather than the full signature.

Since the gas payer information is not part of the payload signed by the sender, we should not incorporate any of this information in the transaction hash. Doing so could arbitrarily allow an attacker to resend a signed transaction by changing the gas payer.

#### Interaction with MPP

MPP should be respected above a `gasPayerSignature`. If both exist, MPP should cover the gas fee. Only when neither a `gasPayerSignature` nor an MPP credit is available should the user pay for the gas.

### Alternative Discussed

#### `reserved` field update
| Index | Attribute | Type | Optional |
| --- | --- | --- | --- |
| 0 | `origin` |  address | yes |

Adding `origin` as a `reserved` field was initially discussed as a solution to the replay protection issue. It was deemed redundant information as only the `gasPayerSignature` payload must include the `origin`, not the transaction itself. The chosen specification provides the same solution with 1 byte instead of 20.

## Replay Protection Discussion

Two replay attacks exist from modification of the signature, either by changing the sender's signature or the gas payer's.

#### Sender-based attack
- Consider the attack where a user submits a transaction to a gas payer for signature. The payer verifies and signs this single transaction as acceptable. The end user then replicates and signs this transaction with multiple addresses. All replications (one per address) will be able to be successfully submitted even though the gas payer may have only intended a single transaction to be placed.

This is mitigated by including the `origin` in the payload that is signed by the gas payer. By doing so it is ensured that the produced `gasPayerSignature` is only valid if the transaction is signed by the specified `origin`.

#### Gas payer-based attack
- Take an already signed and committed transaction. By simply removing the gas payer's signature or replacing the gas payer's signature with that of a different payer, we can resubmit a new transaction that will duplicate the original intended function.

Since the transaction hash does not include any of the information about the gas payer, resubmitting the transaction with a new gas payer would yield an equivalent hash, thus rejecting it. Our `isGasPaid` field prevents an attacker from removing from or adding to a transaction since this field is included in the transaction that is signed by the sender.

# Example Usage
With VIP-191 we expect at least two types of gas payers to emerge: `Generic Delegators` and `Application Sponsors`. Both types of payers will consist of a centralized service that takes in unsign transactions via an API and returns gas-payer-signed transactions per their internal instruction sets.

### Application Sponsors
This will grant the ability for dApps to pay fees on behalf of their users for specific transactions (similar but more extensible than MPP).

Consider the case where `VET+` must be used within a dApp's flow. The dApp does not own the contracts for `VET+`, however would like to pay for the wrapping and approving process in relation to usage with its platform. This cannot be done with pure MPP, however with this proposal the dApp would be able to pay for transaction fees in specifically those scenarios.

#### Experience
![image](https://user-images.githubusercontent.com/747165/54079714-bb839080-4296-11e9-8112-613508130679.png)
- dApp frontend messages dApp-backend to say that it would like to send a specified transaction that is gas-payer-signed by the dApp
- dApp backend authenticates request and applies any other risk checks before gas-payer-signing the transaction (without adding a fee) and returning to the dApp frontend
- dApp frontend sends the gas-payer-signed transaction to the Wallet
- Wallet sees that the transaction is already gas-payer-signed and skips all internal gas checks (balance, MPP, or Generic Delegator)
- Wallet signs and broadcasts transaction

**Note:** In this process the dApp (such as CometVerse) will handle this process for each gas-payer-signed transaction, only needing the wallet to handle recognizing the present signature for proper UX.

### Generic Delegator
A `Generic Delegator` is a third-party service that accepts all transactions for delegation and in exchange attaches an additional transfer clause to the transaction which charges a fee in whichever token the end user prefers. This allows an end user to still pay for their transaction fee, but to pay in another currency such as `VET` or a stablecoin. 

In this case wallets may plug in to a pool of Generic Delegators (ideally with a standardized API) to provide lowest-cost service to their users such that users may send transactions using whichever currency they may hold. Following this we expect a fee market to develop ensuring that the users receive at-market price for fee delegation.

#### Experience
![image](https://user-images.githubusercontent.com/747165/54079713-b3c3ec00-4296-11e9-9ade-a893432b66fe.png)
- Wallet receives unsigned transaction for signature from dApp
- Wallet checks its VTHO balance (or MPP credit), sees it does not have enough VTHO but does have enough of X token
- Wallet sends unsigned transaction to one or many Generic Delegators
- Each Generic Delegator attaches what it deems a reasonable fee in X token via an additional transfer clause and signs
- Delegator returns the gas-payer-signed transaction to the Wallet
- Wallet selects the gas-payer-signed transaction that is both correct and has the lowest fee
- Wallet signs and broadcasts selected transaction

**Note:** In this process the Wallet (such as Sync or Comet) will handle this process for each transaction as a generic feature

# Implementation

#### Thor Logic
```
gasPayer = user

if tx.hasGasPayerSignature() && !tx.reserved[0]:
    revert()
        
if tx.reserved[0]:
    if !tx.hasGasPayerSignature():
        revert()
    else:
        gasPayer = recover(tx.id(), tx.gasPayerSignature())
        if gasPayer == tx.Signer():
            revert()

 
if isMPPed(tx):
    gasPayer = tx.Sponsor()
```

#### Thorify (web3) and Connex update
In order to support application sponsors, we will need to add the option of passing in a `gasPayerSignature` to web3 when sending a transaction.
