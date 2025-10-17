---
VIP: TBD
Title: Transaction URI Format for VeChain
Description: A standard way of representing VeChain transactions as URIs for QR codes and deep links
Author: Kyle Boe <kyle@boe.codes>
Discussions: https://vechain.discourse.group/t/add-vip-transaction-uri-standard/445
Category: Information
Status: Idea
CreatedAt: 2025-05-14
---

# Overview

This VIP proposes a standardized URI format for representing various VeChain transactions, including payment requests in VET and VIP-180 tokens, multi-clause transactions, and fee-delegated transactions. These URIs can be embedded in QR codes, hyperlinks in web pages, emails, or chat messages to enable cross-application communication between loosely coupled applications. The format allows for instant invocation of the user's preferred wallet application with the correct transaction parameters, requiring only confirmation by the authenticated user.

# Motivation

The convenience of representing payment requests through clear, contact-less standards has been a significant factor in the adoption of digital payment methods like Apple Pay, Google Wallet, and other tap-to-pay solutions. Bringing a similarly convenient mechanism to VeChain would accelerate its acceptance as a payment platform among end-users and enterprises.

VeChain's unique transaction model, which includes features like multi-clause transactions and fee delegation, requires a specialized URI format that can represent these advanced capabilities while maintaining simplicity and usability.

This standard enables:

1. Easy generation of QR codes for payment requests
2. Deep linking to wallet applications from websites and mobile apps
3. Standardized communication between dApps and wallets
4. Support for VeChain's advanced transaction features

# Rationale

The proposed format is designed to resemble Ethereum's ERC-681 URI format as closely as possible, while extending it to support VeChain-specific features. This approach leverages existing familiarity among users and developers while accommodating VeChain's unique capabilities.

Key design considerations include:
1. Compatibility with existing URI schemes where possible
2. Support for all VeChain transaction types and features
3. Compact representation for efficient QR code encoding
4. Clear separation between simple and complex use cases
5. Extensibility for future enhancements

# Specification

## URI Structure Visualization

```
vechain:0xAddress@chainTag/function?param1=value1&param2=value2#delegate=url
        |         |        |       |                            |
        |         |        |       |                            +-- Optional delegation
        |         |        |       +-- Parameters (optional)
        |         |        +-- Function name (optional)
        |         +-- Chain tag (identifies network)
        +-- Target address (contract or recipient)
```

## URI Format

VeChain transaction URIs contain "vechain" in their schema (protocol) part and are constructed as follows using ABNF-like syntax notation:

```
request                 = schema_prefix ":" clauses [ "#" delegate_param ]
schema_prefix           = "vechain"
delegate_param          = "delegate=" STRING
clauses                 = clause *( ":" clause )
clause                  = target_address [ "@" chain_tag ] [ "/" function_name ] [ "?" parameters ]
target_address          = vechain_address
chain_tag               = 1*HEXDIG
function_name           = STRING
vechain_address         = ( "0x" 40*HEXDIG )
parameters              = parameter *( "&" parameter )
parameter               = key "=" value
key                     = "value" / "gas" / "gasPrice" / "data" / "expiration" / "blockRef" / "nonce" / "dependsOn" / "gasPriceCoef" / TYPE
value                   = number / vechain_address / STRING / base64_string
number                  = [ "-" / "+" ] *DIGIT [ "." 1*DIGIT ] [ ( "e" / "E" ) [ 1*DIGIT ] ]
base64_string           = *base64char [ base64_padding ]
base64char              = ALPHA / DIGIT / "-" / "_"
base64_padding          = "=" / "=="
```

Where `TYPE` is a standard ABI type name, as defined in the Ethereum Contract ABI specification. `STRING` is a URL-encoded unicode string of arbitrary length, where delimiters and the percentage symbol (`%`) are mandatorily hex-encoded with a `%` prefix.

## Parameter Definitions

- `value`: Amount of VET to transfer in wei (the smallest unit)
- `gas`: Maximum amount of gas allowed for the transaction
- `gasPrice`: Gas price for the transaction
- `data`: Contract call data in hexadecimal format
- `expiration`: Number of blocks after which the transaction expires
- `blockRef`: Reference to a specific block
- `nonce`: Random number set by the wallet/user
- `dependsOn`: ID of the transaction on which the current transaction depends
- `gasPriceCoef`: Coefficient used to calculate the gas price

