---
Title: NFT Subscriptions
Description: Ability to manage subscriptions for NFTs
Author: Mario Micklisch (@ifavo)
Discussions: https://vechain.discourse.group/t/add-vip-nft-subscriptions/49
Category:  Application
Status: Draft
CreatedAt: 2023-09-01

---

## Overview

This VIP outlines a set of common methods for recurring and expirable subscriptions attached to NFTs. It is based on ERC-5643 and requires VIP-181.
  
## Motivation

Ownership of NFTs can be used to verify access to certain areas. With this VIP recurring payments can be verified during access verification.

One example is an on-going subscription for a dApp:
- When payments are stopped, then access can be blocked while still providing access to past information using the NFT information.
- When payment is restored, access can be restored with all past information attached to the NFT, including past payments.

## Rationale

The adoption of ERC-5643 is chose due its existing wider usage and support in the ethereum system.
This standard aims to make on-chain subscriptions as simple as possible by adding the minimal required functions and events for implementing on-chain subscriptions. It is important to note that in this interface, the NFT itself represents ownership of a subscription, there is no facilitation of any other fungible or non-fungible tokens.

## Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

```solidity
interface IERC5643 {
    /// @notice Emitted when a subscription expiration changes
    /// @dev When a subscription is canceled, the expiration value should also be 0.
    event SubscriptionUpdate(uint256 indexed tokenId, uint64 expiration);

    /// @notice Renews the subscription to an NFT
    /// Throws if `tokenId` is not a valid NFT
    /// @param tokenId The NFT to renew the subscription for
    /// @param duration The number of seconds to extend a subscription for
    function renewSubscription(uint256 tokenId, uint64 duration) external payable;

    /// @notice Cancels the subscription of an NFT
    /// @dev Throws if `tokenId` is not a valid NFT
    /// @param tokenId The NFT to cancel the subscription for
    function cancelSubscription(uint256 tokenId) external payable;

    /// @notice Gets the expiration date of a subscription
    /// @dev Throws if `tokenId` is not a valid NFT
    /// @param tokenId The NFT to get the expiration date of
    /// @return The expiration date of the subscription
    function expiresAt(uint256 tokenId) external view returns(uint64);

    /// @notice Determines whether a subscription can be renewed
    /// @dev Throws if `tokenId` is not a valid NFT
    /// @param tokenId The NFT to get the expiration date of
    /// @return The renewability of a the subscription
    function isRenewable(uint256 tokenId) external view returns(bool);
}
```

The `expiresAt(uint256 tokenId)` function MAY be implemented as `pure` or `view`.

The `isRenewable(uint256 tokenId)` function MAY be implemented as `pure` or `view`.

The `renewSubscription(uint256 tokenId, uint64 duration)` function MAY be implemented as `external` or `public`.

The `cancelSubscription(uint256 tokenId)` function MAY be implemented as `external` or `public`.

The `SubscriptionUpdate` event MUST be emitted whenever the expiration date of a subscription is changed.

The `supportsInterface` method MUST return `true` when called with `0x8c65f84d`.

### Subscription Management

Subscriptions represent agreements to make advanced payments in order to receive or participate in something. In order to facilitate these agreements, a user must be able to renew or cancel their subscriptions hence the `renewSubscription` and `cancelSubscription` functions. It also important to know when a subscription expires - users will need this information to know when to renew, and applications need this information to determine the validity of a subscription NFT. The `expiresAt` function provides this functionality. Finally, it is possible that a subscription may not be renewed once expired. The `isRenewable` function gives users and applications that information.

### Easy Integration

Because this standard is fully EIP-721 compliant, existing protocols will be able to facilitate the transfer of subscription NFTs out of the box. With only a few functions to add, protocols will be able to fully manage a subscription's expiration, determine whether a subscription is expired, and see whether it can be renewed.

## Backwards Compatibility

This standard can be fully EIP-721 compatible by adding an extension function set.

