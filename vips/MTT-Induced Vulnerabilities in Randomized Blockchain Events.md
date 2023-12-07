---
Title: Addressing MTT-Induced Vulnerabilities in Randomized Blockchain Events
Description: This proposal identifies a vulnerability in vechain's MTT system allowing users to manipulate outcomes of randomized events.
Author: data,  ([@databeforedishonor])
Discussions: https://vechain.discourse.group/t/mtt-induced-vulnerabilities-in-randomized-events/88
Category: Information
Status: Idea
CreatedAt: 2023-12-06

---

## Overview
The introduction of Multi-Task Transactions (MTT) in the Vechain blockchain has led to a unique security vulnerability, particularly in randomized blockchain events. This vulnerability arises because users can create multi-clause transactions where the outcome of one clause (e.g., a randomized event like a coin flip) determines the success or failure of subsequent clauses. If the desired outcome is not achieved in the first clause, the entire transaction fails, allowing users to circumvent the randomness of the event.

## Motivation
Randomized games, NFT mints, and similar events on the blockchain rely on the integrity and unpredictability of their outcomes. However, the MTT feature in Vechain allows users to exploit these events. By structuring transactions with dependent clauses, users can effectively negate the randomness, ensuring favorable outcomes or avoiding unfavorable ones. This undermines the fairness and security of decentralized applications relying on randomization and needs to be addressed.

## Rationale
The exploitation of the MTT system is straightforward and requires minimal coding skills. Various tools available on different platforms can be used to manipulate these outcomes. This vulnerability not only affects community NFT mints and games but also could impact the broader integrity of the blockchain, where randomization plays a crucial role in various future applications.

## Specification
A proposed solution is to standardize smart contracts for randomization, ensuring that the outcome of a randomized event is only determined after the completion of the transaction. This can be achieved with the assistance of oracles, which provide external data (in this case, the randomized outcome) to the blockchain after the transaction's clauses have been executed.

## Test Cases
A test case scenario might involve a coin flip game contract where the outcome (heads or tails) is determined after the transaction is completed. The test should demonstrate that the randomness is unaffected by the structure of the transaction and that subsequent clauses cannot influence or predict the outcome.

## Reference Implementation
A reference implementation would include a smart contract that integrates with an oracle service to provide the random outcome. This contract would ensure that the outcome is determined and recorded only after all transaction clauses are executed.

## Security Considerations
This proposal aims to enhance the security of randomization processes on the blockchain. By decoupling the outcome determination from the transaction execution, it prevents manipulation and ensures that the randomness inherent in games and other applications remains intact. However, reliance on external oracles introduces a dependency on third-party services, which should be considered and managed to prevent new vulnerabilities.

Copyright and related rights waived via CC0.
