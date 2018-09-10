You can find all the database related functions under the global variable "db"

## 1.  Create a table
 You can create a table that will be available for your Smart Contract only during the deployment of the Smart Contract. The method that we will be using is:
 
 `createTable(tableName: string)`
 
 example:
 ```js
actions.create = function (payload) {
	// let's create a table called users
	db.createTable('users');
}
```

 ## 2.  Get a table
 You can retrieve a table that was initialized during the deployment of the Smart Contract by using the "db" object available as a global variable. The method that we will be using is:

 `getTable(tableName: string) returns null if the table doesn't exist`
 
  example:
 ```js
actions.addUser = function (payload) {
	// let's call the table called users
	const users = db.getTable('users');
}
```

 ## 3.  Add a record in a table
Once you retrieve a table via the getTable method, you can perform actions on it similarly to MongoDB. So to add a record, we will be using:

`insert(record: object)`
 
  example:
 ```js
actions.addUser = function (payload) {
  const users = db.getTable('users');

  const newUser = {
    'id': sender
  };

  users.insert(newUser);
}
```

 ## 4.  Find a record in a table
Similarly to MongoDB, to find a record, we will be using two different methods:

 `findOne(params: object) returns the object found if it exists, null if otherwise`
 
  `find(params: object) returns an array of objects that match (empty array of no results)`

See [the LokiJS docs](https://github.com/techfort/LokiJS/wiki/Query-Examples) for the available params
  
  examples:
 ```js
actions.addUser = function (payload) {
  const users = db.getTable('users');

  let user = users.findOne({ 'id': sender });

  if (user) {
    // do something with the user
  } 
}
```

 ```js
actions.addUser = function (payload) {
  const users = db.getTable('users');

  let results = users.find({ 'name': 'Dan' });

  if (results.length > 0) {
    // do something with the results
  } 
}
```

 ## 5.  update a record in a table
Similarly to MongoDB, to update a record, we will be using:

  `update(record: object)`
  
  example:
 ```js
actions.updateUser = function (payload) {
  const { username } = payload;
  
  if (username && typeof username === 'string'){
    const users = db.getTable('users');
    let user = users.findOne({ 'id': sender });
    if (user) {
      user.username = username;
      users.update(user);
    }
  }
}
```

 ## 6.  Remove a record from a table
Similarly to MongoDB, to update a record, we will be using:

  `remove(record: object)`
  
  example:
 ```js
actions.removeUser = function (payload) {
  const { userId } = payload;

  if (userId && typeof userId === 'string') {
    let users = db.getTable('users');
    let user = users.findOne({ 'id': userId });
    if (user)
      users.remove(user);
  }
}
```

## 7.  Find a record in a table of another Smart Contract
The "db" objects gives you the ability to perform queries on tables that are held by other Smart Contracts:

 `findOneInTable(contract: string, table: string, query: object) returns the object found if it exists, null if otherwise`
 
  `findInTable(contract: string, table: string, query: object) returns an array of objects that match (empty array of no results)`
  
  examples:
 ```js
actions.addUser = function (payload) {
  const users = db.getTable('users');

  const book = db.findOneInTable('books_contract', 'books', { 'owner': sender });

  if (book) {
    // do something with the book
  } 
}
```

 ```js
actions.addUser = function (payload) {
  const users = db.getTable('users');

  const books = db.findInTable('books_contract', 'books', { 'owner': sender });

  if (results.length > 0) {
    // do something with the results
  } 
}
```