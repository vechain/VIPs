---
VIP: 242
Title: ETH Shanghai Upgrade Specification
Description: This VIP defines the specification of the EVM changes to align with Ethereum Shanghai upgrade.
Author: Tony Li (@libotony)
Discussions: https://vechain.discourse.group/t/add-vip-eth-shanghai-upgrade-specification/98
Category: Core
Status: Draft
CreatedAt: 2024-01-11
---

## Overview

This VIP specifies the changes required to align the EVM with the Shanghai release of Ethereum.

## Specification

Changes included in the Network Upgrade:

- [EIP-3198: BASEFEE opcode](https://eips.ethereum.org/EIPS/eip-3198)
- [EIP-3855: PUSH0 instruction](https://eips.ethereum.org/EIPS/eip-3855)
- [EIP-3541: Reject new contracts starting with the 0xEF byte](https://eips.ethereum.org/EIPS/eip-3541)
- [EIP-1108: Reduce alt_bn128 precompile gas costs](https://eips.ethereum.org/EIPS/eip-1108)
- [EIP-2565: ModExp Gas Cost](https://eips.ethereum.org/EIPS/eip-2565)

EIP-3198 introduces the `OP_BASEFEE` opcode, which is part of the [EIP-1559: Fee market change](https://eips.ethereum.org/EIPS/eip-1559) implemented in EVMs. However, since we have a different fee model, we will set `OP_BASEFEE` to 0. The other EIPs will remain unchanged.

## Reference Implementation

The reference implementation will be added once it is completed.

Copyright and related rights waived via [CC0](./LICENSE.md).
