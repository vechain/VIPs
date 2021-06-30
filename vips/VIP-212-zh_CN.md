---
VIP: 212
Title: Direct State Access
Category: Core
Author: Qian Bin <cola.tin.com@gmail.com>
Status: Draft
CreatedAt: 2021-06-28
---

# 概述

本提案定义了称为**直接状态访问**的方法加速访问世界状态。

# 详情

唯链和以太坊一样使用默克尔·帕特里夏树（MPT）管理世界状态。MPT作为一种前缀树，它的时间复杂度为O(log<sub>16</sub>N)。随着状态数量的增加，状态访问的速度会降低。假设状态数量为一百万，那么一次状态访问平均需要5次MPT节点查询。**直接状态访问**利用状态扁平化，结合[VIP-211](./VIP-211-zh_CN.md)，减少节点查询，达到加速状态访问的目的。


## 状态扁平化

MPT管理的状态包含多个历史版本，呈树状结构。遍历MPT某个版本的所有叶子节点，可以得到对应的扁平化状态集。为了使状态集跟随MPT的更新而更新，以及应对可能出现的分叉情况，需要反复遍历MPT，这相当于全量更新，可行性较低。

下面给出一种类似于增量更新的实际可行的方案。它包含以下两个并行任务：

1. 记录MPT更新日志

    定义一个专用的存储区用于存储MPT的更新日志。假设有以下操作更新了MPT
    ```go
    trie.Update(key, value)
    ```
    这个操作产生了一条更新日志 *(key, value)*。在记录日志之前，先对 *key* 做一个变换：
    ```
    key = blockID || key
    ```
    即：把 *blockID* 与 *key* 连接，作为新的 *key*。*blockID* 是本次提交对应的区块ID。

    由于 *blockID* 以区块编号作前缀，因此在区块粒度上，日志是有序的。这个过程类似于WAL（Write-ahead logging）。

1. 日志归并

    日志归并是一个周期性任务，产生持续更新的状态集。它包含以下步骤：

    1. 选定一个区块编号 *N*，等待，直到 *N* 大概率不可逆转。

    1. 遍历日志，以 *[ , big-endian(N)]* 为范围。
        
        判断 *key[:32]* 代表的 *blockID* 是否属于canonical-chain，如果成立，那么将 *(key[32:], value)* 写入状态集。接着把 *(key, value)* 从日志中删除。
        

    任务结束后，得到与区块编号 *N* 对应的状态集 *S<sub>N</sub>*。

## 直接状态访问

访问最新区块对应MPT中的某个状态，需要依次查询 *node<sub>0</sub>, node<sub>1</sub> ... node<sub>n</sub>* 共n+1个节点，其中 *node<sub>0</sub>* 是根节点，*node<sub>k+1</sub>* 是 *node<sub>k</sub>* 的子节点。在应用了[VIP-211](./VIP-211-zh_CN.md)之后，节点具有了提交编号，分别为 *C<sub>0</sub>, C<sub>1</sub> ... C<sub>n</sub>*，其中 *C<sub>0</sub>* 等于最新区块编号。根据[VIP-211](./VIP-211-zh_CN.md)的性质2：

> **父节点的提交编号总是大于等于子节点的提交编号**

可以得到 *C<sub>0</sub> ≥ C<sub>1</sub> ... ≥ C<sub>n</sub>*。当 *C<sub>0</sub> ≥ N* 时（*C<sub>0</sub>* 等于最新区块编号，所以显然成立），状态集 *S<sub>N</sub>* 有效。在依次查询节点的过程中，加入判断 *N ≥ C<sub>k</sub>*。不难看出，当 *N ≥ C<sub>k</sub>* 成立时，*S<sub>N</sub>* 必定包含正确版本的状态，这时可以跳过子节点查询，直接返回 *S<sub>N</sub>* 中的状态。

# 结论

*S<sub>N</sub>* 的时间复杂度为O(1)，在 *S<sub>N</sub>* 保持持续更新时，**直接状态访问**可以使MPT查询的时间复杂度近似达到O(1)。





