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
For now, the market only supports sell side orders. The ability to place bids will be added later. Also, only Steem accounts can place market orders. Smart contracts that hold tokens cannot currently use the market.

Sellers have the ability to put NFT instances up for sale, change the price of existing orders, and cancel existing orders. Buyers have the ability to buy (hit) existing sell orders.
### sell:
Puts one or more NFT instances up for sale. Separate market orders will be created for each individual token listed for sale. As with the regular token market, tokens up for sale are "locked" by transferring them to the NFT market contract for safekeeping. Tokens held by the market will be returned to their owners if the corresponding market order gets canceled. Unlike the regular token market, NFT market orders do not expire. Once listed for sale, a market order will persist until it is either canceled or bought.
* requires active key: yes

* can be called by: Steem account that holds the token(s) to be sold

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * nfts (array of string): list of NFT instance IDs to sell
  * price (string): price that each individual token should be sold at
  * priceSymbol (string): the regular token symbol that the seller wants to be paid in. Does not necessarily have to be STEEMP. Note that you cannot create multiple sell orders for the same NFT instance ID with different price symbols.
  * fee (integer): a whole number ranging from 0 to 10000 inclusive. Represents a percentage of the price that will be taken as a fee and sent to a designated market account specified by buyers. These fees provide an incentive for construction of third-party marketplace apps by giving such apps a way to monetize themselves. To calculate the fee percentage, divide this number by 10000. For example, a fee value of 500 is 5% (0.05). A value of 1234 is 12.34% (0.1234).
  
A maximum of 50 tokens can be put up for sale in a single call of this action. Note that tokens cannot be put on the market if they are currently being delegated to another account.

* example:
```
{
    "contractName": "nftmarket",
    "contractAction": "sell",
    "contractPayload": {
        "symbol": "TESTNFT",
        "nfts": [ "1","2","3" ],
        "price": "2.000",
        "priceSymbol": "ENG",
        "fee": 500
    }
}
```
A successful sell action will emit a "sellOrder" event for each market order created:
``account: the seller, ownedBy: u, symbol, nftId: NFT instance ID for this order, timestamp: time of order creation in milliseconds, price, priceSymbol, fee, orderId: ID of the newly created order``
example:
```
{
    "contract": "nftmarket",
    "event": "sellOrder",
    "data": {
        "account": "aggroed",
        "ownedBy": "u",
        "symbol": "TESTNFT",
        "nftId": "1",
        "timestamp": 1527811200000,
        "price": "2.00000000",
        "priceSymbol": "ENG",
        "fee": 500,
        "orderId": 1
    }
}
```

### changePrice:
Changes the price of one or more existing sell orders. Note that the price symbol cannot be changed (to do that you should cancel the sell order and place a new one).
* requires active key: yes

* can be called by: Steem account that originally placed the order(s) to be modified

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * nfts (array of string): list of NFT instance IDs (NOT order IDs) whose price you wish to change
  * price (string): new price that each individual token should be sold at
  
A maximum of 50 orders can be modified in a single call of this action. All orders must have the same price symbol.

* example:
```
{
    "contractName": "nftmarket",
    "contractAction": "changePrice",
    "contractPayload": {
        "symbol": "TESTNFT",
        "nfts": [ "1","2","3" ],
        "price": "15.666"
    }
}
```
A successful action will emit a "changePrice" event for each market order updated:
``symbol, nftId: NFT instance ID for this order, oldPrice, newPrice, priceSymbol, orderId: ID of the updated order``
example:
```
{
    "contract": "nftmarket",
    "event": "changePrice",
    "data": {
        "symbol": "TEST",
        "nftId": "1",
        "oldPrice": "3.14159000",
        "newPrice": "15.66600000",
        "priceSymbol": "ENG",
        "orderId": 1
    }
}
```

### cancel:
Cancels one or more existing sell orders. Upon an order's successful cancelation, the corresponding NFT instance held by the NFT market contract will be transferred back to the owning account.
* requires active key: yes

* can be called by: Steem account that originally placed the order(s) to be canceled

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * nfts (array of string): list of NFT instance IDs (NOT order IDs) whose orders you wish to cancel
  
A maximum of 50 orders can be cancelled in a single call of this action. All orders must have the same price symbol.

* example:
```
{
    "contractName": "nftmarket",
    "contractAction": "cancel",
    "contractPayload": {
        "symbol": "TESTNFT",
        "nfts": [ "1","2","3" ]
    }
}
```
A successful action will emit a "cancelOrder" event for each market order cancelled:
``account: the seller, ownedBy: u, symbol, nftId: NFT instance ID for this order, timestamp: time of order creation in milliseconds, price, priceSymbol, fee, orderId: ID of the cancelled order``
example:
```
{
    "contract": "nftmarket",
    "event": "cancelOrder",
    "data": {
        "account": "aggroed",
        "ownedBy": "u",
        "symbol": "TESTNFT",
        "nftId": "1",
        "timestamp": 1527811200000,
        "price": "2.00000000",
        "priceSymbol": "ENG",
        "fee": 500,
        "orderId": 1
    }
}
```

## Buy (hit an existing sell order)
### buy:
Buys one or more NFT instances that are currently listed for sale. 
* requires active key: yes

* can be called by: Steem account

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * nfts (array of string): list of NFT instance IDs (NOT order IDs) that you want to buy
  * marketAccount (string): Steem account to receive the market fee percentage of the total sale price
  
A maximum of 50 NFT instances can be bought in a single call of this action. You cannot fill your own orders, and all orders must have the same price symbol.

* example:
```
{
    "contractName": "nftmarket",
    "contractAction": "buy",
    "contractPayload": {
        "symbol": "TESTNFT",
        "nfts": [ "1","2","3","4" ],
        "marketAccount": "peakmonsters"
    }
}
```
A successful purchase will emit a "hitSellOrder" event for each NFT instance sold:
``symbol, priceSymbol, account: the buyer, ownedBy: u, sellers: data structure giving info on all sellers, paymentTotal: total sale price of all orders sold (after subtracting the market fee), feeTotal: total market fee of all orders sold``
example:
```
{
    "contract": "nftmarket",
    "event": "hitSellOrder",
    "data": {
        "symbol": "TESTNFT",
        "priceSymbol": "ENG",
        "account": "cryptomancer",
        "ownedBy": "u",
        "sellers": [
            { "account": "aggroed",
              "ownedBy": "u",
              "nftIds": [ "1","2","3" ],
              "paymentTotal": "8.95353150"
            },
            { "account": "marc",
              "ownedBy": "u",
              "nftIds": [ "4" ],
              "paymentTotal": "7.60000000"
            }
        ],
        "paymentTotal": "16.55353150",
        "feeTotal": "0.87123850"
    }
}
```
In the above example, 4 NFT instances are bought at once by @cryptomancer, from separate sellers. Three of those tokens were sold by @aggroed, who received a payment of 8.95353150 ENG, and one token was sold by @marc who received a payment of 7.60000000 ENG. The total payment amount distributed to the sellers for all 4 tokens was 16.55353150 ENG (8.95353150 + 7.60000000), and the market fee was 0.87123850 ENG. To get the total sale price of 17.42477000 ENG, add together paymentTotal and feeTotal. In this case, the fee for each order was 5%.

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
