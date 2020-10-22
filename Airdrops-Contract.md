Documentation written by [ali-h](https://github.com/ali-h)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* [Actions available](#actions-available)
  * [initAirdrop](#initairdrop)
* [Tables available](#tables-available)
  * [params](#params)
  * [pendingAirdrops](#pendingairdrops)

# Introduction

The airdrops contract allows any user to airdrop hive-engine tokens. User can set up an airdrop with a custom list to send tokens or stake tokens without having to manually transact.

# Actions available:

## initAirdrop:
Initiate a new airdrop. A fee of 0.1 BEE for each account in the airdrop list is required (i.e 100 BEE/1000 accounts). Note that there must be enough tokens in the sender's accounts to initiate the airdrop. If distribution type is stake, Token must have enabled staking.

* requires active key: yes
* can be called by: anyone (holding tokens)
* parameters:
  * symbol (string): token to airdrop
  * type (string 'transfer' || 'stake'): distribution type, either stake or transfer
  * list (array): array of [account, quantity], `quantity` will be transfered/staked to `account`.

* examples:
```
{
    "contractName": "airdrops",
    "contractAction": "initAirdrop",
    "contractPayload": {
        "symbol": "TKN",
        "type": "stake",
        "list": [
          ["ali-h", "100"],
          ["aggroed", "71.5"],
          ["satoshi.real", "0.555223"]
        ]
    }
}
```

A successful action will emit a "initAirdrop" event, e.g.:
```
{
    "contract": "airdrops",
    "event": "initAirdrop",
    "data": {
        "airdropId": "9a21888421bb6520fc6e5e33ceb966af72212a11"
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

## params:
contains contract parameters such as the current fees
* fields
  * feePerTransaction = the cost in BEE per transaction
  * transactionPerBlock = number of maximum transaction in each block per airdrop
  * maxAirdropsPerBlock = number of maximum airdrops in each block

## pendingAirdrops:
contains information about pending airdrops
* fields
  * airdropId = id of airdrop (unique)
  * symbol = symbol of token to airdrop
  * type = distribution type
  * list = list of accounts to transact
  * blockNumber = block number in which airdrop was initiated
