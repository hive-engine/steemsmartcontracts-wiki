
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
  - SwapStatusId (integer): Indicating the current status of the Swap Request 
    - 1 = Init, 2 = In Progress, 3 = Success, 4 = Failure, 5 = Awaiting token conversion, 6 = Converted tokens received
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

```
Deposit address: steem-engine
Memo: SWAP.BTS dswap 1cdc54b3-45b2-44fe-a254-912f1cfa7892
```

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

### SwapRequest (GET)
You can always pull the last status of a Swap Request by making the following request.

- endpoint: /SwapRequest/$id
- method: GET

 - request parameters:
	- id (string): Swap Request id returned when the swap request is created

- example request: 
```
{{ApiUrl}}/SwapRequest/60de18bc5f1d5568d94ac9d2
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
  - SwapStatusId (integer): Indicating the current status of the Swap Request 
    - 1 = Init, 2 = In Progress, 3 = Success, 4 = Failure, 5 = Awaiting token conversion, 6 = Converted tokens received
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

### Swap Request Transactions (GET)
A Swap request contains multiple swap request transactions including the initial deposit, first swap, second swap and finally ending with sending the output tokens to the user requesting the swap. You can retrieve all transactions within a swap request by making the following call.

- endpoint: /SwapRequest/SwapRequestTransactions/$id
- method: GET

 - request parameters:
	- id (string): Swap Request id returned when the swap request is created

- example request: 
```
{{ApiUrl}}/SwapRequest/60dce3d5da6234c4a8218faf
```

- response parameters, an array of Swap Request Transactions containing:
  - Id (string): The internal swap request transaction id
  - SwapRequestId (string): The Swap Request id
  - CreatedAt (string): Contains the datetime string when exactly the transaction is created
  - ModifiedAt (string): Contains the datetime string when the transaction is modified
  - Account (string): the account which is used in the transaction (can be the requestor or dswap account)
  - TokenInput (string): symbol of the token you want to sell
  - TokenInputAmount (decimal): quantity of tokens you want to sell
  - TokenOutput (string): symbol of the token you want to buy
  - TokenOutputAmount (decimal): quantity of tokens you want to buy
  - SwapStepId (integer): The step id of the current process (internal)
    - 5 = Validate, 20 = Convert to Base Token, 30 = Convert to Swap Output, 40 = Transfer to destination
  - ChainTransactionId (string): The transaction id of the transfer or swap captured in the chain
  - SwapStatusId (integer): Indicating the current status of the Swap Request 
    - 1 = Init, 2 = In Progress, 3 = Success, 4 = Failure, 5 = Awaiting token conversion, 6 = Converted tokens received
  - Message (string): Contains the failure reason if the swap has failed or any other message which is relevant to the swap.
  - JsonRequest (string): The transaction request captured in JSON format (can be verified on-chain)

```
[
    {
        "Id": "60dce4679f4ff49dcf23c34f",
        "SwapRequestId": "60dce3d5da6234c4a8218faf",
        "CreatedAt": "2021-06-30T21:38:47.977Z",
        "ModifiedAt": "2021-06-30T21:38:50.079Z",
        "Account": "dswap",
        "TokenInput": "PAL",
        "TokenInputAmount": 0.1,
        "TokenOutput": "DEC",
        "TokenOutputAmount": 0.605,
        "SwapStepId": 5,
        "ChainTransactionId": "d13175ff0818b6042db4d25a79084df9c4c413a0",
        "SwapStatusId": 3,
        "Message": "",
        "JsonRequest": "getTransactionInfo",
        "JsonResponseMainChain": null,
        "JsonResponseSideChain": null
    },
    {
        "Id": "60dce46a9f4ff49dcf23c350",
        "SwapRequestId": "60dce3d5da6234c4a8218faf",
        "CreatedAt": "2021-06-30T21:38:50.467Z",
        "ModifiedAt": "2021-06-30T21:39:02.164Z",
        "Account": "dswap",
        "TokenInput": "PAL",
        "TokenInputAmount": 0.1,
        "TokenOutput": "SWAP.HIVE",
        "TokenOutputAmount": 0.00220440,
        "SwapStepId": 20,
        "ChainTransactionId": "509e8ebd17000923501097c022d73573e7f0eb29",
        "SwapStatusId": 3,
        "Message": "",
        "JsonRequest": "{\"contractName\":\"market\",\"contractAction\":\"marketSell\",\"contractPayload\":{\"symbol\":\"PAL\",\"quantity\":\"0.1\",\"to\":null,\"memo\":null}}",
        "JsonResponseMainChain": null,
        "JsonResponseSideChain": null
    }
]
```

### Get Swap Requests of an account
You can retrieve all swap requests which are requested by an account. 

- endpoint: /SwapRequest
- method: GET

 - request parameters:
	- account (string): requests belonging to this account
	- limit (integer): how many requests you want to retrieve
	- offset (integer): optional, can be used for paging
	- status (integer): optional, can be used to retrieve swap requests with a certain status (success, failure etc.)
	  - 1 = Init, 2 = In Progress, 3 = Success, 4 = Failure, 5 = Awaiting token conversion, 6 = Converted tokens received

- example request: 
```
{{ApiUrl}}/SwapRequest?account=lion200&limit=10&offset=0&status=3
```

- example response:
```
[
    {
        "Id": "60dce4d59f4ff49dcf23c353",
        "CreatedAt": "2021-06-30T21:40:37.882Z",
        "ModifiedAt": "2021-06-30T21:41:20.538Z",
        "Account": "lion200",
        "TokenInput": "PAL",
        "TokenInputAmount": 0.100,
        "TokenOutput": "DEC",
        "TokenOutputAmount": 0.605,
        "TokenOutputAmountActual": 0.605,
        "SwapSourceId": "5fab0821cdef24759c5ae9a9",
        "ChainTransactionId": "4c7d8c4b94dc8c66e0aa114898d98db9138b6a07",
        "SwapStatusId": 3,
        "Chain": 1,
        "Message": null,
        "MaxSlippageInputToken": 0.0,
        "MaxSlippageOutputToken": 0.0,
        "BaseTokenAmount": 0.0022039908,
        "BaseTokenAmountActual": 0.00220440,
        "TokenInputMemo": ""
    },
    {
        "Id": "60dce3d5da6234c4a8218faf",
        "CreatedAt": "2021-06-30T21:36:19.867Z",
        "ModifiedAt": "2021-06-30T21:39:26.497Z",
        "Account": "lion200",
        "TokenInput": "PAL",
        "TokenInputAmount": 0.1,
        "TokenOutput": "DEC",
        "TokenOutputAmount": 0.605,
        "TokenOutputAmountActual": 0.605,
        "SwapSourceId": "5fab0821cdef24759c5ae9a9",
        "ChainTransactionId": "d13175ff0818b6042db4d25a79084df9c4c413a0",
        "SwapStatusId": 3,
        "Chain": 1,
        "Message": null,
        "MaxSlippageInputToken": 0.0,
        "MaxSlippageOutputToken": 0.0,
        "BaseTokenAmount": 0.0022044,
        "BaseTokenAmountActual": 0.00220440,
        "TokenInputMemo": ""
    }
]
```

### Swap Request Count
If you want to implement paging, or just want to know the counts of the swap requests, you can call the following method to retrieve the swap request counts.

- endpoint: /SwapRequest/SwapRequestCount/$account/$status
- method: GET

 - request parameters:
	- account (string): requests belonging to this account
	- status (integer): optional, can be used to retrieve swap requests with a certain status (success, failure etc.)
	  - 1 = Init, 2 = In Progress, 3 = Success, 4 = Failure, 5 = Awaiting token conversion, 6 = Converted tokens received

- example request: 
```
{{ApiUrl}}/SwapRequest/SwapRequestCount/lion200/3
```

- example response:
```
88
```
