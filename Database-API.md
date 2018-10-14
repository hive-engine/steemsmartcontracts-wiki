
You can find all the database related functions under the global variable "db"

## 1.  Create a table
 You can create a table that will be available for your Smart Contract only during the deployment of the Smart Contract. Use the indexes array to add one or more indexes to the table.
The method that we will be using is:
 
 `createTable(tableName: string, indexes: Array<String>)`
 
 example:
 ```js
actions.create = async (payload) => {
	// let's create a table called users
	await db.createTable('users', ['ages']);
}
```

 ## 2.  Add a record in a table
To add a record, we will be using:

`insert(tableName: string, record: object)`
 
  example:
 ```js
actions.addUser = async (payload) => {
  const newUser = {
    'id': sender
  };

  await db.insert('users', newUser);
}
```

 ## 4.  Find a record in a table
Similarly to MongoDB, to find a record, we will be using two different methods:

 `findOne(tableName: string, query: object) returns the object found if it exists, null if otherwise`
 
 `find(tableName: string, query: object, limit: integer  =  1000, offset: integer  =  0, index: string  =  '', descending: boolean  =  false) returns an array of objects that match (empty array of no results)`

See [the LokiJS docs](https://github.com/techfort/LokiJS/wiki/Query-Examples) for the available params
  
  examples:
 ```js
actions.addUser = async (payload) => {
  let user = await db.findOne('users', { 'id': sender });

  if (user) {
    // do something with the user
  } 
}
```

 ```js
actions.addUser = function (payload) {
  let results = await db.find('users', { 'name': 'Dan' });

  if (results.length > 0) {
    // do something with the results
  } 
}
```

 ## 5.  update a record in a table
Similarly to MongoDB, to update a record, we will be using:

  `update(tableName: string, record: object)`
  
  example:
 ```js
actions.updateUser = async (payload) => {
  const { username } = payload;
  
  if (username && typeof username === 'string'){
    let user = await db.findOne('users', { 'id': sender });
    if (user) {
      user.username = username;
      await db.update('users', user);
    }
  }
}
```

 ## 6.  Remove a record from a table
Similarly to MongoDB, to remove a record, we will be using:

  `remove(tableName: string, record: object)`
  
  example:
 ```js
actions.removeUser = async (payload) => {
  const { userId } = payload;

  if (userId && typeof userId === 'string') {
    let user = await db.findOne('users', { 'id': userId });
    if (user)
      await db.remove('users', user);
  }
}
```

## 7.  Find a record in a table of another Smart Contract
The "db" objects gives you the ability to perform queries on tables that are held by other Smart Contracts:

```js
/**
   * retrieve a record from the table of a contract
   * @param {String} contract contract name
   * @param {String} table table name
   * @param {JSON} query query to perform on the table
   * @returns {Object} returns a record if it exists, null otherwise
*/
```
 `findOneInTable(contract: string, table: string, query: object) returns the object found if it exists, null if otherwise`
 
```js
/**
   * retrieve records from the table of a contract
   * @param {String} contract contract name
   * @param {String} table table name
   * @param {JSON} query query to perform on the table
   * @param {Integer} limit limit the number of records to retrieve
   * @param {Integer} offset offset applied to the records set
   * @param {String} index name of the index to use for the query
   * @param {Boolean} descending the records set is sorted ascending if false, descending if true
   * @returns {Array<Object>} returns an array of objects if records found, an empty array otherwise
*/
```
  `findInTable(contract: string, table: string, query: object, limit: integer, offset: integer, index: string, descending: boolean) returns an array of objects that match (empty array of no results)`
  
  examples:
 ```js
actions.addUser = async (payload) => {

  const book = await db.findOneInTable('books_contract', 'books', { 'owner': sender });

  if (book) {
    // do something with the book
  } 
}
```

 ```js
actions.addUser = async (payload) => {
  const books = await db.findInTable('books_contract', 'books', { 'owner': sender });

  if (results.length > 0) {
    // do something with the results
  } 
}
```