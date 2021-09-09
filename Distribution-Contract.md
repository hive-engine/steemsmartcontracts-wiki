Documentation written by [donchate](https://github.com/donchate)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
  * [Execution](#execution)
  * [Fixed Distribution](#fixed-distribution)
  * [Diesel Pools Distribution](#diesel-pools-distribution)
* [Actions available](#actions-available)
  * [create](#create)
  * [update](#update)
  * [flush](#flush)
  * [setActive](#setactive)
  * [deposit](#deposit)
* [Contract Integration](#contract-integration)
* [Tables available](#tables-available)
  * [params](#params)
  * [batches](#batches)

# Introduction
This contract allows a decentralized organization (i.e DAO) to collect and
distribute payments to a roll of users or contracts. The contract supports multiple strategies
for determination of the recipients.
The distribution can be deactivated (suspended), during which time
new deposits are rejected and payouts are not made. There is also a means for the creator to
manually release (_flush_) any outstanding balance in the distribution.
Distribution payout thresholds for each configured token are checked and paid out according to the configured number of ticks.

## Execution
This contract is executed by virtual transaction, providing a scheduled distribution option, as well
as manually by calling relevant actions from another application.

## Fixed Distribution
This strategy will collect tokens to a threshold amount, and distribute them according to a fixed percentage share.

## Diesel Pools Distribution
This strategy calculates the recipient list based on liquidity provider share in a given diesel pool. Accounts may also be excluded from the share calculation and a holding duration bonus applied.

# Actions available

### create:
Create a distribution batch for each unique set of tokens and recipient shares that is required.
A fee of 100 BEE is required.

* requires active key: yes
* can be called by: anyone
* parameters:
  * strategy (string): distribution strategy, one of ```[ 'fixed', 'pool' ] ```
  * numTicks (string): number of periods to distribute deposited tokens
  * _Pool strategy_
    * tokenPair (string): marketpool tokenPair for recipient determination
    * excludeAccount (array, optional): list of account names to be excluded from pool share calculation
    * bonusCurve (object, optional): settings for optional bonus curve applied to LP shares
      * numPeriods (string): number of periods over which to apply bonus curve, after which there is no more growth (integer 1-5555)
      * periodBonusPct (string): percentage bonus per period LP is held (integer 1-100)
  * _Fixed strategy_
    * tokenMinPayout (list): List of accepted tokens
      * tokenMiners[].symbol (string): Valid token symbol
      * tokenMiners[].quantity (decimal > 0): Payout threshold (_must have no more than 3 decimal places_)
    * tokenRecipients (list): List of payment recipients
      * tokenMiners[].account (string): Valid graphene account string
      * tokenMiners[].type (string): _user_ or _contract_ only
      * tokenMiners[].pct (integer >= 1 and <= 100): Percentage share

* example (fixed strategy):
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
* example (pool strategy)
```
{ 
  strategy": "pool",
  "numTicks": "30",
  "tokenPair": "TKNA:TKNB",
  "excludeAccount": ["donchate"],
  "bonusCurve": { 
    "numPeriods": "100",
    "periodBonusPct": "1"
  },
  "isSignedWithActiveKey": true
}
```

### update
You may update the payment thresholds or recipients using this action. Strategy cannot be changed. Including a numTicks parameter will restart the distribution schedule and reset numTicks and numTicksLeft. Changes to bonus curve are applied on the next tick, bonus curve can be disabled by setting the property to an empty object ```{}```.
A fee of 100 BEE is required.

* requires active key: yes
* can be called by: distribution creator
* parameters:
  * numTicks (string): number of periods to distribute deposited tokens
  * _Pool strategy_
    * tokenPair (string): marketpool tokenPair for recipient determination
    * excludeAccount (array, optional): list of account names to be excluded from pool share calculation
    * bonusCurve (object, optional): settings for optional bonus curve applied to LP shares
      * numPeriods (string): number of periods over which to apply bonus curve, after which there is no more growth (integer 1-5555)
      * periodBonusPct (string): percentage bonus per period LP is held (integer 1-100)
  * _Fixed strategy_
    * tokenMinPayout (list): List of accepted tokens
      * tokenMiners[].symbol (string): Valid token symbol
      * tokenMiners[].quantity (decimal > 0): Payout threshold (_must have no more than 3 decimal places_)
    * tokenRecipients (list): List of payment recipients
      * tokenMiners[].account (string): Valid graphene account string
      * tokenMiners[].type (string): _user_ or _contract_ only
      * tokenMiners[].pct (integer >= 1 and <= 100): Percentage share

### flush
You can manually trigger a distribution by using this action. It will release the outstanding
token balance to tokenRecipients immediately and end payment periods. No fee required.

* requires active key: yes
* can be called by: distribution creator
* parameters:
  * id (int): ID of distribution to flush

* example:
```
{
  "id": 1, 
  "isSignedWithActiveKey": true
}
```

### setActive
For enabling and disabling the distribution contract. When disabled, no payments will be made
and no new deposits are accepted. Newly created distributions need to be activated.
No fee required.

* requires active key: yes
* can be called by: distribution creator
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
Deposit tokens to an existing distribution. Anyone can deposit tokens to a distribution and deposits are non-refundable.

* requires active key: yes
* can be called by: anyone
* parameters:
  * id (int): ID of distribution to deposit
  * symbol (string): token symbol to deposit
  * quantity (string): token amount to deposit

* example
```
{
  "id": 1,
  "symbol": "BEE",
  "quantity": "100",
  "isSignedWithActiveKey": true
}
```

# Contract Integration
To configure your contract to handle payments from the distribution contract, add an action called ```receiveDistTokens```.
Upon making a token transfer to a contract's balance, the contract will also call this action and pass a payload, as well as the symbol and quantity of tokens transferred.
This action should be configured to only accept calls from the ```distribution``` contract. It should also verify the receipt of tokens, and then perform contract logic accordingly.

* example custom_json action:
```
{
    "contractName": "mycontract",
    "contractAction": "receiveDistTokens",
    "contractPayload": {
        "data": { "id": "100" },
        "symbol": "TKN",
        "quantity": "1.00000000",
    }
}
```

* example contract action:
```
actions.receiveDistTokens = async (payload) => {
  const {
    data, symbol, quantity,
    callingContractInfo,
  } = payload;

  if (!api.assert(callingContractInfo && callingContractInfo.name === 'distribution', 'not authorized')) return;
  ...
}
```

# Tables available:

## params:
contains internal contract parameters
* fields
  * distCreationFee: the cost in BEE to create a pool
  * distUpdateFee: the cost in BEE to update a pool
  * distTickHours: tick (payment) frequency, defaulted to 24hours for daily payout
  * maxDistributionsLimit: number of distributions to process per block
  * processQueryLimit: database query limit
  * updateIndex: db schema revision index

## batches:
contains the instance configurations
* fields
  * _id: MongoDB internal primary key
  * active: whether or not the distribution instance is active
  * strategy: distribution recipient strategy
  * lastTickTime: last time distribution was processed
  * numTicks: configured number of payments for repeating distributions
  * numTicksLeft: number of payments remaining in the distribution
  * creator: account or contract that created the instance  
  * _Fixed strategy fields_
    * tokenMinPayout: JSON object of configured tokens and their payment thresholds
    * tokenRecipients: JSON object of configured recipients and their share
  * _Pool strategy fields_
    * tokenPair: assigned tokenPair for pool calculation
    * excludeAccount: accounts excluded from pool calculation
    * bonusCurve: settings for optional bonus curve applied to LP shares
