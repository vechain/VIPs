---
VIP: 192
Title: Simple Self-signed Certificate
Category: Interface
Author: Qian Bin <bin.qian@vechain.com>
Status: Draft
Created: 2019-03-12
---

# Overview


该提案提供一种消息签名机制，使应用和用户之间可以建立脱离区块链的信任关系。


# Specification


## Fields

* *purpose* - `string`: 从'identification', 'agreement'中取值，今后可能会扩展。当应用请求用户对消息签名时，必须明确告知其目的。典型场景：
    
    | purpose | scenario |
    | --- | --- |
    | identification | 应用请求用户证明其持有某个地址的私钥。这时，*payload*内容不重要。 |
    | agreement | 应用请求用户同意某项书面约定，约定内容包含在*payload*中。如，隐私条款。 |

    

* *payload* - `object`: 消息内容主体
    
    * *type* - `string`: 指明*content*的类型，固定为'text'，以后可能支持更多类型。
    * *content* - `string`: 消息内容。

* *timestamp* - `number`: 时间戳，以秒为单位，同unix timestamp以及区块头的时间戳
* *domain* - `string`: 应用url的主机名。
* *signer* - `string`: 用户签名私钥对应的地址的16进制表示，以'0x'开头。
* *signature* - `string`: 签名的16进制表示，以'0x'开头。

## Encoding

由于消息与区块链无关，对存储空间不敏感，因此选择灵活和通用的JSON编码。为了使编码结果具有确定性，作如下规定：

1. JSON对象的key按照字母表顺序升序排列；
2. 编码结果必须是紧凑的，即：无缩进、无换行；
3. 涉及字符编码，均使用utf-8；
4. *signer*和*signature*字段转换为小写形式。


## Hashing

采用VeChain广泛使用的blake2b 256-bit哈希算法。待签名消息包含除*signature*以外的所有字段。包含*signature*的消息哈希可以作为消息的ID。

## Signing

使用secp256k1。


## Verification

1. 计算待签名消息的哈希；
2. 根据签名恢复出公钥，并推导出地址；
3. 与*signer*字段比对，若一致则签名有效。注意：如果以16进制字符串形式比对，则忽略大小写。

