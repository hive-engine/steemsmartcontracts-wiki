## 1.  Restrict actions to the owner of the Smart Contract only
There is a global variable called "owner" that holds the id of the owner of the Smart Contract. By using this variable, you can restrict certain actions to the owner exclusively.

For example:
 ```js
actions.removeUser = function (payload) {
  if (sender !== owner) return;

  const { userId } = payload;

  if (userId && typeof userId === 'string') {
    let users = db.getTable('users');
    let user = users.findOne({ 'id': userId });
    if (user)
      users.remove(user);
  }
}
```



## 2.  Execute a Smart Contract from another Smart Contract
From a Smart Contract, you can call another Smart Contract simply by using the following global function:

`executeSmartContract(contract: string, action: string, payload: JSON stringified)`

The Smart Contract will be executed with the same sender as the current sender.

 For example:
 ```js
actions.addUser = function (payload) {
  let users = db.getTable('users');

  const newUser = {
    'id': sender
  };

  users.insert(newUser);

  executeSmartContract('books_contract', 'addBook', '{ "title": "The Awesome Book" }')
}
```

## 3.  Check funds sent to a Smart Contract via a Steem transfer operation
When a transaction is sent via a Steem transfer you can access to two variables in the payload of the action:

- recipient: give you the username Steem of the recipient
- amountSTEEMSBD: give you the amount of STEEM or SBD sent (ie. "10.000 STEEM" or "10.000 SBD")

## 4.  Check type of signature for custom_json operation
When a transaction is sent via a custom_json operation you can access to the following variable via the payload:

- isSignedWithActiveKey: true if the custom_json was signed with the active key of the account, false otherwise
