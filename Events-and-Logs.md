When you execute a Smart Contract you are not able to return results however you are able to emit events that will be stored in the transaction, errors generated during the deployment or the execution of a Smart Contract are logged in the transaction as well.

These information are stored under the key "logs" of a transaction.

Examples of transactions that have events or errors:

```js
Transaction {
  refBlockNumber: 123456789,
  transactionId: 'TXID1234',
  sender: 'harpagon',
  contract: 'contract',
  action: 'deploy',
  payload:
   '{"name":"test_contract","params":"","code":"CiAgICAgIGFjdGlvbnMuY3JlYXRlID0gZnVuY3Rpb24gKHBheWxvYWQpIHsKICAgICAgICAvLyBJbml0aWFsaXplIHRoZSBzbWFydCBjb250cmFjdCB2aWEgdGhlIGNyZWF0ZSBhY3Rpb24KICAgICAgICBlbWl0KCdjb250cmFjdF9jcmVhdGUnLCB7ICJjb250cmFjdE5hbWUiOiAidGVzdF9jb250cmFjdCIgfSkKICAgICAgfQogICAg"}',
  hash:
   'f5694249af602a1d0b3f3b4855f4b1766eb01059b00022c3a7f202c2a3ab87e8',
  logs:
   '{"events":[{"event":"contract_create","data":{"contractName":"test_contract"}}]}' 
}
```

```js
Transaction {
  refBlockNumber: 123456789,
  transactionId: 'TXID1234',
  sender: 'harpagon',
  contract: 'contract',
  action: 'deploy',
  payload:
   '{"name":"test_contract","params":"","code":"..."}',
  hash:
   '5c39a8878244d0b71b1030e5f1b89eb0e0310cdc0d40b4b8f7064334c52ed79d',
  logs:
   '{"error":{"name":"SyntaxError","message":"Unexpected identifier"}}' 
}
```



## Emit an event from a Smart Contract
Emit an event from a Smart Contract is pretty straight forward and is performed by using the global function:

`emit(event: string, data: JSON obj)`

  example:
 ```js
actions.create = function (payload) {
  // Initialize the smart contract via the create action
  emit('contract_create', { "contractName": "test_contract" })
}
```