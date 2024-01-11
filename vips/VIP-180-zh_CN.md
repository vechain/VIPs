---
VIP: 180
Title: Fungible Token Standard
Category: Application
Author: Vechain Foundation
Status: Superseded
CreatedAt: 2018-09-01
---

## ⚠️ 取代 ⚠️

**此标准 (VIP-180) 已被以太坊上的 [ERC20](https://github.com/ethereum/ERCs/blob/master/ERCS/erc-20.md) 标准取代。强烈建议不要开发和使用 VIP-180，建议现有实现迁移到 ERC20。**

有关在 VechainThor 网络上使用 ERC20 作为同质化代币的详细指南，请参阅文档[此处](https://github.com/ethereum/ERCs/blob/master/ERCS/erc-20.md)。

# 概述

VIP180标准协议基于[ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)的修改，对同质化代币进行规范并提供了转移数字货币的基本能力。

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


### decimals

    function decimals() public view returns(uint8)

获取代币使用的小数位位数。

Return: 小数位位数


### totalSupply

    function totalSupply() public view returns(uint256)

获取代币总供应量。

Return: 总供应量


### balanceOf

    function balanceOf(address _owner) public view returns(uint256)

获取指定地址所持有的代币余额。

Params:

+ _owner: 地址

Return: 代币余额


### transfer

    function transfer(address _to, uint256 _value) public

转移`_value`数量的代币到`_to`地址，并且触发`Transfer`事件。任何失败的交易都必须`throw`异常。

Params:

+ _to: 收款地址
+ _value: 转移金额


### transferFrom

    function transferFrom(address _from, address _to, uint256 _value) public

从`_from`地址转移`_value`数量的代币到`_to`地址，并且触发`Transfer`事件。任何失败的交易都必须`throw`异常。

Params:

+ _from: 扣款地址
+ _to: 收款地址
+ _value: 扣款金额


### approve

    function approve(address _spender, uint256 _value) public

允许`_spender`多次提取您的帐户代币，最高达`_value`的代币，并触发`Approval`事件。如果再次调用此函数，它将以`_value`覆盖当前的余量。

Params:

+ _spender: 被允许的地址
+ _value: 额度


### allowance

    function allowance(address _owner, address _spender) public view returns(uint256)

查询`_spender`地址仍然被允许从`_owner`提取的代币数量。

Params:

+ _owner: 允许地址
+ _spender: 被允许的地址

Return: 剩余额度


## 事件

### Transfer

    event Transfer(address indexed _from, address indexed _to, uint256 _value)

当代币发生下列情况是，必须触发该事件：

+ 发生转移时
+ 增发代币时
+ 销毁代币时


### Approval

    event Approval(address indexed _owner, address indexed _spender, uint256 _value)

任何成功调用`approve`方法后，必须触发该事件。
