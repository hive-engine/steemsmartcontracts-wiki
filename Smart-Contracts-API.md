## 1.  Restrict actions to the owner of the Smart Contract only
There is a global variable called "owner" that holds the id of the owner of the Smart Contract. By using this variable you can restrict certain actions to the owner exclusively:
  example:
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
From a Smart Contract you can call another Smart Contract simply by using the following global function:

`executeSmartContract(contract: string, action: string, payload: JSON stringified)`

The Smart Contract will be executed with the same sender as the current sender

  example:
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