# DSwap DCA Api methods

# Table of Contents

* [SwapRequestDCA (MARKET)](#swaprequestdca)
* [CancelSwapRequestDCA](#cancelswaprequestdca)
* [SwapDCARequests GET](#swapdcarequests-get)
* [SwapDCADetail GET](#swapdcadetail-get)
* [DCACancelRequests GET](#dcacancelrequests-get)
  
## API urls:
QA: https://qa-dswap-api.dswap.trade/api/

PROD: https://dswap-api.dswap.trade/api/

## Actions available:

### SwapRequestDCA: 
This method is the call to submit the DCA request to initiate a market dca. 

- endpoint: /SwapRequest/DCA?api-version=3.0
- method: POST

 - request parameters:
	- Account (string): the account creating the swap dca request
	- TokenInput (string): symbol of the token you want to sell	
	- TokenInputAmount (decimal): quantity of tokens you want to sell
	- TokenOutput (string): symbol of the token you want to buy
	- SwapSourceId (string): will be used in the future to identify the source of the swap request. Currently single source.
	- Chain (integer): Id of the Chain (1 = HIVE)
	- RecurrenceType (string): One of the following values: "minute", "hour", "day", "week", "month"
	- RecurrenceTypeAmount (integer): 1 for every minute/hour/day/week/month if you wish to do 1 swap every minute/hour/day/week/month. 
	- OrderCount (integer): Number of orders to be created for the DCA request. Between 2 and 20 for now.
	- TokenInputMemo (string): Needs to contain a GUID which matches the GUID passed during transfer of funds in the chain.

#### Swap Request DCA Hive Engine Tokens
- example swap dca request Hive Engine tokens: 
```
{    
    "Account": "lion200",
    "TokenInput": "DEC",
    "TokenInputAmount": 200,
    "TokenOutput": "BEE",
    "SwapSourceId": "5fab0821cdef24759c5ae9a9",
    "Chain": 1,
	"RecurrenceType": "minute",
	"RecurrenceTypeAmount": 1,
	"OrderCount": 10,
    "TokenInputMemo": "{{GUID}}"	
}
```

- response parameters:
  - Id (string): The swap dca request id which can be used to retrieve last statuses and the corresponding transactions
  - Account (string): the account creating the swap request
  - TokenInput (string): symbol of the token you want to sell	
  - TokenInputAmount (decimal): quantity of tokens you want to sell
  - TokenOutput (string): symbol of the token you want to buy  
  - SwapSourceId (string): will be used in the future to identify the source of the swap request. Currently single source.
  - ChainTransactionId: if the swap is between two hive-engine tokens, this contains the transaction id of the transaction where input token is sent to the Dswap account  
  - Chain (integer): Id of the Chain (1 = HIVE)  
  - TokenInputMemo (string): Contains given token deposit input memo
  - RecurrenceType (string): One of the following values: "minute", "hour", "day", "week", "month"
  - RecurrenceTypeAmount (integer): 1 for every minute/hour/day/week/month if you wish to do 1 swap every minute/hour/day/week/month. Max value = 20 for now
  - OrderCount (integer): Number of orders to be created for the DCA request. Between 2 and 20 for now.
	
- example response: 
```
{
    "Id": "60de18bc5f1d5568d94ac9d2",
    "Account": "lion200",
    "TokenInput": "DEC",
    "TokenInputAmount": 200,
    "TokenOutput": "BEE",
    "SwapSourceId": "5fab0821cdef24759c5ae9a9",
    "ChainTransactionId": "4c7d8c4b94dc8c66e0aa114898d98db9138b6a07",    
    "Chain": 1,
	"RecurrenceType": "minute",
	"RecurrenceTypeAmount": 1,
	"OrderCount": 10,
    "TokenInputMemo": "{{GUID}}"
}
```

### CancelSwapRequestDCA: 
This method is the call to cancel an initialized or in-progress DCA request. 

- endpoint: /SwapRequest/DCACancel?api-version=3.0
- method: POST

 - request parameters:
	- Account (string): the account for which the DCA request is in progress
	- Chain (integer): Id of the Chain (1 = HIVE)
	- ChainTransactionId: if the swap is between two hive-engine tokens, this contains the transaction id of the transaction where input token is sent to the Dswap account  
	- DCAId (string): The Id of the DCA request to be cancelled
	- SourceId (string): will be used in the future to identify the source of the swap request. Currently single source.
	- TokenInputMemo (string): Needs to contain a GUID which matches the GUID passed during transfer of funds in the chain to cancel the DCA. 
	- Message (string): Leave empty. Used for additional info.

#### Cancel Swap Request DCA Hive Engine Tokens
- example cancel dca request Hive Engine tokens: 
```
{    
    "Account": "lion200",    
    "Chain": 1,
	"ChainTransactionId": "",
	"DCAId": "12dasdhvhnwj333",
	"SourceId": "5fab0821cdef24759c5ae9a9",		
    "TokenInputMemo": "{{GUID}}",
	"Message": "",
}
```

- response parameters:
  - Id (string): The swap dca request id which can be used to retrieve last statuses and the corresponding transactions
  - Account (string): the account creating the swap dca request
  - Chain (integer): Id of the Chain (1 = HIVE)  
  - ChainTransactionId (string): if the swap is between two hive-engine tokens, this contains the transaction id of the transaction where input token is sent to the Dswap account  
  - CreatedAt (Datetime): The date time value of the creation dat of the dca request.    
  - DCAId (string): Id of the DCA request to be cancelled
  - SourceId (string): will be used in the future to identify the source of the swap request. Currently single source.  
  - StatusId (integer): Indicating the current status of the DCA Request 
    - 1 = Init, 2 = In Progress, 3 = Success, 4 = Failure, 7 = Expired, 9 = Cancelled, 10 = SuccessPartial, 11 = CancelRequested
  - TokenInputMemo (string): Contains given token deposit input memo  
	
- example response: 
```
{
    "Id": "60de18bc5f1d5568d94ac9d2",
    "Account": "lion200",
    "Chain": 1,
    "ChainTransactionId": "",
	"DCAId": "12dasdhvhnwj333",
    "CreatedAt": "2021-07-01T19:34:18.6540055+00:00",
    "SourceId": "5fab0821cdef24759c5ae9a9",
    "StatusId": 9,	
    "TokenInputMemo": "{{GUID}}"
}
```

### SwapDCARequests GET
Retrieve all DCA requests for given Account and provided filter values.

- endpoint: /SwapRequest/DCARequests?account=$account
- method: GET

 - request parameters:
	- account (string): Account name for which the dca requests need to be retrieved
	- limit (number): how many dca requests to be retrieved
	- offset (number): provide an offset to implement paging
	- statuses (string): provide statuses values if you want to retrieve dca requests having certain statuses
	- api-version (string): provide the version of the api you want to use. eg. "3.0"

- example request: 
```
{{ApiUrl}}/SwapRequest/DCARequests?account=lion200&limit=20&offset=&statuses=1&statuses=3&statuses=5&api-version=3.0
```

- response parameters:
  - Id (string): The dca request id
  - Account (string): the account creating the swap request
  - Chain (integer): Id of the Chain (1 = HIVE)
  - ChainTransactionId (string): if the swap is between two hive-engine tokens, this contains the transaction id of the transaction where input token is sent to the Dswap account  
  - CreatedAt (string): Contains the datetime string when exactly the swap request is created  
  - TokenInput (string): symbol of the token you want to sell	
  - TokenInputAmount (decimal): quantity of tokens you want to sell
  - TokenOutput (string): symbol of the token you want to buy
  - TokenOutputAmountActual (decimal): the actual quantity of tokens bought with the dca. 
  - SwapSourceId (string): will be used in the future to identify the source of the swap request. Currently single source.  
  - SwapStatusId (integer): Indicating the current status of the Swap Request 
    - 1 = Init, 2 = In Progress, 3 = Success, 4 = Failure, 7 = Expired, 9 = Cancelled, 10 = SuccessPartial, 11 = CancelRequested  
  - TokenInputMemo (string): Contains given token deposit input memo
  - RecurrenceType (string): One of the following values: "minute", "hour", "day", "week", "month"
  - RecurrenceTypeAmount (integer): 1 for every minute/hour/day/week/month if you wish to do 1 swap every minute/hour/day/week/month. Max value = 20 for now
  - OrderCount (integer): Number of orders to be created for the DCA request. Between 2 and 20 for now.
  
	
- example response: 
```
{
    "Id": "60de18bc5f1d5568d94ac9d2",
    "CreatedAt": "2021-07-01T19:34:18.6540055+00:00",
    "ModifiedAt": null,
    "Account": "lion200",
    "TokenInput": "DEC",
    "TokenInputAmount": 200,
    "TokenOutput": "BEE",
    "TokenOutputAmountActual": 0.5,
    "SwapSourceId": "5fab0821cdef24759c5ae9a9",
    "ChainTransactionId": "4c7d8c4b94dc8c66e0aa114898d98db9138b6a07",
    "SwapStatusId": 1,
    "Chain": 1,
    "RecurrenceType": "minute",
	"RecurrenceTypeAmount": 1,
	"OrderCount": 10,
    "TokenInputMemo": "{{GUID}}"
}
```

### SwapDCADetail GET
You can retrieve the details of a DCA request, swap requests happened as part of the DCA and refunds as part of the DCA if any

- endpoint: /SwapRequest/DCADetail/$id
- method: GET

 - request parameters:
	- id (string): id of the DCA request

- example request: 
```
{{ApiUrl}}/SwapRequest/DCADetail/1asdasd2313a
```

- example response:
```
{
	"SwapRequestDCA": 
    {
        "Id": "60de18bc5f1d5568d94ac9d2",
		"Account": "lion200",
		"TokenInput": "DEC",
		"TokenInputAmount": 200,
		"TokenOutput": "BEE",
		"SwapSourceId": "5fab0821cdef24759c5ae9a9",
		"ChainTransactionId": "4c7d8c4b94dc8c66e0aa114898d98db9138b6a07",    
		"Chain": 1,
		"RecurrenceType": "minute",
		"RecurrenceTypeAmount": 1,
		"OrderCount": 10,
		"TokenInputMemo": "{{GUID}}"
    },
	"SwapRequests": [
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
	],
	"DCARefunds": [
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
	]
}
```

### DCACancelRequests GET
Retrieve all DCA cancel requests submitted for given account and status

- endpoint: /SwapRequest/DCACancelRequests?account=$account
- method: GET

 - request parameters:
	- account (string): account name
	- status (integer): status of the dca cancel requests (Init (1) if just started)

- example request: 
```
{{ApiUrl}}/SwapRequest/DCACancelRequests?account=$account&status=1&api-version=3.0
```

- response parameters, an array of Swap Request Transactions containing:
  - Id (string): The internal swap request transaction id
  - DCAId (string): The Swap Request id
  - CreatedAt (string): Contains the datetime string when exactly the transaction is created
  - DCAId (string): Id of the DCA request which is requested to be cancelled
  - Account (string): the account which is used in the transaction (can be the requestor or dswap account)
  - ChainTransactionId (string): The transaction id of the transfer or swap captured in the chain
  - SwapSourceId (string): will be used in the future to identify the source of the swap request. Currently single source.  
  - StatusId (integer): Indicating the current status of the Swap Request 
    - 1 = Init, 2 = In Progress, 3 = Success, 4 = Failure, 5 = Awaiting token conversion, 6 = Converted tokens received
  - Message (string): Contains the failure reason if the swap has failed or any other message which is relevant to the swap.
  - TokenInputMemo (string): Memo provided in the original DCA request

```
[
    {
        "Id": "60dce4679f4ff49dcf23c34f",
        "DCAId": "60dce3d5da6234c4a8218faf",
        "CreatedAt": "2021-06-30T21:38:47.977Z",
        "ModifiedAt": "2021-06-30T21:38:50.079Z",
        "Account": "dswap",
        "ChainTransactionId": "d13175ff0818b6042db4d25a79084df9c4c413a0",
        "SwapStatusId": 3,
        "Message": "",
        "TokenInputMemo": "getTransactionInfo"
    },
    {
        "Id": "60dce46a9f4ff49dcf23c350",
        "DCAId": "60dce3d5da6234c4a8218faf",
        "CreatedAt": "2021-06-30T21:38:47.977Z",
        "ModifiedAt": "2021-06-30T21:38:50.079Z",
        "Account": "dswap",
        "ChainTransactionId": "d13175ff0818b6042db4d25a79084df9c4c413a0",
        "SwapStatusId": 3,
        "Message": "",
        "TokenInputMemo": "getTransactionInfo"
    }
]
```
