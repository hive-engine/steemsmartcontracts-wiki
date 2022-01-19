Documentation written by [eonwarped](https://github.com/eonwarped)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* Actions:
  * [createPool](#createpool)
  * [updatePool](#updatepool)
  * [setActive](#setactive)
  * [changeNftProperty](#changenftproperty)
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
  * tokenMiners (list): List of token miners (at most 2 allowed)
    * tokenMiners[].symbol (string): Symbol of token that is mining.
    * tokenMiners[].multiplier (integer >= 1 and <= 100): Multiplier for mining token.
  * nftTokenMiner: NFT Token miner (optional)
    * nftTokenMiner.symbol (string):  Symbol of token that is mining.
    * nftTokenMiner.typeField (string): Property of nft to use to distinguish nft types.
    * nftTokenMiner.properties (list): List of properties for pwoer computation.
      * nftTokenMiner.properties.op: ADD or MULTIPLY whether to add or multiply the
         power values for this property.
      * nftTokenMiner.properties.name: Name of property.
      * nftTokenMiner.properties.burnChange: Configuration for authorized burn adjustments.
        * nftTokenMiner.properties.burnChange.symbol: Symbol of burned token.
        * nftTokenMiner.properties.burnChange.quantity: Quantity of burned token for fee. The fee
            to change this property is the amount desired multiplied by this quantity.
    * nftTokenMiner.typeMap (object): Map type to power attributes. Value should match
         order of the properties list, and they should be BigNumber-compatible strings.
    * nftTokenMiner.equipField (string): Property of nft to use to denote whether an nft should be
         considered as equipped. If not specified, a token is considered equipped when delegated to
         the mining contract or delegated to another user. Otherwise, the source of truth for who
         currently equips the nft is on the property identified by `equipField`, which must be a
         string property.
    * nftTokenMiner.miningPowerField (string): Property of nft to use to add additional mining power.
         Can be specified along with `typeMap`, and contract will add the two together to determine the
         final mining power of a given nft. The NFT property should be a string property, with a valid
         BigNumber format. Any invalid mining power will be ignored (treated as "0").

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
            { "symbol": "MTKN", "multiplier": 4}
        ],
        "nftTokenMiner": {
            "symbol": "TSTNFT",
            "typeField": "name",
            "properties": [
                {"op": "ADD", "name": "power", "burnChange": {"symbol": "NOTKN", "quantity": "1"}},
                {"op": "MULTIPLY", "name": "boost"}
            ],
            "typeMap": {
                "bear": ["-1.0", "0.8"],
                "bull": ["1.0", "2.0"]
            },
            "equipField": "equip",
            "miningPowerField": "miningPower"
        }
    }
}
```

A successful action will emit a "createPool" event, e.g.:
```
{
    "contract": "mining",
    "event": "createPool",
    "data": {
        "id": "TKN:TKN,MTKN:TSTNFT",
    }
}
```
### updatePool
Update a mining pool. An update fee of 100 BEE is required. The symbols themselves cannot be changed.
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
* nftTokenMiner: NFT Token miner (optional)
    * nftTokenMiner.symbol (string):  Symbol of token that is mining.
    * nftTokenMiner.typeField (string): Property of nft to use to distinguish nft types.
    * nftTokenMiner.properties (list): List of properties for pwoer computation.
      * nftTokenMiner.properties.op: ADD or MULTIPLY whether to add or multiply the
         power values for this property.
      * nftTokenMiner.properties.name: Name of property.
      * nftTokenMiner.properties.burnChange: Configuration for authorized burn adjustments.
        * nftTokenMiner.properties.burnChange.symbol: Symbol of burned token.
        * nftTokenMiner.properties.burnChange.quantity: Quantity of burned token for fee. The fee
            to change this property is the amount desired multiplied by this quantity.
    * nftTokenMiner.typeMap (object): Map type to power attributes. Value should match
         order of the properties list, and they should be BigNumber-compatible strings.
    * nftTokenMiner.equipField (string): Property of nft to use to denote whether an nft should be
         considered as equipped. If not specified, a token is considered equipped when delegated to
         the mining contract or delegated to another user. Otherwise, the source of truth for who
         currently equips the nft is on the property identified by `equipField`, which must be a
         string property.
    * nftTokenMiner.miningPowerField (string): Property of nft to use to add additional mining power.
         Can be specified along with `typeMap`, and contract will add the two together to determine the
         final mining power of a given nft. The NFT property should be a string property, with a valid
         BigNumber format. Any invalid mining power will be ignored (treated as "0").


* examples:
```
{
    "contractName": "mining",
    "contractAction": "updatePool",
    "contractPayload": {
        "id": "TKN:TKN:TSTNFT",
        "lotteryWinners": 1,
        "lotteryIntervalHours": 1,
        "lotteryAmount": "1",
        "tokenMiners": [
            { "symbol": "TKN", "multiplier": 1 }
        ],
        "nftTokenMiner": {
            "symbol": "TSTNFT",
            "typeField": "name",
            "properties": [
                {"op": "ADD", "name": "power", "burnChange": {"symbol": "NOTKN", "quantity": "1"}},
                {"op": "MULTIPLY", "name": "boost"}
            ],
            "typeMap": {
                "bear": ["-1.0", "0.8"],
                "bull": ["1.0", "2.0"]
            },
            "equipField": "equip",
            "miningPowerField": "miningPower"
        }
    }
}
```

### setActive
Disable or enable a mining pool. Lotteries will not occur if not active. No fee is required
to change the status, but must be the issuer of the token.
* requires active key: yes
* can be called by: token issuer
* parameters:
  * id (string): ID of pool to modify.
  * active (boolean): Whether the pool is active.

* examples:
```
{
    "contractName": "mining",
    "contractAction": "setActive",
    "contractPayload": {
        "id": "TKN:TKN",
        "active": true
    }
}
```

### changeNftProperty
Change an nft mining property. The property must be configured with a `burnChange`, see `createPool`
or `updatePool` above.
* requires active key: yes
* parameters:
  * id (string): ID of pool to modify.
  * type (string): Type of NFT to modify.
  * propertyName (string): Name of property to modify.
  * changeAmount (BigInteger): Amount to change property by. Note that for MULTIPLY properties the
      resulting property amount cannot be outside the range (0.01, 100)

* examples:
```
{
    "contractName": "mining",
    "contractAction": "changeNftProperty",
    "contractPayload": {
         "id": "TKN::TSTNFT",
         "type": "bear",
         "propertyName": "power",
         "changeAmount": "10"
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
  * nftTokenMiner = nft token miner config
  * active = whether the pool is active
  * updating.inProgress = whether mining power calculation is in process

## miningPower:
contains information about mining power for each account in pool
* fields
  * id = ID of mining pool
  * account = name of account
  * power.$numberDecimal = amount of mining power for the account in the pool.
  * balances = list of balances used to calculate mining power, in same order as the tokenMiners in the pool.
  * nftBalances = list of NFT property balances used to calculate mining power, in same order as the nftTokenMiner properties.

