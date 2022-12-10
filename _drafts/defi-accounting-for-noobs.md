---
title: "Beancount: Easy DeFi Accounting For Noobs"
categories: [accounting]
tags: [DeFi, accounting, beancount]

---


> **Disclaimer**
>
> I'm neither a professional accountant, tax advisor or lawyer. The content of this post is meant to be educational and should not be considered as professional advice of any kind. 
{: .prompt-warning}

## Intro
This blog post will give you a practical introduction to accounting in the context of DeFi.

I've created this post to share some of my learnings as an amateur, self-taught bookkeeper and expand the community of accounting interested DeFi users. Hopefully I can help some of you with your daily crypto / DeFi trading activities and maybe foster a small community accounting interested nerds like me. 😄

**Note:** I will use the labels "bookkeeping" / "bookkeepers" and "accounting" / "accountants" interchangeably despite them being considered not entirely the same in accounting circles.


## Why To Learn Some Basic Accounting
Before diving in to the actual content I wanted to lay out some reasons why you might want to learn the basics of accounting if you're not already motivated / interested.

### 🥸 Tax Compliance
Unless you live in a tax haven you're likely required to accurately report your income and pay taxes. This includes profit from crypto transactions. Considering that centralized institutions like exchanges are likely cooperating with tax authorities, the transparency of DeFi and the statue of limitations on tax crimes that spans several years, I wouldn't rely on the incompetence of tax authorities for long. If the government is good at one thing it's making sure people pay up.

### 😎 Tax Avoidance
Different than the crime of tax _evasion_, tax **avoidance** is the entirely legal action of carefully planning and coordinating your financial activities to minimize your tax burden. Basic accounting combined with some basic knowledge about your local tax law can already lead to massive savings. I will discuss some basic approaches but it is hard to generalize because every jurisdiction has its own laws with its own edge cases.

### 📊 Financial Tracking
Besides understanding your tax burden, accounting will allow you to properly track your finances. You'll be able to accurately track your entire networth, see the source and trends of your expenses, losses and income. If your goal is to become more wealthy or simply have better security and certainty over the state of your finances, accounting can help you with that. 

### The Downsides Of An External Accountant
While you could hire an accountant to do everything for you, unless you have a pretty sizable networth it will hardly be worth the cost. Most accountants are not familiar with crypto let alone obscure DeFi protocols. If your activity consists of more than just basic spot trades you'll have to spend time, money and effort explaining to them how you're earning pickled sausage coins by borrowing using your non fungible, ranged liquidity position as collateral.

Furthermore not having direct control of your "books" (accounting lingo for where all your transactions, inventory etc. are kept track of) limits your ability to optimize your taxes, making you completely reliant on advice from your accountant or other advisors
 
### When To Get Help From Experts 💼
While this whole post is about accounting basics and how to easily keep track of ones' books there are situations where you can and probably should consult with an expert. When it comes to filing statements I would suggest you consult a professional or use some software.

In fact if you're just doing basic trades on exchanges and the popular DeFi protocols you should probably just use one of the many existing crypto tax softwares.

## Crypto Tax Software
If you just need to file your taxes and don't really care about accounting you can just use existing tax
software. It'll allow you to automatically track your portfolio, import your transactions and some
will even give you advice on tax optimizing actions such as tax loss harvesting.

## 📚 Basics: Double-Entry Bookkeeping

Accounting is all about tracking financial information. Unlike simple "cash accounting" where only
cash is tracked the more rigorous "double-entry bookkeeping" method allows us to accurately track
tall types of fund flow but also ensures that the source and destination are always accounted for.


### Transactions

Every single event that affects your finances is booked as a transaction, for example:
- A cash transfer from one bank account to another
- Purchase of an item with a credit card
- Buying 100$ worth of ETH
- Receiving interest

In double-entry bookkeeping everything is an account. Accounts have balances that increase, decrease
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

#### Expense & Income

Considering that all transactions must contain at least 2 postings and that they must balance how
does one account for income, don't the funds come from nowhere? What about expenses? Don't funds
disappear? 🤔