## Transaction Types & Examples

### Simple VET Transfer

For a simple VET transfer, the URI format is:

```
vechain:0xRecipientAddress@chainTag?value=amountInWei
```

**Example**: Transfer 1.5 VET on the mainnet (chainTag: 0x4a):
```
vechain:0x7567d83b7b8d80addcb281a71d54fc7b3364ffed@4a?value=1.5e18
```

--------------

### Basic Token Transfer

For ERC-20/VIP-180 token transfers, the URI format is:

```
vechain:0xTokenContractAddress@chainTag/transfer?address=recipientAddress&uint256=amountInTokenUnits
```

**Example:** Transfer 10 VTHO tokens:
```
vechain:0x0000000000000000000000000000456e65726779@4a/transfer?address=0x7567d83b7b8d80addcb281a71d54fc7b3364ffed&uint256=10000000000000000000
```

--------------

### Multi-Clause Transactions

For multi-clause transactions, the URI format concatenates multiple clause definitions with a colon separator:

```
vechain:0xContractAddress@chainTag/function1?address=someAddress:0xOtherContractAddress@chainTag/function2?value=amountInWei
```

**Example:** Multi-clause transaction with two clauses:
```
vechain:0x0000000000000000000000000000456e65726779@4a/transfer?address=0x7567d83b7b8d80addcb281a71d54fc7b3364ffed&uint256=10000000000000000000:0x1234567890abcdef1234567890abcdef12345678@4a/transfer?address=0xabcdefabcdefabcdefabcdefabcdefabcdef&uint256=5000000000000000000
```

**Note on QR Code Size Limitations**: Multi-clause transactions can result in lengthy URIs that may exceed the practical capacity of QR codes at lower error correction levels. Implementers should consider the following:

- Use the minimum necessary error correction level for QR codes
- For complex transactions with many clauses, consider alternative delivery methods
- When possible, optimize parameter values for brevity (e.g., using scientific notation for large numbers)
- Consider implementing a URI shortening service specifically for transaction URIs

--------------

### Fee Delegation (VIP-191)

For transactions with VIP-191 fee delegation, the URI format includes delegation parameters:

```
vechain:0xContractAddress@chainTag/function?uint256=someValue#delegate=delegationUrl
```

Where `delegationUrl` is a URL of the delegation service.

**Example:** Fee delegation with VIP-191:
```
vechain:0x0000000000000000000000000000456e65726779@4a/transfer?address=0x7567d83b7b8d80addcb281a71d54fc7b3364ffed&uint256=1e18#delegate=https://delegation.service/123456
```

# Test Cases

The following test cases should be used to verify implementations of this URI format:

## Valid URI Test Cases

1. **Simple VET Transfer**
   ```
   vechain:0x7567d83b7b8d80addcb281a71d54fc7b3364ffed@4a?value=1.5e18
   ```
   - Should parse as a transfer of 1.5 VET to address 0x7567d83b7b8d80addcb281a71d54fc7b3364ffed on mainnet

2. **VET Transfer with Gas Parameters**
   ```
   vechain:0x7567d83b7b8d80addcb281a71d54fc7b3364ffed@4a?value=1e18&gas=21000&gasPriceCoef=0
   ```
   - Should parse as a transfer of 1 VET with specified gas limit and price coefficient

3. **Token Transfer**
   ```
   vechain:0x0000000000000000000000000000456e65726779@4a/transfer?address=0x7567d83b7b8d80addcb281a71d54fc7b3364ffed&uint256=10000000000000000000
   ```
   - Should parse as a transfer of 10 VTHO tokens to the specified address

4. **Multi-Clause Transaction**
   ```
   vechain:0x0000000000000000000000000000456e65726779@4a/transfer?address=0x7567d83b7b8d80addcb281a71d54fc7b3364ffed&uint256=1e19:0x7567d83b7b8d80addcb281a71d54fc7b3364ffed@4a?value=5e17
   ```
   - Should parse as a transaction with two clauses: a token transfer and a VET transfer

