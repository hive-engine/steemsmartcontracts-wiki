
# DSwap Api methods

## Actions available:
### CalculateSwapOutput: 
Calculates the output token (the token you want to buy) amount and the conversion rate (in SWAP.HIVE) based on given input token (token you want to sell) and input token amount.

- endpoint: /SwapRequest/CalculateSwapOutput
- method: POST

 - request parameters:
	- TokenInput (string): symbol of the token you want to sell
	- TokenInputAmount (decimal): quantity of tokens you want to sell
	- TokenOutput (string): symbol of the token you want to buy
	- Chain (integer): Id of the Chain (1 = HIVE)

- example request: 
```
{
    TokenInput: "PAL",
    TokenInputAmount: 10,
    TokenOutput: "DEC",
    Chain: 1
}
```

- response parameters:
  - TokenInput (string): symbol of the token you want to sell
  - TokenInputAmount (decimal): quantity of tokens you want to sell
  - TokenOutput (string): symbol of the token you want to buy
  - TokenOutputAmount (decimal): calculated amount of tokens you can buy based on given input token amount
  - BaseTokenAmount (decimal): conversion rate in SWAP.HIVE (a swap consists basically of 2 swaps, 1: from INPUT --> SWAP.HIVE, 2: from SWAP.HIVE --> OUTPUT)
  - Chain (integer): Id of the Chain (1 = HIVE)
	
- example response: 
```
{
    "TokenInput": "PAL",
    "TokenInputAmount": 10.0,
    "TokenOutput": "DEC",
    "TokenOutputAmount": 42.857513122556102498795908538,
    "BaseTokenAmount": 0.21000010,
    "Chain": 1
}
```

### CalculateSwapInput: 
Calculates the input token (the token you want to sell) amount and the conversion rate (in SWAP.HIVE) based on given output token (token you want to buy) and output token amount.

- endpoint: /SwapRequest/CalculateSwapInput
- method: POST

 - request parameters:
	- TokenInput (string): symbol of the token you want to sell	
	- TokenOutput (string): symbol of the token you want to buy
	- TokenOutputAmount (decimal): quantity of tokens you want to buy
	- Chain (integer): Id of the Chain (1 = HIVE)

- example request: 
```
{
    TokenInput: "PAL",
    TokenOutput: "DEC",
    TokenOutputAmount: 42.857,
    Chain: 1
}
```

- response parameters:
  - TokenInput (string): symbol of the token you want to sell
  - TokenInputAmount (decimal): calculated amount of tokens you need to sell to buy given amount of output token amount
  - TokenOutput (string): symbol of the token you want to buy
  - TokenOutputAmount (decimal): quantity of tokens you want to buy
  - BaseTokenAmount (decimal): conversion rate in SWAP.HIVE (a swap consists basically of 2 swaps, 1: from INPUT --> SWAP.HIVE, 2: from SWAP.HIVE --> OUTPUT)
  - Chain (integer): Id of the Chain (1 = HIVE)
	
- example response: 
```
{
    "TokenInput": "PAL",
    "TokenInputAmount": 10.0000000,
    "TokenOutput": "DEC",
    "TokenOutputAmount": 42.857,
    "BaseTokenAmount": 0.21000010,
    "Chain": 1
}
```

### SwapRequest: 
This method is the call to the actual SWAP. 

- endpoint: /SwapRequest
- method: POST

 - request parameters:
  - Account (string): the account creating the swap request
	- TokenInput (string): symbol of the token you want to sell	
	- TokenInputAmount (decimal): quantity of tokens you want to sell
	- TokenOutput (string): symbol of the token you want to buy
	- TokenOutputAmount (decimal): quantity of tokens you want to buy
	- SwapSourceId (string): will be used in the future to identify the source of the swap request. Currently single source.
	- ChainTransactionId: 
	  - if the swap is between two hive-engine tokens, this must contain the transaction id of the transaction where input token is sent to the Dswap account
	  - if the input token is a crypto token (like BTC, LTC, EOS, STEEM etc.) this field is not required
	- Chain (integer): Id of the Chain (1 = HIVE)
	- MaxSlippageInputToken (decimal): Given maximum slippage for the first swap from INPUT TOKEN --> SWAP.HIVE
	- MaxSlippageOutputToken (decimal): Given maximum slippage for the second swap from SWAP.HIVE --> OUTPUT TOKEN
	- BaseTokenAmount (decimal): The calculated conversion rate retrieved from CalculateSwapOutput or CalculateSwapInput methods at the time of the Swap request
	- TokenInputMemo (string): Needs to contain a GUID in case the input token is a crypto token, in order to identify the swap after user manually makes the deposit using this generated GUID as memo. Below you can find different example requests clarifying this.

