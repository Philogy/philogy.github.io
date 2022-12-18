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
Before diving in to the actual content I wanted to lay out some reasons why you might want to learn the basics of accounting if you're not already motivated / interested.

### 🥸 Tax Compliance
Unless you live in a tax haven you're likely required to accurately report your income and pay taxes. This includes profit from crypto transactions. Considering that centralized institutions like exchanges are cooperating with tax authorities, the transparency of DeFi and the statue of limitations on tax crimes that spans several years, I wouldn't rely on the incompetence of tax authorities for long. If the government is good at one thing it's making sure people pay up.

### 😎 Tax Avoidance
Different than the crime of tax _evasion_, tax **avoidance** is the legal action of carefully
planning and coordinating your financial activities to minimize your tax burden. Basic accounting
combined with some basic knowledge about your local tax law can already lead to massive savings.
I will discuss some basic approaches but it is hard to generalize because every jurisdiction has
its own laws with its own edge cases.


### 📊 Financial Tracking
Besides understanding your tax burden, accounting will allow you to properly track your finances.
You'll be able to accurately track your entire networth, see the source and trends of your expenses,
losses and income. If your goal is to become more wealthy or simply have better security and certainty
over the state of your finances, accounting will help you with that. 

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

## 📚 Basics: Double-Entry Bookkeeping

Now that we've gotten the "why" out of the way let's get to the "how".

Accounting is all about tracking financial information. Unlike simple "cash accounting" where only
cash going in and out is tracked, the more rigorous "double-entry bookkeeping" method allows us to
easily and accurately track all flow of funds, even when they don't have a direct impact on your
cash. This is important because as I'll explain later, under most tax codes you're **not only taxed
when you get cash out**, but when you realize a gain generally.

To understand _double-entry bookkeeping / accounting_ let's take a look at the basic concepts
& components:

### The Ledger

The ledger or "books" is where you keep track of your financial activity. Even without a consistent
system like double-entry bookkeeping the mere act of tracking your activity in one place can already
be a great way to get an overview of your activity.


### Transactions

In double-entry bookkeeping every single event that affects your finances in some persisting manner
is booked as a transaction, for example:
- A cash transfer from one bank account to another
- Purchase of an item with a credit card
- Receiving a bill
- Buying 100$ worth of ETH
- Receiving interest
- Paying a bill you received 2 months ago

In double-entry bookkeeping everything is an **account**. Accounts have balances that increase, decrease
and even become negative. Every transaction consists of **postings**, each posting affects one
account:


|Posting|Account|Amount|
|-----|-----|----|
|posting 1| Bob's Bank | -500.00 USD|
|posting 2| Sam's Bank | +500.00 USD|

It's called "double-entry bookkeeping" because every transaction needs at least 2 postings and they
need to balance. The following are **not valid** transactions:

|Account|Amount|
|-----|----|
| Bob's Bank | -500.00 USD|

|Account|Amount|
|-----|----|
| Bob's Bank | -500.00 USD|
| Sam's Bank | +400.00 USD|

A transaction can have more than 2 postings however, but they **must balance**:

|Account|Amount|
|-----|----|
| Bank A | -500.00 USD|
| Bank B | +350.00 USD|
| Bank C | +150.00 USD|

The fact that transactions must balance is a fundamental property of the double-entry bookkeeping
system, it ensures that the source and destination of every flow of funds is somehow accounted.

### Accounts

#### Assets

As demonstrated in the example transactions above asset accounts are the most straightforward, the
hold stuff you have. Whether it's cash, crypto or your laptop the things in your posession are
assets, even if you owe them to someone else.

#### Expense & Income

Considering that all transactions must contain at least 2 postings and that they must balance, how
does one account for income, don't the funds come from nowhere? What about expenses? Don't funds
disappear? 🤔

Well yes, but no. That's where so-called **expense and income accounts** come in. You see in
double-entry bookkeeping, accounts are more than bank accounts, they do not simply hold assets,
instead they are more general. You can imagine them more like trackers / buckets for stats about
your finances. Asset accounts track the "stuff you have", income "stuff you've earned" and expenses
"stuff you've given out":

**You do some freelance work and get wired $1,200.00**

|Account|Amount|
|-----|----|
| Freelance Income| -1,200.00 USD|
| Bank A | +1,200.00 USD|

**You withdraw from an exchange paying a $4.12 fee**

(Not accounting the actual withdrawal)

|Account|Amount|
|-----|----|
| Exchange A | -4.12 USD|
| Withdrawal Fees (Expense)| +4.12 USD|

**You withdraw $200.00 from Bank A to B, paying a $3.08 fee**

|Account|Amount|
|-----|----|
| Bank A | -200.00 USD|
| Withdrawal Fees (Expense)| +3.08 USD|
| Bank B | +196.92 USD|

Unintuitively this means that income accounts have a negative balance.

#### Liabilities
Just like expense and income accounts are sort of opposites, liability accounts are the opposite to
asset accounts. Liabilities are things you owe, future benefits you'll have to give up, like
a mortgage, credit card or unpaid invoice. Some examples:

**You take on debt by buying groceries with your credit card**

|Account|Amount|
|-----|----|
| Grocery Expenses | +75.89 USD|
| Credit Card -XXXX | -75.89 USD|

**You pay off your credit card at the end of the month also paying off newly accrued interest**

|Account|Amount|
|-----|----|
| Cash Bank A | -1211.80 USD |
| Credit Card -XXXX (liability) | +1186.81 USD |
| Credit Card Interest (expense) | +124.99 USD |

**You borrow $250 worth of BTC with margin on Binance** 

|Account|Amount|
|-----|----|
| Binance Margin Debt (BTC) | -250.00 USD |
| Binance Margin Assets (BTC) | +250.00 USD |

#### Equity
The final, and possibly strangest account class is equity. Equity represents the "leftover" in your
accounting. If you took the base value of everything you have and paid of all your debts
(liabilities) equity is what you'd be left with.

Luckily you typically don't have to worry about equity accounts too much, in personal accounting you
only touch them at the start for your opening balances and the end of a quarter / year to book
income and expenses into. The process of moving income and expense balances into equity is called
"closing".

Some examples:

**You start a new ledger accounting $4,200.23 cash and NFTs you bought at a cost of $1,200.69**

|Account|Amount|
|-----|----|
| Cash at Bank A | +4,200.23 USD |
| NFTs | +1,200.69 USD |
| Opening Balances (Equity) | -5400.92 USD|

**You close your income and expenses accounts for the year, booking a net profit of $940.92**

|Account|Amount|
|-----|----|
| Crypto Capital Gains | +1,181.03 USD |
| Transaction Fees | -240.11 USD |
| Retained Capital Gains (Equity) | -940.92 USD|

### Balance Sheet

Your balance sheet

## 🪙 Inventory, Lots and Booking Methods

### Inventory & Lots
Now that we've covered the absolute basics we can get into the juicy stuff, accounting crypto
assets, more generally referred to as inventory. While I'll be talking about crypto specifically the
principles apply to other forms of inventory such stocks, real estate, collectibles, domains etc.

Since accounting is typical done in some fiat base currency e.g. USD or EUR, we need some extended
system to track and manage other assets we may own such as crypto. When you acquire a token it'll be
at some price and cost. When received the token is booked on the balance sheet at its cost: 

**You buy 1.5 ETH on an exchange on the 15. of March 2022, at a price of 2,580.00 USD**

|Account|Amount|
|-----|----|
| Exchange A (Cash) | -3,870.00 USD |
| Exchange A (ETH) | +3,870.00 USD|

Note the main ledger only really cares about postings in the base currency. Purchased assets, their
price and cost are tracked separately with so-called lots. One lot represents a position / purchase,
for the transaction above, in a separate part of our ledger we could add a lot in our assets
inventory:

|Date|Asset|Units|Price|Cost|
|----|-----|-----|-----|----|
|2022-03-15| ETH | 1.5 | +2,580.00 USD | +3,870.00 USD |

**Later on the 18. October you buy another 0.8 ETH at a price of 1,330.70 USD / ETH**

Transaction:

|Account|Amount|
|-----|----|
| Exchange A (Cash) | -1,064.56 USD |
| Exchange A (ETH) | +1,064.56 USD |

Inventory after transaction:

|Date|Asset|Units|Price|Cost|
|----|-----|-----|-----|----|
|2022-03-15| ETH | 1.5 | 2,580.00 USD | +3,870.00 USD |
|2022-10-18| ETH | 0.8 | 1,330.70 USD | +1,064.56 USD |

What if you were to sell 1.0 ETH now, which lots would you book from? This is an important question
because depending on which lots you deduct from you'll have a different realized gain / loss. You
see in double-entry bookkeeping we don't actually track our unrealized "paper" gains / losses, you
only track your realized gains, this is called the "realization principle" of accounting. This is
done not only because it's more practical but because most tax codes only require you to report and
tax realized gains and losses.

The difference between the value you acquired the asset for, your "cost basis" and the value you're
disposing "selling" it for is the realized gain or loss. If the price were 3,000.00 USD / ETH and
you sold 1.0 and you only booked from the older March lot you'd only have a gain of 420.00 USD vs.
1,419.44 USD if you booked 0.8 from the December lot and 0.2 from the March lot.

### Booking Methods

Choosing what lots to book from or expressed more simply "which coins are sold" in a given sale
depends on your tax laws, some countries give you more freedom to choose and some less. There can
also be differences between what booking rules private individual vs. corporations can use.

For the following rules I'll be using the following inventory (not real prices) as the example:

|Date|Asset|Units|Price|Cost|
|----|-----|-----|-----|----|
|2022-03-01| ETH | 1.5 | 2,000.00 USD | +3,000.00 USD |
|2022-06-01| ETH | 1.5 | 3,000.00 USD | +4,500.00 USD |
|2022-09-01| ETH | 1.5 | 1,500.00 USD | +2,250.00 USD |
|2022-11-01| ETH | 1.0 | 1,000.00 USD | +1,000.00 USD |


#### FIFO (First In First Out)