5. **Fee-Delegated Transaction**
   ```
   vechain:0x0000000000000000000000000000456e65726779@4a/transfer?address=0x7567d83b7b8d80addcb281a71d54fc7b3364ffed&uint256=1e18#delegate=https://delegation.service/123456
   ```
   - Should parse as a token transfer with fee delegation

## Invalid URI Test Cases

1. **Missing Chain Tag**
   ```
   vechain:0x7567d83b7b8d80addcb281a71d54fc7b3364ffed?value=1e18
   ```
   - Should be rejected as the chain tag is missing

2. **Invalid Address Format**
   ```
   vechain:0x7567d83b7b8d80addcb281a71d54fc7b336@4a?value=1e18
   ```
   - Should be rejected as the address is not a valid 20-byte hex string

3. **Invalid Parameter Type**
   ```
   vechain:0x0000000000000000000000000000456e65726779@4a/transfer?address=0x7567d83b7b8d80addcb281a71d54fc7b3364ffed&uint256=abc
   ```
   - Should be rejected as "abc" is not a valid uint256 value

4. **Invalid Delegate URL**
   ```
   vechain:0x7567d83b7b8d80addcb281a71d54fc7b3364ffed@4a?value=1e18#delegate=invalid-url
   ```
   - Should be rejected as the delegate parameter is not a valid URL

## Validation Procedure

Implementations should:

1. Verify the URI scheme is "vechain"
2. Validate all addresses are properly formatted (0x followed by 40 hex characters)
3. Ensure the chain tag is a valid hex value
4. Validate all numeric parameters can be parsed as numbers
5. For token transfers, verify the function name and parameter types match the ERC-20/VIP-180 standard
6. For multi-clause transactions, verify each clause is properly formatted
7. For fee-delegated transactions, verify the delegate URL is properly formatted

# Reference Implementation

The following JavaScript implementation demonstrates how to parse and generate VeChain transaction URIs:

