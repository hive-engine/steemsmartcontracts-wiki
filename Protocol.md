To send an action to the sidechain simply post a message on the Steem blockchain that follows this pattern:

```
##STEEMCONTRACTSBEGIN##contract name##action to perform##payload for the Smart Contract##STEEMCONTRACTSEND##
```

 - ##STEEMCONTRACTSBEGIN## this part is used to detect the beginning of the action
 - ##STEEMCONTRACTSEND## this part is used to detect the end of the action
 - "contract name" is the name of the Smart Contract (String)
 - "action to perform" is the action that you want to perform on the above Smart Contract (String)
 - "payload for the Smart Contract" is the payload that will be passed to the Smart Contract (JSON stringified)