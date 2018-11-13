> VIP: 180
> Title: VIP-180 Fungible Token Standard
> Category: Application
> Author: VeChain Foundation
> Status: Final
> CreatedAt: 2018-09-01
--------------------- 

# Overview

The VIP-180 Standard is a superset of the [ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md). It outlines a set of common APIs that all tokens can follow on the VeChainThor Network to transfer tokens, as well as allow tokens to be approved so they can be spent by another on-chain thrid party safely.

# Specification

## Methods

### name

    function name() public view returns(string)

Gets the name of the token.

Return: the name of the token


### symbol

    function symbol() public view returns(string)

Gets the symbol of the token.

Return: the symbol of the token


### decimals

    function decimals() public view returns(uint8)

Gets the number of decimals the token uses.

Return: the number of decimals


### totalSupply

    function totalSupply() public view returns(uint256)

Gets the total token supply.

Return: the total token supply


### balanceOf

    function balanceOf(address _owner) public view returns(uint256)

Gets the balance of the `_owner` account.

Params:

+ _owner: the address of owner

Return: the balance of owner


### transfer

    function transfer(address _to, uint256 _value) public

Transfer `_value` amount of tokens to address `_to `, and **MUST** emit the `Transfer` event. The function **MUST** throw if the transaction failed.

Params:

+ _to: the receiver address
+ _value: the amount of tokens


### transferFrom

    function transferFrom(address _from, address _to, uint256 _value) public

Transfers `_value` amount of tokens from address `_from` to address `_to`, and **MUST** emit the `Transfer` event. The function **MUST** throw if the transaction failed.

Params:

+ _from: the payment address
+ _to: the receiver address
+ _value: the amount of tokens


### approve

    function approve(address _spender, uint256 _value) public

Allows `_spender` to withdraw from your account multiple times, up to the `_value` amount, and **MUST** emit the `Approval` event. If this function is called again it overwrites the current allowance with `_value`.

Params:

+ _spender: the spender address
+ _value: the amount of tokens


### allowance

    function allowance(address _owner, address _spender) public view returns(uint256)

Gets the amount which `_spender` is still allowed to withdraw from `_owner`.

Params:

+ _owner: the owner address
+ _spender: the spender address

Return: the remaining amount of tokens


## Events

### Transfer

    event Transfer(address indexed _from, address indexed _to, uint256 _value)

**MUST** trigger when the following scenario occurs:

+ when transferring tokens
+ when minting tokens
+ when burning tokens


### Approval

    event Approval(address indexed _owner, address indexed _spender, uint256 _value)

**MUST** trigger when call the `approve` method successfully.
