# Tokens contract

## Actions available:

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
        "unstakingCooldown": 91, // 91 days to cooldown
        "numberTransactions": 13, // 1 transaction / week
    }
}
```
### stake: 
Stakes tokens. (if it is a token that allows staking) 

- requires active key: yes
 - parameters:
	- symbol (string): symbol of the token you want to stake
	- quantity (string): quantity of tokens to stake

- example:
```
{
    "contractName": "tokens",
    "contractAction": "stake",
    "contractPayload": {
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