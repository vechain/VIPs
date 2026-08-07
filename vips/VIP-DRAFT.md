---
Title: On-Chain Agent Identity
Description: Defines a persistent on-chain identity layer for AI agents.
Author: Aquilla Write <quill.ai@vechain.org>
Discussions: TODO
Category: Core
Status: Draft
CreatedAt: 2026-04-28
---

## Overview

This proposal defines an on-chain identity layer for AI agents so they can be treated as first-class, verifiable actors on VeChain and other chains. The identity is intended to support persistent recognition, permissioning, auditability, asset transfer, and cross-chain interoperability for autonomous agents. It establishes the minimum protocol-level structure required for agents to be represented as durable on-chain entities.

## Motivation

AI agents that operate across multiple blockchains need a persistent identity that can be verified on chain. Without such an identity, other systems cannot reliably determine whether an actor is the same agent over time, whether it is authorized for a given action, or whether its activity can be audited. A lack of standardized identity also prevents safe composition of agents into higher-level workflows and marketplaces.

## Rationale

A dedicated on-chain identity layer is preferable to relying only on externally owned accounts, application-specific profiles, or off-chain registries.

Externally owned accounts do not provide a standard way to represent an agent as an autonomous actor with independently managed permissions and metadata. Application-specific profiles fragment identity across ecosystems and make verification inconsistent. Off-chain registries can record agent information, but they do not provide native on-chain enforceability or auditability.

This design is intended to provide a chain-native anchor for identity while remaining compatible with existing wallet and SDK workflows. It should also allow future extensions for cross-chain verification without requiring each application to invent its own trust model.

## Specification

<!-- TODO: Define the technical design for on-chain agent identity, including how identities are structured, referenced, verified, and made compatible with wallets, SDKs, and cross-chain operation. -->

The specification must define, at minimum, the following elements:

### 1. Identity Object

An agent identity must be represented by a stable on-chain identifier.

The specification must define:

- the identifier format
- whether the identifier is derived from an address, a contract, a token, or a separate registry entry
- how the identifier remains stable over time
- how the identifier is referenced in transactions, events, and application logic

### 2. Registration and Lifecycle

The specification must define:

- how an agent identity is created
- who is allowed to create it
- how ownership or control is assigned
- how control can be rotated
- how an identity can be revoked, paused, or deactivated
- whether an identity can be recovered after loss of control

### 3. Verification

The specification must define:

- how a caller verifies that an identity is valid
- how signatures from agents are produced and verified
- whether verification requires a smart contract, a wallet, or both
- how a verifier checks that a presented identity is current and not revoked

### 4. Metadata

The specification must define:

- what metadata is stored on chain
- what metadata is stored off chain
- how metadata integrity is guaranteed
- how metadata updates are authorized
- whether the metadata includes capabilities, policy references, model references, or endpoint references

### 5. Permissions and Authorization

The specification must define:

- how an agent is granted permissions
- how permissions are scoped
- how permissions are revoked
- how an application checks permission before execution
- whether permissions can be delegated to sub-agents or session keys

### 6. Cross-Chain Compatibility

The specification must define:

- how an agent identity can be referenced across chains
- how chain-specific proofs or attestations are bound to the identity
- how another chain can verify the same agent identity or a related identity
- how replay protection is handled across chains

### 7. Wallet and SDK Compatibility

The specification must define:

- the required wallet-facing interface
- the required SDK-facing interface
- how applications discover identity data
- how applications construct agent-authenticated messages or transactions

### 8. Events and Interfaces

If implemented as a smart contract system, the specification must include full interface definitions, including Solidity interface blocks for:

- identity registration
- identity update
- identity revocation
- permission grant
- permission revoke
- metadata update
- verification helper functions

<!-- TODO: Specify the exact Solidity interfaces, storage layout, event signatures, and function signatures. -->

## Test Cases

<!-- TODO: Add input/output pairs that validate registration, verification, updates, revocation, permission checks, and cross-chain referencing. -->

## Reference Implementation

<!-- TODO: Provide pseudocode or a reference contract implementation once the technical design is finalized. -->

## Security Considerations

This proposal introduces a security-critical identity primitive for autonomous agents. The final design must address the risk of identity spoofing, unauthorized control transfer, stale metadata, replay across chains, and incorrect permission delegation. It must also consider key compromise, recovery abuse, registry censorship, and inconsistent verification behavior between clients. Any implementation must define clear revocation semantics, strong integrity guarantees for metadata, and predictable failure modes for applications that depend on agent identity.

Copyright and related rights waived via [CC0](./LICENSE.md).

PR summary — This draft proposes a core on-chain identity layer for AI agents on VeChain. It frames the need for persistent, verifiable agent identities to support permissioning, auditability, asset transfer, and cross-chain interoperability, and it highlights the technical areas that still need formal specification before implementation.

TODO list —  
- **Specification**: Define the full technical design for agent identity, including identifier format, registration lifecycle, verification rules, metadata model, permissioning, cross-chain binding, and wallet/SDK interfaces.  
- **Specification**: Add precise Solidity interface blocks, event signatures, function signatures, and any required storage or encoding rules.  
- **Test Cases**: Provide input/output pairs covering registration, verification, metadata updates, revocation, permission checks, and cross-chain references.  
- **Reference Implementation**: Add pseudocode or a contract reference implementation after the design is finalized.  
- **Discussions**: Replace `TODO` with a forum thread or GitHub discussion URL if one exists.