---
title: "Paradigm CTF 2022: Write-up collection"
date: 2022-08-22 12:36
categories: [security-write-ups]
tags: [solidity, EVM, security, smart contracts]

---

> **Disclaimer:** Tokens mentioned in this article are not endorsed in anyway, they were just part of the challenges.
{: .prompt-danger}

## Intro
I participated in the Paradigm CTF 2022 where I was personally able to solve 6 out of the 21 total challenges: ✅ [Hint Finance](#hint-finance), [Just-in-Time](#just-in-time), Merkledrop, Rescue, Sourcecode and Vanity (The check-marks represent which challenges I've written up in this post, some I may add later and for others I'll link to other write-ups). I only got first blood on Hint Finance 🏅. I made a few 0xmonaco cars, but at the end we went with the car my teammates made since I was only able to work on mine for a few hours.

Shout out to my team the [notfellows](https://twitter.com/notfellows)! 😄

### General EVM Challenge Structure 
For the 13 EVM related challenges (not counting 0xmonaco, underhanded 2022) the goal was to get the `Setup` contract's `isSolved` method to return `true`  so that the flag could be retrieved from the server. The contracts were deployed to a private chain which was forked off of mainnet. This is an important detail as not all contracts were provided by the CTF, some were just the mainnet contracts which you then had to look at on etherscan.
 

## 🚩 Hint Finance
> "Earn rewards without risk by depositing tokens into one of HintFinance's vaults!"

The famous  last words of many projects.

### Hint Finance - Intro

In the Hint Finance challenge the goal was to drain 3 identical vault contracts of at least 99% of their underlying token. While the vaults were identical, the tokens stored in the vaults were different.

The vault contract was set up as a typical reward contract whereby you deposit some amount of tokens X in return for vault shares which can be redeemed for your tokens X again, along some rewards in several tokens Y. Besides the `deposit` and `withdraw` methods the contracts also had a `flashloan` method allowing you to borrow any token the vault holds.

There were also a couple of methods related to rewards but I quickly realized that those were just distractions as you have no way of setting the reward tokens and all the configured reward tokens were not the actual target tokens you're looking to drain

### Hint Finance - Initial ideas 🤔
Initially I looked at the flashloan method trying to find a way to deposit/withdraw during the loan but due to its design I had to conclude that this was not possible. This is because the flashloan method enforces not only that the tokens be returned but also that the amount of total outstanding vault shares remain constant. This means you can't keep new shares, have existing shares be redeemed or otherwise mess with the share price besides increasing it by returning more tokens than necessary. 

### Hint Finance - ERC777 does it again 😬
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

### Hint Finance - The Final Token, how????!!! 😡
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

I was right, not only is there an alternative function but it was already uploaded to 4byte (I could've been searching much longer if I had to check it manually) and it was also present on the SAND token contract: the `approveAndCall(...)` method! This is exactly what I needed, if you craft the call payload well the vault will simply approve some address you create to spend tokens on its behalf and the tokens will be mine 🤑.

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

The `flashloan` method calls the caller, not necessarily the token directly meaning we need the SAND token to be the caller somehow and give the vault a specific payload. Thankfully we can just leverage `approveAndCall` again, approving the vault and calling the `flashloan` method.
 
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

Next you'll notice that the approve amount that the SAND contract receives will be the address of the factory. Considering the SAND token has 18 decimals and an address is about ~1e30 when interpreted as a number that'll be more than enough to drain the contract in one go.

A further aspect is that `amount` can't be just anything. If it's `0` and points at an address it'll imply such a large byte-string that `approveAndCall` will just revert when it tries to copy it from calldata to memory. Instead we set it to `0xa0` so that when `approveAndCall` tries to get the data it'll find the `data`  provided by the vault.

Another error I got is `"first param != sender"` from the SAND contract, this is because the `approveAndCall` method enforces that the data has a minimum length and that the first argument is the caller's address. To ensure the correct length and address I set the return payload of the nested call to `approveAndCall` to be:
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

## 🚩 Just In Time (JIT)
Write-up coming soon!

 
<br>
<br>

Found a grammar, spelling or factual error or would like to suggest an improvement? Feel free to
raise an [issue](https://github.com/Philogy/philogy.github.io/issues) or reach
out to me on [twitter](https://twitter.com/real_philogy).