A quite common rule, applicable in both the US and Germany for example, dictates that when you sell
coins you must choose the "first" i.e. oldest coins. In our example above this would mean the March
lot. Under this rule the sale of 2.0 ETH at a price of 2,500.00 USD would in a 500.00 USD gain:

> This calculation is just meant as an illustration, once you start using software it'll be taken
> care of automatically. Regardless the calculation is quite simple, consisting of some
> multiplication, addition and subtraction.
{: .prompt-info}

|Lot Date|Price|Lot Units Reduced|Units Left|Lot Gain|Net Gain|
|----|----|-----|------|-----|----|
||2,500.00 USD||2.0 ETH||0.00 USD|
|2022-03-01|2,000.00 USD|1.5 ETH|0.5 ETH|+750.00 USD|+750.00 USD|
|2022-03-01|3,000.00 USD|0.5 ETH|0.0 ETH|-250.00 USD|+500.00 USD|

After a 2.0 ETH sale under FIFO our inventory would look like this:

|Date|Asset|Units|Price|Cost|
|----|-----|-----|-----|----|
|2022-06-01| ETH | 1.0 | 3,000.00 USD | +4,500.00 USD |
|2022-09-01| ETH | 1.5 | 1,500.00 USD | +2,250.00 USD |
|2022-11-01| ETH | 1.0 | 1,000.00 USD | +1,000.00 USD |

#### LIFO (Last In First Out)

Similar to the FIFO method, in LIFO the newest coins are sold first. 

#### HIFO (Highest In First Out)

The "highest in first out" method is more or less the most advantageous option. It requires you to
book from the most expensive lots first, minimizing gains and maximizing your losses. From some
basic research[^1] it looks like you can actually apply this rule if you're filing in the US and
fully tracking your inventory.

#### Exotic Methods
There are some more rarer methods you may be able to use in certain countries like "average cost
booking" but I haven't found much detail on where and when you can use such methods. In general
there could be many conceivable booking methods but ideally you should use the one that's actually
valid for filing your taxes so that you have an accurate overview of what your taxable gain might
actually look like.

#### Universal vs. Per Account Tracking
Depending on your countries' rules you may have to track your inventory on a universal vs. account
basis. Account / Per Wallet application means that every separate account would have its own
inventory and order, however this could quickly become infeasible if you start transferring assets
between accounts which is why for crypto universal tracking is much more practical. When you track
your inventory "universally" it just means you have one inventory and queue per asset regardless
where you transfer or hold the assets.

#### Cost Basis And Booking When Shorting

If you're shorting crypto by actually borrowing cryptocurrencies / tokens on margin exchanges or
lending platforms you need to track a separate "negative inventory" for your liabilities. This will
allow you to calculate the profit and loss of your short trades. An example:

1. **You borrow 10,000 UST from Aave at a price 0.93 USD / UST and sell it for USDC:**

Transactions:

|Account|Amount|
|-----|----|
| UST Liabilities | -9,300.00 USD |
| USDC Assets | +9,300.00 USD |

UST Liabilities Inventory:

|Date|Asset|Units|Price|Cost|
|----|-----|-----|-----|----|
|2022-05-11| UST | -10,000.00 | 0.93 USD | -9,300.00 USD |

**You sell your USDC back for the UST at 0.06 USD / UST and repay your loan (assuming no
interest):**

|Account|Amount|
|-----|----|
| USDC Assets | -558.00 USD |
| Shorting Gains (Income) | -8,742.00 USD |
| UST Liabilities | +9,300.00 USD |

You'll notice that despite the price being down to $0.06 / UST, $9,300.00 is still booked to the UST
liabilities account because that was the "cost basis" for the loan.

## Putting it in Practice: Beancount

Before I show you more examples of how to book more exotic transactions I want to show you this very
basic, free and open-source accounting tool called `beancount`. It's very simple, taking only
a text-file ledger as input and allowing you to generate different reports and insights. It's mainly
a command-line based tool so it's well suited for developers but it doesn't require any programming
knowledge unless you want to extend it with custom plugins or write your own querries.

### Installation

Installing `beancount` is very simple, but it requires you to have Python and its package manager
`pip` already installed. If you don't have those already I'd recommend just doing a quick search
"how to install python and pip on (windows / mac / linux)". If you're on Linux Python and pip may
already be installed so just check. Once `pip` is installed you can install `beancount` by running
the following in a command line:

```bash
pip install beancount # or pip3 install beancount
```

Next to beancount I'd highly recommend also installing the [`fava` web interface for beancount](https://beancount.github.io/fava/).
It allows you to have a visual interface in your browser to see

### Base Currency

Note that while the examples for the rest of this post will use US-Dollars ($ / USD) as a base
currency you can do your accounting in any base currency you'd like, whether it be Euros, Indian
Rupees, Canadian Dollars or even cryptocurrencies like BTC / ETH.

[^1]: [CNBC - This rarely used tax loophole is helping some bitcoin holders save tons of cash](https://www.cnbc.com/2022/01/15/bitcoin-tax-loophole-how-hifo-accounting-reduces-irs-bill.html)
