> VIP: 181
> Title: VIP-181 Non-fungible Token Standard
> Category: Standard
> Author: VeChain
> Status: Final
> CreatedAt: 2018-09-01
--------------------- 

# Overview

The VIP-181 Standard outlines a set of common methods that all NFT tokens can follow on the VeChainThor Network to transfer tokens, as well as allow tokens to be approved so they can be spent by another on-chain thrid party safely.


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


### totalSupply

    function totalSupply() public view returns(uint256)

Gets the total token supply.

Return: the total token supply


### balanceOf


    function balanceOf(address _owner) public view returns(uint256)

Gets the amount of NTFs the `_owner` possesses.

Params:

+ _owner: the address of owner

Return: the balance of owner


### ownerOf

    function ownerOf(uint256 _tokenId) public view returns(address)

Gets the owner of `_tokenId`.

Params: 

+ _tokenId: the token id

Return: the owner address


### transferFrom

    function transferFrom(address _from, address _to, uint256 _tokenId) public

Transfer ownership of an NFT from address `_from` to address `_to`, and **MUST** emit the `Transfer` event. The function **MUST** throw if the transaction failed.

Params:

+ _from: the payment address
+ _to: the receiver address
+ _tokenId: the token id


### approve

    function approve(address _spender, uint256 _tokenId) public

Allows `_spender` to control your token with `_tokenId`, and **MUST** emit the `Approval` event. If this function is called again it overwrites the current allowance with `_tokenId`.

Params:

+ _spender: the spender address
+ _tokenId: the token id


### getApproved

    function getApproved(uint256 _tokenId) public view returns(address)

Get the approved address for `_tokenId`

Return: the approved address


## setApprovalForAll

    function setApprovalForAll(address _operator, bool _approved) public

Enable or disable approval for `_operator` address to manage all your assets, and **MUST** emit the `ApprovalForAll` event.

Params:

+ _operator: the approved address
+ _approved: True if the operator is approved, false to revoke approval


## isApprovedForAll

    function isApprovedForAll(address _owner, address _operator) public view returns(bool)

Check if an address is an authorized operator for another address

Params:

+ _owner: the address of owner
+ _operator: the address that acts on behalf of the owner

Return: the result of authorization


## Events

### Transfer

    event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId)

**MUST** trigger when the following scenario occurs:

+ when transferring tokens
+ when minting tokens
+ when burning tokens


### Approval

    event Approval(address indexed _owner, address indexed _spender, uint256 indexed _tokenId)

**MUST** trigger when call the `approve` method successfully.


### ApprovalForAll

    event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved)

**MUST** trigger when call the `setApprovalForAll` method successfully.

