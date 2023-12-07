---
Title: Support CCIP for Connex
Description: CCIP adds trusted off-chain-sources
Author: Mario Micklisch (@ifavo)
Discussions: https://vechain.discourse.group/t/add-vip-support-ccip-for-connex/85
Category:  Core
Status: Idea
CreatedAt: 2023-12-07

---

## Overview

[ERC-3668: CCIP Read: Secure offchain data retrieval](https://eips.ethereum.org/EIPS/eip-3668) suggests a mechanism that is used widely on the ETH ecosystem to allow a contract to fetch external data.
Providing the functionality as option for connex based interactions with VechainThor improves data access.
 
## Rationale

Short lived data or data available at places not accessable via contracts (other chains, web, etc.) is made accessable using CCIP reading.
By supporting ERC-3668 Connex will improve developer experience, following a widely used standard and simplify the process of accessing off-chain-information.

With CCIP support off-chain-sources can be shared between different platforms, potentially opening new potentials for dApp developers.
  
## Specification

Connex has an option to enable CCIP as feature.

**When enabled**, it checks errors from contract calls for the signature of:

```sol
error OffchainLookup(address sender, string[] urls, bytes callData, bytes4 callbackFunction, bytes extraData)
```

If detected, it follows the ERC-3668 process to lookup the data and continue with the contract call.

**When disabled**, it will pass on the error without additional processing.


## Test Cases

1. Create a Connex instance
3. Deploy test contract: https://github.com/smartcontractkit/ccip-read/blob/master/packages/ethers-ccip-read-provider/contracts/Token.sol
4. Set URLs to a custom resolving
5. With Disabled CCIP-Reading: When `balanceOf` is called, Then an error is thrown
6. With Enabled CCIP-Reading: When `balanceOf` is called, Then the CCIP process is followed And `balanceOfWithSig` returns successful
  
## Reference Implementation


## Security Considerations

- If the contracts follow the verification of data, security on the blockchain side will not have an impact.
- Contracts and dApps could forward users to access backends using Connex. This could also happen with regular fetch requests in a dApp. It will expose client information (User-Agent, IP) to a configured backend.

Copyright and related rights waived via [CC0](./LICENSE.md).
