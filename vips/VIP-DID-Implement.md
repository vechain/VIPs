---
Title: DID Protocal
Description: Implement DID (Decentralized Identifiers) on Vechain
Author: Mog Lu <mog.lu@vechain.org>
Discussions: 
Category:  Application
Status: Draft
CreatedAt: 2023-09-05
---

## Overview

Implement the function of DID on vechain. The data of the DID documents is stored in the smart contracts, athe developer can verify the DID on-chain or off-chain. The DID technical standards reference [Decentralized Identifiers (DIDs) v1.0](https://w3c.github.io/did-core/).

## Motivation

As an important infrastructure of web3, DID allows users to have unique identities on the blockchain instead of using addresses as identifiers. Multiple verification methods based on DID can improve account security.

## Rationale

1. Smart Contracts

![contracts](https://www.plantuml.com/plantuml/png/JO_13e8m38RlUuhT2Ju314paG3IIU9_iLvQmZRILwDixcJ7sr7xzsx-jM0o9Ty5wiwX2jrsh3u-hAdBQcs3k85MMkeb1ACvphYuWTtEZ4HU3AXFeDlBg8LEqYIIKoO1WGVzatxH3VSOOLr1jBO4vjuyoWVIwYRJVDl4knSVmPCXuiFh95eeBI1nOHz9u2WAdO_90bKo_Jxq1)

The properties of the DID document will be stored in multiple smart contracts. DApp can read the data in the contract. Some verifications support on-chain verification, such as `VerifySender` and `VerifyECDSARecoverAddress`.

- DID Manager

  `DID Manager ` provides methods to register and set DID status.

- DID Hub

  All DID component smart contracts are managed by the `DID hub`, which supports the subsequent addition of new smart contract components to expand the functionality of DID.

- Controller

   Data of [DID Controller](https://w3c.github.io/did-core/#did-controller) will be stored in `Controller` smart constract.A DID controller is an entity that is authorized to make changes to a DID document.

- Verification

  Data of [DID Verification](https://w3c.github.io/did-core/#verification-methods) will be stored in `Verification` smart constract.The data contains the type of verification, the value of verification, and the method of verification on-chain if supported.

  For example, the verification type is `VerifySender`. the value of verification is an `address` and it support verification on-chain. when dApp contract call `verify` method, the `Verification` will check the `tx_sender` is same as the value of verification.

- IVerifyMethod

    This a constract interface.

        function verify(bytes[] memory _keyValue,VerifyArgs memory _verArgs) external returns (bool);

    If the type support verification on-chain, the developer of verification can implement this interface and register to `Verification` smart contract.

- Parames

    This is an extended DID properties, It can store parameter for each DID.

    WARNING, this parameter is not defined in the [Decentralized Identifiers (DIDs) v1.0](https://w3c.github.io/did-core/), other DID implementations may not support this property.


## Specification
  
<!--
  The Specification section should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.
  
  TODO: Remove this comment before submitting
-->

## Test Cases
  
<!--
  This section is optional for non-Core VIPs.

  The Test Cases section should include expected input/output pairs, but may include a succinct set of executable tests. It should not include project build files. No new requirements may be be introduced here (meaning an implementation following only the Specification section should pass all tests here.)
  
  If the test suite is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/vip-####/`. External links will not be allowed

  TODO: Remove this comment before submitting
-->
  
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
