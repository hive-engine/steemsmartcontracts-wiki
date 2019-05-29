
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
    "contractAction": "enableStaking",
    "contractPayload": {
        "symbol": "TKN",
        "undelegationCooldown": 7, // 7 days to cooldown
    }
}

{
    "contractName": "tokens",
    "contractAction": "enableStaking",
    "contractPayload": {
        "symbol": "TKN",
        "undelegationCooldown": 30, // 30 days to cooldown
    }
}

{
    "contractName": "tokens",
    "contractAction": "enableStaking",
    "contractPayload": {
        "symbol": "TKN",
        "undelegationCooldown": 91, // 13 weeks to cooldown
    }
}
```

### delegate: 
Delegates tokens. (if it is a token that allows delegation)
If a delegation already exists for that account it will update it with the new quantity if it is greater than the previous delegation (to decrease a delegation you need to undelegate)

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
    - to (string): account that will see its delegation decreased/removed
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

## Other actions available
### create: ...
### issue: ...
### transfer: ...
### transferToContract: ...
### updateUrl: ...
### updateMetadata: ...

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