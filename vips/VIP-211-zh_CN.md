---
VIP: 211
Title: Numbering Commits for State Trie
Category: Core
Author: Qian Bin <cola.tin.com@gmail.com>
Status: Draft
CreatedAt: 2021-06-18
---

# 概述

本提案在保持兼容的前提下，向默克尔·帕特里夏树（Merkle Patricia Trie，简称MPT）加入“提交编号”（Commit Number）的特性。由此提升MPT性能优化的潜力，包括但不限于状态清理（State Pruning）、直接状态访问（Direct State Access）。

# 实现说明

每个区块对应一次状态树提交，因此把区块编号直接用作提交编号再合适不过。提交编号的类型定义为`uint32`，与区块编号一致。下面通过代码片段阐述涉及修改的部分。

## 构造函数

原先通过root就可以定义一棵树，现在除了root还需要“提交编号”。

```go
// 旧的定义
// func New(root thor.Bytes32, db Database) (*Trie, error)
//
// 新的定义
func New(root thor.Bytes32, commitNum uint32, db Database) (*Trie, error)
```

## Commit方法

和构造函数类似，`Commit`方法也要加上`commitNum`参数。在存储新生成的节点时，`commitNum`会关联到新节点。

```go
// 旧的定义
// func (t *Trie) Commit() (thor.Bytes32, error)
//
// 新的定义
func (t *Trie) Commit(commitNum uint32) (thor.Bytes32, error)
```    

## 编码和存储

MPT节点原先的存储方式简单直接：

```
hash(RLP(node)) => RLP(node)
```

即：通过RLP编码将节点转换成blob，然后求blob的hash，最后把hash和blob作为键值对直接存储到数据库中。

有了“提交编号”之后，我们把原先的hash和blob键值对扩展为：

```
prefix || hash(RLP(node)) => RLP(node) || trailing
```
其中`prefix`通过`commitNum`编码成big-endian的4字节。`trailing`包含当前节点的子节点的提交编号，为了紧凑，使用RLP编码。存储的节点有两种类型：`shortNode`和`fullNode`。`shortNode`只有一个子节点，因此把唯一子节点的提交编号RLP编码就成为它的`trailing`。`fullNode`有16个子节点，可以简单的将子节点的提交编号数组RLP编码作为`trailing`。


## 提示

通过扩展`hashNode`类型可以简化实现。

```go
// 旧的定义
// type hashNode []byte
//
// 新的定义
hashNode struct {
    hash      []byte
    commitNum uint32
}
```

# 意外的惊喜

不难发现，改进后的状态树提交产生的新节点的存储键，在字典序上总是大于老节点的。这意味着，状态树对数据库产生的大量输出是有序的，这对LSM数据库来说无疑可以极大地提高写入速率，降低写放大。
