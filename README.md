VeChain Improvement Proposals
====

VIP stands for VeChain Improvement Proposal. It is a design document providing information, or describing a new feature. The VIP should provide a concise technical specification of the feature.

The VIP author is responsible for building consensus within the community and documenting dissenting opinions.


## Proposals

| No.                      | Title                          | Owner   | Category    | Status |
| ------------------------ | ------------------------------ | ------- | ----------- | ------ |
| [180](./vips/VIP-180.md) | Fungible Token Standard        | VeChain | Application | Final  |
| [181](./vips/VIP-181.md) | Non-Fungible Token Standard    | VeChain | Application | Accepted  |
| [190](./vips/VIP-190.md) | Personal Sign Standard         | Totient Labs | Interface | Final  |
| [191](./vips/VIP-191.md) | Designated Gas Payer           | Totient Labs | Core | Draft  |
| [192](./vips/VIP-192.md) | Simple Self-signed Certificate | Qian Bin | Interface | Draft |
| [193](./vips/VIP-193.md) | Committee-Based POA | Ziheng Zhou (peter.zhou@vechain.com) and Zhijie Ren (zhijie.ren@vechain.com) | Core | Draft |
| [200](./vips/VIP-200.md) | Implementation of SURFACE BFT Protocol | Ziheng Zhou (peter.zhou@vechain.com) and Zhijie Ren (zhijie.ren@vechain.com) | Core | Draft |
| [201](./vips/VIP-201.md) | Simple Gas Payer Standard | Xiqing Chu <xiqing.chu@vechain.com> & Mog Lu <mog.lu@vechain.com> | Information | Draft |
| [210](./vips/VIP-210.md) | Semi-Fungible Token(SFT) Standard | VeChain | Application | Draft |
| [214](./vips/VIP-214.md) | Per Epoch VRF-Based Random Beacon Generation | Zhijie Ren (zhijie.ren@vechain.com) and Ziheng Zhou (peter.zhou@vechain.com) | core | Draft |
| [220](./vips/VIP-220.md) | Finality with One Bit (FOB) | Zhijie Ren (zhijie.ren@vechain.com) and Ziheng Zhou (peter.zhou@vechain.com) | core | Draft |
| [230](./vips/VIP-230.md) | Support `debug_traceCall` | Andreas Papageorgiou (andreas.papageorgiou@electiconsulting.com) | Interface | Draft |


## Contributing

### Formats and Templates

Each VIP should have the following parts:

+ Header: The metadata about the VIP, including the VIP number, title and the author. See [Header Preamble](#Header Preamble) for details.
+ Overview: A short description of the technical issue being addressed.
+ Rationale: The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made.
+ Specification: The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.


### Header Preamble

+ VIP: # of proposal
+ Title: VIP title
+ Category: < Core | Application | Interface | Information >
+ Author: List of authors' real names and optionally, email addrs
+ Status: < Draft | Accepted | Deferred | Withdrawn | Final >
+ CreatedAt: Date created on, in ISO 8601 (yyyy-mm-dd) format


### Category

There are four kinds of VIP:

+ Core: Improvements requiring a consensus fork
+ Application: Application-level standards and conventions, including contract standards
+ Interface: Improvements around client API specifications and standards, and also certain language-level standards like method names and contract ABIs
+ Information: Describes a design issue, or provides general guidelines or information to the community, but does not propose a new feature


## Status Terms

+ Draft: A VIP in draft status must be implemented to be considered for promotion to the next status.
+ Accepted: A VIP that is done with its initial iteration and ready for review by a wide audience.
+ Deferred: A VIP that is not being considered for immediate adoption. May be reconsidered in the future for a subsequent hard fork.
+ Withdrawn: A VIP editor may consider to withdraw the proposal for some reasons.
+ Final: This VIP represents the current state-of-the-art. A Final VIP should only be updated to correct errata.
