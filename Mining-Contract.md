Documentation written by [eonwarped](https://github.com/eonwarped)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* Actions:
  * [createPool](#createpool)
  * [updatePool](#updatepool)
* [Tables available](#tables-available)
  * [params](#params)
  * [pools](#pools)
  * [miningPower](#miningpower)

# Introduction

The mining contract allows a token issuer to set up "mining pools" that give
token stakers a chance to earn tokens via a lottery system. The issuer creates
a mining pool, specifying how often a lottery occurs, how many winners, and how
many tokens should be distributed each round, and the contract system handles
the rest. This is how "Proof of Stake" and "Proof of Staked Mining" works on
the scotbot level, and this contract can be viewed as a replacement of that
functionality. 

# Actions available:

### createPool:
Create a new mining pool. A creation fee of 1000 BEE is required. Note you
can specify the same token as the mining token, in which case it behaves like
'proof of stake', or you can specify a different token, in which case it
behaves like 'proof of staked mining'. You can also specify up to two mining
tokens with different multipliers. For example, for a pool with TEST as the
mined token and TEST, TESTM as the mining tokens with multipliers 1 and 4,
respectively, someone with 10 TEST stake and 10 TESTM stake will have a
mining power of 50. Note that delegated stake counts for this calculation.
For each lottery, the probability that someone will be a winner is their
mining power divided by the total mining power.

Note also that there can only be one pool per combination of mined token and
mining tokens. Upon creation, the mining ID derived from these symbols is
assigned to the pool.

* requires active key: yes
* can be called by: token issuer
* parameters:
  * lotteryWinners (integer >= 1 and <= 20): Number of lottery winners per round.
  * lotteryIntervalHours (integer >= 1 and <= 720): How often in hours to run a lottery.
  * lotteryAmount (string): Amount to pay out per round. Split among `lotteryWinners` winners.
  * minedToken (string): Which token to mine.
  * tokenMiners (list): List of token miners (only 1 or 2 allowed)
    * tokenMiners[].symbol (string): Symbol of token that is mining.
    * tokenMiners[].multiplier (integer >= 1 and <= 100): Multiplier for mining token.

* examples:
```
{
    "contractName": "mining",
    "contractAction": "createPool",
    "contractPayload": {
        "lotteryWinners": 1,
        "lotteryIntervalHours": 1,
        "lotteryAmount": "1",
        "minedToken": "TKN",
        "tokenMiners": [
            { "symbol": "TKN", "multiplier": 1},
            { "symbol": "MTKN", "multiplier": 4},
        ],
    }
}

A successful action will emit a "createPool" event, e.g.:
```
{
    "contract": "mining",
    "event": "createPool",
    "data": {
        "id": "TKN-TKN",
    }
}
```
### updatePool
Update a mining pool. An update fee of 300 BEE is required. Can be used to activate or 
deactivate the pool as well as update multipliers. The symbols themselves cannot be changed.
* requires active key: yes
* can be called by: token issuer
* parameters:
  * id (string): ID of pool to modify.
  * lotteryWinners (integer >= 1 and <= 20): Number of lottery winners per round.
  * lotteryIntervalHours (integer >= 1 and <= 720): How often in hours to run a lottery.
  * lotteryAmount (string): Amount to pay out per round. Split among `lotteryWinners` winners.
  * tokenMiners (list): List of token miners (only 1 or 2 allowed)
    * tokenMiners[].symbol (string): Symbol of token that is mining.
    * tokenMiners[].multiplier (integer >= 1 and <= 100): Multiplier for mining token.
  * active (boolean): Whether pool is active. When inactive, no further lotteries will take place
      until it is reactivated.

* examples:
```
{
    "contractName": "mining",
    "contractAction": "createPool",
    "contractPayload": {
        "id": "TKN:TKN",
        "lotteryWinners": 1,
        "lotteryIntervalHours": 1,
        "lotteryAmount": "1",
        "active": false,
        "tokenMiners": [
            { "symbol": "TKN", "multiplier": 1 }
        ],
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

## params:
contains contract parameters such as the current fees
* fields
  * poolCreationFee = the cost in BEE to create a pool
  * poolUpdateFee = the cost in BEE to update a pool

## pools:
contains information about the pool
* fields
  * lotteryWinners = number of lottery winners per round
  * lotteryIntervalHours = number of hours between each round
  * lotteryAmount = amount to distribute per round
  * tokenMiners = list of mining tokens and their multipliers
  * active = whether the pool is active
  * updating.inProgress = whether mining power calculation is in process

## miningPower:
contains information about mining power for each account in pool
* fields
  * id = ID of mining pool
  * account = name of account
  * power.$numberDecimal = amount of mining power for the account in the pool.
  * balances = list of balances used to calculate mining power, in same order as the tokenMiners in the pool.

