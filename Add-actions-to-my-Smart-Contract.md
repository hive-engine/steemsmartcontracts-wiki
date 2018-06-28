 To make an action available in your Smart Contract you will need to hook up your code to the global variable **actions**

for example, if I want to have an action called **addUser**:

```js
actions.addUser = function (payload) {
	// the code of my action here
	// the parameter payload is passed to all the actions
	// payload it is a JSON containing the payload that was passed by the user via the protocol
}
```