```javascript
/**
 * VeChain Transaction URI Parser
 */
class VeChainUriParser {
  /**
   * Parse a VeChain transaction URI
   * @param {string} uri - The URI to parse
   * @returns {Object} The parsed transaction data
   */
  static parse(uri) {
    try {
      // Basic validation
      if (!uri || typeof uri !== 'string') {
        throw new Error('URI must be a non-empty string');
      }

      // Validate URI scheme
      if (!uri.startsWith('vechain:')) {
        throw new Error('Invalid URI scheme. Expected "vechain:"');
      }

      // Ensure there's content after the scheme
      if (uri.length <= 8) {
        throw new Error('URI is incomplete. Missing address or parameters.');
      }

      // Extract delegate parameter if present
      let delegateUrl = null;
      let mainPart = uri.substring(8); // Remove 'vechain:' prefix

      if (mainPart.includes('#')) {
        const parts = mainPart.split('#');
        mainPart = parts[0];

        // Parse delegate parameter
        const delegatePart = parts[1];
        if (delegatePart.startsWith('delegate=')) {
          delegateUrl = delegatePart.substring(9);

          // Validate delegate URL format
          try {
            new URL(delegateUrl); // This will throw if the URL is invalid
          } catch (e) {
            throw new Error(`Invalid delegation URL: ${delegateUrl}`);
          }
        } else {
          throw new Error(`Unknown fragment parameter: ${delegatePart}`);
        }
      }

      // Parse clauses
      const clauseStrings = mainPart.split(':');

      // Ensure there's at least one clause
      if (clauseStrings.length === 0 || !clauseStrings[0]) {
        throw new Error('URI must contain at least one clause');
      }

      const clauses = clauseStrings.map(clauseString => this.parseClause(clauseString));

      return {
        clauses,
        delegateUrl
      };
    } catch (e) {
      // Add context to the error
      if (!e.message.includes('VeChain URI')) {
        e.message = `VeChain URI parsing error: ${e.message}`;
      }
      throw e;
    }
  }

  /**
   * Parse a single clause from the URI
   * @param {string} clauseString - The clause string to parse
   * @returns {Object} The parsed clause data
   */
  static parseClause(clauseString) {
    try {
      // Extract target address and chain tag
      let targetAddress, chainTag, functionName, parameters = {};

      // Validate clause string
      if (!clauseString || typeof clauseString !== 'string') {
        throw new Error('Clause must be a non-empty string');
      }

      // Parse function name if present
      if (clauseString.includes('/')) {
        const parts = clauseString.split('/');
        if (parts.length !== 2) {
          throw new Error(`Invalid function format in clause: ${clauseString}`);
        }

        const addressPart = parts[0];
        const functionPart = parts[1];

        // Parse address and chain tag
        if (addressPart.includes('@')) {
          const addressParts = addressPart.split('@');
          if (addressParts.length !== 2) {
            throw new Error(`Invalid address@chainTag format: ${addressPart}`);
          }

          targetAddress = addressParts[0];
          chainTag = addressParts[1];

          // Validate chain tag format
          if (!/^[0-9a-fA-F]+$/.test(chainTag)) {
            throw new Error(`Invalid chain tag format (must be hex): ${chainTag}`);
          }
        } else {
          targetAddress = addressPart;
        }

        // Validate address format
        if (!/^0x[0-9a-fA-F]{40}$/.test(targetAddress)) {
          throw new Error(`Invalid address format: ${targetAddress}`);
        }

        // Parse function name and parameters
        if (functionPart.includes('?')) {
          const functionParts = functionPart.split('?');
          if (functionParts.length !== 2) {
            throw new Error(`Invalid function?parameters format: ${functionPart}`);
          }

          functionName = functionParts[0];

          // Validate function name
          if (!functionName || functionName.includes('/') || functionName.includes('?')) {
            throw new Error(`Invalid function name: ${functionName}`);
          }

          // Parse parameters
          const paramString = functionParts[1];
          parameters = this.parseParameters(paramString);
        } else {
          functionName = functionPart;

          // Validate function name
          if (!functionName || functionName.includes('/') || functionName.includes('?')) {
            throw new Error(`Invalid function name: ${functionName}`);
          }
        }
      } else {
        // No function name, just address and possibly parameters
        if (clauseString.includes('?')) {
          const parts = clauseString.split('?');
          if (parts.length !== 2) {
            throw new Error(`Invalid address?parameters format: ${clauseString}`);
          }

          const addressPart = parts[0];
          const paramString = parts[1];

          // Parse address and chain tag
          if (addressPart.includes('@')) {
            const addressParts = addressPart.split('@');
            if (addressParts.length !== 2) {
              throw new Error(`Invalid address@chainTag format: ${addressPart}`);
            }

            targetAddress = addressParts[0];
            chainTag = addressParts[1];

            // Validate chain tag format
            if (!/^[0-9a-fA-F]+$/.test(chainTag)) {
              throw new Error(`Invalid chain tag format (must be hex): ${chainTag}`);
            }
          } else {
            targetAddress = addressPart;
          }

          // Validate address format
          if (!/^0x[0-9a-fA-F]{40}$/.test(targetAddress)) {
            throw new Error(`Invalid address format: ${targetAddress}`);
          }

          // Parse parameters
          parameters = this.parseParameters(paramString);
        } else {
          // Just address and possibly chain tag
          if (clauseString.includes('@')) {
            const addressParts = clauseString.split('@');
            if (addressParts.length !== 2) {
              throw new Error(`Invalid address@chainTag format: ${clauseString}`);
            }

            targetAddress = addressParts[0];
            chainTag = addressParts[1];

            // Validate chain tag format
            if (!/^[0-9a-fA-F]+$/.test(chainTag)) {
              throw new Error(`Invalid chain tag format (must be hex): ${chainTag}`);
            }
          } else {
            targetAddress = clauseString;
          }

          // Validate address format
          if (!/^0x[0-9a-fA-F]{40}$/.test(targetAddress)) {
            throw new Error(`Invalid address format: ${targetAddress}`);
          }
        }
      }

      return {
        to: targetAddress,
        chainTag,
        functionName,
        parameters
      };
    } catch (e) {
      // Add context to the error
      if (!e.message.includes('clause')) {
        e.message = `Clause parsing error: ${e.message}`;
      }
      throw e;
    }
  }

  /**
   * Parse parameters from a parameter string
   * @param {string} paramString - The parameter string to parse
   * @returns {Object} The parsed parameters
   */
  static parseParameters(paramString) {
    const parameters = {};
    const paramPairs = paramString.split('&');

    for (const pair of paramPairs) {
      const [key, value] = pair.split('=');
      parameters[key] = this.parseParameterValue(key, value);
    }

    return parameters;
  }

  /**
   * Parse a parameter value based on its key
   * @param {string} key - The parameter key
   * @param {string} value - The parameter value as a string
   * @returns {any} The parsed parameter value
   */
  static parseParameterValue(key, value) {
    try {
      // Handle numeric values
      if (key === 'value' || key === 'gas' || key === 'gasPrice' ||
          key === 'gasPriceCoef' || key === 'expiration' || key === 'nonce') {

        // Validate numeric format
        if (!/^[+-]?\d*\.?\d*(?:[eE][+-]?\d+)?$/.test(value)) {
          throw new Error(`Invalid numeric format for parameter '${key}': ${value}`);
        }

        // Check for scientific notation
        if (value.includes('e') || value.includes('E')) {
          return Number(value);
        }

        // Check for decimal point
        if (value.includes('.')) {
          return Number(value);
        }

        // Otherwise, treat as BigInt for large integers
        return BigInt(value);
      }

      // Handle address values
      if (key === 'address') {
        // Validate Ethereum address format
        if (!/^0x[0-9a-fA-F]{40}$/.test(value)) {
          throw new Error(`Invalid address format: ${value}`);
        }
        return value;
      }

      // Handle uint256 values (common in token transfers)
      if (key === 'uint256') {
        // Validate that the value can be parsed as a BigInt
        try {
          return BigInt(value);
        } catch (e) {
          throw new Error(`Invalid uint256 value: ${value}`);
        }
      }

      // Handle data values
      if (key === 'data') {
        // Validate hex format for data
        if (value.startsWith('0x') && !/^0x[0-9a-fA-F]*$/.test(value)) {
          throw new Error(`Invalid hex data format: ${value}`);
        }
        return value;
      }

      // Default to string
      return decodeURIComponent(value);
    } catch (e) {
      if (e.name === 'URIError') {
        throw new Error(`Invalid URI encoding in parameter '${key}': ${value}`);
      }
      throw e;
    }
  }

  /**
   * Generate a VeChain transaction URI
   * @param {Object} transaction - The transaction data
   * @returns {string} The generated URI
   */
  static generate(transaction) {
    const { clauses, delegateUrl } = transaction;

    // Generate clause strings
    const clauseStrings = clauses.map(clause => this.generateClauseString(clause));

    // Combine clauses with colon separator
    let uri = 'vechain:' + clauseStrings.join(':');

    // Add delegate parameter if present
    if (delegateUrl) {
      uri += '#delegate=' + encodeURIComponent(delegateUrl);
    }

    return uri;
  }

  /**
   * Generate a clause string from clause data
   * @param {Object} clause - The clause data
   * @returns {string} The generated clause string
   */
  static generateClauseString(clause) {
    const { to, chainTag, functionName, parameters } = clause;

    // Start with the target address
    let clauseString = to;

    // Add chain tag if present
    if (chainTag) {
      clauseString += '@' + chainTag;
    }

    // Add function name if present
    if (functionName) {
      clauseString += '/' + functionName;
    }

    // Add parameters if present
    if (parameters && Object.keys(parameters).length > 0) {
      clauseString += '?' + this.generateParameterString(parameters);
    }

    return clauseString;
  }

  /**
   * Generate a parameter string from parameter data
   * @param {Object} parameters - The parameter data
   * @returns {string} The generated parameter string
   */
  static generateParameterString(parameters) {
    const paramPairs = [];

    for (const [key, value] of Object.entries(parameters)) {
      paramPairs.push(key + '=' + this.formatParameterValue(value));
    }

    return paramPairs.join('&');
  }

  /**
   * Format a parameter value for inclusion in a URI
   * @param {any} value - The parameter value
   * @returns {string} The formatted parameter value
   */
  static formatParameterValue(value) {
    // Handle BigInt values
    if (typeof value === 'bigint') {
      return value.toString();
    }

    // Handle numeric values
    if (typeof value === 'number') {
      return value.toString();
    }

    // Handle string values
    if (typeof value === 'string') {
      // Check if it's an address (don't encode)
      if (value.startsWith('0x') && value.length === 42) {
        return value;
      }

      // Otherwise, URI encode
      return encodeURIComponent(value);
    }

    // Default to string representation
    return String(value);
  }
}

// Example usage:
// const uri = 'vechain:0x7567d83b7b8d80addcb281a71d54fc7b3364ffed@4a?value=1.5e18';
// const transaction = VeChainUriParser.parse(uri);
// console.log(transaction);
//
// const regeneratedUri = VeChainUriParser.generate(transaction);
// console.log(regeneratedUri);
```

