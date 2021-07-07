---
VIP: 213
Title: The Ultimate State Pruner
Category: Core
Author: Qian Bin <cola.tin.com@gmail.com>
Status: Draft
CreatedAt: 2021-07-04
---

# 概述

本提案基于[VIP-211](./VIP-211-zh_CN.md)，利用新节点对旧节点的遮蔽，实现高效的状态清理。


# 详情

## 基本原理
为了简化过程的描述，用二叉树代替MPT的十六叉树作为示例。

现在有状态树 *T<sub>N</sub>*，其提交编号为 *N*，遍历 *T<sub>N</sub>*，有节点 *a, b ... g*，如下：
<pre>
T<sub>N</sub>
        a
       / \
      b   e
    / |   | \
   c  d   f  g
</pre>


若干次更新 *T<sub>N</sub>* 后得到 *T<sub>N<sup>'</sup></sub>*：

<pre>
T<sub>N<sup>'</sup></sub>
        a<sup>'</sup>
       / \
      b<sup>'</sup>  e
    / |   | \
   c<sup>'</sup> d<sup>'</sup>  f  g
</pre>

其中 *a, b, c, d* 节点更新为 *a<sup>'</sup>, b<sup>'</sup>, c<sup>'</sup> d<sup>'</sup>*，旧节点 *a, b, c, d* 被新节点遮蔽。在 *T<sub>N<sup>'</sup></sub>* 对应的区块 *N<sup>'</sup>* 大概率不可逆转时，旧节点可以被安全的删除。


## 实现过程


### 路径编码

为了便于实现，以及对性能的考量，需要在[VIP-211](./VIP-211-zh_CN.md)的基础上再次扩展节点的存储格式：

    pack(path) || commitNum || hash(RLP(node)) => ...

即：在节点存储键之前加上节点的路径信息。

*pack(path)* 记作 *PP*，是路径的64位定长编码，编码算法用Go代码实现如下：

```go
func pack(path []byte) uint64 {
    n := len(path)
    if n > 15 {
        n = 15
    }

    var packed uint64
    for i := 0; i < 15; i++ {
        if i < n {
            packed |= uint64(path[i])
        }
        packed <<= 4
    }
    return packed | uint64(n)
}
```
不难看出，编码结果的字典序与MPT的遍历顺序一致。这带来的好处是，能大大提高遍历MPT的速度。


### 跟踪新节点

用 *C* 表示节点提交编号，遍历T<sub>N<sup>'</sup></sub>，加入过滤条件 *C > N*，并利用[VIP-211](./VIP-211-zh_CN.md)的性质2，

> **父节点的提交编号总是大于等于子节点的提交编号**

即可快速找出所有新节点，并构建映射 *m*，*m*定义为

    PP => C

表示新节点的路径到其提交编号的映射。

### 清理旧节点

遍历数据库，得到所有节点的序列。按以下步骤依次处理节点：

* 从当前节点的存储键解出路径 *PP* 和提交编号 *C*，令 *C<sup>'</sup> = m[PP]*。

* 若 *C < C<sup>'</sup>*，表明当前节点是可被清理的旧节点，直接删除之。

* 否则，当前节点不被新节点遮蔽，或者当前节点本身就是新节点。




