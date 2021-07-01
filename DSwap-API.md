
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

#### Swap Request Hive Engine Tokens
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

#### Swap Request BTC, LTC, BCH, DOGE, SWIFT (BTC forked tokens)
An important notice if the INPUT TOKEN is a BTC-forked crypto token is that the Deposit address for the INPUT TOKEN must be retrieved from the converter endpoint for the DSwap account. IMPORTANT: You need to send the dswap account name as "destination" parameter. Because that's the account being used to retrieve and convert deposits.

- example retrieving deposit address from the converter:
```
const request = await this.http.fetch(`${environment.CONVERTER_API_HE}convert/`, {
	method: 'POST',
	body: json({ 
		from_coin: 'BTC', 
		to_coin: 'SWAP.BTC', 
		destination: 'dswap' }) 
});

const response = await request.json();
```
The response of the converter contains a uniquely generated address to which the user needs to deposit the BTC, LTC etc. token:
```
response:
{
	"ex_rate":1.0,
	"destination":"dswap",
	"pair":"BTC -> SWAP.BTC (1.0000 SWAP.BTC per BTC)",
	"address":"bc1qs7eu9m3feu2e5gsnjtjp5wnj8yyv7kg6lmrs88"
}
```

- The swap request for this SWAP from BTC --> DEC would look like:
```
{    
    "Account": "lion200",
    "TokenInput": "BTC",
    "TokenInputAmount": 10,
    "TokenOutput": "DEC",
    "TokenOutputAmount": 1764507.455,
    "SwapSourceId": "5fab0821cdef24759c5ae9a9",
    "ChainTransactionId": "", 
    "Chain": 1,
    "MaxSlippageInputToken": 5,
    "MaxSlippageOutputToken": 5,
    "BaseTokenAmount": 6450.599625154223,
    "TokenInputMemo": "bc1qs7eu9m3feu2e5gsnjtjp5wnj8yyv7kg6lmrs88" -- This is the unique deposit address generated & retrieved from the converter API endpoint above.
}
```

#### Swap Request BTS, EOS
For BitShares EOS like crypto tokens the converter API is returning a generic deposit address where the user can make his/her deposit.
In order to make this deposit unique and match this deposit to a swap request, you need to add a unique identifier in the MEMO of your deposit.

- For BTS and EOS, the crypto converter request looks like:
```
const request = await this.http.fetch(`${environment.CONVERTER_API_HE}convert/`, {
	method: 'POST',
	body: json({ 
		from_coin: 'BTS', 
		to_coin: 'SWAP.BTS', 
		destination: 'dswap' }) 
});

const response = await request.json();
```
And the response looks like:
```
Response: 
{
	"ex_rate":1.0,
	"destination":"dswap",
	"pair":"BTS -> SWAP.BTS (1.0000 SWAP.BTS per BTS)",
	"memo":"SWAP.BTS dswap",
	"account":"steem-engine"
}
```

As you can see in the response of the converter. The deposit needs to happen to "steem-engine" address with memo "SWAP.BTS dswap".
Because this memo is very generic and we actually need to make this deposit unique to the Swap Requestor, the memo should contain a GUID (unique identifier), which should be looking like: 

Deposit address: steem-engine

Memo: SWAP.BTS dswap **1cdc54b3-45b2-44fe-a254-912f1cfa7892**

This unique identifier also needs to be provided to the Swap Request as **TokenInputMemo** which would result in the following request:
```
{
	"Account":"lion200",
	"Chain":1,
	"ChainTransactionId":"",
	"TokenOutput":"DEC",
	"TokenOutputAmount":5.944,
	"TokenInput":"BTS",
	"TokenInputAmount":1,
	"SwapSourceId":"5fab0821cdef24759c5ae9a9",
	"BaseTokenAmount":0.02,
	"MaxSlippageInputToken":5,
	"MaxSlippageOutputToken":5,
	"TokenInputMemo":"1cdc54b3-45b2-44fe-a254-912f1cfa7892" -- Unique identifier
}
```

After creating the swap request, the user still needs to deposit BTS by using the following details obviously: 
```
Deposit address: steem-engine (Retrieved from the Converter API)
Memo: SWAP.BTS dswap 1cdc54b3-45b2-44fe-a254-912f1cfa7892
```

#### Swap Request HBD, SBD, STEEM
The swap request requirements for HBD, SBD, STEEM are comparable to the swap request requirements for BTS and EOS.
The only difference is that the deposit address for BTS and EOS retrieved from the Converter API is **steem-engine** and for HBD, SBD and STEEM this deposit address is **graphene-swap**.

So the Swap Request paramters are just like the request for BTS and EOS containing a unique identifier given in TokenInputMemo field.
The only difference is that the user needs to manually deposit HBD, SBD or STEEM using following details:

```
Deposit address: graphene-swap (Retrieved from the Converter API)
Memo: SWAP.STEEM dswap 785d9da6-b3f2-4732-8dd2-672131ee9991
```