- example swap request Hive Engine tokens: 
```
{    
    "Account": "lion200",
    "TokenInput": "PAL",
    "TokenInputAmount": 0.100,
    "TokenOutput": "DEC",
    "TokenOutputAmount": 0.605,
    "SwapSourceId": "5fab0821cdef24759c5ae9a9",
    "ChainTransactionId": "4c7d8c4b94dc8c66e0aa114898d98db9138b6a07",
    "Chain": 1,
    "MaxSlippageInputToken": 0,
    "MaxSlippageOutputToken": 0,
    "BaseTokenAmount": 0.0022039908,
    "TokenInputMemo": ""
}
```

- response parameters:
  - Id (string): The swap request id which can be used to retrieve last statuses and the corresponding transactions
  - CreatedAt (string): Contains the datetime string when exactly the swap request is created
  - ModifiedAt (string): Contains the datetime string when  the swap request is modified
  - Account (string): the account creating the swap request
	- TokenInput (string): symbol of the token you want to sell	
	- TokenInputAmount (decimal): quantity of tokens you want to sell
	- TokenOutput (string): symbol of the token you want to buy
	- TokenOutputAmount (decimal): quantity of tokens you want to buy
	- TokenOutputAmountActual (decimal): the actual quantity of tokens bought with the swap. this can be different to the TokenOutputAmount value based on given MaxSlippageOutputToken value.
	- SwapSourceId (string): will be used in the future to identify the source of the swap request. Currently single source.
	- ChainTransactionId: 
	  - if the swap is between two hive-engine tokens, this must contain the transaction id of the transaction where input token is sent to the Dswap account
	  - if the input token is a crypto token (like BTC, LTC, EOS, STEEM etc.) this field contains the transaction id of the pegged token deposit transaction
	- Chain (integer): Id of the Chain (1 = HIVE)
	- Message: Contains the failure reason if the swap has failed or any other message which is relevant to the swap.
	- MaxSlippageInputToken (decimal): Given maximum slippage for the first swap from INPUT TOKEN --> SWAP.HIVE
	- MaxSlippageOutputToken (decimal): Given maximum slippage for the second swap from SWAP.HIVE --> OUTPUT TOKEN
	- BaseTokenAmount (decimal): The calculated conversion rate retrieved from CalculateSwapOutput or CalculateSwapInput methods at the time of the Swap request
	- BaseTokenAmountActual (decimal): The actual conversion rate used in the swap. Can be different from BaseTokenAmount value based on given MaxSlippageInputToken value.
	- TokenInputMemo (string): Contains given token deposit input memo
	
- example response: 
```
{
    "Id": "60de18bc5f1d5568d94ac9d2",
    "CreatedAt": "2021-07-01T19:34:18.6540055+00:00",
    "ModifiedAt": null,
    "Account": "lion200",
    "TokenInput": "PAL",
    "TokenInputAmount": 0.100,
    "TokenOutput": "DEC",
    "TokenOutputAmount": 0.605,
    "TokenOutputAmountActual": 0.0,
    "SwapSourceId": "5fab0821cdef24759c5ae9a9",
    "ChainTransactionId": "4c7d8c4b94dc8c66e0aa114898d98db9138b6a07",
    "SwapStatusId": 1,
    "Chain": 1,
    "Message": null,
    "MaxSlippageInputToken": 0.0,
    "MaxSlippageOutputToken": 0.0,
    "BaseTokenAmount": 0.0022039908,
    "BaseTokenAmountActual": 0.0,
    "TokenInputMemo": ""
}
```
