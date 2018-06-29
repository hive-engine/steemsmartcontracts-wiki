 To make an action available in your Smart Contract, you will need to hook up your code to the global variable **actions**.

For example, if I want to have an action called **addUser**:

```js
actions.addUser = function (payload) {
	// the code of my action here
	// the parameter payload is passed to all the actions
	// payload it is a JSON containing the payload that was passed by the user via the protocol
}
```

The "create" action is a reserved action that is triggered only once during the whole life of the Smart Contract, and this happens when the Smart Contract is deployed. This is the only place where you can initialize the storage for your Smart Contract. For example:

```js
actions.create = function (payload) {
	// initialize some stuffs here
}
```