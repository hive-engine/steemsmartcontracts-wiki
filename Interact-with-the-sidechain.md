
There are two ways to interact with the sidechain:
### 1) via custom_json
Send a custom_json operation to the Steem blockchain with the following parameters:

id: "ssc"
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

"id": "ssc",

"json": {

"contractName": "NAME OF THE CONTRACT",

"contractAction": "ACTION OF THE CONTRACT TO PERFORM",

"contractPayload": { OBJECT  THAT  WILL  BE  PASSED  TO  THE  CONTRACT  ACTION }

}

}
```