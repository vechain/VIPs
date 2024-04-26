---
Title: Increase the base fee coefficient
Description: This VIP proposes a change to the base fee coefficient to make it uncapped.
Author: Darren Kelly (darren.kelly@vechain.org)
Discussions: https://vechain.discourse.group/t/increase-the-base-fee-coefficient/109
Category:  Core
Status: Draft
CreatedAt: 2024-04-26

---

## Overview

Currently the gas price coefficient range is from 0-255, which limits the range of values that can be used. This VIP proposes to uncap the base fee coefficient.

## Motivation

The reason for increasing the range for the base fee coefficient is to provide network participants with the ability to give higher priority to their transactions. With the current implement network participants are bound to twice the gas price, 0.42 VTHO.

## Specification
  
The most simple way to achieve this is to perhaps just change the divisor from 255 to be 1. This maintains backwards compatibility and also reduces the code changes. The base fee coefficient would remain bounded between 0 and 255 however this range would be sufficiently large enough.

The current formula is:

```md
Transaction Fee = Base Gas Fee + ( Base Gas Fee * ( Base Fee Coefficient / 255 ) )
```

The new formula would be:

```md
Transaction Fee = Base Gas Fee + ( Base Gas Fee * ( Base Fee Coefficient / 1 ) )
```

Copyright and related rights waived via [CC0](./LICENSE.md).
