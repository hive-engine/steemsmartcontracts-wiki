Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

# Table of Contents

* [Enabling the market](#enabling-the-market)
  * actions:
  * [enableMarket](#enablemarket)
* [Managing Sell Orders](#managing-sell-orders)
  * actions:
  * [sell](#sell)
  * [changePrice](#changeprice)
  * [cancel](#cancel)
* [Buy (hit an existing sell order)](#buy-hit-an-existing-sell-order)
  * actions:
  * [buy](#buy)
* [Tables available](#tables-available)
  * [SYMBOLsellBook](#symbolsellbook)
  * [SYMBOLopenInterest](#symbolopeninterest)
  * [SYMBOLtradesHistory](#symboltradeshistory)

# Actions available:
## Enabling the market
### enableMarket:
Updates the user friendly name of an NFT.
* requires active key: no

* can be called by: Steem account that owns the NFT

* parameters:
  * name (string): name of the token (letters, numbers, whitespace only, max length of 50)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)

* example:
```
{
    "contractName": "nft",
    "contractAction": "updateName",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "My Awesome NFT"
    }
}
```
## Managing Sell Orders
### sell:
Updates the url of the token's project web site.
* requires active key: no

* can be called by: Steem account that owns the NFT

* parameters:
  * url (string): new url for the token (0 <= length <= 255)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)

* example:
```
{
    "contractName": "nft",
    "contractAction": "updateUrl",
    "contractPayload": {
        "symbol": "TESTNFT",
        "url": "https://mynewurl.com"
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions. The one exception is the NFT instance table, as the _id serves as the token ID used to refer to that particular token in data queries and various smart contract actions.

## SYMBOLinstances
Every NFT symbol has its own separate table to store NFT instances (issued tokens). The instance table name for a particular symbol is formed by taking the symbol and adding "instances" to the end of it. Thus, if you have an NFT called MYNFT, the MYNFTinstances table will store all the issued MYNFT tokens.
* fields
  * _id = the token ID number
  * account = the Steem account or smart contract that holds this particular token
  * ownedBy = indicates if this token is held in a Steem account or smart contract. For a Steem account, the value will be "u".  For a smart contract, the value will be "c".
  * lockedTokens = describes all regular Steem Engine tokens which are locked in this particular NFT instance. If there are no locked tokens, the value will be {}
  * properties = values of all the data properties for this particular NFT instance. If there are no data properties set, the value will be {}
  * **(optional)** delegatedTo = if this token is delegated, will contain information about which account or contract the token is delegated to. If there is no delegation, this field will not exist (will be undefined).
  * **(optional)** previousAccount = the Steem account or smart contract that previously held this particular token. Will only be set if the token has been burned or transferred at least once. If a token was bought on the market, previousAccount will be the NFT market contract itself.
  * **(optional)** previousOwnedBy = same meaning as ownedBy, but for the Steem account or smart contract that previously held this particular token. Will only be set if previousAccount is set.

The delegatedTo field has its own structure as follows:
* account = the Steem account or smart contract that the token is delegated to
* ownedBy = indicates if the token is delegated to a Steem account or smart contract. For a Steem account, the value will be "u".  For a smart contract, the value will be "c".
* **(optional)** undelegateAt = if this token is pending undelegation, this will be the timestamp when the undelegation will be completed (in milliseconds). If there is no pending undelegation, this field will not exist (will be undefined)

examples of typical token data:
```
{
    _id: 4,
    account: 'marc',
    ownedBy: 'u',
    lockedTokens: {},
    properties: {}
}

{
    _id: 605,
    account: 'gamecontract',
    ownedBy: 'c',
    lockedTokens: { ENG: '10', SPT: '0.5' },
    properties: { color: 'red', frozen: true }
}

{
    _id: 222,
    account: 'aggroed',
    ownedBy: 'u',
    lockedTokens: { ENG: '10', TKN: '0.01' },
    properties: { color: 'orange' },
    delegatedTo: {
        account: 'cryptomancer',
        ownedBy: 'u'
    }
}

{
    _id: 12345,
    account: 'testContract',
    ownedBy: 'c',
    lockedTokens: {},
    properties: { color: 'blue', edition: 1 },
    delegatedTo: {
        account: 'contract2',
        ownedBy: 'c',
        undelegateAt: 1528243200000
    },
    previousAccount: 'aggroed',
    previousOwnedBy: 'u'
}
```
