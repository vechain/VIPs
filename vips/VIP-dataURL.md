---
Title: Data URL Token Metadata
Description: A standard for storing token metadata directly on-chain using data URLs
Author: Kyle Boe <kyle@boe.codes>
Discussions:
Category: Application
Status: Draft
CreatedAt: 2024-02-19
---

## Overview

This VIP proposes an extension to existing token standards (ERC-721/ERC-1155) to enable storing token metadata directly on-chain using data URLs. This approach provides an alternative to external metadata storage solutions like IPFS or centralized CDNs, offering improved data availability and permanence while maintaining compatibility with existing token standards and tools.

## Motivation

Current token implementations typically store metadata off-chain, either on centralized servers or decentralized storage networks like IPFS. This approach introduces potential issues:

1. Metadata availability depends on external services remaining operational
2. Links to external resources can break over time
3. Centralized storage requires hosting costs and maintenance
4. Network latency when retrieving metadata

By storing metadata directly on-chain using data URLs, we can:

1. Ensure metadata permanence and availability matches that of the blockchain
2. Eliminate dependencies/cost of external storage services
3. Reduce network calls when retrieving token data
4. Maintain backwards compatibility with existing standards and tools

## Rationale

Data URLs are a well-established web standard that allows embedding data directly in URL strings. Modern browsers and tools universally support this format, making it an ideal choice for on-chain metadata storage. The format is particularly efficient for JSON data when combined with base64 encoding.

This proposal builds on existing token standards rather than creating new ones, allowing for gradual adoption while maintaining compatibility with existing infrastructure.

## Specification

### Data URL Format

Token metadata should be encoded as a data URL following [RFC 2397](https://www.rfc-editor.org/rfc/rfc2397#section-2). The format is outlined in a more user-friendly way in the [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Schemes/data).

The content should adhere to the [Metadata Standards](https://docs.opensea.io/docs/metadata-standards) as adopted by OpenSea and other platforms.

### Implementation

Contracts implementing this standard should override the `_baseURI()` function to return the appropriate JSON data URL prefix:

```solidity
function _baseURI() internal pure override returns (string memory) {
    return "data:application/json;base64,";
}
```

The `tokenURI` function, as outlined in [ERC721URIStorage](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/extensions/ERC721URIStorage.sol), concatenates this base URI with the base64-encoded JSON metadata:

```solidity
function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
  _requireOwned(tokenId);

  string memory _tokenURI = _tokenURIs[tokenId];
  string memory base = _baseURI();

  // If there is no base URI, return the token URI.
  if (bytes(base).length == 0) {
      return _tokenURI;
  }
  // If both are set, concatenate the baseURI and tokenURI (via string.concat).
  if (bytes(_tokenURI).length > 0) {
      return string.concat(base, _tokenURI);
  }

  return super.tokenURI(tokenId);
}
```

Thus returning the full data URL for the token metadata.

## Reference Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/Base64.sol";

contract DataUrlToken is ERC721, ERC721URIStorage {
    constructor() ERC721("DataUrlToken", "DUT") {}

    function _baseURI() internal pure override returns (string memory) {
        return "data:application/json;base64,";
    }

    function mintToken(uint256 tokenId, string memory base64EncodedMetadata) public {
        _mint(msg.sender, tokenId);
        _setTokenURI(tokenId, base64EncodedMetadata);
    }
}
```

## Security Considerations

1. **Base64 Encoding**: Embedding of Base64 encoded data comes with some potential risks. Without proper validation, a malicious actor could potentially inject harmful data into the data URL. Implementers should ensure that the data is properly validated before encoding or decoding. Web browsers have built-in protections against [certain types of attacks](https://blog.mozilla.org/security/2017/11/27/blocking-top-level-navigations-data-urls-firefox-59/), but it's still important to properly validate reads & writes.

Copyright and related rights waived via [CC0](./LICENSE.md).