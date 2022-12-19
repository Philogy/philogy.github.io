---
title: "Beancount: Easy DeFi Accounting For Noobs (Part 1)"
categories: [accounting]
tags: [DeFi, accounting, beancount]

---


> **Disclaimer**
>
> I'm neither a professional accountant, tax advisor or lawyer. The content of this post is meant to
> be educational and should not be considered as professional advice of any kind. It is recommended
> you consult a professional before applying any tax strategies discussed herein.
{: .prompt-warning}

## Intro
This blog post will give you a practical introduction to accounting in the context of DeFi.

I've created this post to share some of my learnings as an amateur, self-taught bookkeeper and
expand the community of accounting interested DeFi users. Hopefully I can help some of you with your
daily crypto / DeFi trading activities and maybe foster a small community of accounting interested
nerds like me. 😄

> **A Note to Real Accountants**
>
> I'll be simplifying if not straight up misusing certain tax and accounting terminology for the
> sake of simplicity and accessibility, corrections and extended explanations may be found at the
> very bottom of the post under "asdlfsaldfkjkl (TODO)". If I've made an uncorrected mistake please
> let me know so I can improve the post, thx!
{:.prompt-info}


## Reasons To Learn Basic Accounting

Before diving in to the actual content I wanted to lay out some reasons why you might want to learn
the basics of accounting if you're not already motivated / interested. If you already know why you
want to learn feel free to skip to the next [section]().

- Tax Compliance 🥸: If you want to easily and accurately determine your income you need
  a consistent system
- Tax Avoidance 😎: If you want to save on taxes you need to understand your tax laws and how you
  can legally leverage them to your benefit
- Insight & Understanding 📊: Accounting allows you to notice and quantify trends in your finances
  as well as understand how you're spending and earning it, if you want to be able to optimize
  something you have to be able to measure it first 

### The Downsides Of An External Accountant
While you could hire an accountant to do everything for you, unless you have a pretty sizable
networth it will hardly be worth the cost. Most accountants are not familiar with crypto let alone
obscure DeFi protocols. If your activity consists of more than just basic spot trades you'll likely
have to spend time, money and effort explaining to a normal accountant how the different DeFi
protocols, farms, NFTs and airdrops work.

![Confused math lady meme with the caption "Your accountant trying to understand how you made y-figures off food coins"](/assets/images/confused-accountant-meme.jpg)

Furthermore not having direct control of your "books" (accounting lingo for where all your
transactions, inventory etc. are kept track of) limits your ability to optimize your taxes, making
you completely reliant on advice from your accountant or other advisors.
 
## 💼 Crypto Tax Software & When To Get Help From Experts
While this whole post is about accounting basics and how to easily keep track of ones' books there
are situations where you can and probably **should consult** with an expert. When it comes to filing
tax statements for your yearly / quarterly reporting or better understanding the tax laws and how
they apply to your specific situation you should definitely consult professionals.

In fact if you're just doing basic trades on exchanges and the popular DeFi protocols you should
probably just use one of the many existing crypto tax softwares. They'll save you a lot of effort
by automating the importing and classification of transactions.

My personal approach is to do the accounting myself while consulting a tax advisor on the general
laws and how they might apply to my situation. When it comes time to report my taxes I also compile
a bunch of summaries from my accounting and hand it over to the tax advisor for them to do the
reports.

## 🫘 Beancount

Beancount is an open-source, easy-to-use text based accounting software. Unlike commercial
accounting programs it's base feature set is relatively limited. I find this to be very attractive
as it makes it easy to use without getting distracted by overviews, charts and features I don't
need.

While beancount is simple at its core you _can_ choose to make it as sophisticated you want by
installing and/or writing your own plugins or by simply leveraging the command-line tools that come
along beancount. Beancount has a very nice API making it easy to extend with Python.

