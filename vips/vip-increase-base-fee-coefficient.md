---
Title: Increase the base fee coefficient
Description: This VIP proposes to increase the base fee coefficient range from 0-255 (2^8) to a larger range of 0-65535 (2^16)
Author: Darren Kelly (darren.kelly@vechain.org)
Discussions: TODO
Category:  Core
Status: Draft
CreatedAt: 2024-04-26

---

## Overview

Currently the gas price coefficient range is from 0-255, which limits the range of values that can be used. This VIP proposes to increase the base fee coefficient to a `uint16` to allow for a larger range of values.

## Motivation

Client's of the blockchain can ensure that their transactions receive priority by setting a higher gas price coefficient. However, the current range of values is limited to 0-255. This VIP proposes to increase the base fee coefficient to a uint16 to allow for a larger range of values.

## Specification
  
The implementation will change the `GasPriceCoef` of the transaction types from `uint8` to `uint16` in transaction models.

## Test Cases
  
Build, sign and send a transaction with a gas price coefficient of 65535. The transaction should be included in the next block.

Copyright and related rights waived via [CC0](./LICENSE.md).
