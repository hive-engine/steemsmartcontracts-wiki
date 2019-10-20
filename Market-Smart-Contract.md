
# Market contract

## Actions available:
### buy: 
Buys tokens with STEEMP.
The STEEMP that you are trying to exchange are locked into the smart contract. 

 - parameters:
	- symbol (string): symbol of the token you want to buy
	- quantity (string): quantity of tokens to buy
	- price (string): maximum number of STEEMP/token you are willing to pay

- example:
```
{
    "contractName": "market",
    "contractAction": "buy",
    "contractPayload": {
        "symbol": "TKN",
        "quantity": "100000",
        "price": "0.00001"
    }
}
```
	
### sell: 
Sells tokens for STEEMP.
The tokens that you are trying to exchange are locked into the smart contract. 

 - parameters:
	- symbol (string): symbol of the token you want to sell
	- quantity (string): quantity of tokens to sell
	- price (string): minimum number of STEEMP/token you are willing to sell

- example:
```
{
    "contractName": "market",
    "contractAction": "sell",
    "contractPayload": {
        "symbol": "NKT",
        "quantity": "100",
        "price": "0.034"
    }
}
```

### cancel: 
Cancels an order.
Will unlock any tokens that haven't been bought/sold.

 - parameters:
	- type (string: "buy"|"sell"): type of order
	- id (string): Steem transaction ID that initiated the order

- example:
```
{
    "contractName": "market",
    "contractAction": "cancel",
    "contractPayload": {
        "type": "sell",
        "id": "1b7e32719b76f07e144d15c8d3045545b896e90b"
    }
}
```

## Tables available:

### buyBook:
list of all the pending buy orders

-	fields:
	-	txId = transactionId

	- account = account owning the order

	- symbol = symbol of the token to exchange

	- quantity = quantity of tokens to buy

	- price = price (STEEMP/token)

	- tokensLocked = number of remaining STEEMP that are locked into the smart contract

- indexes available: symbol, account, priceDec (index on price), expiration, txId

### sellBook:
list of all the pending sell orders

-	fields:
	-	txId = transactionId

	- account = account owning the order

	- symbol = symbol of the token to exchange

	- quantity = remaining quantity of tokens to sell

	- price = price (STEEMP/token)

- indexes available: symbol, account, priceDec (index on price), expiration, txId

### tradesHistory:
list of all the trades (max 24hrs of history)
-	fields:
	- type = 'buy' or 'sell'

	- symbol = symbol of the token traded

	- quantity = quantity of tokens traded

	- price = price (STEEMP/token)

	- timestamp = unix timestamp of the trade (this timestamp is based on the timestamp of the reference Steem block)

- indexes available: symbol

### metrics:
list of the metrics per token
-	fields:
	- "symbol": symbol of the token
    - "volume": 24h volume traded
    - "volumeExpiration": unix timestamp when the volume is not relevant anymore (when no trades during the last 24h for example)
    - "lastPrice": last price for which the token was traded
    - "lowestAsk": lowest price from the sell order book
    - "highestBid": highest price from the buy order book
    - "lastDayPrice": price of the token 24h ago
    -  "lastDayPriceExpiration": unix timestamp when lastDayPrice is not relevant anymore (when no trades during the last 24h for example)
    - "priceChangeSteem": 24h price change in Steem
    - "priceChangePercent": 24h price change in %

- indexes available: symbol
