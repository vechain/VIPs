---
VIP: 211
Title: Numbering Commits for State Trie
Category: Core
Author: Qian Bin <cola.tin.com@gmail.com>
Status: Draft
CreatedAt: 2021-06-18
---

# 概述

本提案通过扩展默克尔·帕特里夏树（Merkle Patricia Trie，简称MPT）的存储格式，赋予节点**提交编号**的属性。

# 详情

唯链的每个区块包含一次MPT状态树提交，在执行提交时，将当前区块编号作为**提交编号**，包含到提交产生的新节点之中。为此，需要扩展节点的存储格式。

原先存储节点时，以节点的哈希作为键，节点本身作为值：

```
hash(node) => node
```

扩展为：

```
prefix || hash(node) => node || trailing
```

其中 *prefix* 长度为4个字节，由 *commitNum* 经big-endian编码得到。*trailing* 包含子节点的提交编号，为了紧凑，使用RLP编码。节点有两种类型：*shortNode* 和 *fullNode*。*shortNode* 只有一个子节点，因此把其唯一子节点的提交编号（`uint32`）RLP编码就成为它的 *trailing*。*fullNode* 有16个子节点，可以简单的将子节点的提交编号数组（`[16]uint32`）RLP编码作为 *trailing*。

相应的，哈希指针需要被扩展。在代码中，哈希指针用`hashNode`描述：

```go
type hashNode []byte
```

扩展为：

```go
type hashNode struct {
    hash      []byte
    commitNum uint32
}
```

此外，接口也需要有所改动：

构造函数
```diff
-// 旧的定义
-func New(root thor.Bytes32, db Database) (*Trie, error)
+// 新的定义
+func New(root thor.Bytes32, commitNum uint32, db Database) (*Trie, error)
```

提交方法
```diff
-// 旧的定义
-func (t *Trie) Commit() (thor.Bytes32, error)
+// 新的定义
+func (t *Trie) Commit(commitNum uint32) (thor.Bytes32, error)
```

# 结论

新的MPT具有如下重要性质：

1. **新节点的的提交编号总是大于老节点的提交编号**

1. **父节点的提交编号总是大于等于子节点的提交编号**
