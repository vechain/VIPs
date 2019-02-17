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

We will accomplish this by adding a new `gasPayerSignature` field encoded as index `0` of the `reserved` field.

The `gasPayer` can then be recovered from this field, similarly to the transaction signature as a whole.

MPP should be respected above a `gasPayerSignature`. If both exist, MPP should cover the gas fee. Only when neither a `gasPayerSignature` nor an MPP credit is available should the user pay for the gas.

# Expected Usage
- User constructs unsigned transaction
- User sends unsigned tx to one or many Gas Relayers
- Gas Relayer optionally attaches a fee via an additional transfer clause
- If acceptable, Gas Relayer signs the unsigned transaction (sans `gasPayerSignature` field) to create the `gasPayerSignature`
- Gas Relayer returns the `gasPayerSignature` and optional fee transfer clause to the end user
- User verifies the the added fee is acceptable, or chooses the lowest fee from multiple relayers
- User constructs final transaction including `gasPayerSignature`, signs, and then transmits to the node

**Note:** This process would be handled automatically by the wallet, dApp, or Connex/web3 provider.
### Generic Relayer
We expect a generic relayer to accept any transaction, while adding an additional clause to charge the user in VET or a VIP180 token to cover the cost of gas (as well as a fee for the service).

In this case wallets may plug in to a pool of gas relayers to provide lowest-cost service to their users such that users can send transactions without necessarily holding VTHO.

We expect a fee market to develop ensuring that the users recieve at-market price for fee delegation.

### Application Specific
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