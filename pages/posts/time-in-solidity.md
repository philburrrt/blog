---
title: Dealing with time in solidity
date: 2022/5/25
description: Solidity uses Unix time, which might be less complicated than you think 
tag: solidity
author: Philbert Burrow
---

# Dealing with time in solidity

Before anything:

UNIX TIME !!!

That's really all you need to know. 

Unix time is basically the number of seconds passed since January 1st, 1970. The counter acts as a time oracle in computing.

You may read that ```block.timestamp``` or (depricated) ```now``` returns the seconds since the last epoch. At first, I interpreted this to mean since the last Ethereum epoch, but really it just means since 1970.

That being said, here is a locker that I created utilizing the Unix time references.

```
contract Locker is Ownable, ReentrancyGuard {

    using SafeMath for uint256;
    address public rootToken;

    struct LockTimer {
        uint256 start;
        uint256 end;
        uint256 amount;
    }

    mapping(address => LockTimer) timers;

    function setRootToken(address _rootToken) public onlyOwner {
        rootToken = _rootToken;
    }

    function lock(uint256 _end, uint256 amount) public {
    
        require(_end > 0 && _end <= 5);
        require(amount <= IERC20(rootToken).balanceOf(msg.sender));
        IERC20(rootToken).transferFrom(msg.sender, address(this), amount);

        timers[msg.sender].amount += amount;

        if(_end == 1) {
            timers[msg.sender].end = block.timestamp + 4 weeks;
            timers[msg.sender].start = block.timestamp;
        }
        else if(_end == 2) {
            timers[msg.sender].end = block.timestamp + 8 weeks;
            timers[msg.sender].start = block.timestamp;
        }
        else if(_end == 3) {
            timers[msg.sender].end = block.timestamp + 16 weeks;
            timers[msg.sender].start = block.timestamp;
        }
        else if(_end == 4) {
            timers[msg.sender].end = block.timestamp + 32 weeks;
            timers[msg.sender].start = block.timestamp;
        }
        else if(_end == 5) {
            timers[msg.sender].end = block.timestamp + 64 weeks;
            timers[msg.sender].start = block.timestamp;
        }

    }

    function withdraw(uint256 amount) external nonReentrant {
        require(timers[msg.sender].end > block.timestamp);
        require(timers[msg.sender].amount <= amount);
        IERC20(rootToken).transfer(msg.sender, amount);
        timers[msg.sender].amount -= amount;
    }

    function checkTimer(address _address) public view returns (uint256, uint256) {
        return(
            timers[_address].start,
            timers[_address].end
        );
    }
```

Notes:

- ```weeks``` is short for 'the amount of seconds in one week'
- You don't want users to have to count seconds, so it's best to give them options. I did so using 1-5, which will be programmed to a button stating the amount of weeks added on the UI
