---
Title: Support Contracts for VeChainThor Node-Ownership
Description: Allow contracts to own VeChainThor Nodes to share ownership or benefits.
Author: Mario Micklisch (@ifavo)
Discussions: https://vechain.discourse.group/t/add-vip-support-contracts-for-vechainthor-node-ownership/50
Category:  Core
Status: Draft
CreatedAt: 2023-09-04

---

## Overview

Contracts are widely used, for example in Multi-Sig-Wallets or for DAOs. VeChainThor Nodes can not be transferred to contracts due a restriction in the node-contract. 
Lifting the transfer restriction to allow ownership for contracts will allow a widely shared use of VeChainThor Node, especially will it allow to form DAOs arround a single VeChainThor Node.

  
## Rationale

<!--
  The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.
  
  The current placeholder is acceptable for a draft.
  
  TODO: Remove this comment before submitting
-->
  
## Specification

The [Transfer-Function](https://github.com/vechain/ThorNode-contracts/blob/b3e1765ed7b1599b301602a0e0f72587cf24be1b/contracts/XOwnership.sol#L106) calls [`_transfer`]( https://github.com/vechain/ThorNode-contracts/blob/b3e1765ed7b1599b301602a0e0f72587cf24be1b/contracts/XOwnership.sol#L179-L195).

Within `_transfer` the token recipient is [verified to be not a contract](https://github.com/vechain/ThorNode-contracts/blob/b3e1765ed7b1599b301602a0e0f72587cf24be1b/contracts/XOwnership.sol#L183):

```sol
        require(!_isContract(_to), "_to mustn't a contract");
```

Removing this line will allow contracts to own VeChainThor Nodes.

## Test Cases

1. Deploy a Test-Receiver-Contract
2. Create new VeChainThor Node-Token
3. call `.transfer(<contract address>, <token id>)`
4. `ownerOf(<token id>)` equals Test-Receiver-Contract
  
## Reference Implementation
  
<!--
  This section is optional.

  The Reference Implementation section should include a minimal implementation that assists in understanding or implementing this specification. It should not include project build files. The reference implementation is not a replacement for the Specification section, and the proposal should still be understandable without it.
  
  If the reference implementation is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/vip-####/`. External links will not be allowed.

  TODO: Remove this comment before submitting
-->
  
## Security Considerations

<!--
  All VIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. For example, include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. VIP submissions missing the "Security Considerations" section will be rejected. An VIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

Copyright and related rights waived via [CC0](./LICENSE.md).
