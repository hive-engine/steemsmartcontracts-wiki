# NFT Airdrops Smart Contract
Documentation written by [bennierex](https://github.com/bennierex)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

## Table of Contents

- [NFT Airdrops Smart Contract](#nft-airdrops-smart-contract)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Actions available](#actions-available)
    - [`newAirdrop`](#newairdrop)
  - [Tables available](#tables-available)
    - [`params`](#params)
    - [`pendingAirdrops`](#pendingairdrops)

## Introduction

This Smart Contract allows NFT's to be airdropped, much like the Airdrops Contract for (fungible) tokens. This way, a substantial amount of NFT's can be transferred to many different accounts, without having to manually create each transaction.

## Actions available

### `newAirdrop`
Initiate a new airdrop. A fee of 0.01 BEE (can be changed by contract owner) for each NFT in the airdrop list is required (i.e 100 BEE/10,000 NFTs). Note that there must be enough tokens in the sender's account to initiate the airdrop.

* requires active key: **yes**
* can be called by: **anyone** (holding tokens and the NFT's to be airdropped)
* parameters:
  * `symbol` (string, required)  
    NFT symbol to airdrop
  * `list` (array, required)  
    array of objects `{to: account, ids: [array of ids]}`; the NFT's with the specified `ids` (array of strings) of type `symbol` will be transferred to `account`.
  * `startBlockNumber` (integer, optional, default: next blocknumber)  
    minimum Hive-Engine block number after which the airdrop distribution can start.
  * `softFail` (boolean, optional, default: `true`)  
    specifies whether or not the airdrop should fail (`false`) or continue (`true`) when encountering a transfer error.
  * `fromType` (string, optional, default: 'user')  
    in case the airdrop is initiated by another contract, this can be set to 'contract' to specify the NFT's to be airdropped are owned by the calling contract.  
    **NOTE:** This is currently _not_ supported by the NFT contract, and therefore not enabled via the param `enabledFromTypes`.

**Example**
```json
{
    "contractName": "nftairdrops",
    "contractAction": "newAirdrop",
    "contractPayload": {
        "symbol": "TUNZ",
        "list": [
            {"to": "bait002", "ids":["11", "12"]},
            {"to": "aggroed", "ids": ["13", "6", "4"]},
            {"to": "bennierex", "ids":["39"]},
            {"to": "cryptomancer", "ids":["16"]},
            {"to": "nftmarket", "toType": "contract", "ids": ["8", "9", "10"]}
        ],
        "startBlockNumber": 10000005
    }
}
```

**A successful initiation will emit a "newNftAirdrop" event**
```json
{
  "contract": "nftairdrops",
  "event": "newNftAirdrop",
  "data": {
    "airdropId": "9a21888421bb6520fc6e5e33bec966af72212a11",
    "sender": "bennierex",
    "senderType": "user",
    "symbol": "TUNZ",
    "startBlockNumber": 10000005
  }
}
```

After being initialized, distribution will automatically start from either the next block, or the block specified by `startBlockNumber`.

**Distribution emits an "nftAirdropDistribution" event**
```json
{
  "contract": "nftairdrops",
    "event": "nftAirdropDistribution",
    "data": {
      "airdropId": "9a21888421bb6520fc6e5e33bec966af72212a11",
      "symbol": "TUNZ",
      "transactionCount": 50
    }
}
```

**A finished airdrop emits an "nftAirdropFinished" event**
```json
{
  "contract": "nftairdrops",
    "event": "nftAirdropFinished",
    "data": {
      "airdropId": "9a21888421bb6520fc6e5e33bec966af72212a11",
      "symbol": "TUNZ"
    }
}
```

**Failure emits an "nftAirdropFailed" event**
```json
{
  "contract": "nftairdrops",
    "event": "nftAirdropFailed",
    "data": {
      "airdropId": "9a21888421bb6520fc6e5e33bec966af72212a11",
      "symbol": "TUNZ"
    }
}
```

## Tables available
**Note:** all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

### `params`
Contains contract parameters such as the current fees and transaction limits.
* `feePerTransaction` (default: 0.01 BEE)  
  Sets the fee (in `UTILITY_TOKEN_SYMBOL`) required to be paid per NFT transaction.
* `maxTransactionsPerAirdrop` (default: 10000)  
  Sets the maximum number of NFT's that can be airdropped in a single airdrop. (this is more likely limited by maximum Hive custom_json operation length)
* `maxTransactionsPerAccount` (default: 50)  
  Sets the maximum number of NFT's that can be airdropped to a single account.
* `maxTransactionsPerBlock` (default: 500)  
  Sets the maximum transactions that can be processed each block. Setting this to a too low value will cause airdrops with many transactions to take a long time to complete. If more than one airdrop is processed in a single block (see `maxAirdropsPerBlock`), the transaction limit is spread evenly over all the airdrops. Even if some of the airdrops do not use up all transactions.
* `maxAirdropsPerBlock` (default: 1)  
  Sets the maximum number of airdrops that can be processed in a single block. See also `maxTransactionsPerBlock` on how this affects the processing of pending airdrops.
* `processingBatchSize` (default: 50)  
  This setting is used internally when transferring NFT's to/from the contract. It is important to keep this setting at or below the `MAX_NUM_NFTS_OPERABLE` constant of the NFT Smart Contract, or else NFT airdrops with more transactions than the limit imposed by the NFT Smart Contract will fail.
* `enabledFromTypes` (default: ['user'])  
  A list of allowed values for the `fromType` payload value. Should match any or all values of the `ALLOWED_FROM_TYPES` constant.

### `pendingAirdrops`
Contains information about pending airdrops
* `isValid` (boolean)  
  Whether or not the airdrop data is still valid or should be removed from the db.
* `softFail` (boolean)  
  Specifies if the initiator requested the airdrop to fail upon encountering a transfer error.
* `blockNumber` (integer)  
  The Hive-Engine blocknumber after which to start the distribution.
* `airdropId` (string)  
  The unique ID for this airdrop.
* `symbol` (string)  
  The NFT symbol.
* `from` (string)  
  The account name that initated the airdrop.
* `fromType` (string, allowed values: 'user', 'contract')  
  The account type that initated the airdrop (user or contract).
* `list` (array of objects)  
  A list of the remaining airdrop transactions (will be empty when the airdrop has finished).
* `nftIds` (array of strings)  
  The id's for the NFT's that have still to
* `totalFee` (string)  
  The total fee that has been reserved/burned to initiate this airdrop.