# Security Considerations

Implementing the VeChain Transaction URI format requires careful attention to several security considerations:

## Phishing Risks

1. **Malicious URIs**: Bad actors may create URIs that attempt to trick users into authorizing unintended transactions. Wallet applications should:
   - Display clear, human-readable transaction details before granting authorization
   - Show the full recipient address and transaction amount
   - Highlight unusual or potentially dangerous transaction parameters

2. **URI Validation**: Implementations must strictly validate all URI components to prevent malformed transactions that might behave unexpectedly.

## QR Code Scanning Security

1. **QR Code Validation**: Applications that scan QR codes should validate the encoded URI before processing it.
2. **Preview Before Action**: Always show users what action will be performed before executing it.

## Deep Linking Vulnerabilities

1. **Origin Validation**: When processing URIs from deep links, applications should validate the origin of the request when possible.
2. **Intent Confirmation**: Users should explicitly confirm their intent to process a transaction from an external application.
3. **State Preservation**: Ensure the application state is properly preserved when handling deep links to prevent session hijacking.

## Parameter Validation

1. **Type Checking**: All parameters must be strictly validated for type correctness.
2. **Value Range Checking**: Numeric parameters should be checked for reasonable ranges.
3. **Address Validation**: All addresses should be validated for correct format and checksum.
4. **Function Signature Validation**: For contract interactions, validate that the function name and parameter types match the expected ABI.