Furthermore as part of its core feature set it has some very nice handling of "inventory" or as it
prefers to call it in its [documentation](https://beancount.github.io/docs/) commodities. While not
specifically made for crypto it's very well suited to aid in that.

### Installation

Installing `beancount` is very simple, but it requires you to have Python and its package manager
`pip` already installed. If you don't have those already I'd recommend just doing a quick search
"how to install python and pip on (windows / mac / linux)". If you're on a Linux distro, Python and
pip may already be installed so just check. Once `pip` is installed you can install `beancount` by
running the following in a command line:

```bash
pip install beancount # or pip3 install beancount
```

Next to beancount I'd highly recommend also installing the [`fava` web interface for beancount](https://beancount.github.io/fava/).
It allows you to have a visual interface in your browser to see your finances and even allows you to
make basic entries.

### ✏️ Editing Files
Beancount and the CLI (command line) tools that come installed with it are mainly there to help you
gain insight and create summaries based on your entries. Beancount entries are tracked in
`.beancount` files which are created and edited manually, the `.beancount` format is a simple,
human-readable text format for accounting financial data. For editing such files I'd highly
recommend using an editor like [VS Code](https://code.visualstudio.com/), while it's typically used
by programmers note you don't need to know how to code.

If you choose to go with VS Code you should install the Beancount plugin, it'll give you syntax
highlighting, auto alignment, auto completion and more to make it even easier to manage your
entries.

![Beancount VS Code Plugin](/assets/images/beancount-vscode-plugin.png)


## 📚 Accounting Basics: Double-Entry Bookkeeping

Now that we've installed the tool I'll be demonstrating the basic accounting concepts, how they
relate to crypto / DeFi accounting and how to apply them using beancount.

### 🗒️Your First Ledger

Your "ledger" or "books" is the central store of all your transactions. In beancount it's a set of
files with the `.beancount` extension. Let's go ahead and create our first ledger, I'll call it
`Main.beancount` but the name doesn't really matter:

![Beancount VS Code Plugin](/assets/images/beancount-ledger-creation.png)

If you just want to make a small note or remark for yourself you can add a so-called comment by
prepending the line with semi-colon `;`:

```
; This is a comment
; Doesn't matter what I say here beancount won't recognize it and it won't break my file!
```
{: file='Main.beancount'}

Next you'll want to specify your base currency, unless you're a DAO which wants to do its accounting
in a stablecoin or a coin like ETH, BTC or RAI you should pick your local fiat currency. While
I don't live in the US for the sake of this post I'll go with US-Dollars:

```
; Define the base currency (remember this is a comment):
option "operating_currency" "USD"
```
{: file='Main.beancount'}

Now that the bare basics are set up you can start `fava` by opening a terminal and running
`fava Main.beancount` in the folder your ledger is in:

![Fava started in VS Code terminal](/assets/images/beancount-fava-started.png)

You can then go the shown address in your browser to get a visual overview of what you're changing,
but ignore it for now because you're ledger is empty.

### Transactions & Accounts

In double entry accounting, a transaction is recorded anytime an event persistently changes your
finances e.g.:

- A cash transfer from one bank account to another
- Purchase of an item with a credit card
- Receiving a bill
- Buying 100$ worth of ETH
- Receiving interest
- Paying a bill you received 2 months ago

The "receiving bill" example may be an unintuitive one, how does "receiving a bill" affect my finances? Well
when you receive a bill you're being made aware of money you now owe someone. While this post is
aimed at DeFi accounting these examples are meant to demonstrate what transactions roughly
represent in the double-entry accounting framework.

Let's add a simple transaction to our ledger denoting a $500 transfer from a "Bank A" to "Bank B":

```
2022-12-18 *
    Assets:Bank-A                              -500.00 USD
    Assets:Bank-B                               500.00 USD
```
{: file='Main.beancount'}

A transaction in beancount starts with a date in iso format `<year>-<month>-<day>` and a `*`. It's then
followed by rows of postings, which each posting affecting a specific account. You can also add an
optional "payee" and "narration" explaining where and what you're doing, they must be in quotes. For example:

```
2022-12-18 * "JP Morgan" "Rebalancing to my Chase account"
    Assets:Bank-A                              -500.00 USD
    Assets:Bank-B                               500.00 USD
```
{: file='Main.beancount'}

This will help you later understand what a transaction is and why it was done so it's highly you
recommended you add it to every transaction, the payee and narration must always be in quotes.

If you've saved your file and have the VS Code plugin or fava running you'll notice they're
complaining about errors in your ledger file (note I have relative line numbers on which is why they
look weird on my end):
![Fava started in VS Code terminal](/assets/images/beancount-invalid-reference-error.png)

This happens because beancount needs you to define the accounts you're going to use, this
requirement helps you easily catch typos later. To define an account use the "open" directive, the
errors should disappear:
```
2022-12-18 open Assets:Bank-A
2022-12-18 open Assets:Bank-B

2022-12-18 * "JP Morgan" "Rebalancing to my Chase account"
    Assets:Bank-A                              -500.00 USD
    Assets:Bank-B                               500.00 USD
```
{: file='Main.beancount'}

You can call accounts whatever you'd like, colons `:` in the name are similar to the slashes `/` in
file paths, they denote "parent" accounts, like folders allowing you to group together related
accounts:

```
2020-12-18 open Assets:Cash:JPM:Savings
2020-12-18 open Assets:Cash:JPM:Checking
2020-12-18 open Assets:Cash:BofA
2021-03-11 open Assets:Crypto:Tokens
2021-03-11 open Assets:Crypto:Stables
```
{: file='Main.beancount'}

```ml
// running `bean-report Main.beancount accounts | treeify`
-- Assets
   |-- Cash
   |   |-- BofA
   |   |-- JPM
   |       |-- Checking
   |       |-- Savings
   |-- Crypto
       |-- Stables
       |-- Tokens
```
{: file='Account Tree'}

Transactions can have more than two postings however, meaning you can express transactions that
change more than 1 account:

```
; Opening (doesn't have to on the same day)
2022-12-18 open Assets:Cash:Bank-A      
2022-12-18 open Assets:Cash:Physical    
2022-12-18 open Expenses:Fees:Bank-A:ATM

; Transaction
2022-12-18 * "Bank A" "ATM Withdrawal"
    Assets:Cash:Bank-A                       -502.20 USD
    Assets:Cash:Physical                      500.00 USD
    Expenses:Fees:Bank-A:ATM                    2.20 USD
```
{: file='Main.beancount'}

### 🪨 Balance

A core rule of double-entry accounting which is also strictly enforced in beancount is that **all
transactions must balance**. Specifically they must balance to zero. It's this rule combined with
the use of different account types that makes double-entry accounting so powerful. It ensures that
everything is kept track of in some way or another, if you create a transaction that doesn't balance
beancount will give you an error.

However to make your life easier beancount allows you to omit the value of one posting and will
balance the transaction for you automatically:

```
; Opening (doesn't have to on the same day)
2022-12-18 open Assets:Cash:Bank-A      
2022-12-18 open Assets:Cash:Physical    
2022-12-18 open Expenses:Fees:Bank-A:ATM

; Transaction
2022-12-18 * "Bank A" "ATM Withdrawal"
    Assets:Cash:Bank-A                       -502.20 USD
    Assets:Cash:Physical                      500.00 USD
    Expenses:Fees:Bank-A:ATM
```
{: file='Main.beancount'}

You can check the filled in values in fava under "Journal" or by running
`bean-report Main.beancount print` in your command line (`bean-report` is one of the CLI tools
automatically installed alongside `beancount`).

### Account Types

If all transactions must balance then "how do I account for income, expenses, purchases from debt
or even assets I already had when I started?" you may ask. That's where the different account types
come in.

In double-entry accounting accounts are not only where money is (asset accounts) but also where it
comes from (income accounts), where it went (expense accounts), where it's going to go (liability
accounts) and where it has come from in the past (equity accounts).

These 5 base types: Assets, Income, Expenses, Liabilities and Equity are also enforced by beancount.
Any valid account name must start with one of these as its root.

#### Assets & Liabilities

Assets and liabilities are opposites and the simplest account types. Asset are everything you have
in your posession, even if you owe to someone and liabilities are everything you owe someone. For
example when you borrow $10,000.00 worth of coins on a lending protocol you'd book both assets and
liabilities, because you're both getting the tokens you're borrowing but you're creating debt:

```
; Opening
2022-01-01 open Assets:Crypto:Tokens
2022-01-01 open Liabilities:Crypto

; Transaction
2022-03-01 * "Aave" "Borrow Tokens"
    Assets:Crypto:Tokens                   10,000.00 USD
    Income:Crypto:Interest                -10,000.00 USD
```
{: file='Main.beancount'}

#### Expenses & Income
Counterintuitively income accounts have a negative balance, this is because they're a source of
funds, meaning they're deducted when used. If you do a lot of transactions with different DeFi
protocols, maybe even some freelance work and/or have a fixed job, having a good breakdown of income
types by keeping track of corresponding income accounts can give you a good overview of how you made
your money. Similarly expense accounts give you insight into how your money is spent.

Here are two examples:

```
; Opening
2022-01-01 open Assets:Crypto:Tokens
2022-01-01 open Assets:Cash:Bank-A
2022-01-01 open Income:Crypto:Interest
2022-01-01 open Income:Freelance:Consulting
2022-01-01 open Expenses:Fees:Stripe

; Transactions
2022-03-01 * "Aave" "Withdraw Interst"
    Assets:Crypto:Tokens                       30.00 USD
    Income:Crypto:Interest                    -30.00 USD

2022-03-01 * "Bob" "Payment for February"
    Income:Freelance:Consulting            -3,000.00 USD
    Assets:Cash:Bank-A                      2,972.23 USD
    Expenses:Fees:Stripe
```
{: file='Main.beancount'}

#### Equity
Equity accounts are the weirdest account type, they basically represent what you'd have left over if
you took all your assets and paid off your debts. They're mainly used when you start a new ledger or
"close a time period". In accounting "closing" a time period basically means reseting the income
& expense accounts and moving any net profit / loss into equity:

```
; Opening
2022-01-01 open Equity:Opening-Balances
2022-01-01 open Assets:Cash:Bank-A
2022-01-01 open Income:Crypto:Interest
2022-01-01 open Income:Freelance:Consulting
2022-01-01 open Expenses:Fees:Stripe

; Transactions
2022-01-01 * "Opening balances"
  

; inbetween transactions

2022-03-01 * "Bob" "Payment for February"
    Income:Freelance:Consulting            -3,000.00 USD
    Assets:Cash:Bank-A                      2,972.23 USD
    Expenses:Fees:Stripe
```
{: file='Main.beancount'}

Equity in total should be negative, if it's positive it means you owe more than you have
(Liabilities > Assets) and are theoretically insolvent.
