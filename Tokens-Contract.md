

# Tokens contract

## Actions available:

### enableStaking: 
Enables the staking feature for a token.

- requires active key: yes
 - parameters:
	- symbol (string): symbol of the token you want to enable the staking feature
	- unstakingCooldown (integer): number of days that a user will have to wait until the tokens are added to their balance (between 1 and 365 days)

- example:
```
{
    "contractName": "tokens",
    "contractAction": "enableStaking",
    "contractPayload": {
        "symbol": "TKN",
        "unstakingCooldown": 7
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

	- unstakeCompleteTimestamp: timestamp when the unstake will be completed (in milliseconds)