## Privacy Considerations

1. **Sensitive Data**: URIs should not contain sensitive user data.
2. **Delegation Services**: When using fee delegation, be aware that the delegation service may be able to track user activities.

## Implementation Recommendations

1. **Sandboxed Parsing**: Parse URIs in a sandboxed environment to prevent potential exploits.
2. **Rate Limiting**: Implement rate limiting for URI processing to prevent DoS attacks.
3. **Timeout Mechanisms**: Include expiration parameters for time-sensitive transactions.
4. **Secure Defaults**: Use secure default values for optional parameters.


# Sources/References

This VIP draws inspiration and references from several existing standards and specifications:

1. **Bitcoin URI Scheme (BIP-21)**
   - [BIP-21 Specification](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki)
   - Established the concept of cryptocurrency payment URIs
   - Provided foundation for parameter formatting

2. **Ethereum URI Format (ERC-681)**
   - [ERC-681 Specification](https://eips.ethereum.org/EIPS/eip-681)
   - Adapted the URI concept for smart contract interactions
   - Influenced our approach to function calls and parameters

3. **VeChain Fee Delegation (VIP-191)**
   - [VIP-191 Specification](https://github.com/vechain/VIPs/blob/master/vips/VIP-191.md)
   - Defines the fee delegation mechanism referenced in this proposal
   - Informs the delegation parameter format

4. **VeChain Transaction Model**
   - [VeChain Whitepaper](https://www.vechain.org/whitepaper)
   - Provides the foundation for multi-clause transaction support
   - Defines the transaction parameters used in this URI format

5. **URI Specification (RFC 3986)**
   - [RFC 3986](https://tools.ietf.org/html/rfc3986)
   - Defines the general syntax for Uniform Resource Identifiers
   - Guides the overall structure and encoding rules

6. **Ethereum Contract ABI Specification**
   - [Ethereum Contract ABI](https://docs.soliditylang.org/en/v0.8.30/abi-spec.html)
   - Referenced for parameter type definitions
   - Informs the encoding of function parameters

# Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
