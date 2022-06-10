---
title: Testing with Foundry
date: 2022/6/10
description: A simple Foundry test, annotated
tag: solidity
author: Philbert Burrow
---

# Testing with Foundry

I've been working with solidity for a month or two now. Started with hardhat, just because it had the most users and recently migrated all of my current Projects to foundry. It seems like an obvious move for many reasons.

The docs have been pretty solid, but I couldn't find many details about testing. And it can be a bit more intimidating than Hardhat as most are slightly familiar with javascript and ethers.

Inevitably, I found that [this video](https://www.youtube.com/watch?v=pgh74-XulXg&ab_channel=Chainlink) taught me what I needed to know to get started within the first 25 minutes. But, maybe I can break that down a bit more quickly via a code block.

```
pragma solidity ^0.8.13;

import "ds-test/test.sol";

import "../src/StakeContract.sol"; // staking contract that we're testing
import "./mock/MockERC20.sol"; // ERC20 we created which automatically mints tokens to the deployer address

contract ContractTest is DSTest {

    StakeContract public stakeContract; // declare to interface with the staking contract
    MockERC20 public token; // declare to interface with the mock token
    uint256 public constant AMOUNT = 1e18;

    function setUp() public {
        stakeContract = new StakeContract(); // deploy the staking contract at the beginning of tests
        token = new MockERC20(); // deploy the mock token at the beginning of tests
    }

    function testStaking() public {

        token.approve(address(stakeContract), AMOUNT); 
        bool success = stakeContract.stake(AMOUNT, address(token)); // if this doesn't revert, success will be true
        assertTrue(success); // test will fail if unsuccessful

    }
}
```

Here are the two contracts referenced in this test. If you add these and enter command ``forge test``, it should pass.

**StakeContract.sol**

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

error StakeContract__TransferFailed();

contract StakeContract {

    mapping(address => uint256) public s_balances;

    function stake(uint256 amount, address token) external returns (bool) {

        s_balances[msg.sender] = s_balances[msg.sender] + amount;

        bool success = IERC20(token).transferFrom(msg.sender, address(this), amount);
        if(!success) revert StakeContract__TransferFailed();
        return success;
    }
}
```

**MockERC20.sol**

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.13;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MockERC20 is ERC20 {
    constructor() ERC20("MockERC20", "MERC") {

        _mint(msg.sender, 100000e18);

    }
}
```