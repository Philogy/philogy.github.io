---
title: "Curta CTF ZSafe Write-up"
date: 2024-01-16 17:09
categories: [smart-contract-ctf]
tags: [solidity, EVM, smart contracts, security]
---

This is a write-up for the Curta CTF [ZSafe](https://www.curta.wtf/puzzle/base:5) challenge. An
actual solution wasn't submitted due to personal time constraints and [issues with foundry
scripts](https://github.com/foundry-rs/foundry/issues/6825) that made it impossible to submit using
it. That said I could've rewrote my solution to deploy via Python but I did not have any more time
to spend on this CTF unfortunately so I thought I'd just draft up the solution as a write-up here
and "submit it" by committing to it on Twitter.


## Part 1: Bypassing the upgrade whitelist.

To be able to manipulate how the proxy returns values for the `check` function you need to upgrade the underlying proxy to code you control. However the whitelist is _almost_ air-tight, it checks the codehash against contracts it knows (and are actually air-tight) so to upgrade it to something malicious you must take advantage of the self-destruct + create2 metamorphic contract pattern. This pattern lets you modify bytecode of contracts in place, allowing you to deploy your own contract that initially has a matching code hash of one of the accepted contracts but is then changed to a more malicious implementation.

## Part 2: Self-destructing the Clone

Now that you've deployed your own contract with the identical bytecode and upgraded to it you need to actually trigger a self-destruct, however, the code you've copied doesn't have one. You can still self-destruct, you'll just need to do so in another context. Specifically within a DELEGATECALL. Notice that the `upgradeToAndCall` function in SafeUpgradeable allows you to not only set the implementation but make an "initializing call" if:

1. The new implementation is whitelisted
2. You pass the `onlyProxy` check

The first is easy to do, you can simply call the `initialize` method. For the second let's look at the `_checkProxy` method underlying the modifier:

```solidity
if (address(this) == __self || ERC1967Utils.getImplementation() != __self) {
    revert("No hacc");
}
```

We need to pass two checks:

a) That the contract's address is **not** `__self`
b) The set proxy implementation **is** `__self`

What is `__self`?

```solidity
address private immutable __self = address(this);
```

Well `__self` is an immutably stored variable containing `address(this)`. Immutables are stored as part of the bytecode and set at initialization meaning you can't change them without affecting the code hash. When we copy the code for our imposter upgrade we copy the *original* address, meaning we implicitly pass the first check as our new contract's address is by definition not the previous one.

For the second check we can't directly set the implementation without doing an upgrade, this is a circular dependency. As the code stands we can't pass it. However notice that the code hash only encapsulates the *runtime bytecode*, not the deployment bytecode. Meaning that we just have to set the implementation in the deploy code for our imposter contract.

With all the hurdles passed we trigger an upgrade, delegate-calling to a contract that self-destructs on our behalf. Assuming we've set up the factory and create2 deployment correctly we can then re-deploy any code we'd like to the address that's now the implementation of the `SafeChallenge.proxy` contract.

## Part 3: Passing the check itself

To reach the desired unlocked state we need to provide inputs into `unlock` function such that the validation passes. The `unlock` function runs the `check` function 3 separate times on 3 pairs of inputs that we can provide. It also calls the proxy's p1 and p2 methods for extra information, data that we can now control with our implanted implementation.

We need all the inputs to result in 2 separate signatures (v, r, s) for 2 distinct messages. Assuming keccak isn't broken these are guaranteed to be different due to their differing salt (0xdead and 0xbeef respectively). Furthermore, the owner is only queried once and cannot change. The contract checks that *both* signatures were made by the same address and therefore the same key.

The neat thing about controlling the proxy however is that we can modify our owner(), p1() and p2() methods such that they return *different* values for every call of `check` despite being view functions. Also, note that p2() is called two separate times to derive the s value for the first and second signature meaning we can provide up to 6 different values via p2().

The second signature in `check` poses an issue however, `transform_s2` derives the s value of the second signature via the keccak hash function. Cryptographic hash functions are impossible to reverse (that we know of) so the second signature at least can't be one that was generated *from* a private key. However, we don't need it to be. Thanks to the math underyling ECDSA we can reverse the equation to compute the private key from a final signature, reversing the process.

The ECDSA signature generation algorithm is defined as follows (minus some checks):

1. Pick a random k in [1, n-1].
2. Compute the point K = k * G
3. Set the r parameter of the signature to the x coordinate of K
4. Compute s = k^-1 * (z - r * d_A) (where z is our message hash and d_A our private key)

We can retrieve the private key to generate a valid signature for a given s by reversing the last equation:

s = k^-1 * (z - r * d_A)

=> d_A = (s * k - z) * r^-1

(Note that the reason you can't normally do this is because k is private and unique, sharing or reusing k leaks your private key).

For our second signature, we can now simply pick a random k and p2, hash the p2 to get s, compute r and get the private key. Now we simply sign the first hash using the derived private key and voila we have two distinct signatures (v, r1, s1) and (v, r2, s2) with s2 being the result of a hash.

We now just compute p1 for each check such that r2 = r1 + p1 (unchecked, p1 = r2 - r1) and configure p2 to return s1 then s2. We can set the externally supplied s to 0 so that it does not affect the output of p2 when XOR-ed in `transform_s1` and `transform_s2`.

To make owner(), p1() and p2() stateful so that the return something different on every call, without violating the conditions the EVM enforces in the STATICCALL we can take advantage of the fact that SLOADs are technically stateful because of their gas accounting. The first time a unique slot is loaded it costs 2100 gas, the second time it only costs 100 gas. So to make a given function stateful we can just store our values in an array and return the first value that costs roughly more than 2100 gas to load:

```solidity
function p2() external view returns (uint256) {
    uint256 i = 0;
    unchecked {
        while (true) {
            uint256 preGas = gasleft();
            uint256 p = p2s[i];
            uint256 afterGas = gasleft();

            if (preGas - afterGas > 2100) return p;

            i++;
        }
    }
    revert();
}
```

Stitch it all together and voila, solved!

gg and thanks for reading
