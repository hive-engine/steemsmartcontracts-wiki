Documentation written by [ali-h](https://github.com/ali-h)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* [Actions available](#actions-available)
  * [addPair](#addPair)
  * [buy](#buy)
  * [sell](#sell)
  * [marketBuy](#marketBuy)
  * [marketSell](#marketSell)
* [Tables available](#tables-available)
  * [pairs](#pairs)
  * [buyBook](#buyBook)
  * [sellBook](#sellBook)
  * [tradesHistory](#tradesHistory)
  * [metrics](#metrics)

# Introduction

dMarket contract allows users to trade tokens with different pairs other than SWAP.HIVE (the original market contract).

It stores different pairs and tokens that can be traded. A global pair means every token is allowed to be traded for it, only Hive Engine team can create a global pair. There are two global pairs allowed by default (SWAP.BTC & BEE). More pairs can be added by users. It allows tribes and other projects to allow tokens to be traded for their native tokens (ex. DEC:PAL, ORB:PAL, BEE:PAL, SWAP.DOGE:PAL, etc..).

# Actions available:

## addPair:
Used to add a new pair. A fee of 100 BEE is required to create a new pair.
* requires active key: yes
* can be called by: anyone
* parameters:
  * pair (string): symbol of the token to be used as price. (replacing traditional SWAP.HIVE)
  * symbol (string): token to add in the pair

* example: PAL token is allowed to be traded for SWAP.BTC
```
{
    "contractName": "dmarket",
    "contractAction": "addPair",
    "contractPayload": {
        "pair": "SWAP.BTC",
        "symbol": "PAL"
    }
}
```

A successful action will emit a "addPair" event, e.g.
```
{
    "contract": "dmarket",
    "event": "addPair",
    "data": {
      "pair": "SWAP.BTC",
      "symbol": "PAL"
    }
}
```

## buy:
Buy any token for any existing pair.
* requires active key: yes
* can be called by: anyone
* parameters:
  * symbol (string): symbol of the token you want to buy
  * pair (string): symbol of the token to be used as price (e.g. SWAP.BTC)
  * quantity (string): quantity of tokens to buy
  * price (string): maximum number of pair/token you are willing to pay

* examples: Buys BEE tokens for the price of 0.00000912 SWAP.BTC per token
```
{
    "contractName": "dmarket",
    "contractAction": "buy",
    "contractPayload": {
        "symbol": "BEE",
        "pair": "SWAP.BTC",
        "quantity": "120",
        "price": "0.00000912"
    }
}
```

## sell:
Sell any token for any existing pair.
* requires active key: yes
* can be called by: anyone
* parameters:
  * symbol (string): symbol of the token you want to sell
  * pair (string): symbol of the token to be used as price (e.g. SWAP.BTC)
  * quantity (string): quantity of tokens to sell
  * price (string): minimum number of pair/token you are willing to sell

* examples: Sells BEE tokens for the price of 0.00000925 SWAP.BTC per token
```
{
    "contractName": "dmarket",
    "contractAction": "sell",
    "contractPayload": {
        "symbol": "BEE",
        "pair": "SWAP.BTC",
        "quantity": "120",
        "price": "0.00000925"
    }
}
```

### marketBuy:
Executes a market buy with the pair token. This action will try to buy as many tokens as it can with the amount you specify, regardless of market price. As such, multiple sell orders could be filled by this action, and you may experience price slippage if the order book is thin. 
* requires active key: yes
* can be called by: anyone
* parameters:
  * symbol (string): symbol of the token you want to buy
  * pair (string): symbol of the token to be used as price (e.g. SWAP.BTC)
  * quantity (string): quantity of pair token to spend

* examples: Buy as many BEE tokens as possible with 0.0015 SWAP.BTC
```
{
    "contractName": "dmarket",
    "contractAction": "marketBuy",
    "contractPayload": {
        "symbol": "BEE",
        "pair": "SWAP.BTC",
        "quantity": "0.0015"
    }
}
```

### marketSell:
Executes a market sell for pair token. This action will try to sell as many tokens as the quantity you specify, regardless of market price. As such, multiple buy orders could be filled by this action, and you may experience price slippage if the order book is thin. 
* requires active key: yes
* can be called by: anyone
* parameters:
  * symbol (string): symbol of the token you want to sell
  * pair (string): symbol of the token to be used as price (e.g. SWAP.BTC)
  * quantity (string): quantity of tokens to sell

* examples: Sells 150 BEE tokens for SWAP.BTC
```
{
    "contractName": "dmarket",
    "contractAction": "marketSell",
    "contractPayload": {
        "symbol": "BEE",
        "pair": "SWAP.BTC",
        "quantity": "150"
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

## pairs:
contains all the pairs info and tokens that are allowed to be traded for them
* fields
  * symbol = pair token symbol (the token that will be used as price)
  * precision = precision of the pair token
  * allowedSymbols = array of the tokens that are allowed to be traded for this pair

## buyBook:
list of all the pending buy orders
* fields
  * txId = transactionId
  * account = account owning the order
  * symbol = symbol of the token to exchange
  * pair = symbol of the token to exchange for (token used as price)
  * quantity = quantity of tokens to buy
  * price = price (pair/token)
  * tokensLocked = number of remaining pair tokens locked into the smart contract

## sellBook:
list of all the pending sell orders
* fields
  * txId = transactionId
  * account = account owning the order
  * symbol = symbol of the token to exchange
  * pair = symbol of the token to exchange for (token used as price)
  * quantity = quantity of tokens to sell
  * price = price (pair/token)

## tradesHistory:
list of all the trades (max 24 hrs of history)
* fields
  * type = 'buy' or 'sell'
  * buyer = account that is buying the tokens
  * seller = account that is selling the tokens
  * symbol = symbol of the token traded
  * pair = symbol of the token used as price
  * quantity = quantity of tokens traded
  * price = price (pair/token)
  * timestamp = unix timestamp of the trade (this timestamp is based on the timestamp of the reference hive block)
  * volume = amount of pair token transacted
  * buyTxId = order transaction ID of the order hit on the buy side
  * sellTxId = order transaction ID of the order hit on the sell side

## metrics:
list of the metrics per token
* fields
  * symbol: symbol of the token
  * pair: pair symbol that these metrics are for
  * volumeExpiration = unix timestamp when the volume is not relevant anymore (when no trades during the last 24h for example)
  * lastPrice = last price for which the token was traded
  * lowestAsk = lowest price from the sell order book
  * highestBid = highest price from the buy order book
  * lastDayPrice = price of the token 24h ago
  * lastDayPriceExpiration = unix timestamp when lastDayPrice is not relevant anymore (when no trades during the last 24h for example)
  * priceChangePair = 24h price change in pair
  * priceChangePercent = 24h price change in %