Well yes, but no. That's where so-called **expense and income accounts** come in. You see in
double-entry bookkeeping, accounts are more than bank accounts, they do not simply hold assets,
instead they are more general. You can imagine them more like trackers / buckets for income and
expenses. Some example transactions:

**You do some freelance work and get wired $1,200.00**

|Account|Amount|
|-----|----|
| Bank A | +1,200.00 USD|
| Freelance Income| -1,200.00 USD|

**You withdraw from an exchange paying a $4.12 fee**

|Account|Amount|
|-----|----|
| Exchange A | -4.12 USD|
| Withdrawal Fees (Expense)| +4.12 USD|

**You transfer $200.00 between 2 exchanges paying a $3.08 fee**

|Account|Amount|
|-----|----|
| Exchange A | -200.00 USD|
| Withdrawal Fees (Expense)| +3.08 USD|
| Exchange B | +196.92 USD|

#### Liabilities
Just like expense and income accounts are sort of opposites, liability accounts are the opposite to
asset accounts. Liabilities are things you owe, future benefits you'll have to give up, like
a mortgage, credit card or unpaid invoice. Some examples:

**You take on debt by buying groceries with your credit card**

|Account|Amount|
|-----|----|
| Grocery Expenses | 75.89 USD|
| Credit Card -XXXX | -75.89 USD|

**You pay off your credit card at the end of the month, paying off newly accrued interest**

|Account|Amount|
|-----|----|
| Credit Card -XXXX | 1186.81 USD |
| Cash Bank A | -1211.80 USD |
| Credit Card Interest | 24.99 USD |

#### Equity
The final, and possibly strangest account class is equity. Equity represents the leftover in the
balance sheet. If you took your assets and paid off all your liabilities, equity is the value you'd
be left with. At least personally I've rarely used equity accounts. They're typically only touched
when you start a new accounting ledger or at the end of quarters / years when you "close" the
expense and income accounts. In businesses they're also touched when dividends are issued or
investment capital is raised.

### Other Important Accounting Lingo
- **Balance sheet**: A summary of balances, usually a snapshot of a person's or company's financials
  at a given moment in time
- **The Accounting Equation**: $Assets = Liabilities + Equity$, ensures that everything balances as
  expected
- **Books / Ledger**: Refers to the place where the transactions associated to an entity are stored,
  typically digital but can also be on paper

#### Debits & Credits
When real accountants build transactions they say they're "crediting" or "debiting" accounts. Debits
are equivalent to adding an amount and crediting to decreasing an account. Furthermore accounts are
referred to as credit / debit accounts depending if they "increase" in the positive or negative
direction. Asset and Expense accounts are debit accounts while Liability, Income and Equity accounts
are credit accounts.

We'll do away with this dichotomy because simply thinking in terms of negative / positive numbers is
easier and that's also what the software I'll demonstrate later uses. The two paradigms are pretty
much equivalent.

## 🪙 Inventory, Lots and Booking Methods
Now that we've covered the absolute basics we can get into the juicy stuff, accounting crypto
assets, more generally referred to as inventory. While I'll be talking about crypto specifically the
principles apply to other forms of inventory such stocks, real estate, collectibles, domains etc.

Since the accounting is done in some base currency e.g. USD or EUR, we need some extended system to track and manage
other assets we may own such as crypto. When you acquire a token it'll be at some price and cost.
When received the token is booked on the balance sheet at its cost: 

**You buy 1.5 ETH on exchange at a price of $1,200.00 / ETH**

|Account|Amount|
|-----|----|
| Exchange A (Cash) | -1,800.00 USD |
| Exchange A (ETH) | +1,800.00 USD|

Note that transactions still need to balance in the base currency, regardless of what asset is
actually being transacted. Furthermore you don't account any gain / loss until it's actually
realized by selling the asset or somehow else disposing of the asset, this is the so-called
"realization principle". Now note we still need a way to track the price / cost of non-cash assets
on our balance sheet. That's where lots come in, in the above example in a separate location you'd
add a lot.

