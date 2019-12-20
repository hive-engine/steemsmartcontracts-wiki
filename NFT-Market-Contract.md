Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

# Table of Contents

* [Enabling the market](#enabling-the-market)
  * actions:
  * [enableMarket](#enablemarket)
  * [setGroupBy](#setgroupby)
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
Before token holders can trade an NFT, the market must be enabled for a particular NFT symbol. Until this is done, no market orders can be placed. There are two steps required to enable the market:
1. Call the enableMarket action on the nftmarket contract.
2. Call the setGroupBy action on the nft contract.

These actions require active key authority, and can only be called by the Steem account that owns/created the NFT. The order in which you perform the actions is not important, but both of them must be done before any market orders can be placed. Market enablement is permanent, you CANNOT disable a market once it has been enabled.
### enableMarket:
Enables a market by creating necessary database tables for the given symbol.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)

* example:
```
{
    "contractName": "nftmarket",
    "contractAction": "enableMarket",
    "contractPayload": {
        "symbol": "TESTNFT"
    }
}
```
A successful enableMarket action will emit an "enableMarket" event:
``symbol``
example:
```
{
    "contract": "nftmarket",
    "event": "enableMarket",
    "data": {
        "symbol": "TESTNFT"
    }
}
```

### setGroupBy:
**Note:** this is an action on the nft contract, NOT nftmarket.

After you have created some data properties via the addProperty action, you can call setGroupBy in order to define a list of data properties by which market orders for NFT instances should be grouped by. You can think of this grouping as an index used to organize orders on the market, similar to how PeakMonsters groups Splinterlands cards according to type & gold foil status. NFT instances which have the same values for the designated data properties are considered equivalent as far as the market is concerned.

Consider the following points carefully before calling this action:

* Data properties which never change once set (i.e. read-only properties) are the best ones to use for this grouping.
* Long text strings do not make ideal properties to group by. Integers and boolean types make the best grouping.
* Numbers with fractional parts (for example 3.1415926) should be avoided due to possible rounding issues. Integers without fractional parts are ideal for grouping.
* This grouping **can only be set once!** You can't change it later on, so don't call this action until you are completely ready to do so.
* Token holders will not be able to place market orders until you have defined a valid grouping via this action.

* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * properties (array of string): list of data property names to set as the grouping. The schema for each property must have been previously created via the addProperty action.

* example:
```
{
    "contractName": "nft",
    "contractAction": "setGroupBy",
    "contractPayload": {
        "symbol": "TESTNFT",
        "properties": [ "level","isFood" ]
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
