

# Tokens contract

## Actions available:

## Staking
### enableStaking: 
Enables the staking feature for a token.

- requires active key: yes
 - parameters:
	- symbol (string): symbol of the token you want to enable the staking feature
	- unstakingCooldown (integer): number of days that a user will have to wait until the tokens are added to their balance (between 1 and 365 days)
	- numberTransactions (integer): number of transactions that an unstake will be divided in (between 1 and 365 days)

- examples:
```
{
    "contractName": "tokens",
    "contractAction": "enableStaking",
    "contractPayload": {
        "symbol": "TKN",
        "unstakingCooldown": 7, // 7 days to cooldown
        "numberTransactions": 7, // 1 transaction / day
    }
}

{
    "contractName": "tokens",
    "contractAction": "enableStaking",
    "contractPayload": {
        "symbol": "TKN",
        "unstakingCooldown": 30, // 30 days to cooldown
        "numberTransactions": 60, // 2 transactions / day
    }
}

{
    "contractName": "tokens",
    "contractAction": "enableStaking",
    "contractPayload": {
        "symbol": "TKN",
        "unstakingCooldown": 91, // 13 weeks to cooldown
        "numberTransactions": 13, // 1 transaction / week
    }
}
```
### stake: 
Stakes tokens. (if it is a token that allows staking) 

- requires active key: yes
 - parameters:
	- to (string): account that will receive the stake 
	- symbol (string): symbol of the token you want to stake
	- quantity (string): quantity of tokens to stake

- example:
```
{
    "contractName": "tokens",
    "contractAction": "stake",
    "contractPayload": {
        "to": "harpagon",
        "symbol": "TKN",
        "quantity": "3.5"
    }
}
```
### unstake: 
Unstakes tokens. (if it is a token that allows staking)
There is a cooldown period that is setup per token (see "enableStaking" action)

- requires active key: yes
 - parameters:
	- symbol (string): symbol of the token you want to unstake
	- quantity (string): quantity of tokens to unstake

- example:
```
{
    "contractName": "tokens",
    "contractAction": "unstake",
    "contractPayload": {
        "symbol": "TKN",
        "quantity": "3.5"
    }
}
```

### cancelUnstake: 
Cancels an initiated unstake.

- requires active key: yes
 - parameters:
	- txID (string): transaction id when the unstake was initiated

- example:
```
{
    "contractName": "tokens",
    "contractAction": "cancelUnstake",
    "contractPayload": {
        "txID": "ABCDE0123"
    }
}
```

## Delegation
### enableDelegation: 
Enables the delegation feature for a token.

- requires active key: yes
 - parameters:
	- symbol (string): symbol of the token you want to enable the delegation feature
	- undelegationCooldown (integer): number of days that a user will have to wait until the tokens that were delegated are added back to their balance (between 1 and 365 days)

- examples:
```
{
    "contractName": "tokens",
    "contractAction": "enableDelegation",
    "contractPayload": {
        "symbol": "TKN",
        "undelegationCooldown": 7, // 7 days to cooldown
    }
}

{
    "contractName": "tokens",
    "contractAction": "enableDelegation",
    "contractPayload": {
        "symbol": "TKN",
        "undelegationCooldown": 30, // 30 days to cooldown
    }
}

{
    "contractName": "tokens",
    "contractAction": "enableDelegation",
    "contractPayload": {
        "symbol": "TKN",
        "undelegationCooldown": 91, // 13 weeks to cooldown
    }
}
```

### delegate: 
Delegates tokens. (if it is a token that allows delegation)
If a delegation already exists for that account it will add the new quantity to it (to decrease a delegation you need to undelegate)

- requires active key: yes
 - parameters:
	- to (string): account that will receive the delegation 
	- symbol (string): symbol of the token you want to delegate
	- quantity (string): quantity of tokens to delegate

- example:
```
{
    "contractName": "tokens",
    "contractAction": "delegate",
    "contractPayload": {
        "to": "harpagon",
        "symbol": "TKN",
        "quantity": "3.5"
    }
}
```

### undelegate: 
Undelegates tokens partially or totally. (if it is a token that allows delegation)
There is a cooldown period that is setup per token (see "enableDelegation" action)

- requires active key: yes
 - parameters:
    - from (string): account that will see its delegation decreased/removed
	- symbol (string): symbol of the token you want to undelegate
	- quantity (string): quantity of tokens to undelegate

- example:
```
{
    "contractName": "tokens",
    "contractAction": "undelegate",
    "contractPayload": {
	    "to": "harpagon",
        "symbol": "TKN",
        "quantity": "3.5"
    }
}
```

## Tokens parameters

### updatePrecision: 
Update the precision of a token (it can only be increased)

- requires active key: yes
 - parameters:
	- symbol (string): symbol of the token to update
	- precision (integer): new precision for the token (can only be greater than the current precision)

- example:
```
{
    "contractName": "tokens",
    "contractAction": "updatePrecision",
    "contractPayload": {
        "symbol": "TKN",
        "precision": 8
    }
}
```

## Other actions available
### create:
Creates a token (a token creation fee might be enforced)

