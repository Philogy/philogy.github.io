---
title: "Curta CTF ZSafe Write-up"
date: 2024-01-16 17:09
categories: [smart-contract-ctf]
tags: [solidity, EVM, smart contracts, security]
math: true
---

This is a write-up for the Curta CTF [ZSafe](https://www.curta.wtf/puzzle/base:5) challenge. I
initially didn't submit a solution due to personal time constraints and [issues with foundry
scripts](https://github.com/foundry-rs/foundry/issues/6825) that made it impossible to submit
directly using them. Instead I committed to the solve on [Twitter](https://x.com/real_philogy/status/1747381887679484166).

You can see my full solution in my repo [here](https://github.com/Philogy/curta-zsafe-solution).

## Intro

![ZSafe Challenge Structure](/assets/images/zSafe-challenge-structure.svg)

The end goal is to have the `SafeCurta` puzzle contract's `verify` function evaluate `true` for your seed
which is determined by `generate`. To do so it checks that your associated `SafeChallenge` factory
contract returns `isUnlocked`. This means our setup is:

1. Deploy the `SafeChallenge` contract
2. Find a way to pass the checks in its `unlock` function to achieved the `isUnlocked: true` state.

To unlock the `SafeChallenge` contract (referred to as the lock from here on out) we must pass the
`unlock` method 3 `r` values and 3 `s` values that when combined with the values of the `proxy`
contract gives us 6 valid signatures for the set `owner`:

```solidity
function unlock(bytes32[3] calldata r, bytes32[3] calldata s) external {
    for (uint256 i = 0; i < 2; ++i) {
        require(uint256(r[i]) < uint256(r[i + 1]));
    }

    for (uint256 i = 0; i < 3; ++i) {
        check(r[i], s[i]);
    }

    isUnlocked = true;
}

function check(bytes32 _r, bytes32 _s) internal {
    uint8 v = 27;

    address owner = proxy.owner();

    bytes32 message1_hash = keccak256(abi.encodePacked(seed, address(0xdead)));
    bytes32 r1 = transform_r1(_r);
    bytes32 s1 = transform_s1(_s);
    address signer = ecrecover(message1_hash, v, r1, s1);

    require(signer != address(0), "no sig match :<");
    require(signer == owner, "no owner match :<");

    bytes32 message2_hash = keccak256(abi.encodePacked(seed, address(0xbeef)));
    bytes32 r2 = transform_r2(_r);
    bytes32 s2 = transform_s2(_s);
    address signer2 = ecrecover(message2_hash, v, r2, s2);

    require(signer2 != address(0), "no sig match :<");
    require(signer2 == owner, "no owner match :<");
}
```

We quickly realize that this is impossible. As defined in the whitelisted `SafeSecret` and `SafeSecretAdmin`
contracts, they do not give us enough freedom with the return values to be able to create valid
signatures. This is mainly because of the lock's `transform_s2` method which derives the $s$ value
for the second signature of every `check`:

```solidity
function transform_s2(bytes32 s) internal view returns (bytes32) {
    return keccak256(abi.encodePacked(uint256(s) ^ proxy.p2(), seed));
}
```

The fact that it derives the second $s$ value from a hash means that we cannot reverse some useful final
$s$ value to see what `p2()` value `SafeSecretAdmin` would need to return. Our only path is to
choose some random `p2()` see what hash it results in and try to reverse the other values, but this
is mathematically impossible without breaking the keccak hash function or dlog (fundamental
cryptographic hardness assumption related to ECDSA).

So our only path forward is to somehow manipulate the `proxy` with a custom implementation such that
`p2()` returns 2 separate values per `check`.


## Part 1: Bypassing the upgrade whitelist.

To be able to manipulate how the proxy returns values for the `check` function you need to upgrade
the proxy to code we control. However the whitelist only lets us change the implementatino to
a contract matching the code hash of the `SafeSecret` or `SafeSecretAdmin` contracts. However once
the address is set the code is never revalidated, meaning if we are somehow able to change the code
of a deployed contract in-place we can just upgrade the `proxy` to some "valid" code, change it and
voila we have our malicious proxy implementation.

This can be achieved with the metamorphic contract pattern. Summarized what it does is deploy
a contract using create2 such that the address of the contract is independent of its final runtime
bytecode, instead it retrieves/builds its final bytecode in the constructor by using external
information. By combining this with `SELFDESTRUCT` we can wipe the code and then redeploy different
code _to the same address_:

```solidity
contract ReinitFactory is IReinitFactory {
    bool private _copy;
    address private _target;

    function fakeDeploy(address target, address victim) external returns (address upgrade) {
        _copy = true;
        _target = target;
        upgrade = address(new FakeUpgrade{salt: bytes32(0)}());
    }

    function deployActual(address real, address victim) external {
        _copy = false;
        _target = real;
        new FakeUpgrade{salt: bytes32(0)}();
    }

    function mode() external view override returns (bool, address) {
        return (_copy, _target);
    }
}
```
{: file="ReinitFactory.sol"}

```solidity
contract FakeUpgrade {
    error SetFailed();

    constructor() {
        (bool copy, address target) = IReinitFactory(msg.sender).mode();

        if (copy) {
            assembly {
                let size := extcodesize(target)
                extcodecopy(target, 0, 0, size)
                return(0, size)
            }
        } else {
            // Create minimal proxy bytecode.
            bytes memory code =
                abi.encodePacked(hex"3d3d3d3d363d3d37363d73", address(target), hex"5af43d3d93803e602a57fd5bf3");

            assembly {
                return(add(code, 0x20), mload(code))
            }
        }
    }
}
```
{: file="FakeUpgrade.sol"}

So we can now use these contracts to deploy a contract that initially has a matching codehash to one
on the whitelist. Note that for the branch where we insert our actual bytecode I only deploy
a minimal proxy pointing to the actual code. While this adds some indirection it saves me a lot of
gas when deploying the solution as I only have to deploy my final solution bytecode once instead of
twice.

## Part 2: Self-destructing the Impostor

Now that we've deployed our own contract with the identical bytecode and upgraded to it we need
to actually trigger a self-destruct, however, the code we've copied doesn't have one. We can still
self-destruct, we'll just need to do so in another context. Specifically within a DELEGATECALL.
Notice that the `upgradeToAndCall` function in `SafeUpgradeable` allows us to not only set the
implementation but make an "initializing call" if:

1. The new implementation is whitelisted
2. You pass the `onlyProxy` check

(code slightly simplified from actual CTF)

```solidity
function upgradeToAndCall(address newImplementation, bytes memory data) public payable virtual onlyProxy {
    _authorizeUpgrade(newImplementation);
    ERC1967Utils.upgradeToAndCall(newImplementation, data);
}

function _authorizeUpgrade(address newImplementation) internal {
    require(whitelist[newImplementation.codehash], "wtf no whitelisted no hacc pls");
}
```
{: file="SafeUpgradeable.sol"}

The first is easy to do, to be allowed to DELEGATECALL into an arbitrary contract we can simply
whitelist it by calling the `initialize` method. For the second let's look at the `_checkProxy`
method underlying the modifier:

```solidity
function _checkProxy() internal view virtual {
    if (address(this) == __self || ERC1967Utils.getImplementation() != __self) {
        revert("No hacc");
    }
}
```

We need to pass two checks:

a) That the contract's address is **not** `__self`
b) The set proxy implementation **is** `__self`

Well, what is `__self`?

```solidity
address private immutable __self = address(this);
```

Well `__self` is an immutably stored variable containing `address(this)`. Immutables are stored as
part of the bytecode and set at initialization meaning you can't change them without affecting the
code hash. When we copy the code for our imposter upgrade we copy the *original* address, meaning
we implicitly pass the first check as our new contract's address is by definition not the previous
one.

For the second check we can't directly set the implementation without doing an upgrade, this is a
circular dependency. As the code stands we can't pass it. However notice that the code hash only
encapsulates the *runtime bytecode*, not the deployment bytecode. Meaning that we just have to set
the implementation in the deploy code for our imposter contract.

With all the hurdles passed we trigger an upgrade, `DELEGATECALL`-ing to a contract that
self-destructs on our behalf. Assuming we've set up the factory and create2 deployment correctly
we can then re-deploy any code we'd like to the address that's now the implementation of the
`SafeChallenge.proxy` contract:

![Visualization of Malicious Upgrade Procedure](/assets/images/zSafe-malicious-upgrade.excalidraw.svg)

## Part 3: Passing the check itself

Now that we're able to actually fully control the target bytecode we need to actually create
signature values that'll pass the 3 calls to `check` in the lock (`SafeChallenge`) contract.

To reach the desired unlocked state we need to provide inputs into the `unlock` function such that
the validation passes. The `unlock` function runs the `check` function 3 separate times on 3 pairs
of inputs that we can provide. It also calls the proxy's p1 and p2 methods for extra information,
data that we can now control with our implanted implementation.

We need all the inputs to result in 2 separate signatures $(v, r, s)$ for 2 distinct messages.
Assuming keccak isn't broken these are guaranteed to be different due to their differing salt
(`0xdead` and `0xbeef` respectively). Furthermore, the owner is only queried once and cannot change.
The contract checks that *both* signatures were made by the same address and therefore the same
key.

The neat thing about controlling the proxy however is that we can modify our `owner()`, `p1()` and
`p2()` methods such that they return *different* values for every call of `check` despite being view
functions. The `p2()` method is also called two separate times to derive the s value for the first
and second signature meaning we can provide up to 6 different values via `p2()`.

As initially introduced, the second signature in `check` poses an issue, `transform_s2` derives
the $s$ value of the second signature via the keccak hash function. Cryptographic hash functions
are impossible to reverse (that we know of) so the second signature at least can't be one that
was generated *from* a private key. However, we don't need it to be. Thanks to the math
underyling ECDSA we can reverse the equation to compute the private key from a final signature,
reversing the process.

The ECDSA signature generation algorithm is defined as follows (minus some checks):

1. Pick a random $k \in [1, n-1]$.
2. Compute the coordinates $(x, y)$ of the point $K = k\cdot G$
3. Set the $r = x$
4. Compute $s = k^{-1} \cdot (z - r \cdot d_A)$ (where $z$ is our message hash and $d_A$ our private key)

We can retrieve the private key to generate a valid signature for a given s by reversing the last equation:

$$s = k^{-1} \cdot (z - r \cdot d_A)$$

$$ d_A = (s \cdot k - z) \cdot r^{-1}$$

> The reason you can't normally reverse an ECDSA signature to retrieve the original private key is because
> the underlying $k$ parameter is not meant to be known. If you have a cryptographic library that
> somehow leaks or reuses $k$ it allows you to extract the private keys that created signatures
> using it. In our situation since we control the process, we can choose and therefore know $k$.
{: .prompt-info}

For our second signature, we can simply pick a random $k$ and `p2()` (denoted as $p_{2,2}$), hash
$p_{2,2}$ to get $s_2$, compute $r$ from $k$ and get the private key using the above derivation.
Now we simply sign the first hash using the derived private key and voila we have two distinct
signatures $(v, r_1, s_1)$ and $(v, r_2, s_2)$ with $s_2$ being the result of a hash.

We now just compute $p_1$ for each check such that $r_2 = r_1 + p_1$ ($p_1 = r_2 - r_1 \mod 2^{256}$) and
configure `p2()` to return $s_1$ then the original $p_{2,2}$ we used to compute $s_2$. We can set the
externally supplied `s` values to 0 so that it does not affect the output of `p2()` when XOR-ed in
`transform_s1` and `transform_s2`.

One last final issue is that the lock contract sees the proxies `p1()`, `p2()` and `owner()` methods
as view-functions, meaning solc will compile the contract down to use `STATICCALL` to call the
methods. This means we cannot modify in our methods. Without having access between the calls (which
we do not), how do we make these functions stateful such that they "know" how often they were called
so that they can return different values each time?

One way would be to set and check for a hardcoded gas remaining value, as gas will be used
throughout the call and different amounts will be remaining. However trying to plan for and deal
with specific gas uses is very finicky and I would not recommend such an approach.

Instead we can make these view methods stateful by taking advantage of the EVM's storage cold/warm gas
accounting. When a storage value is first read as part of a transaction it incurs a larger "warming"
cost of 2100 to account for the node's cost of fetching the value from its DB on disk, every
subsequent time the value is `SLOAD`ed however, it only incurs a cost of 100 gas as it's already
"warm" and cached locally for the duration of the transaction. This means we can use the gas cost of
loading variables to determine whether they've been read already, giving us a stateful but
`STATICCALL`-compatible way to return different values:


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

Voila! Solved. We then simply stitch together the deployment and submission of our different
transactions, test against a local fork before deploying on-chain. (Or in my case, accidentally
deploy directly to mainnet, test locally before finally redeploying the fixed solution to mainnet).

## Conclusion

Overall this was a very interesting challenge by [Zellic's jazzy](https://twitter.com/ret2jazzy),
taking advantage of some interesting math behind ECDSA and some false assumptions people commonly
make about the immutability of code. Highly recommend you try it out, feel free to check out the
code to my solution [on github](https://github.com/Philogy/curta-zsafe-solution) as well. It
contains not only the challenge and solution contracts but the python script I used to actually run
the above math and generate the different values.

Feel free provide feedback on this post on [Twitter (X)](https://twitter.com/real_philogy) or
simply follow for more smart contract development and CTF content in the future! 😁
