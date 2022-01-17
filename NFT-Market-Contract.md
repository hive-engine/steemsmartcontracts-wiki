Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

**Note**: although this smart contract is available on both Steem Engine and Hive Engine, not all features are supported on Steem Engine.

# Table of Contents

* [Enabling the market](#enabling-the-market)
  * actions:
  * [enableMarket](#enablemarket)
  * [setGroupBy](#setgroupby)
* [Configuring the market](#configuring-the-market)
  * actions:
  * [setMarketParams](#setmarketparams)
* [Managing Sell Orders](#managing-sell-orders)
  * actions:
  * [sell](#sell)
  * [changePrice](#changeprice)
  * [cancel](#cancel)
* [Buy (hit an existing sell order)](#buy-hit-an-existing-sell-order)
  * actions:
  * [buy](#buy)
* [Tables available](#tables-available)
  * [params](#params)
  * [SYMBOLsellBook](#symbolsellbook)
  * [SYMBOLopenInterest](#symbolopeninterest)
  * [SYMBOLtradesHistory](#symboltradeshistory)

# Actions available:
## Enabling the market
Before token holders can trade an NFT, the market must be enabled for a particular NFT symbol. Until this is done, no market orders can be placed. There are two steps required to enable the market:
1. Call the enableMarket action on the nftmarket contract.
2. Call the setGroupBy action on the nft contract.

These actions require active key authority, and can only be called by the Steem or Hive account that owns/created the NFT. The order in which you perform the actions is not important, but both of them must be done before any market orders can be placed. Market enablement is permanent, you CANNOT disable a market once it has been enabled.
### enableMarket:
Enables a market by creating necessary database tables for the given symbol.
* requires active key: yes

* can be called by: Steem or Hive account that owns/created the NFT definition

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

* can be called by: Steem or Hive account that owns/created the NFT definition

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

## Configuring the market
**Note**: this section only applies to Hive Engine. These features are not available on Steem Engine.

After you have enabled the market, there are some optional settings that can be configured which will affect the behavior of the entire market. You can skip this step if desired; otherwise this configuration can be set & updated at any time by using the following action:
### setMarketParams:
Only available on Hive Engine. Sets market-wide configuration, or updates existing configuration. Once set, a piece of configuration can never be deleted, it can only be updated to a new value.
* requires active key: yes

* can be called by: Hive account that owns/created the NFT definition

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * **(optional)** officialMarket (string): Steem or Hive account to receive the market fee percentage of the total sale price whenever NFT instances are bought or sold on the market. If set, will always override the [marketAccount parameter of the buy action](#buy), which will instead be considered the agent account. Cannot be the same as the buyer / contract caller when the buy action is being used (i.e. a market can't buy from itself).
  * **(optional)** agentCut (integer): a whole number ranging from 0 to 10000 inclusive. Represents a percentage of the sale/purchase fee that will be sent to a designated agent account specified by buyers. These agent fees provide an incentive for construction of third-party marketplace apps by giving such apps a way to monetize themselves, while also protecting the interests of the NFT creator who may also wish to profit from the market. To calculate the agent cut percentage, divide this number by 10000. For example, a value of 500 is 5% (0.05). A value of 1234 is 12.34% (0.1234).
  * **(optional)** minFee (integer): a whole number ranging from 0 to 10000 inclusive. If set, will be enforced as the minimum allowed value to be set as the [fee parameter of the sell action](#sell).

Note that all of these settings are optional. You can have minFee set without using the agent parameters, or vice versa. However, in order for the agent cut to be applied to fees, both officialMarket and agentCut must be defined. Furthermore, it is perfectly valid and may even be desirable in some circumstances to have agentCut be set to 0 or 10000 (100%).

Keep in mind that agentCut is a percentage of the sale fee, **NOT** the total sale price. So if the fee set by a sell action is 500 (5%), and the agentCut is 2000 (20%), then agents will get 20% of that 5% fee, and the other 80% will go to the account specified by the officialMarket setting.

* example:
```
{
    "contractName": "nftmarket",
    "contractAction": "setMarketParams",
    "contractPayload": {
        "symbol": "TESTNFT",
        "officialMarket": "niftymart",
        "agentCut": 2000,
        "minFee": 500
    }
}
```
A successful setMarketParams action will emit a "setMarketParams" event with the updated info (not all fields may be set, depending on what settings are in use):
``symbol, oldOfficialMarket, officialMarket, oldAgentCut, agentCut, oldMinFee, minFee``
examples:
```
// example of setting minFee for the first time
{
    "contract": "nftmarket",
    "event": "setMarketParams",
    "data": {
        "minFee": 50
    }
}

// example of changing all settings at once
{
    "contract": "nftmarket",
    "event": "setMarketParams",
    "data": {
        "symbol": "TESTNFT",
        "oldOfficialMarket": "splinterlands",    // will only be here if officialMarket was previously set and is being changed to a new value
        "officialMarket": "peakmonsters",        // will only be here if officialMarket is being changed to a new value or set for the first time
        "oldAgentCut": 1200,                     // will only be here if agentCut was previously set and is being changed to a new value
        "agentCut": 1100,                        // will only be here if agentCut is being changed to a new value or set for the first time
        "oldMinFee": 50,                         // will only be here if minFee was previously set and is being changed to a new value
        "minFee": 250                            // will only be here if minFee is being changed to a new value or set for the first time
    }
}
```

## Managing Sell Orders
For now, the market only supports sell side orders. The ability to place bids will be added later. Also, only Steem accounts can place market orders. Smart contracts that hold tokens cannot currently use the market.

Sellers have the ability to put NFT instances up for sale, change the price of existing orders, and cancel existing orders. Buyers have the ability to buy (hit) existing sell orders.
### sell:
Puts one or more NFT instances up for sale. Separate market orders will be created for each individual token listed for sale. As with the regular token market, tokens up for sale are "locked" by transferring them to the NFT market contract for safekeeping. Tokens held by the market will be returned to their owners if the corresponding market order gets canceled. Unlike the regular token market, NFT market orders do not expire. Once listed for sale, a market order will persist until it is either canceled or bought.
* requires active key: yes

* can be called by: Steem or Hive account that holds the token(s) to be sold

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * nfts (array of string): list of NFT instance IDs to sell
  * price (string): price that each individual token should be sold at
  * priceSymbol (string): the regular token symbol that the seller wants to be paid in. Does not necessarily have to be STEEMP. Note that you cannot create multiple sell orders for the same NFT instance ID with different price symbols.
  * fee (integer): a whole number ranging from 0 to 10000 inclusive. Represents a percentage of the price that will be taken as a fee and sent to a designated market or agent account specified by buyers. These fees provide an incentive for construction of third-party marketplace apps by giving such apps a way to monetize themselves. To calculate the fee percentage, divide this number by 10000. For example, a fee value of 500 is 5% (0.05). A value of 1234 is 12.34% (0.1234). Must be >= minFee, if minFee has been set using the [setMarketParams action](#setmarketparams). 
  
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

* can be called by: Steem or Hive account that originally placed the order(s) to be modified

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

* can be called by: Steem or Hive account that originally placed the order(s) to be canceled

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
Buys one or more NFT instances that are currently listed for sale. The buyer must have enough tokens in his account to pay for all NFT instances; there is no concept of a partial fill, either you buy everything requested or nothing.
* requires active key: yes

* can be called by: Steem or Hive account

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * nfts (array of string): list of NFT instance IDs (NOT order IDs) that you want to buy
  * marketAccount (string): Steem or Hive account to receive the market fee percentage of the total sale price. Cannot be the same as the buyer / contract caller. Will be overridden by the officialMarket setting, if such has been set using the [setMarketParams action](#setmarketparams). This account will receive an agent fee instead, if agentCut has been set using the [setMarketParams action](#setmarketparams). So this field can be either the market account or agent account depending on how settings for a particular market are configured, or ignored altogether if officialMarket is set but agentCut is not set.
  * **(optional, only available on Hive Engine)** expPrice (string): The expected total price of all NFT instances to be bought in this transaction. If set, a check will be done to make sure this value matches the total price of the corresponding sell orders. If there is a price mismatch, then the purchase will fail. It is recommended that you always use this parameter in order to protect against last minute order changes.
  * **(optional, only available on Hive Engine)** expPriceSymbol (string): The expected price symbol of the NFT instances to be bought in this transaction. If set, a check will be done to make sure this value matches the price symbol of the corresponding sell orders. If there is a symbol mismatch, then the purchase will fail. It is recommended that you always use this parameter in order to protect against last minute order changes.
  
A maximum of 50 NFT instances can be bought in a single call of this action. You cannot fill your own orders, and all orders must have the same price symbol.

* example:
```
{
    "contractName": "nftmarket",
    "contractAction": "buy",
    "contractPayload": {
        "symbol": "TESTNFT",
        "nfts": [ "1","2","3","4" ],
        "marketAccount": "peakmonsters",
        "expPrice": "17.42477",
        "expPriceSymbol": "BEE"
    }
}
```
A successful purchase will emit a single "hitSellOrder" event with data on all NFT instances sold:
``symbol, priceSymbol, account: the buyer, ownedBy: u, sellers: data structure giving info on all sellers, paymentTotal: total sale price of all orders sold (after subtracting any market & agent fee), marketAccount: account that market fee goes to, feeTotal: total market fee of all orders sold, agentAccount: account that agent fee goes to, agentFeeTotal: total agent fee of all orders sold``
example:
```
{
    "contract": "nftmarket",
    "event": "hitSellOrder",
    "data": {
        "symbol": "TESTNFT",
        "priceSymbol": "BEE",
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
        "marketAccount": "splintermart",
        "feeTotal": "0.78411465",
        "agentAccount": "peakmonsters",
        "agentFeeTotal": "0.08712385"
    }
}
```
In the above example, 4 NFT instances are bought at once by @cryptomancer, from separate sellers. Three of those tokens were sold by @aggroed, who received a payment of 8.95353150 BEE, and one token was sold by @marc who received a payment of 7.60000000 BEE. The total payment amount distributed to the sellers for all 4 tokens was 16.55353150 BEE (8.95353150 + 7.60000000), with a market fee of 0.78411465 BEE and agent fee of 0.08712385 BEE. To get the total sale price of 17.42477000 BEE, add together paymentTotal, feeTotal, and agentFeeTotal. In this case, the fee for each order was 5%, and the agent cut was 10% of that fee (so splitting the 5% total fee into two parts, 90% of the fee for the market account, and 10% of the fee for the agent account).

Note that marketAccount, feeTotal, agentAccount, and agentFeeTotal are all optional fields, which will be included only if the respective fees exist (it is possible to have no fees at all, or for only a market fee or only an agent fee to be set).

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

## params
**indexes:** symbol

This table only exists on Hive Engine.  Market parameters set by the [setMarketParams action](#setmarketparams) will be stored in this table.
* fields
  * symbol = NFT symbol identifying which market these settings are for
  * officialMarket = official market account to receive purchase fees when NFT instances are sold
  * agentCut (integer) = a whole number between 0 and 10000, inclusive, which represents the percentage of purchase fees that should go to the agent account
  * minFee (integer) = a whole number between 0 and 10000, inclusive, which represents the minimum allowed fee for a sell order

See [setMarketParams action](#setmarketparams) for more details on how these settings are used.

## SYMBOLsellBook
**indexes:** ownedBy, account, nftId, grouping, priceSymbol

Every NFT symbol has its own separate table to store the sell side order book. The table name for a particular symbol is formed by taking the symbol and adding "sellBook" to the end of it. Thus, if you have an NFT called MYNFT, the MYNFTsellBook table will store all active sell orders. Note that this table will not exist if the enableMarket action has not been called yet.
* fields
  * account = the Steem or Hive account that created this particular order
  * ownedBy = indicates if this order was created by a user account or smart contract. As smart contracts are not supported for now, the value will always be "u".
  * nftId (string) = NFT instance ID for this particular order
  * grouping = holds a copy of the data property values which together uniquely identify how this market order should be grouped, according to the list set by the NFT contract setGroupBy action
  * timestamp (integer) = creation time of this order in milliseconds
  * price (string) = price of this order
  * priceDec (decimal) = price of this order as a decimal data type
  * priceSymbol = the token symbol that the seller wants payment in
  * fee (integer) = a whole number between 0 and 10000, inclusive, which represents the market fee percentage for this order

examples of typical sell orders:
```
{
    _id: 4,
    account: 'marc',
    ownedBy: 'u',
    nftId: '35',
    grouping: { level: '', color: '' },
    timestamp: 1527811200000,
    price: '8.000',
    priceDec: { '$numberDecimal': '8.000' },
    priceSymbol: 'TKN',
    fee: 500
}

{
    _id: 112,
    account: 'aggroed',
    ownedBy: 'u',
    nftId: '10',
    grouping: { type: '3', isRare: 'true' },
    timestamp: 1527811200000,
    price: '2.00000000',
    priceDec: { '$numberDecimal': '2.00000000' },
    priceSymbol: 'ENG',
    fee: 1000
}
```
A couple points about the grouping:
1. If an NFT instance does not have one of the grouped by data properties set, the grouping will contain an empty string '' for that data property, as in the first example above.
2. Grouping values will always be strings, regardless of the data property type, as shown in the second example above where the value 'true' is a boolean data type but represented here as a string.

We can also infer that the above examples are orders for two different NFT symbols, because the grouped by data property names are fixed per symbol according to the setGroupBy action (i.e. grouped by names will be the same for all orders of a particular NFT symbol, although of course the data property values can vary to distinguish the various groupings).

## SYMBOLopenInterest
**indexes:** side, priceSymbol, grouping

Every NFT symbol has its own separate table to store open interest metrics. The table name for a particular symbol is formed by taking the symbol and adding "openInterest" to the end of it. Thus, if you have an NFT called MYNFT, the MYNFTopenInterest table will store all open interest for that symbol. Note that this table will not exist if the enableMarket action has not been called yet.

Open interest is defined to be the number of currently open (active) orders, grouped according to the data property values in each order's grouping field. Note that an open interest entry for a particular combination of data property values won't exist until at least 1 order whose NFT instance has that combination of values is created. Once created, open interest entries never get deleted. Thus it is perfectly reasonable to have an entry with 0 current active orders.

The open interest count will be incremented as orders are created, and decremented as orders are cancelled or filled.

* fields
  * side = will be buy or sell depending on which side of the order book an order is on. Currently only sell side orders are supported, so every open interest entry will have this field set to 'sell'.
  * priceSymbol = the token symbol of the order's payment method
  * grouping = holds a copy of the data property values which together represent how these market orders should be grouped, according to the list set by the NFT contract setGroupBy action. Note that the combination of side, priceSymbol, and grouping uniquely identifies an open interest entry.
  * count (integer) = the number of currently open (active) orders for this combination of side, priceSymbol, and grouping. Will be 0 or greater.

examples of typical open interest entries:
```
{
    _id: 29,
    side: 'sell',
    priceSymbol: 'ENG',
    grouping: { level: '29', color: '' },
    count: 567
}

{
    _id: 800,
    side: 'sell',
    priceSymbol: 'STEEMP',
    grouping: { level: '5', color: 'red' },
    count: 0
}
```
**Counting number of open orders:** simply sum together the count field for all open interest entries in the table, querying by priceSymbol and side if desired.

## SYMBOLtradesHistory
**indexes:** priceSymbol, timestamp

Every NFT symbol has its own separate table to store trade history. The table name for a particular symbol is formed by taking the symbol and adding "tradesHistory" to the end of it. Thus, if you have an NFT called MYNFT, the MYNFTtradesHistory table will store all trade history for that symbol. Note that this table will not exist if the enableMarket action has not been called yet.

Only the last 24 hours worth of trade history is kept. Every time a trade is made, a check will be made to prune old information from this table.

* fields
  * type = will be buy or sell. Currently, the market only supports sell side orders. Thus all trades will have this field set to 'buy' (a trade is done when a buyer hits a sell order using the buy action).
  * account = the buyer's Steem or Hive account name
  * ownedBy = will always be u (smart contracts are not currently allowed to participate in the market)
  * counterparties = contains detailed information about the counterparties from which NFT instances are bought or sold (these will always be the sellers for now)
  * priceSymbol = the token symbol of the trade's payment method
  * price = the total price for all NFT instances purchased in this trade (inclusive of market fees)
  * **(optional)** marketAccount = the Steem or Hive account that received the market fee for this trade, if there is such a fee
  * **(optional)** fee = the total fee paid to the market account by the purchaser for this trade, if there is such a fee
  * **(optional)** agentAccount = the Hive account that received the agent fee for this trade, if there is such a fee
  * **(optional)** agentFee = the total agent fee paid to the agent account by the purchaser for this trade, if there is such a fee
  * timestamp (integer) = time of this trade in seconds
  * volume (integer) = number of NFT instances exchanged in this trade

an example of a trade involving multiple NFT instances bought from different sellers:
```
{
    _id: 12345,
    type: 'buy',
    account: 'cryptomancer',
    ownedBy: 'u',
    counterparties: [
        {
            account: 'aggroed',
            ownedBy: 'u',
            nftIds: [ '147' ],
            paymentTotal: '2.98451050'
        },
        {
            account: 'marc',
            ownedBy: 'u',
            nftIds: [ '503' ],
            paymentTotal: '7.60000000'
        }
    ],
    priceSymbol: 'BEE',
    price: '11.14159000',
    marketAccount: 'peakmonsters',
    fee: '0.55707950',
    timestamp: 1527811200,
    volume: 2
}
```
In the above example, 2 NFT instances were bought by @cryptomancer from @aggroed and @marc. The total price for both NFT instances, inclusive of all fees, was 11.14159000 BEE. From that amount, a fee of 0.55707950 BEE was subtracted and sent to the @peakmonsters account. The remaining amount was distributed as payment to the two sellers, with @aggroed receiving 2.98451050 BEE and @marc receiving 7.60000000 BEE. The market fee was 5% for each order hit, and there was no agent fee.

Here's another example involving an agent fee:

```
{
    _id: 45678,
    type: 'buy',
    account: 'cryptomancer',
    ownedBy: 'u',
    counterparties: [
        {
            account: 'aggroed',
            ownedBy: 'u',
            nftIds: [ '8320' ],
            paymentTotal: '2.98451050'
        }
    ],
    priceSymbol: 'BEE',
    price: '3.14159000',
    marketAccount: 'splintermart',
    fee: '0.12566360',
    agentAccount: 'samtheman',
    agentFee: '0.03141590',
    timestamp: 1527811200,
    volume: 1
}
```
In this example, just a single NFT instance was bought by @cryptomancer from @aggroed. The price of the NFT instance was 3.14159000 BEE (including all fees). From that amount, a market fee of 0.12566360 BEE was subtracted and sent to the official market account @splintermart. A separate agent fee of 0.03141590 BEE was also subtracted and sent to the agent account @samtheman. The fee set on the sell order was 5%, for a total fee of ```fee + agentFee``` = 0.15707950 BEE. That fee was split into the market & agent fees, with 80% (0.12566360 BEE) going to the market account and 20% (0.03141590 BEE) going to the agent account. So the agent cut was 20% of the total fee.

**Calculating daily volume metrics:** simply sum together the volume field for all trade history entries in the table, querying by priceSymbol if desired, to get the total number of NFT instances traded. Similarly, sum together the price field to get total value exchanged. Note that price is stored as a string, so you must convert to a floating point number in order to calculate the sum.