- requires active key: yes
 - parameters:
	- name (string): name of the token (letters, numbers, whitespaces only, max length of 50)
	- symbol (string): symbol of the token (uppercase letters only, max length of 10)
	- precision (integer): precision for the token (between 0 and 8)
	- maxSupply (string): maximum supply for the token (between 1 and 9,007,199,254,740,991)
	- url (string): url of the project (max length of 255)

- example:
```
{
    "contractName": "tokens",
    "contractAction": "create",
    "contractPayload": {
        "symbol": "TKN",
        "name": "my token",
        "precision": 8,
        "maxSupply": "1000",
        "url": "myawesomeproject.com"
    }
}
```

### issue:
Issue tokens to an account (can only be triggered by the account that created the token)

- requires active key: yes
 - parameters:
	 - to (string): account that will receive the newly created tokens (3 <= length <= 16)
	- symbol (string): symbol of the token (uppercase letters only, max length of 10)
	- quantity (string): quantity of tokens to issue (must match the requirements of the token (maxSupply limit, precision, etc...)


- example:
```
{
    "contractName": "tokens",
    "contractAction": "issue",
    "contractPayload": {
        "symbol": "TKN",
        "to": "harpagon",
        "quantity": "15.21548745"
    }
}
```
### transfer:
Transfer tokens to an account

- requires active key: yes
 - parameters:
	 - to (string): account that will receive the tokens (3 <= length <= 16)
	- symbol (string): symbol of the token (uppercase letters only, max length of 10)
	- quantity (string): quantity of tokens to issue (must match the requirements of the token (maxSupply limit, precision, etc...)


- example:
```
{
    "contractName": "tokens",
    "contractAction": "transfer",
    "contractPayload": {
        "symbol": "TKN",
        "to": "harpagon",
        "quantity": "15.21548745"
    }
}
```
### transferToContract:
Transfer tokens to a contract

- requires active key: yes
 - parameters:
	 - to (string): contract that will receive the tokens (3 <= length <= 50)
	- symbol (string): symbol of the token (uppercase letters only, max length of 10)
	- quantity (string): quantity of tokens to issue (must match the requirements of the token (maxSupply limit, precision, etc...)


- example:
```
{
    "contractName": "tokens",
    "contractAction": "transferToContract",
    "contractPayload": {
        "symbol": "TKN",
        "to": "market",
        "quantity": "15.21548745"
    }
}
```
### transferOwnership:
Transfer ownership of a token from the current token issuer to an other account

- requires active key: yes
 - parameters:
	 - to (string): contract that will receive the ownership of the token (3 <= length <= 50)
	- symbol (string): symbol of the token (uppercase letters only, max length of 10)


- example:
```
{
    "contractName": "tokens",
    "contractAction": "transferOwnership",
    "contractPayload": {
        "symbol": "TKN",
        "to": "harpagon"
    }
}
```
### updateUrl:
Update the url of a token

- requires active key: no
 - parameters:
	 - url (string): the new url for the token (0 <= length <= 255)
	- symbol (string): symbol of the token (uppercase letters only, max length of 10)


- example:
```
{
    "contractName": "tokens",
    "contractAction": "updateUrl",
    "contractPayload": {
        "symbol": "TKN",
        "url": "mynewurl.com"
    }
}
```

### updateMetadata:
Update the metadata of a token

- requires active key: no
 - parameters:
	 - metadata (JSON): the metadata for the token (the JSON when stringified can't exceed 1000 characters)
	- symbol (string): symbol of the token (uppercase letters only, max length of 10)


- example:
```
{
    "contractName": "tokens",
    "contractAction": "updateMetadata",
    "contractPayload": {
        "symbol": "TKN",
        "metadata": {
	        "url": "mynewurl.com",
	        "description": "my description"
	    }
    }
}
```

## Tables available:

### balances:
balances of the users

-	fields:
	- account = account owning the balance
	- symbol = symbol of the token
	- balance = quantity of tokens
	- stake = quantity of tokens staked
	- pendingUnstake = quantity of tokens being unstaked
	- delegationsIn = quantity of tokens being delegated to that account
    - delegationsOut = quantity of tokens being delegated to other accounts
	- pendingUndelegations = quantity of tokens being undelegated

### delegations:
opened delegations

-	fields:
	- from = account that initiated the delegation
	- to = account that received the delegation
	- symbol = symbol of the token delegated
	- quantity = quantity of tokens delegated

### pendingUnstakes:
pending unstakes

-	fields:
	- account: account that initiated the unstake
	- symbol:  symbol of the token being unstaked
	- quantity: quantity of tokens being unstaked
	- quantityLeft: quantity of tokens still to unstake
	- nextTransactionTimestamp: timestamp when the next unstake transaction will happen (in milliseconds)
	- numberTransactionsLeft: number of transactions left for this unstake
	- txID: transaction ID used to initiate the unstake

### pendingUndelegations:
pending undelegations

-	fields:
	- account: account that initiated the undelegation
	- symbol:  symbol of the token being undelegated
	- quantity: quantity of tokens being undelegated
	- completeTimestamp: timestamp when the undelegation will be completed (in milliseconds)
	- txID: transaction ID used to initiate the undelegation