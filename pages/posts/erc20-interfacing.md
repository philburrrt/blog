---
title: Getting a contract interface with a specific ERC20 token
date: 2022/5/23
description: Breaking down ERC20 deposits in staking contracts & general interfacing. 
tag: solidity
author: Philbert Burrow
---

# Getting a contract interface with a specific ERC20 token

Creating a smart contract which interacts with a specific ERC20 token, one that might not exist yet, felt particularly unintuitive. 

Here's the **TestToken** that I used throughout my learning experience:

```solidity=
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract TestToken is ERC20, Ownable {
    constructor() ERC20("TestToken", "TT") {}

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }
}
```


## Contract that checks a user's balance of a specific token

The thing that I wish I would have known here is that, basically, importing IERC20 is your ticket to interacting with ERC20 contracts. Presumably, that goes for any popular token format.

Other than that, I created a ```setRootToken()``` function which allows me to set the address of the accepted token after the contract is deployed. This is nice because I will be deploying the token and locker contract at the same time.


### Locker.sol

```solidity=
// SPDX-License-Identifier: MIT

pragma solidity 0.8.12;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract Locker is Ownable {

    address public rootToken;

    constructor () {}

    function setRootToken(address _rootToken) public onlyOwner {
        rootToken = _rootToken;
    }

    function getUserBalance(address _user) external view returns(uint256) {
        return IERC20(rootToken).balanceOf(_user);
    }

```

## Contract that allows a user to deposit a specific ERC20 and records the balance

So this is really powerful and doesn't take much more code. 

We can't call the getUserBalance() function within depositToken() because it technically does not exist when compiling. I removed that function and added its code to depositToken().

Lastly, create a map of addresses and their locked balances, using the quantity entered when depositing.

### Locker.sol

```solidity=
// SPDX-License-Identifier: MIT

pragma solidity 0.8.12;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract Locker is Ownable {

    address public rootToken;

    mapping (address => uint256) public lockedTotal;

    constructor () {}

    function setRootToken(address _rootToken) public onlyOwner {
        rootToken = _rootToken;
    }

    function depositToken(uint256 amount) public {
        require(amount <= IERC20(rootToken).balanceOf(msg.sender));
        IERC20(rootToken).transferFrom(msg.sender, address(this), amount);
        lockedTotal[msg.sender] = amount;
    }

}
```
