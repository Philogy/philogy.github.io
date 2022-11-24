---
title: "Smart Contract Design: The Operator and FBA Patterns"
categories: [smart-contract-development]
tags: [solidity, EVM, security, smart contracts]

---

# Test

**Core content:**
- Main EOA -> contract interaction patterns
- Existing operator patterns
- Advantage of operator pattern:
    - Improved composability
    - Gas savings on peripheries


## Intro
Building and reviewing several smart contracts I've noticed that a certain set of patterns or
moreover their lack leads to particular UX (user experience) and DX (developer experience)
headaches. In this post I'd like to document them, share some insight and hopefully help you build
better smart contract applications.

If applied correctly the patterns laid out herein will help your applications be more efficient,
composable and secure.

## Why Periphery Contracts Are Awesome
Periphery contracts also referred to as "utility" contracts are separate contracts that can
trustlessly add features and/or improve the UX of existing smart contracts by bundling and
automating a users' calls to different smart contracts. The Uniswap 🦄 Router contracts are a good
example: In Uniswap V1-V3 pairs are managed by individual pool contracts, if a certain pair does not
exist a user will have to trade over multiple pairs, however EOAs cannot safely do this directly. A
periphery contract, the Uniswap Router enables EOAs to easily transact across multiple pools.

Separating your application into core and periphery contracts is a tradeoff:


- Separation of concerns
- Makes core more generalizable and reusable
- 

Pros | Cons
------|---
Reason 1 | Reason 2


## Delegating Control
### Delegating Control -- Common use
Before expanding to the generalizable pattern itself I'd like to take a closer look at the commonly
used [ERC20](https://eips.ethereum.org/EIPS/eip-20) and [ERC721](https://eips.ethereum.org/EIPS/eip-721)
token standards. ERC20 and ERC721 both have "approve"-style methods, allowing users to set accounts
that can transfer tokens on their behalf. While many criticize the common token acceptance pattern
emerging from ERC20's approve method I'd make the case that the approve methods of both standards
significantly improve UX if you imagine the alternative of no approval methods.

Without the approve methods (ERC20 & ERC721: `approve`, ERC721: `setApprovalForAll`) EOAs would need
at least 2 setup transactions and 1-3 transactions every time they wanted to trustlessly deposit
assets into a contract. This could be reduced to 0 setup transactions and exactly 2 transactions for
every deposit with the help of private mempools, however that would still be more than the 1 setup
and 1 direct interaction transaction enabled by the approve pattern.

### Delegating Control -- Why Periphery Contracts Are Awesome

Beyond ERC20, ERC721 or other asset contracts the ability to delegate control over certain
functionality in a contract is useful for contracts which manage some user individualized state
similar to asset balances. For example staking and lending protocols whereby different accounts have
"sub-accounts" within these contracts. Per



### Delegating Control -- The Operator Pattern