The new functions introduced in this standard add minimal overhead to the existing EIP-721 interface, which should make adoption straightforward and quick for developers.

## Test Cases

The following tests require Foundry.

```solidity
// SPDX-License-Identifier: CC0-1.0
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../src/ERC5643.sol";

contract ERC5643Mock is ERC5643 {
    constructor(string memory name_, string memory symbol_) ERC5643(name_, symbol_) {}

    function mint(address to, uint256 tokenId) public {
        _mint(to, tokenId);
    }
}

contract ERC5643Test is Test {
    event SubscriptionUpdate(uint256 indexed tokenId, uint64 expiration);

    address user1;
    uint256 tokenId;
    ERC5643Mock erc5643;

    function setUp() public {
        tokenId = 1;
        user1 = address(0x1);

        erc5643 = new ERC5643Mock("erc5369", "ERC5643");
        erc5643.mint(user1, tokenId);
    }

    function testRenewalValid() public {
        vm.warp(1000);
        vm.prank(user1);
        vm.expectEmit(true, true, false, true);
        emit SubscriptionUpdate(tokenId, 3000);
        erc5643.renewSubscription(tokenId, 2000);
    }

    function testRenewalNotOwner() public {
        vm.expectRevert("Caller is not owner nor approved");
        erc5643.renewSubscription(tokenId, 2000);
    }

    function testCancelValid() public {
        vm.prank(user1);
        vm.expectEmit(true, true, false, true);
        emit SubscriptionUpdate(tokenId, 0);
        erc5643.cancelSubscription(tokenId);
    }

    function testCancelNotOwner() public {
        vm.expectRevert("Caller is not owner nor approved");
        erc5643.cancelSubscription(tokenId);
    }

    function testExpiresAt() public {
        vm.warp(1000);

        assertEq(erc5643.expiresAt(tokenId), 0);
        vm.startPrank(user1);
        erc5643.renewSubscription(tokenId, 2000);
        assertEq(erc5643.expiresAt(tokenId), 3000);

        erc5643.cancelSubscription(tokenId);
        assertEq(erc5643.expiresAt(tokenId), 0);
    }
}
```

## Reference Implementation

Implementation: `ERC5643.sol`

```solidity
// SPDX-License-Identifier: CC0-1.0
pragma solidity ^0.8.13;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "./IERC5643.sol";

contract ERC5643 is ERC721, IERC5643 {
    mapping(uint256 => uint64) private _expirations;

    constructor(string memory name_, string memory symbol_) ERC721(name_, symbol_) {}

    function renewSubscription(uint256 tokenId, uint64 duration) external payable {
        require(_isApprovedOrOwner(msg.sender, tokenId), "Caller is not owner nor approved");

        uint64 currentExpiration = _expirations[tokenId];
        uint64 newExpiration;
        if (currentExpiration == 0) {
            newExpiration = uint64(block.timestamp) + duration;
        } else {
            if (!_isRenewable(tokenId)) {
                revert SubscriptionNotRenewable();
            }
            newExpiration = currentExpiration + duration;
        }

        _expirations[tokenId] = newExpiration;

        emit SubscriptionUpdate(tokenId, newExpiration);
    }

    function cancelSubscription(uint256 tokenId) external payable {
        require(_isApprovedOrOwner(msg.sender, tokenId), "Caller is not owner nor approved");
        delete _expirations[tokenId];
        emit SubscriptionUpdate(tokenId, 0);
    }

    function expiresAt(uint256 tokenId) external view returns(uint64) {
        return _expirations[tokenId];
    }

    function isRenewable(uint256 tokenId) external pure returns(bool) {
        return true;
    }

    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
        return interfaceId == type(IERC5643).interfaceId || super.supportsInterface(interfaceId);
    }
}
```

## Security Considerations

This VIP standard does not affect ownership of an NFT and thus can be considered secure.

Copyright and related rights waived via [CC0](./LICENSE.md).
