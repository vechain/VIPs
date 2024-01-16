---
VIP: 181
Title: Non-fungible Token Standard
Category: Application
Author: Vechain Foundation
Status: Superseded
CreatedAt: 2018-09-01
---

## ⚠️ 取代 ⚠️

**此标准 (VIP-181) 已被以太坊上的 [ERC721](https://github.com/ethereum/ercs/blob/master/ERCS/erc-721.md) 标准取代。强烈建议不要继续开发和使用 VIP-181，建议现有实现迁移到 ERC721。**

有关在 VechainThor 网络上使用 ERC721 非同质化代币的详细指南，请参阅[文档](https://github.com/ethereum/ercs/blob/master/ERCS/erc-721.md)。

# 概述

VIP181标准协议是基于[ERC721](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md)的修改，对非同质化代币进行规范并提供了转移数字货币的基本能力。

# 规则

## 方法

### name

    function name() public view returns(string)

获取代币的名字。

Return: 代币名称


### symbol

    function symbol() public view returns(string)

获取代币的符号。

Return: 代币符号


### totalSupply

    function totalSupply() public view returns(uint256)

获取当前代币总供应量。

Return: 总供应量


### balanceOf


    function balanceOf(address _owner) public view returns(uint256)

返回由`_owner`持有的NFTs的数量。

Params:

+ _owner: 地址

Return: 持有的NTFs数量


### ownerOf

    function ownerOf(uint256 _tokenId) public view returns(address)

返回`_tokenId`代币持有者的地址。

Params: 

+ _tokenId: NTFs代币的编号

Return: 持有者地址


### transferFrom

    function transferFrom(address _from, address _to, uint256 _tokenId) public

转移`_from`地址持有的`_tokenId`的所有权至`_to`地址，一次成功的转移操作必须发起`Transer`事件。任何失败的交易都必须`throw`异常。

Params:

+ _from: 转移代币的地址
+ _to: 接收代币的地址
+ _tokenId: 需要转移的代币ID


### approve

    function approve(address _spender, uint256 _tokenId) public

授予`_spender`地址具有`_tokenId`的控制权，方法成功后需触发`Approval`事件。

Params:

+ _spender: 被授权的地址
+ _tokenId: 代币ID


### getApproved

    function getApproved(uint256 _tokenId) public view returns(address)

查询`_tokenId`的代币授权情况。

Return: 被授权的地址


## setApprovalForAll

    function setApprovalForAll(address _operator, bool _approved) public

授予地址`_operator`具有所有NFTs的管理权，成功后需触发`ApprovalForAll`事件。

Params:

+ _operator: 被授权的地址
+ _approved: 是否授权


## isApprovedForAll

    function isApprovedForAll(address _owner, address _operator) public view returns(bool)

查询`_operator`是否被完全授权。

Params:

+ _owner: 授权地址
+ _operator: 被授权的地址

Return: 授权结果


## 事件

### Transfer

    event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId)

当代币发生下列情况是，必须触发该事件：

+ 发生转移时
+ 增发代币时
+ 销毁代币时


### Approval

    event Approval(address indexed _owner, address indexed _spender, uint256 indexed _tokenId)

任何成功调用`approve`方法后，必须触发该事件。


### ApprovalForAll

    event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved)

任何成功调用`setApprovalForAll`方法后，必须触发该事件。

