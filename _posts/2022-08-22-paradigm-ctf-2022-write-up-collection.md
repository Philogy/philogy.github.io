---
title: "Paradigm CTF 2022: Write-Up Collection"
date: 2022-08-22 12:36
categories: [security-write-ups]
tags: [solidity, EVM, security, smart contracts]

---



## Intro
I participated in the Paradigm CTF 2022 where I was personally able to solve 7 out of the 13 EVM-related challenges (not counting the PVP game 0xMonaco and external underhanded solidity contest). I made a few 0xMonaco cars as well, but at the end we went with the car my teammates made since I was only able to work on mine for a few hours. I did not attempt or look at any of the Cairo / Solana challenges as I felt that I would not be able to do so much in the 48h provided.

Shout out to my team the [notfellows](https://twitter.com/notfellows)! 😄

Checkout the [Glossary]({% post_url 2022-08-22-paradigm-ctf-2022-write-up-collection %}#glossary) for links to other write-ups.

### Intro - General EVM Challenge Structure 
For the 13 EVM related challenges (not counting 0xMonaco, underhanded 2022) the goal was to get the `Setup` contract's `isSolved` method to return `true`  so that the flag could be retrieved from the server. The contracts were deployed to a private chain which was forked off of mainnet. This is an important detail as not all contracts were provided by the CTF, some were just the mainnet contracts which you then had to look at on etherscan.
 

## Hint Finance 🚩

> "Earn rewards without risk by depositing tokens into one of HintFinance's vaults!"

The famous  last words of many projects.

### Hint Finance - Intro

In case you're just looking for the code here's the [**full script**](https://gist.github.com/Philogy/61534359bbdccb016ebf86b21a0cb8a6) in foundry.

In the Hint Finance challenge the goal was to drain 3 identical vault contracts of at least 99% of their underlying token. While the vaults were identical, the tokens stored in the vaults were different.

The vault contract was set up as a typical reward contract whereby you deposit some amount of tokens X in return for vault shares which can be redeemed for your tokens X again, along some rewards in several tokens Y. Besides the `deposit` and `withdraw` methods the contracts also had a `flashloan` method allowing you to borrow any token the vault holds.

There were also a couple of methods related to rewards but I quickly realized that those were just distractions as you have no way of setting the reward tokens and all the configured reward tokens were not the actual target tokens you're looking to drain

### Hint Finance - Initial Ideas 🤔
Initially I looked at the flashloan method trying to find a way to deposit/withdraw during the loan but due to its design I had to conclude that this was not possible. This is because the flashloan method enforces not only that the tokens be returned but also that the amount of total outstanding vault shares remain constant. This means you can't keep new shares, have existing shares be redeemed or otherwise mess with the share price besides increasing it by returning more tokens than necessary. 

### Hint Finance - ERC777 Does It Again 😬
After my initial look I remembered that the contracts are forked off mainnet and decided to check their code. There I discovered that 2 of the 3 target tokens were ERC777 tokens.

> ERC777 tokens are token contracts which are backwards compatible with ERC20 but implement added functionality based on the [ERC777 standard](https://eips.ethereum.org/EIPS/eip-777). Most importantly the standard specifies that a token's `transfer` / `transferFrom` methods must function as follows (excluding allowance logic):
> 1. Call the sender's pre-transfer hook if one is registered
> 2. Change balance of sender, reciepient
> 3. Call the sender's post-transfer hook if one is registered
{: .prompt-tip}

Upon discovering that the 2 tokens were ERC777 tokens I went to check the obvious vulnerability that came to mind, thinking "surely it couldn't be this straight forward". I looked at the `withdraw` and `deposit` methods and checked whether I could leverage the pre-/post-transfer hooks to somehow exploit the vault:

```solidity
function deposit(uint256 amount)
    external
    updateReward(msg.sender)
    returns (uint256)
{
    uint256 bal = ERC20Like(underlyingToken).balanceOf(address(this));
    uint256 shares = totalSupply == 0
        ? amount
        : (amount * totalSupply) / bal;
    ERC20Like(underlyingToken).transferFrom(
        msg.sender,
        address(this),
        amount
    );
    totalSupply += shares;
    balanceOf[msg.sender] += shares;
    return shares;
}

function withdraw(uint256 shares)
    external
    updateReward(msg.sender)
    returns (uint256)
{
    uint256 bal = ERC20Like(underlyingToken).balanceOf(address(this));
    uint256 amount = (shares * bal) / totalSupply;
    ERC20Like(underlyingToken).transfer(msg.sender, amount);
    totalSupply -= shares;
    balanceOf[msg.sender] -= shares;
    return amount;
}
```


Looking at the `deposit` method you only get control via the pre-transfer hook. This is not useful as the logic for updating the token balance and issuing new shares happens after the hook is complete, oustside of your control.

The `withdraw` method is the jackpot however as you get the control-flow via the post-transfer hook, **after** the balance was changed but **before** your share balance is updated. Let's break it down with an example where we withdraw everything to illustrate: 

Step|Vault Balance|Your Balance|Total Shares|Your Shares|Share Price
---|---|---|---|---|---
Before `withdraw` call|100.00|0.0|100.0|50.0|1.0
**Start of post-transfer hook**|**50.00** (-50)|**50.0** (+50)|100.0|50.0|0.5
Contract reduces share balance|50.00|50.0|50.0 (-50)|0.0 (-50)|1.0
**After call:**|50.00|50.0|50.0|0.0|1.0

As you'll notice there's a temporary dip in share price at the point where we take over the control-flow. We can "buy the dip" we artificially create by depositing back into the vault:

Step|Vault Balance|Your Balance|Total Shares|Your Shares|Share Price
---|---|---|---|---|---
Before `withdraw` call|100.00|0.0|100.0|50.0|1.0
Start of post-transfer hook|**50.00** (-50)|**50.0** (+50)|100.0|50.0|0.5
After buying shares `deposit(50.0)`|**100.00** (+50)|0.0 (-50)|**200.0** (+100)|**150.0** (+100)|0.5
Contract reduces share balance|100.0|0.0|**150.0** (-50)|**100.0** (-50)|0.667
**After call:**|100.0|0.0|150.0|100.0|**0.667**

Result is we were able to mint shares "out of thin air", diluting the other depositors and allowing us to redeem more tokens. If we repeat this a couple times we'll eventually own 99%+ of the shares and be able to drain the 2 ERC777 (PNT, AMP) tokens from the vault.

I'm not going to detail the implementation of a ERC777 based reentrancy attack, as it's a fairly common thing but you can view my exploit script [here](https://gist.github.com/Philogy/61534359bbdccb016ebf86b21a0cb8a6) if you're interested in the implementation details.

### Hint Finance - The Final Token, How????!!! 😡
After the first 2 tokens were relatively straight forward I started wondering how I was gonna approach the last one, SAND (ID: 1). Its transfer methods had no strange hooks, the contract file names didn't seem to advertise any unusual standards being implemented so what can I do?

Well I went back to the vault contract and made an inventory of all the potential external calls I could somehow use:
- [X] ~~`transferFrom` in  `provideRewardTokens(...)` (only transfers rewards in)~~
- [X] ~~`transfer` in  `getRewards(...)` (only transfers specific reward tokens out)~~
- [X] ~~`transferFrom` in  `deposit(...)` (only transfers in tokens)~~
- [X] ~~`transfer` in  `withdraw(...)` (transfers out but only exploitable with reentrancy)~~
- [X] ~~`transfer` in  `flashloan(...)` (transfers out but cannot keep tokens)~~
- [ ] `onHintFinanceFlashloan` in  `flashloan(...)` (triggers flashloan callback)
- [X] ~~several `balanceOf` (view method, meaning staticcall used under the hood)~~

Hmmm 🤔, there seems to be only two possibilities left: either I'm wrong about one of the other paths or the selector of the weirdly named `onHintFinanceFlashloan` method collides with another useful method. This is possible because the [ABI spec](https://docs.soliditylang.org/en/develop/abi-spec.html#function-selector) says that selectors have to be the first 4-bytes of the function signature.

So I calculated the selector of `onHintFinanceFlashloan`, `0xcae9ca51` and put it in [4byte.directory](https://www.4byte.directory/signatures/?bytes4_signature=0xcae9ca51), a database of function signatures and their selectors. Note that when I looked at 4byte during the challenge `onHintFinanceFlashloan` was not yet uploaded.

I was right, not only is there an alternative function but it was already uploaded to 4byte (I could've been searching much longer if I had to check it manually) and it was also present on the token contract: the `approveAndCall(...)` method! This is exactly what I needed, if you craft the call payload well the vault will simply approve some address you create to spend tokens on its behalf and the tokens will be mine 🤑.

### Hint Finance - Flexible ABI

Before doing a  victory lapse we actually have to exploit this vulnerability which is not that trivial. Let's look at the `flashloan` method as a whole:

```solidity
function flashloan(
    address token,
    uint256 amount,
    bytes calldata data
) external updateReward(address(0)) {
    emit log_named_address("token", token);
    emit log_named_uint("amount", amount);
    emit log_named_bytes("data", data);
    uint256 supplyBefore = totalSupply;
    uint256 balBefore = ERC20Like(token).balanceOf(address(this));
    bool isUnderlyingOrReward = token == underlyingToken ||
        rewardData[token].rewardsDuration != 0;

    ERC20Like(token).transfer(msg.sender, amount);
    IHintFinanceFlashloanReceiver(msg.sender).onHintFinanceFlashloan(
        token,
        factory,
        amount,
        isUnderlyingOrReward,
        data
    );

    uint256 balAfter = ERC20Like(token).balanceOf(address(this));
    uint256 supplyAfter = totalSupply;

    require(supplyBefore == supplyAfter);
    if (isUnderlyingOrReward) {
        uint256 extra = balAfter - balBefore;
        if (extra > 0 && token != underlyingToken) {
            _updateRewardRate(token, extra);
        }
    } else {
        require(balAfter == balBefore); // don't want random tokens to get stuck
    }
}
```

The `flashloan` method calls the caller, not necessarily the token directly meaning we need the token to be the caller somehow and give the vault a specific payload. Thankfully we can just leverage `approveAndCall` again, approving the vault and calling the `flashloan` method.
 
There's a few more tricky bits I had to find out while debugging my exploit as the two colliding methods have different parameters. Let's breakdown the calldata that the vault creates when calling `onHintFinanceFlashloan`:
```
cae9ca51 // selector
0x00: <32-byte padded address> // vault: token, SAND: target
0x20: <32-byte padded address> // vault: factory, SAND: amount
0x40: <32-byte uint>  // vault: amount, SAND: data.offset
0x60: <32-byte padded bool> // vault: isUnderlyingOrReward, SAND: ?
0x80: <32-byte padded byte-string offset (0xa0)> // vault: data.offset, SAND: ?
0xa0: <32-byte padded byte-string length> // vault: data.length, SAND: ?
0xc0..X: <byte-string> // vault: data, SAND: ?
```

Firstly  the recipient of the approval will be the `token`. This means that we can't directly be the recipient of the approval because solidity will revert if you try to call methods like `balanceOf` or `transfer` on an EOA. So I created a custom contract to execute the exploit which had some methods so that it can pretend it's an ERC20 token:
```solidity
function transfer(address, uint256) external returns (bool) {
	// just needs to not revert
	return true;
}

function balanceOf(address) external view returns (uint256) {
	return 1e18; // constant for the sake of simplicity
}
```

Next you'll notice that the approve amount that the token contract receives will be the address of the factory. Considering the token has 18 decimals and an address is about ~1e30 when interpreted as a number that'll be more than enough to drain the contract in one go.

A further aspect is that `amount` can't be just anything. If it's `0` and points at an address it'll imply such a large byte-string that `approveAndCall` will just revert when it tries to copy it from calldata to memory. Instead we set it to `0xa0` so that when `approveAndCall` tries to get the data it'll find the `data`  provided by the vault.

Another error I got is `"first param != sender"` from the token contract, this is because the `approveAndCall` method enforces that the data has a minimum length and that the first argument is the caller's address. To ensure the correct length and address I set the return payload of the nested call to `approveAndCall` to be:
```solidity
bytes memory innerPayload = abi.encodeWithSelector(
	bytes4(0x00000000),
	_sandVault,
	bytes32(0),
	bytes32(0), 
	bytes32(0) 
);
```
The selector is `0x00000000` for simplicity so I add an empty fallback method to my exploit contract so it accepts the final call.

Finally I just nest the calls and use `transferFrom` to transfer the tokens out of the vault. You can view my full solution [here](https://gist.github.com/Philogy/61534359bbdccb016ebf86b21a0cb8a6).

### Hint Finance - Conclusion
The Hint Finance challenge reminds us that we should always be wary of external calls to user controllable addresses and that you should always assume that ERC20 tokens can reenter upon transfers, just like ETH.

## Just In Time (JIT) 🚩
### JIT - Intro 
In case you're just looking for the code here's the [**full script**](https://gist.github.com/Philogy/f4f48a29053344ce467c12ff62ebea8a) I wrote to solve the challenge. Considering that only 5 others solved it I am particularly proud of having solved this one. Especially since I really struggled with it: 

![Message from me "I spent half the day just staring at just-in-time sob emoji 😂😭", Georgios Konstantopoulos responds "lmao"](/assets/images/struggling-with-jit.png) 

But after an estimated ~16h of struggle my perceverance paid off and I managed to conquer the challenge!

In the Just-In-Time challenge (referred to as JIT for the remainder) the condition for receiving the flag, was draining the provided `JIT` contract of ETH. The `JIT` contract performs some logic around a provided input `program`, compiling it to EVM bytecode, subsequently deploying it and then delegate calling it. 

> **`DELEGATECALL`**
>
> Unlike a normal call, when a contract delegate calls to another it gives it complete control allowing it to modify storage, emit events, call other contracts, transfer ETH and even self destruct on its behalf.
>
> While useful for libraries, proxies and complex patterns like [ERC-2535 Diamonds](https://eips.ethereum.org/EIPS/eip-2535), it can open up a lot of attack surface.
> 
> For more details refer to [evm.codes](https://www.evm.codes/#f4) or the
> [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
{: .prompt-tip}

### JIT - Compiler Structure 🧱
If we're going to give this JIT compiler a program that'll compile to malicious code we'll have to first understand it. Skimming over the contract and reading the comments we can identify 6 modules / steps:

1. Loop finder: Finds and matches square-brackets `[]` 
2. Block finder: Based on the loops splits the program into discrete "blocks"  that can be jumped to
3. Optimizer: Shortens repetitive / inefficient program segments
4. Code generator: Generates the main meat of the code, translating jit symbols into EVM bytecode and keeping track of where blocks start and end in the bytecode. 
5. Label filler: Inserts the actual jump locations into the code.
6. Deployer: Inserts the code length into the constructor, deploys and executes the code

Side note: Looking at the basic set of symbols `[],.+-<>`, the "jit language"  seems to be a super set of [Brainfuck](https://en.wikipedia.org/wiki/Brainfuck), a minimalistic, turing complete language.

### JIT - Desired Code 
Before we trick the compiler into generating malicious code we need to ask ourselves what that code should look like so that we know what we're aiming for. There's a relatively small set of opcodes that would allow us to transfer out ETH: `CREATE` , `CALL`, `CALLCODE`, `DELEGATECALL`, `CREATE2` and `SELFDESTRUCT`.

From all these operations `SELFDESTRUCT`  seems to be the best candidate as it only consumes 1 stack element and automatically transfers all ETH, especially useful considering how constrained the compiler is. The precise recipient is irrelevant as our sole win condition is to drain the compiler, making our goal: **have the code reach a `SELFDESTRUCT` opcode with at least 1 element on the stack**.

### JIT - Getting Our Code In 🕳️
Working backwards we can look at the code generator and see that the only opcodes it directly inserts are: `SWAP`s, `PUSH`s, `MLOAD`, `MSTORE`, `JUMPDEST`, `JUMP`, `JUMPI`, `ADD`, `SUB`, `CALLDATALOAD`, `DUP`s, `SHR` and `LOG0`.

However there is a single section that inserts user defined bytes into the final result. However with heavy restrictions. If the parser doesn't recognize a byte in the program it'll insert it directly into the code, wrapped in invalid opcodes:

```solidity
// jit.compile isn't actually provided in the challenge, I simply split the invoke method for testing
jit.compile("+-\xff");
```
Results in:
```
630000001c80600e6000396000f3 (constructor)
6000618000                   (contract header, sets up stack)
5b                           (JUMPDEST, block start)
80516001018152               ("+")
8051600190038152             ("-")
deadffbeef                   (0xff wrapped in invalid opcodes)
5b00                         (contract footer, JUMPDEST STOP)
```
This presents The major issue that whatever opcode we place in the middle is **not reachable**. It can't be reached linearly because of the invalid opcodes in front and it can't be jumped to because the EVM requires a `JUMPDEST` at the location where a jump (`JUMP` / `JUMPI`) lands, meaning we actually need to insert at least 2 bytes somehow:  `5bff` (`JUMPDEST`, `SELFDESTRUCT`).

There are other compiler sections that insert custom bytes but only ever after `PUSH` opcodes meaning they aren't interpreted as their opcodes but as data.

### JIT - Breaking The Wrapper ⛏️
Note that we can put **any** opcode in the wrapper as long as its byte value doesn't overlap with one of the recognized jit symbols (`[],.<>+-RLAS0#`), we can even insert a `PUSH` opcode into the wrapper. What's interesting about the `PUSH` opcode here is that it's not an isolated opcode, the bytes following a `PUSH` get pushed on the stack, so they're not interpreted as functional opcodes but data.

While this still doesn't allows us to reach the code it does allow us to change the meaning of subsequent code. As an example imagine the following EVM code: 
```
INVALID INVALID [SELFDESTRUCT] INVALID INVALID PUSH2 0x5bff
  de      ad          ff         be      ef     61    5bff
```
Now, placing a `PUSH3` in the invalidity wrapper:
```
INVALID INVALID PUSH3 0xbeef61 JUMPDEST SELFDESTRUCT
  de      ad     62     beef61    5b         ff
```
The bytes `5bff` that were previously treated as data to be pushed, are all of a sudden malicious code!

Note in the actual JIT compiler we use the `A` / `S` symbols to insert the 2-bytes, which also inserts added opcodes before its `PUSH2` which is why my final solution uses `PUSH5` (`0x64`).

### JIT - Reaching The Wrapper
We've figured out how to insert some malicious code but now we actually need to jump to it. Looking at the loop finder, block finder and label filler modules they seem quite robust, they only ever insert jump labels that jump between the edges of blocks, which are at the program's square brackets.

Thankfully we can insert a custom jump location by leveraging 3 bugs in the compiler:
1. The loop linking mapping `loops` is not reset between calls
2. The block to code position mapping `basicBlockAddrs` is also not reset between calls
3. Unmatched open-brackets `[` are allowed and do not reset their loop values

Matched square brackets are safe because when they're found their respective links are reset. However for unmatched square brackets the block finder still queries the `loops` mapping allowing us to use values set in previous calls.

### JIT - Tying It All Together 🎁
Now that we know how to exploit the contract we need to implement the exploit, while it's relatively straight forward once you know how, getting the precise positions right is a bit finicky. 

I started with the final program, inserting some padding `#` to ensure that I had enough space to set the necessary values in prior calls: `[################\x64S\x5b\xff`. I then compiled with JIT to see where the location of the critical `JUMPDEST` was. I then crafted 2 pieces of JIT code one setting the value in the `basicBlockAddrs` mapping and the other in `loops`. It's likely possible to do it in 1 instead of 2 pre-solution programs but for simplicity's sake I just did it in 2 as it was easier.

Despite all the effort the above work condenses down to the following small solution:
```solidity
// SPDX-License-Identifier: GPL-3.0-only

pragma solidity ^0.8.13;

import "forge-std/Script.sol";

import {Setup, JIT} from "src/public/contracts/Setup.sol";

contract ExploitJIT is Script { 
	function setUp() public {} 
	
	function run() public { 
		vm.startBroadcast();
		Setup s = Setup(vm.envAddress("SETUP_ADDR"));
		JIT jit = s.TARGET();
		
		jit.invoke("[#######################]", "");
		jit.invoke("########################[]", "");
		jit.invoke("[################\x64S[\xff", "");
		
		vm.stopBroadcast(); 
	} 
}
```

### JIT - Conclusion
Just-In-Time shows us that simplicity is king and that you should avoid using storage mappings for on the fly calculations. Not only is it vulnerable but it costs a lot of gas!

This challenge also taught me that I should never give up and always look for stupid stuff I missed. I was stuck for several hours because I missed the fact that `JIT` used mappings for certain critical data structures and didn't take precautions to reset them in-between calls. Making a detailed list of assumptions I had and checking them could've helped me discover this sooner.


## Glossary
The following is a breakdown of the Paradigm CTF 2022 with links to respsective
write-ups. Challenges are sorted by total solves. Credit to [0xfoobar's thread](https://twitter.com/0xfoobar/status/1561529252029276161) for links to some of the write-ups.

- ✅ Means personally solved (for 0xMonaco just means participated)
- ❌ Did not solve

**EVM (Solidity) Challenges:**

- ✅ 0xMonaco ([winning car by 0age](https://twitter.com/z0age/status/1561685707650990084), [thread 1](https://twitter.com/sudolabel/status/1561512456291172352))
- ❌ Solidity Underhanded 2022
- ❌ Fun Reversing Challenge
- ❌ Stealing Sats
- ❌ Electric Sheep
- ✅ [Just-in-Time]({% post_url 2022-08-22-paradigm-ctf-2022-write-up-collection %}#just-in-time-jit-)
- ❌ Trapdoooor
- ✅ [Hint Finance]({% post_url 2022-08-22-paradigm-ctf-2022-write-up-collection %}#hint-finance-) 🏅 (I got first blood)
- ❌ [Trapdooor](https://twitter.com/elyx0/status/1561532604519747584)
- ❌ Lockbox 2
- ✅ [Vanity](https://twitter.com/danielvf/status/1561508004423471104)
- ✅ Sourcecode 
- ✅ Merkledrop
- ✅ Rescue
- ✅ Random

**Solana Challenges:**
- ❌ [Solhana 3](https://twitter.com/raggedsec/status/1561514247783256064)
- ❌ [Solhana 2](https://twitter.com/raggedsec/status/1561514247783256064)
- ❌ [Solhana 1](https://twitter.com/raggedsec/status/1561514247783256064)
- ❌ [Otter Swap](https://twitter.com/NotDeGhost/status/1561545438897078273)
- ❌ [Otter World](https://twitter.com/NotDeGhost/status/1561545438897078273)

**Cairo Challenges:**
- ❌ Cairo Auction
- ❌ Cairo Proxy
- ❌ Riddle Of The Sphinx


 
<br>
<br>

Found a grammar, spelling or factual error or would like to suggest an improvement? Feel free to
raise an [issue](https://github.com/Philogy/philogy.github.io/issues) or reach
out to me on [twitter](https://twitter.com/real_philogy).

