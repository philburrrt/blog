---
title: My first fatal mistake in prod
date: 2022/06/12
description: Write solidity slowly and carefully
tag: solidity
author: Philbert Burrow
---

# My first fatal mistake in prod

During my experience developing token & related contracts for Hyperfy, I realized that a very simple, custom ERC1155 implementation could help a group of friends distribute GLBs that otherwise would go unused. At the time, I had way too many things on my plate, but was excited about this idea and thought I could quickly execute on it, then dip out.

That was the first mistake I made -- thinking that I could push out a custom contract, in a few hours and nothing would go wrong.

Everything was created in hardhat. I ran enough tests against it to feel comfortable. Deployed to Rinkeby to make sure everything looked good on Opensea, re-read my code about 10 times and then yolo deployed. My friend was pretty anxious to publish something, and I had work to get back to.

After spending $350 deploying to mainnet, my friend asked me how we were going to manage profits from minting. I told him that I'd transfer ownership to our multi-sig and we'd withdraw at our convenience.

Then I realized that ERC1155 does not have a built in withdrawal function.

Thankfully, the first drop was going to be free and for friends only, so it's not like we came anywhere near to getting all of our earnings stuck in a contract. And I realized this about 15 minutes after deploying. But still, it's was a pretty sloppy mistake that made me look like a total noob. I was embarrassed.

But, as any productive person would do, I listened to the criticism my 100x more experienced engineer friends gave me, and took a serious look at the corners I was cutting. I simply had assumed that it would be common sense to have a withdrawal function in a contract that you purchase NFTs from.

So, here's what I changed up after the incident: 

- Started using Foundry instead of Hardhat to improve my solidity knowledge by testing with solidity
- Started reading every contract that I imported from any library
- Created an excel sheet to better organize my testing - grouped by functions
- Placed more emphasis on learning the basics rather than solving specific problems

Moral of the story is, get in a slow and steady mentality when dealing with smart contracts. That way, you won't waste your money or people's time.
