---
Title: Multi-Signature Wallet Standard for vechain
Description: This proposal introduces a multi-signature wallet standard for secure and collaborative fund management.
Author: Sebastian Seijo <sebastian.seijo@vechain.org>, [@sebaseek](https://github.com/sebaseek)
Discussions:
  []
Category: Application
Status: Draft
CreatedAt: 2024-12-24
---

## Overview

This proposal introduces a standardized multi-signature (multi-sig) wallet for the vechain ecosystem. A multi-sig wallet requires multiple approvals from predefined signatories to execute a transaction. It enhances security, transparency, and collaboration, making it ideal for use cases such as DAO treasuries, business accounts, and shared fund management.

## Motivation

vechain lacks a native multi-signature wallet solution, which is a critical feature for:
1. **Enhanced Security:** Protecting funds by requiring multiple authorizations for transactions.
2. **Transparency:** Recording all proposals, approvals, and executions on-chain for auditability.
3. **Collaborative Fund Management:** Enabling DAOs, organizations, and teams to manage shared funds effectively.

By introducing this standard, vechain can align with other leading blockchain ecosystems like Ethereum and Polkadot, empowering businesses and developers with a reliable tool for secure fund management.

## Rationale

The design of this multi-sig wallet standard focuses on simplicity, compatibility, and robustness. Similar standards have proven effective in other blockchain ecosystems. By adopting these practices and tailoring them to vechain's unique ecosystem, this proposal provides a solution that meets the needs of businesses, DAOs, and individual users.

## Specification

The multi-sig wallet will include the following features:

1. **Wallet Creation:**
    - Define signatories and set a required approval threshold during initialization.
    - Example: A wallet with three signatories (Diego, Fernanda, Javier) and a 2/3 approval threshold.

2. **Transaction Proposal:**
    - Any signatory can propose a transaction by specifying:
        - Recipient address.
        - Amount to transfer.

3. **Approval Mechanism:**
    - Signatories review and approve proposed transactions.
    - Transactions execute automatically once the required approvals are met.

4. **Role Management:**
    - Enable adding or removing signatories and updating thresholds securely.

### Example Smart Contract:

```solidity
pragma solidity ^0.8.0;

contract MultiSigWallet {
    address[] public signatories;
    mapping(address => bool) public isSignatory;
    uint256 public requiredApprovals;

    struct Transaction {
        address to;
        uint256 amount;
        uint256 approvals;
        bool executed;
        mapping(address => bool) approvedBy;
    }

    Transaction[] public transactions;

    modifier onlySignatory() {
        require(isSignatory[msg.sender], "Not a signatory");
        _;
    }

    constructor(address[] memory _signatories, uint256 _requiredApprovals) {
        require(_signatories.length > 0, "Signatories required");
        require(_requiredApprovals > 0 && _requiredApprovals <= _signatories.length, "Invalid required approvals");

        for (uint256 i = 0; i < _signatories.length; i++) {
            address signatory = _signatories[i];
            require(signatory != address(0), "Invalid address");
            require(!isSignatory[signatory], "Duplicate signatory");

            isSignatory[signatory] = true;
            signatories.push(signatory);
        }

        requiredApprovals = _requiredApprovals;
    }

    function proposeTransaction(address _to, uint256 _amount) external onlySignatory {
        Transaction storage newTransaction = transactions.push();
        newTransaction.to = _to;
        newTransaction.amount = _amount;
        newTransaction.approvals = 0;
        newTransaction.executed = false;
    }

    function approveTransaction(uint256 _txIndex) external onlySignatory {
        Transaction storage transaction = transactions[_txIndex];
        require(!transaction.executed, "Transaction already executed");
        require(!transaction.approvedBy[msg.sender], "Already approved");

        transaction.approvedBy[msg.sender] = true;
        transaction.approvals++;

        if (transaction.approvals >= requiredApprovals) {
            executeTransaction(_txIndex);
        }
    }

    function executeTransaction(uint256 _txIndex) internal {
        Transaction storage transaction = transactions[_txIndex];
        require(!transaction.executed, "Transaction already executed");
        require(transaction.approvals >= requiredApprovals, "Not enough approvals");

        transaction.executed = true;
        (bool success, ) = transaction.to.call{value: transaction.amount}("");
        require(success, "Transaction failed");
    }

    receive() external payable {}
}
```

## Test Cases

Test cases will be defined to verify the correct operation of the multi-sig wallet implementation, including:

### Basic Wallet Creation Test:

- **Input:** Initialize with three signatories and a 2/3 approval threshold.
- **Output:** Wallet successfully created with the correct setup.

### Transaction Proposal Test:

- **Input:** A signatory proposes a transaction to transfer 100 VET to an external address.
- **Output:** Transaction is added to the list of pending transactions.

### Approval and Execution Test:

- **Input:** Two signatories approve a transaction with a 2/3 approval threshold.
- **Output:** Transaction is executed, and funds are transferred.

### Invalid Scenarios Test:

- **Case 1:** Non-signatories attempting to propose or approve transactions.
- **Case 2:** Approvals exceeding the required threshold.

## Reference Implementation

The provided smart contract serves as a reference implementation for the proposed multi-signature wallet standard. Work needs to be done to fit this into the vechain ecosystem and ensure compatibility with existing tools and services.

## Security Considerations

The following security aspects are critical for this proposal:

### Validation of Signatories:

- Ensure only authorized signatories can propose and approve transactions.

### Replay Attack Prevention:

- Safeguard against the reuse of transaction data for malicious purposes.

### Logging and Auditing:

- Maintain an immutable on-chain record of all proposals, approvals, and executions for transparency and accountability.
