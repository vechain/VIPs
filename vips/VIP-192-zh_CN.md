---
VIP: 192
Title: Simple Self-signed Certificate
Category: Interface
Author: Qian Bin <bin.qian@vechain.com>
Status: Draft
CreatedAt: 2019-03-12
---

# Overview

该提案提供一种签名方案，使**应用**能够安全的向**用户**请求可信的身份证明和条款确认。在追求去中心化的区块链世界中，**用户**--即私钥持有者--地位不同以往，标题中的**证书**（Certificate）比**凭据**（Credential）更贴切。


# Specification

## Fields

* *purpose* - `string`: 从"identification", "agreement"中取值，今后可能会扩展。当应用请求用户对消息签名时，必须明确告知其目的；用户有义务仔细确认这个字段。典型场景：
    
    | purpose | scenario |
    | --- | --- |
    | identification | 应用请求用户证明其持有某个地址的私钥。这时，*payload*内容不重要。 |
    | agreement | 应用请求用户同意某项条款，条款内容包含在*payload*中。如，隐私条款。 |

    

* *payload* - `object`: 消息内容主体
    
    * *type* - `string`: 指明*content*的类型，固定为"text"，以后可能支持更多类型。
    * *content* - `string`: 消息内容，依*type*值而定，*type*取"text"时，为普通文本串。

* *timestamp* - `number`: 时间戳，以秒为单位，同unix timestamp以及区块头的时间戳。用户在签名之前，需要仔细确认这个字段（如果类比成书面合同的话，这个字段相当于合同签署日期）；应用（后端）通常需要检查这个字段，确保时间戳符合业务规则。
* *domain* - `string`: 应用源URL中的主机名部分。相当于书面合同中的甲方，应用（后端）需要严格的和真实domain匹配，防止被第三方意外窃听后的跨域重放攻击。
* *signer* - `string`: 用户私钥对应的地址的16进制表示，以'0x'开头。
* *signature* - `string`: 签名的16进制表示，以'0x'开头。

## Encoding

此方案的设计初衷不涉及与区块链的交互，待签名消息体对存储空间不敏感，因此我们选择简单、灵活、通用的JSON作为编码格式。为了使编码结果具有确定性，对编码过程作如下约定：

1. JSON 对象的 key 按照字母表顺序升序排列；
2. 编码结果必须是紧凑的，即：无缩进、无换行；
3. 涉及字符编码，均使用 `utf-8`；
4. *signer* 和 *signature* 字段编码为16进制，0x开头的小写形式。

## Hashing

默认哈希算法采用 VeChain 广泛使用的 `blake2b 256-bit` 哈希算法。

## Signing

默认签名算法使用 `secp256k1`。待签名消息定义为：包含除 *signature* 以外的所有字段，经过上述编码方式编码后的输出。

## Verification

1. 编码待签名消息，并计算哈希；
2. 根据签名和待签名消息哈希恢复出公钥，并推导出地址；
3. 与*signer*字段比对，若一致则签名有效。注意：如果以16进制字符串形式比对，则忽略大小写。

## Certificate ID

完整的证书包含有效的 *signature* 字段。经过上述编码方式编码后的输出再经过哈希得到证书ID。