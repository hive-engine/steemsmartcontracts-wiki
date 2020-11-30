Documentation written by [donchate](https://github.com/donchate)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* Actions:
  * [create](#create)
  * [update](#update)
  * [flush](#flush)
  * [setActive](#setactive)
  * [deposit](#deposit)
* [Tables available](#tables-available)
  * [params](#params)
  * [batches](#batches)

# Introduction

This contract allows a decentralized organization (i.e DAO) to collect and
distribute payments to a roll of users or contracts. The contract will collect
tokens to a threshold amount, and distribute them according to a percentage share.
Its payout thresholds for each configured token are checked upon every _deposit_ and paid out
immediately when they are exceeded.
The distribution batch only accepts tokens defined in its configuration, which can be updated
by the creator at any time. The distribution can be deactivated (suspended), during which time
new deposits are rejected and payouts are not made. There is also a means for the creator to
manually release (_flush_) any outstanding balance in the distribution.

# Actions available

### create
Create a distribution batch for each unique set of tokens and recipient shares that is required.
A fee of 500 BEE is required.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenMinPayout (list): List of accepted tokens
    * tokenMiners[].symbol (string): Valid token symbol
    * tokenMiners[].quantity (decimal > 0): Payout threshold (_must have no more than 3 decimal places_)
  * tokenRecipients (list): List of payment recipients
    * tokenMiners[].account (string): Valid graphene account string
    * tokenMiners[].type (string): _user_ or _contract_ only
    * tokenMiners[].pct (integer >= 1 and <= 100): Percentage share

* example:
```
{
  "tokenMinPayout": [
    {
      "symbol": "TKN", 
      "quantity": 10
    }
  ], 
  "tokenRecipients": [
    {
      "account": "donchate", 
      "type": "user", 
      "pct": 100
    }
  ], 
  "isSignedWithActiveKey": true
}
```

### update
You may update the payment thresholds or recipients using this action.
A fee of 250 BEE is required.

* requires active key: yes
* can be called by: distribution creator
* parameters:
  * id (int): ID of distribution to update
  * tokenMinPayout (list): List of accepted tokens
    * tokenMiners[].symbol (string): Valid token symbol
    * tokenMiners[].quantity (decimal > 0): Payout threshold (_must have no more than 3 decimal places_)
  * tokenRecipients (list): List of payment recipients
    * tokenMiners[].account (string): Valid graphene account string
    * tokenMiners[].type (string): _user_ or _contract_ only
    * tokenMiners[].pct (integer >= 1 and <= 100): Percentage share

* example:
```
{
  "id": 1, 
  "tokenMinPayout": [
    {
      "symbol": "TKN", 
      "quantity": 100
    },
    {
      "symbol": "TKNA", 
      "quantity": 50
    }    
  ], 
  "tokenRecipients": [
    {
      "account": "donchate", 
      "type": "user", 
      "pct": 50
    }, 
    {
      "account": "dantheman", 
      "type": "user", 
      "pct": 50
    }
  ], 
  "isSignedWithActiveKey": true
}
```

### flush
You can manually trigger a distribution by using this action. It will release the outstanding
token balance to tokenRecipients immediately. No fee required.

* requires active key: yes
* can be called by: distribution creator
* parameters:
  * id (int): ID of distribution to flush
  * symbol (string): Valid and configured token symbol

* example:
```
{
  "id": 1, 
  "symbol": "TKN", 
  "isSignedWithActiveKey": true
}
```

### setActive
For enabling and disabling the distribution contract. When disabled, no payments will be made
and no new deposits are accepted. Newly created distributions need to be activated.
No fee required.

* requires active key: yes
* can be called by: token issuer
* parameters:
  * id (int): ID of distribution to flush
  * active (boolean): true to activate, false to deactivate

* activate a distribution:
```
{
  "id": 1, 
  "active": true, 
  "isSignedWithActiveKey": true
}
```
* deactivate a distribution:
```
{
  "id": 1, 
  "active": false, 
  "isSignedWithActiveKey": true
}
```

### deposit
The contract will perform a transfer of any configured token using this action. A payment distribution will be attempted immediately following a valid deposit.
No fee required.

* requires active key: yes
* can be called by: anyone
* parameters:
  * id (int): ID of distribution instance
  * symbol (string): Symbol of token being deposited
  * quantity (string): string decimal up to staked token max precision

* example request data:
```
{
  "id": 1, 
  "symbol": "TKN", 
  "quantity": "1000"
}
```

# Tables available

## params:
contains internal contract parameters
* fields
  * distCreationFee = the cost in BEE to create a distribution
  * distUpdateFee = the cost in BEE to update a distribution

## batches:
contains the instance configurations
* fields
  * _id = MongoDB internal primary key
  * tokenMinPayout = JSON object of configured tokens and their payment thresholds
  * tokenRecipients = JSON object of configured recipients and their share
  * active = whether or not the distribution instance is active
  * creator = account or contract that created the instance