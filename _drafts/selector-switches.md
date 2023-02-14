---
title: "Building Optimal Selector Switches"
categories: [smart-contract-development]
tags: [solidity, EVM, smart contracts, development]
math: true

---

![Switchboard operator](/assets/images/23-01-better-switches/switchboard-operator.jpg)

_Image: Telecom switchboard operator[^1]_

## Intro
Every[^2] smart contract on Ethereum and other EVM chains starts with its selector switch. Selector
switches tell the contract whether they've implemented the requested method and where the code for
that method is actually found within the contract. They're commonly reffered to as "function
dispatchers" although I like to use the term _selector switch_ as they act like `switch` statements
in common languages and also remind me of old-school telephone switchboard operators which would
manually route your telephone call in the ol' days.

Selector switches are executed upon every contract call meaning inefficiencies & optimizations in
this area of the code will impact all calls and the general gas efficiency of the contract. The most
straightforward approaches to selector switches employed by high-level languages Solidity and Vyper
and even taught in the [Huff tutorial on function dispatching](https://docs.huff.sh/tutorial/function-dispatching/#what-is-the-problem)
are not as optimal as they could be.

This post will look at the common approaches to selector switches and show more optimal versions
you can apply to your low-level contracts. This post also assumes some familiarity with the EVM and
its opcodes (low-level EVM instructions); if you need a quick intro I'd recommend [Huff's
"Understanding the EVM" Tutorial section](https://docs.huff.sh/tutorial/evm-basics/#technical) as it
gives a good overview of how the EVM works.

This post will also go into the methods and techniques I used to build my constant gas, 22-function,
55 gas selector switch.

### Refresher: What Are Selectors?
Selectors are standardized 4-byte identifiers used to identify methods across contracts. Function
selectors are the left 4-bytes of the keccak hash of a function's _signature_. Since the data types
of the parameters are also hashed this allows contracts to differentiate between functions that have
the same name but different set of parameters like the `safeTransferFrom` methods in [ERC721](https://eips.ethereum.org/EIPS/eip-721)
tokens. One thing to watch out for is selector collisions, these can occur because the selector
is only the first 4 bytes of the actual hash, they are quite rare however, occuring mainly in CTFs.
You can see an example of vulnerability arising from a selector collision in my write-up of the
[Paradigm CTF 2022 "Hint Finance" challenge](http://localhost:4000/posts/paradigm-ctf-2022-write-up-collection/#hint-finance-).

#### Example

**Function:**
```solidity
function withdraw(uint256 amount) external {
    // ...
}
```
**Signature:** `withdraw(uint256)`

**Selector:**

`keccak256("withdraw(uint256)")[0:4] =>`

**`0x2e1a7d4d`**`13322e7b96f9a57413e1525c250fb7a9021cf91d1540d5b69f16a49f =>`

`0x2e1a7d4d`

**Call to `withdraw(1e18)`:**

```
selector:      0x2e1a7d4d
amount (1e18): 0x0000000000000000000000000000000000000000000000000de0b6b3a7640000
----------------------------------------------------------------------------------------
calldata:      0x2e1a7d4d0000000000000000000000000000000000000000000000000de0b6b3a7640000
```

The precise specification can be found in the [Contract ABI Specification](https://docs.soliditylang.org/en/latest/abi-spec.html)
which also specifices how arguments are encoded and some more precise details when it comes to
actually computing the selector for different types.

> If you've installed the [foundry](https://book.getfoundry.sh/) smart contract development
> framework using `foundryup` you should have the `cast` tool installed which lets you easily get
> the selector of any method by running [`cast sig <your function>`](https://book.getfoundry.sh/reference/cast/cast-sig) e.g. `cast sig "withdraw(uint)"`
> results in `0x2e1a7d4d`
{: .prompt-tip}

Having a common standard for selectors is extremely useful as it lets different contracts easily
interact with each other without having to look up and implement some custom data and function
encoding scheme for every contract. It also allows higher level languages to offer some neat
abstraction.

### Micro Huff Crashcourse

Since I'll be denoting opcode snippets using Huff in this post I'll do an extremely brief intro to Huff
here, you can find the full list of features here:
[docs.huff.sh/get-started/huff-by-example/](https://docs.huff.sh/get-started/huff-by-example/).

**What is Huff?:** Huff is a low-level assembly language for the EVM, it's basically just _mnemonic
bytecode_ (bytecode but written as aliases "SLOAD", "DUP1" rather than the byte values) with some
syntatic sugar around selectors, constants and jump destinations to improve the developer's quality
of life.

**Opcodes:**
Opcodes in Huff are written in their _mnemonic_ i.e. "word form" and can be upper / lower case. The
only exception are the `PUSH1` - `PUSH32` opcodes. For these the values can be written directly in
hexadecimal and the compiler will find the shortest fitting push opcodes for thme e.g. `0x3212`
compiles to `PUSH2 0x3213` (`0x613213`). Huff also allows for constants which are defined by
`#define constant CONSTANT_NAME = 0x<constant value>` and are referenced by square brackets e.g.
`[PERMIT_TYPEHASH]` and are compiled to `PUSH` opcodes.

**Jump Labels & Destinations:**
Jump labels are convenient way to manage jumps without having to manually recalculate all
destinations everytime a bit of code is changed. Destinations are defined by `<name_of_label>:` e.g.
`withdraw_start:`, the compiler will insert the 1-byte `JUMPDEST` opcode at all labels, whether
they're referenced or not. Jump destinations can be pushed to the stack by just writing their label:
`withdraw_start`, Huff will currently always use a `PUSH2` opcode even if the destination only
requires 1 byte.

**Macros:**
Macros are kind of like functions in higher level languages. They're bits of code you can reuse
and are inserted inline by the compiler whenever referenced. They can also have static arguments
which are also inserted at compile time. Within the macro static arguments are referenced by angled
brackets and the name of the argument e.g. `<zero>`. Macro arguments can be static values, jump
labels or opcodes.

**Other features:**
Huff has a few other features like `fn` macros, tables, helper functions and even a way to define
tests but I won't be using most of them for this post outside of tables which I'll introduce once we
start using them. You can check out the extra features with above link to Huff's "Huff by example"
section.

> Because Huff is so essentially mnemonic opcodes with bells and whistles I'll also be using it to
> illustrate theoretical compiler output in a more human readable way. 
{: .prompt-info}


## Common Selector Switches

The goal of a selector switch is to:
1. Load the selector from calldata
2. Reverts for non-matching selectors
3. Jump to the section of bytecode that executes the logic for that function


### Linear If-Else Switch


"Linear If-Else Switches" are the most straightforward and common switch, generated by Solidity,
Vyper and even presented in Huff's tutorial. While the precise structure and opcodes used between
the languages differ an efficient Huff implementation and very similar to actual output from the
Solidity compiler could look something like this:

```
/*loads bytes 0-31 of calldata and shifts right by 28 bytes to get the 4 byte selector
(28 bytes = 28 * 8 bits = 0xe0 in hexadecimal)*/
// In Huff it's a common convention to denote the state of the stack in a comment on every line
0x0 calldataload 0xe0 shr // [call_selector] 

// 1st If-Else Block
// Copy selector from stack
dup1                      // [call_selector, call_selector]
// Place selector on stack
0x01020304                // [function_selector, call_selector, call_selector]
// Compare, are they eq-ual?
eq                        // [function_selector == call_selector, call_selector]
// Place function destination on stack incase they match
function1_dest            // [function_dest, function_selector == call_selector, call_selector]
// Conditional jump, will jump if the result from `eq` was true (1)
jumpi                     // [call_selector]
// If selectors didn't match `jumpi` will not jump and the code will continue here

// Next if-else blocks
dup1 0x05060708 eq function2_dest jumpi
dup1 0x11223344 eq function2_dest jumpi
...

// if no match just revert
0x0 0x0 revert
```
{: file="If-Else-Switch.huff"}

Shown as a diagram the control flow progresses step by step through every else-if "block" until
a matching selector is identified or the end of the switch is reached, in which case it results in
a revert:

![Linear If-Else Switch Flow Chart](/assets/images/23-01-better-switches/linear-switch.svg)

This approach makes for a decent selector switch, it can easily be generated for any contract
regardles of its selector set and count and fulfills our requirements of a selector:
1. It loads the selector from calldata
2. Reverts if nothing matched
3. It does a direct comparison with all selectors until it finds one, won't jump unless there's
   a 1:1 match

However this approach is far from efficient because the average gas use (assuming the functions are
randomly ordered in the switch and all used equaly) grows linearly with the amount of functions.
Assuming we're considering the gas use as run-time complexity this would have a time complexity of
$O(n)$ in classic CS big-O notation. This is fine and quite efficient for contracts with ~3-6
functions but quickly loses ground to alternative approaches as the amount of functions grows.

**Usage based optimization:**

Without changing the fundamental approach we can optimize the average gas use by reordering the
switch, putting more commonly used functions first. This will allow common calls to go through less
if-else blocks to find their match, improving average gas performance. The problem with this
optimization is that there's no real way to tell the compilers of high-level languages like Solidity
/ Vyper your selector-order preference and you need to anticipate the average use of your contract
which can be tricky.

### Binary Search Switch

The binary search based selector switch is much more efficient than the simpler linear else-if
switch for contracts with more functions. This is achieved by structuring the switch like a binary
search tree, eliminating half the possible selectors at every step. This makes the time complexity
logarithmic, $O(\log n)$: 

![Linear If-Else Switch Flow Chart](/assets/images/23-01-better-switches/binary-tree-switch-small.svg)

Intuitively we can see that the average gas cost of reaching any given method in the contract grows
logarithmically relative to the amount of selectors because we can double the amount of supported
selectors while just adding a constant size step on the path to every selector: 

![Linear If-Else Switch Flow Chart](/assets/images/23-01-better-switches/binary-tree-switch-big.svg)

_(click to expand)_

Instead of 2 x "Split" and 1x "If-Else" branches you need to traverse 3x "Split"
and 1x "If-Else" branch to reach any given function. If the amount of functions is not a power of
2 the tree approach will still work but the amount of branches required to reach a given function
will vary.

But how does it work exactly? How are you able to eliminate half the possible functions at every
step?

Like a binary search tree for integers you can create one for the selectors of the functions
and then compare them with some middle value at every node. This is possible because the EVM has no
notion of datatypes, it doesn't differentiate between strings, unsigned integers, signed integers,
booleans, selectors or jump labels. To the EVM opcodes all values on the stack are 256-bit words,
that can be added, subtracted and compared, against each other. To create our selector binary
search tree we first need to interpret the 4-byte selectors as integers and sort them:

```
0x06fdde03 // name()
0x095ea7b3 // approve(address,uint256)
0x18160ddd // totalSupply()
0x205c2878 // withdrawTo(address, uint256)
0x23b872dd // transferFrom(address,address,uint256)
0x2e1a7d4d // withdraw(uint256)
0x313ce567 // decimals()
0x3644e515 // DOMAIN_SEPARATOR()
0x4a4089cc // withdrawFromTo(address, address, uint256)
0x70a08231 // balanceOf(address)
0x7ecebe00 // nonces(address)
0x87f8ab26 // depositAmount(uint256)
0x9470b0bd // withdrawFrom(address, uint256)
0x95d89b41 // symbol()
0xa9059cbb // transfer(address, uint256)
0xac9650d8 // multicall(bytes[])
0xb2069e40 // depositAmountTo(address, uint256)
0xb760faf9 // depositTo(address)
0xd0e30db0 // deposit()
0xd505accf // permit(address, address, uint256, uint256, uint8, bytes32, bytes32)
0xdd62ed3e // allowance(address, address)
```
{: file='METH-WETH Selectors'}

You then continuously split the list of selectors in halves, saving the mid points as you go along,
so that you can construct the tree:

```
      0x06fdde03 // name()
      0x095ea7b3 // approve(address,uint256)
    <---- Split 1_1_1 0x10000000
        0x18160ddd // totalSupply()
        0x205c2878 // withdrawTo(address, uint256)
      <---- Split 1_1_1_2 0x23000000
        0x23b872dd // transferFrom(address, address, uint256)
        0x2e1a7d4d // withdraw(uint256)
  <---- Split 1_1 0x30000000
      0x313ce567 // decimals()
      0x3644e515 // DOMAIN_SEPARATOR()
    <---- Split 1_1_2 0x40000000
      0x4a4089cc // withdrawFromTo(address, address, uint256)
      0x70a08231 // balanceOf(address)
      0x7ecebe00 // nonces(address)
<---- Split 1 0x80000000
      0x87f8ab26 // depositAmount(uint256)
      0x9470b0bd // withdrawFrom(address, uint256)
    <---- Split 1_2_1 0x95000000
      0x95d89b41 // symbol()
      0xa9059cbb // transfer(address, uint256)
      0xac9650d8 // multicall(bytes[])
  <---- Split 1_2 0xb0000000
      0xb2069e40 // depositAmountTo(address, uint256)
      0xb760faf9 // depositTo(address)
    <---- Split 1_2_2 0xc0000000
      0xd0e30db0 // deposit()
      0xd505accf // permit(address, address, uint256, uint256, uint8, bytes32, bytes32)
      0xdd62ed3e // allowance(address, address)
```
{: file='METH-WETH Selector Splits'}

The code then compares the selector to the mid value to see if it should jump left or right at every
point:

> **Smol Optimization Tip**
>
> Depending on the context within the code you can save gas by replacing `0x00` (which compiles to
> `PUSH1 0x00` and costs 3 gas) with other opcodes that cost 2 gas and return `0x00`:
> - `PC` (program counter): Returns 0 if it's the very first byte of your contract
> - `RETURNDATASIZE`: Returns 0 until your contract has called another contract and the return
>   / revert data was at least 1 byte long at which point it'll return the length in bytes of that
>   returned data
> - `CALLVALUE`: Returns 0 if no ETH was sent along with the call, if you've already done a call value check
>   (`assert(msg.value == 0)`) you can rely on the `CALLVALUE` opcode to always return `0` until the
>   end of the call.
> - `MSIZE` (memory size): Returns 0 as long as you haven't used memory yet
{: .prompt-tip}

```
// -- Get 4 byte selector
pc calldataload 0xe0 shr     // [selector]

// -- Split 1
// Copy selector
dup1                         // [selector, selector]
// Push split value of split 1
0x80000000                   // [split_1_val, selector, selector]
// Compare selector to split value.
lt                           // [split_1_val < selector, selector]
// Prepare split 1_2 dest incase branching off.
split_dest_1_2               // [split_dest_1_2, split_1_val < selector, selector]
// Jump to split 1_2 if comparison suceeded
jumpi                       // [selector]

  // Split 1_1
  dup1 0x30000000 lt split_dest_1_1_2 jumpi
    // Split 1_1_1

    dup1 0x10000000 lt split_dest_1_1_1_2 jumpi
      // Split 1_1_1_1

      dup1 0x06fdde03 eq name_dest    jumpi
      dup1 0x095ea7b3 eq approve_dest jumpi
      returndatasize returndatasize revert

    split_dest_1_1_1_2:
      // Split 1_1_1_2

      dup1 0x23000000 lt split_dest_1_1_1_2_2 jumpi
        // Split 1_1_1_2_1

        dup1 0x18160ddd eq totalSupply_dest jumpi
        dup1 0x205c2878 eq withdrawTo_dest  jumpi
        returndatasize returndatasize revert

      split_dest_1_1_1_2_2:
        // Split 1_1_1_2_2

        dup1 0x23b872dd eq transferFrom_dest jumpi
        dup1 0x2e1a7d4d eq withdraw_dest     jumpi
        returndatasize returndatasize revert

  split_dest_1_1_2:
    // Split 1_1_2

    dup1 0x40000000 lt split_dest_1_1_2_2 jumpi
      // Split 1_1_2_1

      dup1 0x313ce567 eq decimals_dest         jumpi
      dup1 0x3644e515 eq DOMAIN_SEPARATOR_dest jumpi
      returndatasize returndatasize revert

    split_dest_1_1_2_2:
      // Split 1_1_2_2

      dup1 0x4a4089cc eq withdrawFromTo_dest jumpi
      dup1 0x70a08231 eq balanceOf_dest      jumpi
      dup1 0x7ecebe00 eq nonces_dest         jumpi
      returndatasize returndatasize revert

split_dest_1_2:
  // Split 1_2
  dup1 0xb0000000 lt split_dest_1_2_2 jumpi
    // Split 1_2_1

    dup1 0x95000000 lt split_dest_1_2_1_2 jumpi
      // Split 1_2_1_1
      dup1 0x87f8ab26 eq depositAmount_dest jumpi
      dup1 0x9470b0bd eq withdrawFrom_dest  jumpi
      returndatasize returndatasize revert

    split_dest_1_2_1_2;
      // Split 1_2_1_2
      dup1 0x95d89b41 eq symbol_dest    jumpi
      dup1 0xa9059cbb eq transfer_dest  jumpi
      dup1 0xac9650d8 eq multicall_dest jumpi
      returndatasize returndatasize revert

  split_dest_1_2_2:
    // Split 1_2_2

    dup1 0xc0000000 lt split_dest_1_2_2_2 jumpi
      // Split 1_2_2_1
      dup1 0xb2069e40 eq depositAmountTo_dest jumpi
      dup1 0xb760faf9 eq depositTo_dest       jumpi
      returndatasize returndatasize revert

    split_dest_1_2_2_2;
      // Split 1_2_2_2
      dup1 0xd0e30db0 eq deposit_dest   jumpi
      dup1 0xd505accf eq permit_dest    jumpi
      dup1 0xdd62ed3e eq allowance_dest jumpi
      returndatasize returndatasize revert
```
{: file="Binary Tree Switch.huff"}

Unlike the diagram you'll notice that I didn't create split branches all the way down, but instead
put small linear switches at the end once only 2-3 functions were left in the split. This is because
a split branch can only cut the range of selectors in half but cannot confirm whether the selector
matches exactly, you need a directly comparing "if-else" branch at the end of your tree. If you
split your selectors all the way until each split only contains 1 selector the last split is redundant:

```
// Split branch cost:   22 (+1 for jump dest on the `_2` branch)
// If-else branch cost: 22

// Split until only 1 Selector left

split_dest_x:
  dup1 0xd2000000 lt split_dest_x_2 jumpi
      // Split x_2
    dup1 0xd0e30db0 eq deposit_dest jumpi (y + 44 gas to reach)
  split_dest_x_2:
    // Split x_2
    dup1 0xd505accf eq permit_dest  jumpi (y + 45 gas to reach, includes 1 gas for `JUMPDEST`)

// Linear if-else for 2 functions:

split_dest_x:
  dup1 0xd0e30db0 eq deposit_dest jumpi (y + 22 gas to reach)
  dup1 0xd505accf eq permit_dest  jumpi (y + 44 gas to reach)

```

Similar reasoning applies for a split with 3 selectors. Further splitting a split with 3 selectors
left turns the cost of (22, 44, 66) => (45, 66, 67). In fact if you take the average gas use of the
different selectors it doesn't even make sense to break up a split containing 4 selectors: (22, 44,
66, 88; avg: 55) => (44, 66, 45, 67; avg: 55,5). Meaning that my example is not even a perfectly optimal
binary search switch.

The Solidity compiler will automatically create such binary search switches once you have sufficient
functions to justify the cost. Unfortunately this is where we meet the limits of existing high-level
compilers in terms of selector switch generation, to squeeze more gas out of our contracts we must
dive deeper, into the dark depth of opcode level optimization, but don't worry it's less intimidating
than it sounds.

## ⚡Constant Gas Switches

Considering that the generated binary search switches are already optimal in terms of opcode use
we need a completely different approach if we're going to improve the gas cost of existing selector
switches. To do that we'll target the theoretically optimal time complexity of $O(1)$ meaning the
cost of our selector switch doesn't grow at all relative to the amount of selectors.

While $O(1)$ is theoretically optimal we'll also have to make sure the constant amount of gas used is actually
cheaper than an alternative binary search or linear if-else switch. This is important because a naive
$O(1)$ switch would e.g. be to store all jump destinations of the different functions in storage and
load them when the contract gets executed, while this would use a constant amount of gas regardless
of how many functions you have because it only needs a single `SLOAD`, the cost of that operation
(2100) would make it more expensive than nearly any alternative.

### 🧱 Base Cost

Whenever I work on gas optimization I like to determine or at least approximate how efficient
a solution could theoretically be. For gas cost this is usually down to what you need the code to do
and what you're working with. In general when building a selector switch there's two pieces that cannot
be avoided:
1. Isolating the selector from calldata: `<zero push> calldataload` + \[`PUSH1 0xe0 SHR` / `PUSH32 0xffffffff00000000000000000000000000000000000000000000000000000000 AND`\] (11 gas, assuming zero-push is a 2 gas opcode such as `PC` or `RETURNDATASIZE`)
2. Checking if the selector is an exact match: `PUSH4 <expected_selector>` \[`eq` / `sub`\] `PUSH1/2 <code / revert dest> JUMPI` (19 gas)

So we can confidently say that no ABI compliant selector switch, that reverts upon unmatching
selectors can cost less than 30 gas. In fact exactly 30 gas is possible when your contract has exactly
1 function and selector: 

```
// Load selector
pc calldataload 0xe0 shr

// Use subtraction opcode `SUB` as "not equals" comparison. If selectors don't match
// subtraction result will be non-zero and cause `JUMPI` to jump to the revert
<single_selector> sub no_match_error jumpi

// Huff macro that will insert the functions code
SINGLE_FUNCTION()

no_match_error:
  returndatasize returndatasize revert // pushes 2 zeros and then reverts

```
{: file="single-function-switch.huff"}

### Lookup Tables

To convert the selector into a jump label in constant gas we need to construct some type of array
/ map data structure that stores the lookup table as those typically have constant $O(\log n)$ lookup times.


#### Extracting Indices

Considering that jump labels are 2-bytes large and selectors are 4-byte (32-bit) large values that would
require an 8.5GB array or map (`2 ** 32 * 2`) so that every single selector can be mapped to a
unique jump label. To reduce the size of our lookup table and make it fit into the [24kB Spurious Dragon
contract size limit](https://eips.ethereum.org/EIPS/eip-170) we'll need to extract an index that
sits in a smaller range, however this means that different selectors will result in the same index.
This why the final, 19-gas direct selector comparison is necessary as we need to check for collision
to make sure that when our contract is called with a selector that's not part of the ABI it'll
actually revert:

![Lookup switch](/assets/images/23-01-better-switches/lookup-switch.png)

There are different ways to extract a unique index from the 4-byte selector:


##### Hashing

Hashing it the most general purpose way to map a set of selectors to a unique set of indices in
a tight range. One just needs to search for a nonce that when hashed together with the selector
results in a unique index. While it may require some longer searching for larger selector sets the
search time can be reduced by increasing the accepted output range for indices. Code for such an
index extractor would look something like this:

```
// Gets selector.
pc calldataload 0xe0 shr  // [selector]

// Combine nonce with selector, `nonce` should be a number left shifted by 32-bits.
dup1 [NONCE] or           // [hash_preimage, selector]
// Store in memory for hasing.
msize mstore              // [selector]
msize returndatasize sha3 // [hash, selector]
[MASK] and                // [index, selector]

```
{:file='Hash Selector Indexer.huff'}

While this is approach is quite general purpose it's not very efficient. With the required
memory and `SHA3` opcode use it brings the base cost of this approach to 93 gas, not including the code
to actually do the table lookup. Meaning it'll only really pay off when your contract start to have
\>25 (or >40) functions, depending on the type of lookup table you're using.

That's why as an alternative to hashing it can be cheaper to make a custom "hashing function" that
condenses the selector to an index by combining bitwise-xor, bit-shifting and modulus operations in
a more creative fashion. Either approach will require you to iterate through many combinations to
find the most efficient setup.

However if you're lucky and your selector set is smaller you can also use one of the following, more
efficient selector indexing approaches:

- code lookup table (37): 3 (push size) + 3 (push code offset) + 3 (add index code offset) + 3 (push mem offset) + 6 (codecopy) + 2 (push zero) + 3 (mload) + 3 (push mask2) + 3 (apply mask, and) + 8 (jump) = 37
- push table (20): 3 (push table) + 3 (shift using index) + 3 (push table mask) + 3 (apply table mask) + 8 (jump)

##### Direct Bit-Masking

Looking at the selectors of the functions in your ABI as 32-bit binary values you can look to
identify a set of bit positions across a tight range that are unique to all selectors e.g.:

```
               xx   xxx
[0x06fdde03] 00000110111111011101111000000011 - {00___011} name()
[0x95d89b41] 10010101110110001001101101000001 - {01___111} symbol()
[0x313ce567] 00110001001111001110010101100111 - {11___100} decimals()
[0x18160ddd] 00011000000101100000110111011101 - {01___000} totalSupply()
[0x23b872dd] 00100011101110000111001011011101 - {10___110} transferFrom(address,address,uint256)
[0xa9059cbb] 10101001000001011001110010111011 - {10___100} transfer(address, uint256)
[0x70a08231] 01110000101000001000001000110001 - {11___010} balanceOf(address)
[0x095ea7b3] 00001001010111101010011110110011 - {00___101} approve(address,uint256)
[0xdd62ed3e] 11011101011000101110110100111110 - {01___101} allowance(address, address)
[0xd0e30db0] 11010000111000110000110110110000 - {01___011} deposit()
[0xb760faf9] 10110111011000001111101011111001 - {11___101} depositTo(address)
[0x2e1a7d4d] 00101110000110100111110101001101 - {10___000} withdraw(uint256)
[0x853828b6] 10000101001110000010100010110110 - {00___100} withdrawAll()
[0xca9add8f] 11001010100110101101110110001111 - {00___010} withdrawAllTo(address)
[0x205c2878] 00100000010111000010100001111000 - {10___001} withdrawTo(address, uint256)
[0x9470b0bd] 10010100011100001011000010111101 - {01___001} withdrawFrom(address, uint256)
[0x4a4089cc] 01001010010000001000100111001100 - {00___001} withdrawFromTo(address, address, uint256)
[0x3644e515] 00110110010001001110010100010101 - {11___001} DOMAIN_SEPARATOR()
[0x7ecebe00] 01111110110011101011111000000000 - {11___011} nonces(address)
[0xd505accf] 11010101000001011010110011001111 - {01___100} permit(address, address, uint256, uint256, uint8, bytes32, bytes32)
[0xac9650d8] 10101100100101100101000011011000 - {10___010} multicall(bytes[])

```

In the above example there's a set of 5-bits in an 8-bit range that are unique across all selectors.
The tight range is ideal as it means the range of values between the lowest and highest index is not
so wide and the table in our contract can therefore be smaller. Extracting the index from a selector
would require the operator `(selector >> MAGIC_SHIFT) & MAGIC_MASK` which for the above example
would be `(selector >> 22) & 0c7`. In Huff:

```
// Gets selector.
pc calldataload 0xe0 shr // [selector]

// Selector copied on stack because it would be needed for direct comparison later
dup1                     // [selector; selector]
0x16 shr 0xc7 and        // [index ∈ (0x00,0xff); selector ]

```
{:file='Index Extractor.huff'}

##### Bit Splicing
While there will always be a set of bit indices that are unique across all selectors they may not
always be across a tight range, in fact it becomes increasingly less likely as the amount of
functions in your contract increases. This presents an issue as the resulting index needs to be in
a tight range for the table size to be practical. However there are a few tricks you can use to
relatively efficient bring bits together that aren't close to each other.

As an example let's Imagine you have an ABI such that the a set of unique bits across all selectors
looks as follows (marked by `X`):

`0 0 0 0 0 0 0 0 X X X X 0 0 0 0 0 0 0 0 0 0 0 X 0 X 0 0 0 0 0 0`

Extracted with the code: `SELECTOR() 0x6 shr 0x3c005 and`, representing a range of 18-bits. However
we can add a few simple operations to close the gap between the upper bits and the lower bits. By
shifting and then bitwise OR-ing the values together we can create a resulting index that will
always be in a smaller range:

```
Initially extracted: A B C D 0 0 0 0 0 0 0 0 0 0 0 E 0 F
Shifted right by 11: 0 0 0 0 0 0 0 0 0 0 0 A B C D 0 0 0
--------------------------------------------------------
OR-ed together     : A B C D 0 0 0 0 0 0 0 A B C D E 0 F
Lower bits masked  : 0 0 0 0 0 0 0 0 0 0 0 A B C D E 0 F
```

With just 3 operations (shift, or, and) we reduced the range of the resulting index while
keeping the unique bits, in Huff this would look like this (0xb is 11 in hexadecimal):

```
// Gets selector.
pc calldataload 0xe0 shr // [selector]
dup1 0x6 shr 0x3c005 and // [wide_index, selector]

dup1 0xb shr             // [shifted_top_bits, wide_index, selector]
or 0x7f and              // [narrow_index, selector]
```

This is quite efficient as it doesn't use a lot of opcodes and they're all base-2 operations
(bit-wise OR, bit-shift, bit-wise AND) which cost 3 gas vs. other arithmetic operations like `mod`,
`div` and `mul` which could also be used but cost 5 gas each.

#### 🏗️ Building A Lookup Table in the EVM
Similar to how there are several approaches with varying efficiencies when building a selector
indexer that are also several approaches to building and retrieving values from lookup tables.

##### Push Tables
With these tables you can pack up to 16x 2-byte jump destinations into a single value and then have
the table be pushed in its entirety onto the stack using a `PUSH` opcode. This is quite efficient as
"loading" the table itself only requires 3 gas and retrieving a value 12 gas, assuming your index is
already multiplied by 16 to ensure it's the right shift. The big constraint with this approach is
that it only supports indices that are in the range 0-15 which will be an issue if your contract has
more than 16 functions that need to be jumped to. Assuming you're able to find a group of 4-bits
that is unique across all selectors the switch would look as follows:

```
// Gets selector.
pc calldataload 0xe0 shr // [selector]
[JUMP_TABLE]             // [table]
dup2 [IND_SHIFT] shr     // [shifted_selector, table, selector]
// Mask offset by 4 bits to ensure result will be a valid table shift.
0xf0 and                 // [index, table, selector]
shr 0xffff and           // [jump_dest, selector]
jump

// At the jump destinations, check if selector matches:
0x???????? sub error_dest jumpi
FUNCTION_LOGIC()

error_dest:
  0x0 0x0 revert
```
{:file='16 Selector Switch.huff'}

You can fit more than 16-destinations into a push table if the necessary information for each jump
dest is less than 16-bits. For example if your functions are small enough such that you can
practically pad their size in the bytecode to a power of 2 ($2^n$) then you'd only need $16- n$ unique
bits to identify the jump destination of your functions, meaning each label takes up less space
meaning more will fit in a 256-bit `PUSH32` opcode. Note that making such changes will not only
require you to pad your functions but also adjust the above code, adding the required offset after
the table lookup and potentially using different operations to determine the table shift.


##### Table Trees
Push tables can be extended to support more than 16-functions by creating a tree type structure,
whereby the first table selects a sub-table that determines the final destination. While this is
technically still a tree with logarithmic $O(\log n)$ time complexity it's still shown here as it
can be very efficient for contracts with larger selector sets and is practically constant size as
most contracts will at most need a tree with depth 2, or in extreme cases (>256 functions), depth 3.


#### 🏃 Direct Jumping

## Advanc A Practical 67-Gas 21-Function Selector Switch


## Footnotes
[^1]: Joseph A. Carr, Attribution, via Wikimedia Commons
[^2]:
    Every contract written in high level languages that target the EVM like Solidity, Vyper, Fe have selector switches by default, however certain MEV bot contracts written in low level languages / assembly may omit an ABI compliant selector switch to save on even more gas.
