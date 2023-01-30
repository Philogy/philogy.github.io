---
title: "Building Better Selector Switches"
categories: [smart-contract-development]
tags: [solidity, EVM, smart contracts, development]
math: true

---

## Intro
Every[^1] smart contract on Ethereum and other EVM chains starts with its selector switch. Selector
switches tell the contract whether they've implemented the requested method and where the code for
that method is actually found within the contract.

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

### Refresher: What Are Selectors?
Selectors are standardized 4-byte identifiers used to identify methods across contracts. Function
selectors are the left 4-bytes of the keccak hash of a function's _signature_. Since the data types
of the parameters are also hashed this allows contracts to differentiate between functions that have
the same name but different set of parameters like the `safeTransferFrom` method in [ERC721](https://eips.ethereum.org/EIPS/eip-721)
tokens. One thing to watch out for is selector collisions, these can occur because the selector
is only the first 4 bytes of the actual hash, they are quite rare however, occuring mainly in CTFs,
you can see an example of this in my write-up of the [Paradigm CTF 2022 "Hint Finance" challenge](http://localhost:4000/posts/paradigm-ctf-2022-write-up-collection/#hint-finance-).

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
selector:      2e1a7d4d
amount (1e18): 0000000000000000000000000000000000000000000000000de0b6b3a7640000
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

Since I'll be denoting opcode snippets in this post in Huff I'll do an extremely brief intro to Huff
here, you can find the full list of features here:
[docs.huff.sh/get-started/huff-by-example/](https://docs.huff.sh/get-started/huff-by-example/).

**What is Huff?:** Huff is a low-level assembly language for the EVM, it's basically just _mnemonic
bytecode_ (bytecode but written as aliases "SLOAD", "DUP1" rather than the byte values) with some
syntatic sugar around selectors, constants and jump destinations to improve the developer's quality
of life.

**Opcodes:**
Opcodes in Huff are written in their _mnemonic_ i.e. "word form" and can be upper / lower case. The
only exception are the `PUSH1` - `PUSH32` opcodes. For these the values can be written directly in
hexadecimal and the compiler will find the shortest fitting push opcodes for thme e.g. `0x3213`
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
2. Only executes logic if the selectors actually match.
3. Reverts for non-matching selectors
4. Jump to the section of bytecode that executes the logic for that function


### Linear If-Else Switch

![Linear If-Else Switch Flow Chart](/assets/images/23-01-better-switches/linear-switch.svg)

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

This approach makes for a decent selector switch, it can easily be generated for any contract
regardles of its selector set and count and fulfills our requirements of a selector:
1. It loads the selector from calldata
2. It does a direct comparison with all selectors until it finds one, won't jump unless there's
   a 1:1 match
3. Reverts if nothing matched
4. Jumps to a relevant bytecode section if a match was found

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


## Constant Gas Switches

### Lookup Switches

### Direct Switches

### Advanced Tips to Save even More Gas


## Footnotes
[^1]:
    Every contract written in high level languages that target the EVM like Solidity, Vyper, Fe
    have selector switches by default, however certain MEV bot contracts written in low level
    languages / assembly may omit an ABI compliant selector switch to save on even more gas.
