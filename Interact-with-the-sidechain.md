
There are two ways to interact with the sidechain:
### 1) via custom_json
Send a custom_json operation to the Steem blockchain with the following parameters:

id: "ssc-CHAIN-ID"

json: 
```js
{

"contractName": "NAME OF THE CONTRACT",

"contractAction": "ACTION OF THE CONTRACT TO PERFORM",

"contractPayload": { OBJECT THAT WILL BE PASSED TO THE CONTRACT ACTION }

}
```

### 2) via transfer
Transfer funds with the following json as memo:

```js
{

"id": "ssc-CHAIN-ID",

"json": {

"contractName": "NAME OF THE CONTRACT",

"contractAction": "ACTION OF THE CONTRACT TO PERFORM",

"contractPayload": { OBJECT  THAT  WILL  BE  PASSED  TO  THE  CONTRACT  ACTION }

}

}
```

Important: CHAIN-ID is used to identify to which sidechain you want to send the transaction, the sidechain id is defined in the config.json file of the nodes