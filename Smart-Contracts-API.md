The Smart Contract API is available under the "api" global object


## 1.  Restrict actions to the owner of the Smart Contract only
There is a variable called "owner" that holds the id of the owner of the Smart Contract. By using this variable, you can restrict certain actions to the owner exclusively.

For example:
 ```js
actions.removeUser = async (payload) => {
  if (api.sender !== api.owner) return;

  const { userId } = payload;

  if (userId && typeof userId === 'string') {
    let user = await api.db.findOne('users', { 'id': userId });
    if (user)
      await api.db.remove('users', user);
  }
}
```



## 2.  Execute a Smart Contract from another Smart Contract
From a Smart Contract, you can call another Smart Contract simply by using the following global functions:

`executeSmartContract(contract: string, action: string, payload: JSON stringified)`

The Smart Contract will be executed with the same sender as the current sender.

 For example:
 ```js
actions.addUser = async (payload) => {
  const newUser = {
    'id': sender
  };

  await api.db.insert('users', newUser);

  await api.executeSmartContract('books_contract', 'addBook', '{ "title": "The Awesome Book" }')
}

```

`executeSmartContractAsOwner(contract: string, action: string, payload: JSON stringified)`

The Smart Contract will be executed with the sender set as the current contract owner.

 For example:
 ```js
actions.transferTokensToUser = async (payload) => {
  const { to } = payload;
  await api.executeSmartContractAsOwner('tokens', 'transfer', { symbol: 'TKN', quantity: 123.456, to });
}
```

## 3.  Transfer tokens from a smart contract to a user or another contract (available only if the "tokens" smart contract has been deployed)
Smart Contracts have the ability to hold tokens that can only be moved within the Smart Contract itself.
This can be done via the global function:

`transferTokens(to: string, symbol: string, quantity: number, type: string ('user' | 'contract'))`

The transfer will be executed with the 'null' account set as the sender.

 For example:
 ```js
actions.sendRewards = async function (payload) {

    const { to, quantity } = payload;

    await api.transferTokens(to, 'TKN', quantity, 'user');

}
```

## 4.  Check funds sent to a Smart Contract via a Steem transfer operation
When a transaction is sent via a Steem transfer you can access to two variables in the payload of the action:

- recipient: give you the username Steem of the recipient
- amountSTEEMSBD: give you the amount of STEEM or SBD sent (ie. "10.000 STEEM" or "10.000 SBD")

## 5.  Check type of signature for custom_json operation
When a transaction is sent via a custom_json operation you can access to the following variable via the payload:

- isSignedWithActiveKey: true if the custom_json was signed with the active key of the account, false otherwise

## 6.  Steem reference block number
The Steem block number from which the transaction was initiated is available via the following variable:
- api.refSteemBlockNumber: Steem block number

## 7.  Steem transaction id
The transaction id from which the transaction was initiated is available via the following variable:
- api.transactionId: transaction id

## 8. deterministic random number generation
A deterministic random number generator is available via the following function:
- api.random(): returns a random number between 0 and 1 based on the Steem transaction hash, the Steem block hash and the previous Steem block hash

## 9. contract version
The contract version is available via the following variable:
- api.contractVersion: integer that represents the version of the contract (every time a contract is updated, the version is increased by 1 (the first version being 1)

## 10. calling contract information
If the contract action has been called by another contract you'll have the following object available in the payload:
- callingContractInfo: object holding information about the contract that called the contract action
 For example:
 ```js
actions.sendRewards = async function (payload) {

    const { to, quantity, callingContractInfo } = payload;

    if (callingContractInfo.name === 'tokens' && callingContractInfo.version === 3) {
        await api.transferTokens(to, 'TKN', quantity, 'user');
    }
}
```

## Additional libraries available via the smart contracts API
 ### BigNumber.js:
 - keyword: api.BigNumber
 - link to the library's code source: http://mikemcl.github.io/bignumber.js/
 - purpose: make the management of numbers easier as javascript has issues with dealing with decimals

### validator.js:
-  keyword: api.validator
 - link to the library's code source: https://github.com/chriso/validator.js
 - purpose: allows you to perform validations within a smart contract as the Regular Expression are disabled