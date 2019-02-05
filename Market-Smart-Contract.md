# Market contract

## Actions available:
### buy: 
Buys tokens with STEEMP.
The STEEMP that you are trying to exchange are locked into the smart contract. 

 - parameters:
	- symbol (string): symbol of the token you want to buy
	- quantity (number): quantity of tokens to buy
	- price (number): maximum number of STEEMP/token you are willing to pay

- example:
```
{
    "contractName": "market",
    "contractAction": "buy",
    "contractPayload": {
        "symbol": "TKN",
        "quantity": 100000,
        "price": 0.00001
    }
}
```
	
### sell: 
Sells tokens for STEEMP.
The tokens that you are trying to exchange are locked into the smart contract. 

 - parameters:
	- symbol (string): symbol of the token you want to sell
	- quantity (number): quantity of tokens to sell
	- price (number): minimum number of STEEMP/token you are willing to sell

- example:
```
{
    "contractName": "market",
    "contractAction": "sell",
    "contractPayload": {
        "symbol": "NKT",
        "quantity": 000,
        "price": 0.034
    }
}
```

### cancel: 
Cancels an order.
Will unlock any tokens that haven't been bought/sold.

 - parameters:
	- type (string: "buy"|"sell"): type of order
	- id (string): transaction id related to the order creation

- example:
```
{
    "contractName": "market",
    "contractAction": "cancel",
    "contractPayload": {
        "type": "sell",
        "id": "1f32152b0e82c4cc90a00ecfbad229a037435fe2-1"
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

### sellBook:
list of all the pending sell orders

-	fields:
	-	txId = transactionId

	- account = account owning the order

	- symbol = symbol of the token to exchange

	- quantity = remaining quantity of tokens to sell

	- price = price (STEEMP/token)