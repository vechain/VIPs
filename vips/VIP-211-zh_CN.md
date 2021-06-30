---
VIP: 211
Title: Numbering Commits for State Trie
Category: Core
Author: Qian Bin <cola.tin.com@gmail.com>
Status: Draft
CreatedAt: 2021-06-18
---

# 概述

本提案在保持兼容的前提下，赋予默克尔·帕特里夏树（Merkle Patricia Trie，简称MPT）**提交编号**（Commit Number）的特性。

# 详情

唯链的每个区块对应一次MPT状态树提交，每个提交产生一批新的节点，包括一个根节点，其哈希值被区块索引，节点不包含任何关于区块的信息。现在把区块编号作为提交操作的编号，附加到新节点之上，使节点包含了额外的信息，即：这个节点是由哪个区块产生的，从而为后续的优化提供有力的支持。新的MPT具备的重要性质总结如下：

1. **新节点的的提交编号总是大于老节点的提交编号**

1. **父节点的提交编号总是大于等于子节点的提交编号**

为了实现此提案，需要对MPT接口进行以下修改：


## 构造函数

标定一棵状态树，原先只需要根节点哈希，现在还需要提交编号 `commitNum`。

```diff
-// 旧的定义
-func New(root thor.Bytes32, db Database) (*Trie, error)
+// 新的定义
+func New(root thor.Bytes32, commitNum uint32, db Database) (*Trie, error)
```

## Commit方法

和构造函数类似，`Commit` 方法也要加上 `commitNum` 参数。

```diff
-// 旧的定义
-func (t *Trie) Commit() (thor.Bytes32, error)
+// 新的定义
+func (t *Trie) Commit(commitNum uint32) (thor.Bytes32, error)
```    

## 存储逻辑

节点原先的存储方式：

```
hash(RLP(node)) => RLP(node)
```

即：通过RLP编码将节点转换成 *blob*，求 *blob* 的hash，然后把hash和 *blob* 作为键值对直接存储到数据库中。

有了**提交编号**之后，我们把原先的hash和 *blob* 键值对扩展为：

```
prefix || hash(RLP(node)) => RLP(node) || trailing
```
其中 *prefix* 长度为4个字节，由 *commitNum* 经big-endian编码得到。*trailing* 包含当前节点的子节点的提交编号，为了紧凑，使用RLP编码。节点有两种类型：*shortNode* 和 *fullNode*。*shortNode* 只有一个子节点，因此把其唯一子节点的提交编号（`uint32`）RLP编码就成为它的 *trailing*。*fullNode* 有16个子节点，可以简单的将子节点的提交编号数组（`[16]uint32`）RLP编码作为 *trailing*。


## 实现提示

通过扩展 `hashNode` 类型可以简化实现。

```diff
-// 旧的定义
-type hashNode []byte
+// 新的定义
+hashNode struct {
+    hash      []byte
+    commitNum uint32
+}
```
