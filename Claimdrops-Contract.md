Documentation written by [ali-h](https://github.com/ali-h)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* [Actions available](#actions-available)
  * [create](#create)
  * [claim](#claim)
  * [expire](#expire)
* [Tables available](#tables-available)
  * [params](#params)
  * [claimdrops](#claimdrops)

# Introduction

Claimdrops smart contract can be used to create limited time offerings for any token on hive-engine with specific price, pool & limits for everyone or a list of users.

# Actions available:

## create:
Creates a new claimdrops. A fee of 0.1 BEE for each claim plus 50 BEE for creation is required. Each Claimdrop has a pool from which quantity of each claim is deducted. a maximum claim limit for everyone or a list of users can be defined. Price is set in Hive Pegged Token (SWAP.HIVE) which is sent to the owner account/contract set by the user creating the claimdrop. Expiry can be upto 90 days from creation time.

* requires active key: yes
* can be called by: anyone (holding tokens)
* parameters:
  * symbol (string): token to claimdrop
  * price (string): price of 1 token
  * pool (string): max quantity of tokens to claimdrop
  * maxClaims (integer): maximum claims
  * expiry (date): expiration time (ex. 2020-12-10T05:45:00)
  * owner (string): username of owner or name of a contract (earnings and remaining pool will be sent here)
  * ownerType (string 'user' || 'contract'): type of owner
  * list (array): array of accounts who are eligible to claim, with thier maximum limits (i.e [account, limit])
  * limit (array): maximum buy/claim limit for every account
  Note: Only one parameter of `list` and `limit` is required. while `list` limits the claimdrop to accounts in the list, `limit` allows everyone to claim

* examples:
```
{
    "contractName": "claimdrops",
    "contractAction": "create",
    "contractPayload": {
        "symbol": "TKN",
        "price": "0.01",
        "pool": "1000",
        "maxClaims": 800,
        "expiry": "2020-12-28T00:00:00",
        "owner": "ali-h",
        "ownerType": "user",
        "list": [
          ["ali-h", "500"],
          ["aggroed", "500"],
          ["satoshi.real", "500"]
        ]
    }
}
```

with limit for everyone
```
{
    "contractName": "claimdrops",
    "contractAction": "create",
    "contractPayload": {
        "symbol": "TKN",
        "price": "0.01",
        "pool": "1000",
        "maxClaims": 800,
        "expiry": "2020-12-28T00:00:00",
        "owner": "ali-h",
        "ownerType": "user",
        "limit": "500"
    }
}
```

A successful action will emit a "create" event, e.g.
```
{
    "contract": "claimdrops",
    "event": "create",
    "data": {
        "claimdropId": "9a21888421bb6520fc6e5e33bec966af72212a11"
    }
}
```

## create:
Claim tokens. Price in HIVE.SWAP is calculated by the quantity. Price is deducted and sent to the claimdrop owner while claimant receives requested tokens.

* requires active key: yes
* can be called by: accounts eligible for the claimdrop
* parameters:
  * claimdropId (string): id of the claimdrop
  * quantity (string): quantity of tokens to claim

* examples:
```
{
    "contractName": "claimdrops",
    "contractAction": "create",
    "contractPayload": {
        "claimdropId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "quantity": "520"
    }
}
```

A successful action will emit a "claim" event, e.g.
```
{
    "contract": "claimdrops",
    "event": "claim",
    "data": {
        "claimdropId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "quantity": "520"
    }
}
```

## expire:
Expires a claimdrop. once a claimdrop is expired, remaining tokens in the pool (if any) are sent to the owner account.

* requires active key: yes
* can be called by: claimdrop's owner or creator
* parameters:
  * claimdropId (string): id of the claimdrop

* examples:
```
{
    "contractName": "claimdrops",
    "contractAction": "expire",
    "contractPayload": {
        "claimdropId": "9a21888421bb6520fc6e5e33bec966af72212a11",
    }
}
```

A successful action will emit an "expire" event, e.g.
```
{
    "contract": "claimdrops",
    "event": "expire",
    "data": {
        "claimdropId": "9a21888421bb6520fc6e5e33bec966af72212a11",
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

## params:
contains contract parameters such as the current fees
* fields
  * creationFee = the cost in BEE for claimdrop creation
  * feePerClaim = the cost in BEE per claim
  * maxExpiryTime = maximum expiry time in milliseconds

## claimdrops:
contains information about claimdrops
* fields
  * claimdropId = id of the claimdrop (unique)
  * symbol = symbol of token to claimdrop
  * price = price per token
  * remainingPool = remaining quantity of tokens to claim
  * remainingClaims = number of remaining claims
  * claims = array of succesfull claims
  * expiry: expiry timestamp
  * owner: owner of the claimdrop
  * ownerType: type of owner (user or contract)
  * sender: creator of the claimdrop
  * limit: maximul claim limit for everyone
  * list: list of accounts eligible to claim with max limits
