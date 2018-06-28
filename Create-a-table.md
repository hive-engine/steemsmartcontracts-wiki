You can create a table that will be available for you Smart Contract only during the deployment of the Smart Contract. The method that we will be using is:
 
 `createTable(tableName: string)`
 
 example:
 ```js
actions.create = function (payload) {
	// let's create a table called users
	db.createTable('users');
}
```