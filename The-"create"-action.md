The "create" action is a reserved action, it is triggered only once during the whole life of the Smart Contract and this happens when the Smart Contract is deployed. This is the only place where you can initialize the storage for your Smart Contract for example.

```js
actions.create = function (payload) {
	// initialize some stuffs here
}
```