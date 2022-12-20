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
along with beancount. Beancount has a very nice API making it easy to extend with Python.

Furthermore as part of its core feature set it has some very nice handling of inventor" or as it
prefers to call it in its [documentation](https://beancount.github.io/docs/) "commodities". While
not specifically made for crypto it's very well suited for accounting crypto transactions.

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
make basic entries. To install fava run `pip install fava`.

### ✏️ Editing Files
Beancount and the CLI (command line) tools that come installed with it are mainly there to help you
gain insight and create summaries based on your entries. Beancount entries are tracked in
`.beancount` files which are created and edited manually, the `.beancount` format is a simple,
human-readable text format for accounting financial data. For editing such files I'd highly
recommend using an editor like [VS Code](https://code.visualstudio.com/), while it's typically used
by programmers note you don't need to know how to code to be able to use beancount.

If you choose to go with VS Code you should install the Beancount plugin, it'll give you syntax
highlighting, auto alignment, auto completion and more to make it even easier to manage your
entries.

![Beancount VS Code Plugin](/assets/images/beancount-vscode-plugin.png)


## 📚 Accounting Basics: Double-Entry Bookkeeping

Now that we've installed the tool I'll be demonstrating the accounting concepts, how they relate to
crypto / DeFi accounting and how to apply them using beancount.

### 🗒️Your First Ledger

Your "ledger" or "books" is the central store of all your transactions. In beancount it's a set of
files with the `.beancount` extension. Let's go ahead and create our first ledger, I'll call it
`Main.beancount` but the name doesn't really matter:

![Beancount VS Code Plugin](/assets/images/beancount-ledger-creation.png)

If you just want to make a small note or remark for yourself you can add a so-called comment by
prepending the line with a semi-colon `;`:

```
; This is a comment
; Doesn't matter what I say here beancount won't recognize it and it won't break my file!
```
{: file='Main.beancount'}

Next you'll want to specify your base currency, unless you're a DAO that wants to do its accounting
in a stablecoin or crypto coin like ETH or BTC you should pick your the fiat currency you use
day-to-day and report your taxes in. While I don't live in the US for the sake of this post I'll go
with US-Dollars:

```
; Defines the base currency (remember this is a comment):
option "operating_currency" "USD"
```
{: file='Main.beancount'}

Now that the bare basics are set up you can start `fava` by opening a terminal and running
`fava Main.beancount` in the folder your ledger is in:

![Fava started in VS Code terminal](/assets/images/beancount-fava-started.png)

You can then go to the shown address (likely `https://localhost:5000`) in your browser to get a
visual overview of what you're changing, but ignore it for now because your ledger is empty.

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

This will help you later understand what a transaction is and why it was done so it's highly
recommended you add at least narration to your transactions. The payee and narration must always be
in quotes `"`.

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

In double-entry accounting, accounts are not only where money is (asset accounts) but also where it
comes from (income accounts), where it went (expense accounts), where it's going to go (liability
accounts) and where it has come from in the past (equity accounts).

These 5 base types: `Assets`, `Income`, `Expenses`, `Liabilities` and `Equity` are also enforced by
beancount. Any valid account name must start with one of these as its root.

#### Assets & Liabilities

Assets and liabilities are opposites and the simplest account types. Asset are everything you have
in your posession, even if you owe it to someone and liabilities are everything you owe someone. For
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

Similarly if you make a purchase from a credit card:
```
; Opening
2022-01-01 open Liabilities:Credit-Cards:Mastercard-6969
2022-01-01 open Expenses:Living:Groceries

; Transaction
2022-06-01 * "Walmart" "Weekly food shopping"
    Liabilities:Credit-Cards:Mastercard-6969  -79.20 USD
    Expenses:Living:Groceries                  79.20 USD

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

I'll go into closing a bit later.

Equity in total should be negative, if it's positive it means you owe more than you have
(Liabilities > Assets) and are theoretically insolvent.

## 🪙 Inventory, Lots and Booking Methods

### Inventory & Lots

Now that we've covered the basics we can get into how to account crypto assets. You see the
intuitive approach doesn't work, while beancount accepts any type of currency for postings, not only
the base currency we've defined if we try to book a simple trade it'll give us an error:

```
option "operating_currency" "USD"

2022-01-01 open Assets:Cash:Exchange-A
2022-01-01 open Assets:Crypto:Tokens

2022-01-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -2,200.00 USD
    Assets:Crypto:Tokens                        2.00 ETH

```
{: file='Main.beancount'}

Specifically beancount is telling us that `Transaction does not balance: (-2200.00 USD, 2.00 ETH)`.
You see when you book assets in double-entry bookkeeping you book them at cost, like this the
transaction balances in USD:

```
2022-01-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -2,200.00 USD
    Assets:Crypto:Tokens                        2.00 ETH {1,100.00 USD, 2022-01-02}
```
{: file='Main.beancount'}

The price and date are denoted in the curly braces after the asset. This tells beancount that while
we're getting `ETH` we're just converting $2,200.00 into $2,200.00 worth of ETH so that the
transaction balances. Similar to the postings auto-balance you can leave out the date and/or the
price and beancount will automatically fill it in for you (if you're only buying one asset):

```
2022-01-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -2,200.00 USD
    Assets:Crypto:Tokens                        2.00 ETH {}
```
{: file='Main.beancount'}

Booking an asset or "commodity" as beancount likes to call it, like this creates a "lot". A lot is
a single acquisition / coin in your inventory. As you do multiple purchases and sales you will
accumulate different lots each with their own cost and date.

In beancount lots also have an optional label which I use for NFTs:

```
2022-01-01 open Assets:Cash:Exchange-A
2022-01-01 open Assets:Crypto:NFTs

2022-03-02 * "NFT Purchase"
    Assets:Cash:Exchange-A                -10,000.00 USD
    Assets:Crypto:NFTs                          1 GOBBLER {2,000.00 USD, "id:401"}
    Assets:Crypto:NFTs                          1 GOBBLER {2,000.00 USD, "id:402"}
    Assets:Crypto:NFTs                          1 GOBBLER {2,000.00 USD, "id:403"}
    Assets:Crypto:NFTs                          1 GOBBLER {2,000.00 USD, "id:404"}
    Assets:Crypto:NFTs                          1 GOBBLER {2,000.00 USD, "id:405"}
```
{: file='Main.beancount'}

The label has to be part of lot data in the curly braces, separated by a comma and in quotes. What's
in quotes is entirely up to you, I just like to use the `id:<token ID>` convention. While it's very
well suited for crypto you can also use beancount's lot system for other assets:

```
2022-01-01 open Assets:Cash:Bank-A

2022-04-20 open Assets:Real-Estate
2022-04-20 open Liabilities:Mortgages:Home
2022-04-20 * "Closing the purchase of new home"
    Assets:Cash:Bank-A                    -50,000.00 USD
    Liabilities:Mortgages:Home           -450,000.00 USD
    Assets:Real-Estate                          1 PROPERTY {500,000.00 USD, "42th Avenue 69th Street, 10019 NY"}
```
{: file='Main.beancount'}

If you want to see your full inventory you can go to fava under the `Holdings` tab. I don't like
that overview too much however so I made this custom query you can run under fava's `Query` tab or
using beancount's `bean-query` tool:

```sql
SELECT account, cost_date, sum(units(position)) as units, sum(cost(position)) as cost_total, cost_label as label
WHERE currency != 'USD'
GROUP BY currency, account, cost_date, cost_label
ORDER BY account, currency, cost_date
```
{: file='Get non-USD Lots Query'}

### Booking Methods
(source)[^1]

Now that we know how to book cryptocurrencies when we acquire we have to know how to dispose of
them. The main problem or rather question here is in what order to you use your lots?

Consider the inventory resulting from the following purchases:

```
2022-01-01 open Assets:Cash:Exchange-A
2022-01-01 open Assets:Crypto:Tokens
2022-01-01 open Assets:Crypto:NFTs

2022-01-02 * "Simple Trade"
  Assets:Cash:Exchange-A  -2200.00 USD
  Assets:Crypto:Tokens        2.00 ETH {1100 USD, 2022-01-02}

2022-03-02 * "Simple Trade"
  Assets:Cash:Exchange-A  -3300.00 USD
  Assets:Crypto:Tokens        1.50 ETH {2200 USD, 2022-03-02}

2022-06-02 * "Simple Trade"
  Assets:Cash:Exchange-A  -1200.00 USD
  Assets:Crypto:Tokens         0.8 ETH {1500.0 USD, 2022-06-02}
```
{: file='Main.beancount'}

|Account|Date|Units|Cost Total|Label|
|-------|----|-----|----------|-----|
|Assets:Crypto:Tokens|2022-01-02|2.00 ETH|2200.00 USD|None|
|Assets:Crypto:Tokens|2022-03-02|1.50 ETH|3200.00 USD|None|
|Assets:Crypto:Tokens|2022-06-02|0.80 ETH|1200.00 USD|None|

If you were to sell 1 ETH at the current price which "coin(s)" would you sell? The method by which
you choose lots to be used is called the "booking method" and depends on your taxs laws. Some
countries give you more freedom to choose and some less. There can also be differences between what
booking rules private individuals vs. corporations can use.

#### FIFO (First In First Out)
The "First In First Out" booking method, dictates that the oldest lots are to be used first and is
one of the most common booking methods. This method is applicable in both the US and Germany and is
supported by beancount of the box: 

```
2022-01-01 open Assets:Cash:Exchange-A
2022-01-01 open Assets:Crypto:Tokens "FIFO"
2022-01-01 open Assets:Crypto:NFTs

2022-01-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -2,200.00 USD
    Assets:Crypto:Tokens                        2.00 ETH {}

2022-03-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -3,300.00 USD
    Assets:Crypto:Tokens                        1.50 ETH {}


2022-06-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -1,200.00 USD
    Assets:Crypto:Tokens                        0.8 ETH {}

2022-09-13 open Income:Crypto:Gains-PnL
2022-09-13 * "not-FTX" "Sell some ETH"
    Assets:Crypto:Tokens                       -1.00 ETH {}
    Assets:Cash:Exchange-A                  1,717.80 USD
    Income:Crypto:Gains-PnL
```
{: file='Main.beancount'}

The booking method is defined next to the account when it's created. When you dispose of some assets
the curly braces select which lot you're selling. This allows you to pick specific lots / coins
based on the date, cost or label. If you have a booking method configured you can leave the lot
selector empty and beancount will automatically pick the right one for you and fill out the details:


```
> bean-report Main.beancount print

2022-01-01 open Assets:Cash:Exchange-A
2022-01-01 open Assets:Crypto:Tokens                             "FIFO"
2022-01-01 open Assets:Crypto:NFTs

2022-01-02 * "Simple Trade"
  Assets:Cash:Exchange-A  -2200.00 USD
  Assets:Crypto:Tokens        2.00 ETH {1100 USD, 2022-01-02}

2022-03-02 * "Simple Trade"
  Assets:Cash:Exchange-A  -3300.00 USD
  Assets:Crypto:Tokens        1.50 ETH {2200 USD, 2022-03-02}

2022-06-02 * "Simple Trade"
  Assets:Cash:Exchange-A  -1200.00 USD
  Assets:Crypto:Tokens         0.8 ETH {1500.0 USD, 2022-06-02}

2022-09-13 open Income:Crypto:Gains-PnL

2022-09-13 * "not-FTX" "Sell some ETH"
  Assets:Crypto:Tokens       -1.00 ETH {1100 USD, 2022-01-02}
  Assets:Cash:Exchange-A   1717.80 USD
  Income:Crypto:Gains-PnL  -617.80 USD
```
{: file='Beancount'}

Your inventory is then also adjusted automatically:

|Account|Date|Units|Cost Total|Label|
|-------|----|-----|----------|-----|
|Assets:Crypto:Tokens|2022-01-02|**1.00 ETH**|2200.00 USD|None|
|Assets:Crypto:Tokens|2022-03-02|1.50 ETH|3200.00 USD|None|
|Assets:Crypto:Tokens|2022-06-02|0.80 ETH|1200.00 USD|None|

#### LIFO (Last In First Out)
Similar to FIFO the LIFO rule dictates that the last lot i.e. the newest must be used first. This
rule is also available in beancount and can be specified just like the FIFO rule:

```
2022-01-01 open Assets:Crypto:Tokens "LIFO"
```
{: file='Main.beancount'}

#### Average Cost Booking
Unlike FIFO and LIFO, average cost booking requires you to essentially merge all lots into one at
their average price. This method is used in the UK and Canada. This method is not yet supported
by beancount out-of-the-box and requires manual lot recreation or writing a custom plugin:

```
2022-01-01 open Assets:Cash:Exchange-A
2022-01-01 open Assets:Crypto:Tokens
2022-01-01 open Assets:Crypto:NFTs

2022-01-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -2,200.00 USD
    Assets:Crypto:Tokens                        2.00 ETH {}

2022-03-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -3,300.00 USD
    Assets:Crypto:Tokens                       -2.00 ETH {}
    Assets:Crypto:Tokens                        3.50 ETH {}


2022-06-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -1,200.00 USD
    Assets:Crypto:Tokens                       -3.50 ETH {}
    Assets:Crypto:Tokens                        4.30 ETH {}

2022-09-13 open Income:Crypto:Gains-PnL
2022-09-13 * "not-FTX" "Sell some ETH"
    Assets:Crypto:Tokens                       -1.00 ETH {}
    Assets:Cash:Exchange-A                  1,717.80 USD
    Income:Crypto:Gains-PnL
```
{: file='Main.beancount'}

#### Specific ID & HIFO (Highest In First Out)
If you're in the US and you document your purchases very well (which is something that beancount will
enable you to do), then you can apply the "Specific ID" method[^2]. This method allows you to choose
**any lot** from your inventory when you sell. This allows you to come up with and use essentially any
booking method you want, including the HIFO "highest in first out" method. The HIFO method dictates
that you sell your most expensive lots first, this minimizes your realized gains or maximizes your
realized losses, allowing you to minimize your tax burden when you sell.

Selecting custom lots is supported by beancount but methods like HIFO are not supported
out-of-the-box and would either have to be done manually or via a custom plugin:

```
2022-01-01 open Assets:Cash:Exchange-A
2022-01-01 open Assets:Crypto:Tokens
2022-01-01 open Assets:Crypto:NFTs

2022-01-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -2,200.00 USD
    Assets:Crypto:Tokens                        2.00 ETH {}

2022-03-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -3,300.00 USD
    Assets:Crypto:Tokens                        1.50 ETH {}


2022-06-02 * "Simple Trade"
    Assets:Cash:Exchange-A                 -1,200.00 USD
    Assets:Crypto:Tokens                        0.8 ETH {}

2022-09-13 open Income:Crypto:Gains-PnL
2022-09-13 * "not-FTX" "Sell some ETH"
    Assets:Crypto:Tokens                       -1.00 ETH {2022-03-02}
    Assets:Cash:Exchange-A                  1,717.80 USD
    Income:Crypto:Gains-PnL
```
{: file='Main.beancount'}

If you're manually selecting lots you have to make sure that your selection in the curly brackets
`{` `}` is not ambiguous

#### Universal vs. Per Account Booking
FIFO, LIFO and HIFO all require you to keep track of inventory you can book from. While you could
keep track of one inventory for every account (Exchange A, Exchange B, Wallet 1, Wallet 2, etc.)
this separation starts to break down when you transfer assets between locations. That's for crypto
it's usually more practical to keep track a single inventory and do "universal booking" meaning
regardless where you buy / sell your assets you're always booking from the 


## DeFi Accounting

Now that we've covered the basics, understand booking methods and how to account for basic trades
I wanted to go through some more examples related to DeFi and some other smaller nuances.

### Profit & Loss Accounts (PnL Accounts)

For gains related to crypto or other trading activity I create what I like to call "PnL accounts"
they're accounts where book all gains and losses of a certain type so that I can immediately see my
net gain and don't have to check whether or not a given trade realized a loss or gain:

```
2022-01-01 open Assets:Cash:Exchange-A
2022-01-01 open Assets:Crypto:Tokens "FIFO"
2022-01-01 open Assets:Crypto:NFTs
2022-01-01 open Income:Crypto:Gains-PnL

2022-01-01 * "NFT Purchase"
    Assets:Cash:Exchange-A
    Assets:Crypto:NFTs                          1 GOBBLER {4,000.00 USD, "id:400"}
    Assets:Crypto:NFTs                          1 GOBBLER {3,900.00 USD, "id:401"}


2022-03-01 * "ArtGobblers" "Sale"
    Assets:Crypto:NFTs                         -1 GOBBLER {"id:401"}
    Assets:Cash:Exchange-A                  2,000.00 USD
    Income:Crypto:Gains-PnL

2022-08-01 * "DCA into ETH"
    Assets:Cash:Exchange-A                   -500.00 USD 
    Assets:Crypto:Tokens                        0.482 ETH {}

; Includes Fee
2022-11-13 * "Uniswap" "Get some DAI"
    Assets:Crypto:Tokens                       -0.202838 ETH {}
    Assets:Crypto:Tokens                      322.113821 DAI {1.00 USD}
    Income:Crypto:Gains-PnL
```
{: file='Main.beancount'}

This allows me to  easily isolate and track my profit (or losses):


```
2022-01-01 open  Income:Crypto:Gains-PnL
2022-03-01 *     ArtGobblers | Sale          1900.00 USD   1900.00 USD
2022-11-13 *     Uniswap | Get some DAI      -111.70 USD   1788.30 USD
```
{: file='(cmd output of) bean-report Main.beancount journal -a Income:Crypto:Gains-PnL -b -x'}

The above overview can also be seen in fava by clicking on "Go to account" and selecting your PnL
account.

However if you want to breakdown your losses and gains into separate accounts you can do so. You
could even automate the process with a plugin. Making a plugin is actually relatively easy, check
out the [beancount docs on how to script plugins](https://beancount.github.io/docs/beancount_scripting_plugins.html).

### Spread, Slippage & Fees

When you perform trades on exchanges, centralized or decentralized there are different factors that
reduce the value you're able to get out of a trade. By "fees" I'm referring to explicit network
and/or exchange fees that are deducted from your account in exchange for facilitating the trade.
Spreads and slippage on the other hand, while not the same thing, for the sake of accounting they
may as well be as they both represent deviations from your practical price and the actual market
price.

There are two approaches to account for these trade impacts:
1. Direct accounting: calculate fees and slippage / spread and put them into respective loss
   accounts
2. Cost basis inclusion: add any cost associated with a trade to the cost basis of the output asset

Not only is the second option simpler, the first may be incorrect depending on your tax laws. You
see trading fees and price difference costs (slippage, spreads) are usually tax deductible however
if you account them directly by puting them to an expense account, you're moving forward the time of
that deduction. To make it simpler let's just look at a simple example with an exchange that charges
an unrealistic 25% fee:

```
2021-01-01 open Assets:Cash:Exchange-A
2021-01-01 open Expenses:Fees:Exchange-A
2021-01-01 open Assets:Crypto:Tokens                             "FIFO"
2021-01-01 open Assets:Crypto:NFTs
2021-01-01 open Income:Crypto:Gains-PnL

2021-06-13 * "ETH DCA"
  Assets:Cash:Exchange-A    -2500.00 USD
  Expenses:Fees:Exchange-A    500.00 USD
  Assets:Crypto:Tokens          2.00 ETH {1000 USD, 2021-06-13}

2022-07-14 * "ETH Sale"
  Assets:Crypto:Tokens       -0.5 ETH {1000 USD, 2021-06-13}
  Assets:Cash:Exchange-A   450.00 USD
  Income:Crypto:Gains-PnL   50.00 USD
```
{: file='Main.beancount (direct fee inclusion)'}

Versus Cost inclusion:

```
2021-06-13 * "ETH DCA"
  Assets:Cash:Exchange-A  -2500.00 USD
  Assets:Crypto:Tokens        2.00 ETH {1250 USD, 2021-06-13}

2022-07-14 * "ETH Sale"
  Assets:Crypto:Tokens       -0.5 ETH {1250 USD, 2021-06-13}
  Assets:Cash:Exchange-A   450.00 USD
  Income:Crypto:Gains-PnL  175.00 USD
```
{: file='Main.beancount (cost inclusion)'}

You see in the direct inclusion scenario the full expenses is booked in the first year (2021)
although that loss hasn't necessarily had a material impact yet. In the second scenario 1/4 of the
fee is included as part of the loss because 1/4 of the original ETH amount is sold, the fee is
implicitly accounted for when a gain / loss from the affected amount is actually realized.

Which method you can and should use really depends on your tax laws. Accounting the expense
immediately can help reduce your tax bill but may also have you understate your net profit or
overstate your net loss. For these kinds of details it's usually best to consult a professional as
they should be able to answer such questions quite easily and quickly if they have accounting / tax
expertise.

### Crypto Income
Whether it's freelance work or interest, income denominated in crypto is relatively easy to account,
similar to a trade book the crypto to your assets at the current market price and the relevant
income account as source of the funds:

```
2021-01-01 open Assets:Crypto:Tokens "FIFO"
2021-01-01 open Income:Crypto:Gains-PnL
2021-01-01 open Income:Crypto:Interest
2021-01-01 open Income:Freelance:Consulting

2022-03-03 * "Bob" "Payment of invoice #1001"
    Income:Freelance:Consulting
    Assets:Crypto:Tokens                        1.7623 ETH {2,966.84 USD}

; Withdrawal of original collateral not booked because you owned it before and still own it after the withdrawal
; Only the change, the interest is accounted
2022-07-31 * "Aave" "Withdrawal of collateral + interest"
    Income:Crypto:Interest
    Assets:Crypto:Tokens                       34.110 DAI {0.996 USD}
```
{: file='Main.beancount'}

> **Be Aware Of Income Separation**
>
> Depending on your tax laws you may **not be able** to deduct losses from crypto against income
> earned from your fixed job or freelance work because they're classed as different types of income.
>
> This means that if you get paid in crypto and the price drops by 50% you may have to pay taxes on
> the full value of the originially received amount as it was personal income while only being able
> to deduct the 50% price fall from other crypto gains.
>
> If you get paid in crypto it's vital you inform yourself to what extent there's a separation so
> that you can sell and set aside received crypto if necessary.
{: .prompt-danger}

### Liquidity Tokens
LP Tokens are the fungible or non-fungible (NFT) tokens you get when providing liquidity to DEXs
like Uniswap, Sushiswap or Pancakeswap. The tax implications of providing liquidity is hard to
determine in most of the world as there doesn't really exist an equivalent instrument in the TradFi
world that tax authorities or advisors could really compare to.

![Uniswap V3 LP NFT](/assets/images/univ3-lp-token.svg)

But looking at how they work we can at least try to account the intuitively:

1. You give up 1 or 2 assets
2. You receive another asset in return (the LP token/s)
3. While you hold the LP asset it's price can develop independently of the assets, accruing fees and
   going up or down in value based on the underlying
4. When you get rid of the LP asset you receive 1 or 2 assets in different proprtions

Based on these circumstances I think it's fair to say that when you provide liquidity you're
disposing i.e. practically selling your tokens for the LP asset:

```
2021-01-01 open Assets:Crypto:Tokens "FIFO"
2021-01-01 open Income:Crypto:Gains-PnL
2021-01-01 open Income:Crypto:Interest
2021-01-01 open Income:Freelance:Consulting
2021-01-01 open Equity:Opening-Balances


2022-09-03 * "Opening Transaction"
    Assets:Crypto:Tokens                    6,000.00 DAI {1.0001 USD}
    Assets:Crypto:Tokens                        4.23 ETH {1,720.00 USD}
    Equity:Opening-Balances


2022-11-01 open Assets:Crypto:LP "FIFO"


; price directive tells beancount about prices
2022-11-01 price ETH 1,565.00 USD

; Fee cost included
2022-11-01 * "Sushiswap" "Provide ETH / DAI liquidity"
    Assets:Crypto:Tokens                   -3,000.00 DAI {}
    Assets:Crypto:Tokens                       -1.90384929272910 ETH {}
    Assets:Crypto:LP                          629.37 SUSHI-LP {% raw %}{{5979.52 USD, "DAI-ETH"}}{% endraw %}
    Income:Crypto:Gains-PnL
```
{: file='Main.beancount'}

> You can indicate the total cost of the lot rather than the cost / unit by using double curly
> braces:
>
> - `2.0 ETH {1,200.00 USD}`: Book 2 ETH at a cost of 1,200.00 USD each
> - `3.0 ETH {% raw %}{{3,300.00 USD}}{% endraw %}`: Book 3 ETH at a total cost of 3,300.00 USD or
>   1,100.00 USD each
{: .prompt-tip}

When you withdraw your underlying from your LP position you can then book the disposal of the LP
asset although I'd put that into a separate LP account. Accounting this can be a bit tricky if you
consider that to withdraw from your LP you're also spending gas and therefore desposing of ETH,
booking a potentially separate loss / gain:

```
2022-12-04 price ETH 1,242.161 USD

2022-12-04 open Assets:Temp ; holds fee value
2022-12-04 * "Sushiswap" "Withdraw liquidity (network fee)"
    Assets:Crypto:Tokens                       -0.003481 ETH {}
    Assets:Temp                                 4.324 USD ; Fee value
    Income:Crypto:Gains-PnL ; Profit / Loss from disposing ETH for fee

2022-12-04 open Income:Crypto:LP
2022-12-04 * "Sushiswap" "Withdraw liquidity withdrawal"
    Assets:Crypto:LP                         -629.37 SUSHI-LP {"DAI-ETH"}
    Assets:Temp                                -4.324 USD
    Assets:Crypto:Tokens                    3,400.00 DAI {1.00 USD}
    Assets:Crypto:Tokens                        2.7375 ETH {1,242.161 USD}
    Income:Crypto:LP
```
{: file='Main.beancount'}

Note that I split the single transaction (LP withdrawal) over two entries so that I can leverage
beancount's auto-balance feature to calculate the P/L on the ETH and the LP tokens, furthermore for
the sake of simplicity I'm not following my own rule of including the fee in the cost of the output
for the ake of simplicity.


[^1]: [Token Tax -  Accounting methods](https://tokentax.co/blog/crypto-accounting-methods)
[^2]: [Forbes - Article on crypto booking methods](https://www.forbes.com/sites/shehanchandrasekera/2020/09/17/what-crypto-taxpayers-need-to-know-about-fifo-lifo-hifo-specific-id)
