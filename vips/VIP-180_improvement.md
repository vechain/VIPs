---
Title: VIP-180
Description: Fungible Token Standard
Author: Paula Amstutz paula@saynode.ch
Discussions: <URL>
Category: Application
Status: Idea
CreatedAt: 2023-09-07
VIP: 180
---

## Overview

In the documentation for VIP 180, it is mentioned: “The VIP-180 Standard is a superset of the [ERC-20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)”. However, this is not fully true for the functions below:

- function transfer(address \_to, uint256 \_value) public
- function transferFrom(address \_from, address \_to, uint256 \_value) public
- function approve(address \_spender, uint256 \_value)

These functions according to Etherium’s EIP-20 documentation (provided on the link above from you) are as follows:

- function transfer(address \_to, uint256 \_value) public returns (bool success)
- function transferFrom(address \_from, address \_to, uint256 \_value) public returns (bool success)
- function approve(address \_spender, uint256 \_value) public returns (bool success)

## Motivation

VIP-180 doesn’t look like a superset of EIP-20 if it doesn’t behave the same way EIP-20 does. It leads to discrepancies between chains. That is why I have proposed to change the above functions to make VIP-180 a subset of ERC-20.

## Rationale

When we deployed our token on multiple chains, we were facing problems because VIP-180 was not behaving exactly the same as EIP-20. Changing the following functions will make VIPS180 compatible with other EVM based chains.

- function transfer(address \_to, uint256 \_value) public returns (bool success)
- function transferFrom(address \_from, address \_to, uint256 \_value) public returns (bool success)
- function approve(address \_spender, uint256 \_value) public returns (bool success)

## Specification

As it is mentioned that VIP-180 is a superset of EIP-20, it means they should behave in the same way. VIP-180 doesn't return a boolean to specify whether the transaction has been successful or not but ERC-20 does.

## Test Cases

## Reference Implementation

## Security Considerations

This change doesn't have any security implications as it aims only to make VIP-180 compatible with other chains and doesn't introduce major changes in VIP-180 standard.

Copyright and related rights waived via [CC0](./LICENSE.md).
