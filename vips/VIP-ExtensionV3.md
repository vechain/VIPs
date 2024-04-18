---
Title: EVM Clause Context
Description: This VIP proposes an enhancement of the built-in extension contract to provide information about the clauses in the current transaction.
Author: @databeforedishonor and Darren Kelly (darren.kelly@vechain.org)
Discussions: https://vechain.discourse.group/t/mtt-induced-vulnerabilities-in-randomized-events/88
Category:  Core
Status: Draft
CreatedAt: 2024-04-18

---

## Overview

The below VIP-XXX proposal outlines an upgrade to the EVM, which will expose clause information to the smart contract, via the built-in `extension` contract
  
## Motivation

Smart contract developers may want to access information about the current clause being executed. For example, a smart contract can limit the number of clauses in case any malicious actor could exploit the contract via multiple clauses.

## Rationale

The design and decision to expose the clause context in the `extension` contract follows the same pattern as [VIP-191](https://github.com/vechain/VIPs/blob/master/vips/VIP-191.md). This time, the `extension` contract will migrate from V2 to V3 to provide the clause context information.
  
## Specification

The `extension` contract will be extended to provide the following functions:  

```solidity
/// @title ExtensionV3 extends EVM global functions.
contract ExtensionV3 is ExtensionV2 {

    /**
    * @dev Get the index of the current clause in the transaction.
    */
    function txClauseIndex() public view returns (uint) {
        return ExtensionV3Native(this).native_txClauseIndex();
    }

    /**
    * @dev Get the total number of clauses in the transaction.
    */
    function txClauseCount() public view returns (uint) {
        return ExtensionV3Native(this).native_txClauseCount();
    }
}

contract ExtensionV3Native is ExtensionV2Native {
    function native_txClauseCount() public view returns (uint);

    function native_txClauseIndex() public view returns (uint);
}
```

## Test Cases

### Test Case 1: Get the index of the current clause in the transaction

Given that `3341b04d` is the function signature of the `txClauseIndex` function, the following curl command should return the index of the current clause in the transaction.

#### Request 1

```bash
curl --request POST \
  --url 'http://localhost:8669/accounts/*' \
  --header 'Accept: application/json, text/plain' \
  --header 'Content-Type: application/json' \
  --data '{
  "clauses": [
    {
      "to": "0x0000000000000000000000457874656e73696f6e",
      "value": "0x0",
      "data": "0xa6dfac1a"
    },
    {
      "to": "0x0000000000000000000000457874656e73696f6e",
      "value": "0x0",
      "data": "0xa6dfac1a"
    }
  ]
}'
```

#### Response 1

```json
[
  {
    "data": "0x0000000000000000000000000000000000000000000000000000000000000001",
    ...other fields
  },
  {
    "data": "0x0000000000000000000000000000000000000000000000000000000000000002",
    ...other fields
  }
]
```

### Test Case 2: Get the total number of clauses in the transaction

Given that `c26ae5e1` is the function signature of the `txClauseCount` function, the following curl command should return the total number of clauses in the transaction:

#### Request 2

```bash
curl --request POST \
  --url 'http://localhost:8669/accounts/*' \
  --header 'Accept: application/json, text/plain' \
  --header 'Content-Type: application/json' \
  --data '{
  "clauses": [
    {
      "to": "0x0000000000000000000000457874656e73696f6e",
      "value": "0x0",
      "data": "0xa6dfac1a"
    },
    ...clause 2,
    ...clause 3
    ...clause 4
  ]
}'
```

#### Response 2

```json
[
  {
    "data": "0x0000000000000000000000000000000000000000000000000000000000000004",
    ...other fields
  },
  ...output 2,
  ...output 3
  ...output 4
]
```
  
Copyright and related rights waived via [CC0](./LICENSE.md).
