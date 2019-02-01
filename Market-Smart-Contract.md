# Market contract

## Actions available:
### buy: 
Buys STEEMP with tokens.
The tokens that you are trying to exchange are locked into the smart contract. 

 - parameters:
	- symbol (string): symbol of the token you want to buy the STEEMP with
	- quantity (number): quantity of STEEMP to buy
	- price (number): number of tokens / STEEMP

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
Sells STEEMP for tokens.
The STEEMP that you are trying to exchange are locked into the smart contract. 

 - parameters:
	- symbol (string): symbol of the token you want to sell the STEEMP for
	- quantity (number): quantity of STEEMP to sell
	- price (number): number of tokens / STEEMP

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

	- quantity = quantity of STEEMP to buy

	- price = price (token/STEEMP)

	- tokensLocked = number of remaining tokens that are locked into the smart contract

### sellBook:
list of all the pending sell orders

-	fields:
	-	txId = transactionId

	- account = account owning the order

	- symbol = symbol of the token to exchange

	- quantity = quantity of STEEMP to sell

	- price = price (token/STEEMP)