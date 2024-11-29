**Title**: Verifiable Random Function (VRF) for VeChainThor Ecosystem  
**Description**: This proposal introduces Verifiable Random Function (VRF) to the VeChainThor ecosystem for secure and transparent random number generation.  
**Author**: Erik Nucibella <erik.nucibella@vechain.org>, [@akanoce](https://github.com/akanoce)  
**Discussions**: [https://vechain.discourse.group/t/add-vip-vrf-integration-on-vechainthor/238]  
**Category**: Core  
**Status**: Idea  
**CreatedAt**: 2024-11-29

---

## Overview

This proposal recommends integrating Verifiable Random Function (VRF) into the VeChainThor blockchain to provide a secure, cryptographically verifiable source of randomness. This integration would enable decentralized applications (dApps) to utilize tamper-proof random number generation for various use cases, including lotteries, token distributions, and gamified reward mechanisms. The introduction of VRF would especially benefit platforms like [VeBetterDAO](https://vebetter.com), allowing apps to create unprecedent and gamified reward mechanism.

## Motivation

Currently, VeChain lacks a secure on-chain solution for generating random numbers. Existing methods, such as pseudo-random number generation based on block properties, are vulnerable to manipulation and lack verifiability. This undermines the fairness and security of decentralized applications that require randomness, such as lotteries, token drops, and reward systems. There are no oracle-based VRF implementations known at this time.

By introducing VRF, VeChain can provide a secure, tamper-proof source of randomness, ensuring fairness in applications requiring randomization. Platforms like [VeBetterDAO](https://vebetter.com) would benefit from this functionality, enabling transparent lotteries, fair token distributions, and gamified experiences built on unbiased randomness.

## Rationale

Integrating VRF into the VeChainThor blockchain aligns VeChain with best practices from other blockchain ecosystems, such as Ethereum, which uses VRF solutions (e.g., [Chainlink VRF](https://docs.chain.link/vrf)) for secure and transparent random number generation. With the absence of a native VRF solution in VeChainThor, developers must rely on insecure pseudo-random methods, as illustrated by the following example:

```solidity
function getImprovedRandom(uint256 seed) public view returns (uint256) {
    return uint256(keccak256(abi.encodePacked(
        blockhash(block.number - 1),
        block.timestamp,
        block.difficulty, // If available on VeChain
        msg.sender,
        seed
    ))) % 100; // Generates a random number between 0 and 99
}
```

However, pseudo-random approaches suffer from several limitations:

- **Predictability**: Data such as block timestamps and difficulty can be manipulated or anticipated by miners and validators.
- **Vulnerability**: Using on-chain data for randomness introduces opportunities for adversaries to manipulate outcomes, compromising fairness.

By implementing VRF, VeChainThor would provide developers with a robust, decentralized, and verifiable solution, ensuring fairness in randomness-based processes.

Some of the use cases to be unlocked with a VRF:

- Gaming Applications

  - Lotteries: Fair and transparent random selection of winners.
  - Arcade Games: Securely powering random spins, rolls, or draws.

- NFT Ecosystem

  - Randomized trait assignment during minting.
  - Introducing unpredictability to NFT drops or airdrops.

- Gamified Platforms (e.g., VeBetterDAO)

  - Engaging challenges with randomly assigned tasks, multipliers, or rewards.
  - Transparent reward distribution in competitions or leaderboards.

- Governance Mechanisms
  - Fairly selecting committee members or resolving disputes via random processes.

## Specification

[A golang implementation of VRF has already been delivered as part of the POA 2.0 milestone](https://abyteahead.medium.com/poa-2-0-vechains-verifiable-random-function-library-in-golang-5582268d073b)

- **Randomness Request**: Developers can request random numbers using a VRF smart contract or library. The request specifies the seed and desired range (minimum and maximum values).

  ```solidity
  uint256 randomNumber = VRF.requestRandom(seed, min, max);
  ```

  This would generate a random number between the specified `min` and `max` values.

- **Verifiability**: Each random number generated will be accompanied by a cryptographic proof. This proof ensures that the number has not been tampered with, and users or developers can verify the number’s integrity through the proof.

- **Integration with dApps**: The VRF system will be accessible to developers through simple smart contract calls, enabling seamless integration into existing decentralized applications.

### Example: Chainlink VRF Integration

To showcase how VRF implementations behave on ethereum, here is an example based on Chainlink's VRF:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@chainlink/contracts/src/v0.8/VRFConsumerBaseV2.sol";
import "@chainlink/contracts/src/v0.8/interfaces/VRFCoordinatorV2Interface.sol";

contract VRFExample is VRFConsumerBaseV2 {
    VRFCoordinatorV2Interface COORDINATOR;

    uint64 subscriptionId;  // Subscription ID for Chainlink VRF
    bytes32 keyHash;        // Gas lane key for VRF
    uint32 callbackGasLimit = 100000; // Max gas for the callback
    uint16 requestConfirmations = 3; // Confirmations to wait before response
    uint256 public randomResult;

    constructor(
        address vrfCoordinator,
        bytes32 _keyHash,
        uint64 _subscriptionId
    ) VRFConsumerBaseV2(vrfCoordinator) {
        COORDINATOR = VRFCoordinatorV2Interface(vrfCoordinator);
        keyHash = _keyHash;
        subscriptionId = _subscriptionId;
    }

    // Request random number
    function requestRandomWords() public {
        COORDINATOR.requestRandomWords(
            keyHash,
            subscriptionId,
            requestConfirmations,
            callbackGasLimit,
            1 // Number of random numbers to request
        );
    }

    // Callback function to receive the random number
    function fulfillRandomWords(
        uint256, /* requestId */
        uint256[] memory randomWords
    ) internal override {
        randomResult = randomWords[0];
    }
}
```

## Test Cases

Test cases will be defined to verify the correct operation of the VRF implementation, including:

1. **Basic Randomness Test**:

   - **Input**: Seed = 12345, min = 1, max = 100.
   - **Output**: A random number between 1 and 100, accompanied by a proof of verifiability.

2. **Multiple Requests Test**:
   - **Input**: Multiple requests with different seeds.
   - **Output**: A unique random number each time, all verifiable through respective proofs.

## Reference Implementation

A reference implementation will be provided, demonstrating how to request random numbers and verify proofs using VRF.

[Existing libraries or features of the protocol may be reused to achieve the goal](https://abyteahead.medium.com/poa-2-0-vechains-verifiable-random-function-library-in-golang-5582268d073b)

## Security Considerations

The implementation should not impact the security of the VechainThor protocol.

Security is paramount in this proposal. The VRF system uses cryptographic algorithms to ensure that random number generation is tamper-proof and verifiable. Potential security risks include:

- **Oracle Manipulation**: If an external oracle service is used for randomness, the oracle could be compromised.
- **Proof Verification**: It is crucial that developers correctly verify the cryptographic proofs accompanying random numbers. The proposal recommends using established libraries for proof verification to maintain trust in the system.

The design of the VRF system ensures that all randomness is verifiable, providing transparency and security for decentralized applications across the VeChainThor ecosystem.
