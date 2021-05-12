Documentation written by [ali-h](https://github.com/ali-h)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* [Actions available](#actions-available)
  * [setMarketParams](#setMarketParams)
  * [create](#create)
  * [bid](#bid)
  * [cancelBid](#cancelBid)
  * [cancel](#cancel)
  * [settle](#settle)
* [Tables available](#tables-available)
  * [params](#params)
  * [auctions](#auctions)

# Introduction

The NFT Auction contract lets users Auction thier NFT(s) in an automated way for a any token as price.

An auction can run upto 30 days after which it is automatially settled for the lead bid or expired if there are no bids. To take the lead a bid must be 5% greater than the previous bid (if any). After a lead bid, if there are no other bids in the next 24 hours, the auction is settled automatically. Bids can be cancelled anytime except the last 5 minutes when auction is about to be settled. Users can always update thier bids for a larger quantity. Auction owners can manually cancel or settle auction for any bid at any time. When auction is settled, NFTs are sent to the lead bidder and payment is sent to the owner. All other bids are refunded.

# Actions available:

## setMarketParams:
Sets market-wide configuration, or updates existing configuration. Once set, a piece of configuration can never be deleted, it can only be updated to a new value.
* requires active key: yes
* can be called by: Hive account that owns/created the NFT definition
* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * **(optional)** officialMarket (string): Hive account to receive the market fee percentage of the total sale price whenever NFT instances are sold in an auction. If set, will always override the [marketAccount parameter of the bid action](#bid), which will instead be considered the agent account.
  * **(optional)** agentFeePercent (integer): a whole number ranging from 0 to 10000 inclusive. Represents a percentage of the sale fee that will be sent to a designated agent account specified by bidders. These agent fees provide an incentive for construction of third-party marketplace apps by giving such apps a way to monetize themselves, while also protecting the interests of the NFT creator who may also wish to profit from the market. To calculate the agent cut percentage, divide this number by 10000. For example, a value of 500 is 5% (0.05). A value of 1234 is 12.34% (0.1234).
  * **(optional)** minFeePercent (integer): a whole number ranging from 0 to 10000 inclusive. If set, will be enforced as the minimum allowed value to be set as the [fee parameter of the create action](#create).

Note that all of these settings are optional. You can have minFeePercent set without using the agent parameters, or vice versa. However, in order for the agent cut to be applied to fees, both officialMarket and agentFeePercent must be defined. Furthermore, it is perfectly valid and may even be desirable in some circumstances to have agentFeePercent be set to 0 or 10000 (100%).

Keep in mind that agentFeePercent is a percentage of the sale fee, **NOT** the total sale price. So if the fee set by a create action is 500 (5%), and the agentFeePercent is 2000 (20%), then agents will get 20% of that 5% fee, and the other 80% will go to the account specified by the officialMarket setting.

* example:
```
{
    "contractName": "nftauction",
    "contractAction": "setMarketParams",
    "contractPayload": {
        "symbol": "TESTNFT",
        "officialMarket": "niftymart",
        "agentFeePercent": 2000,
        "minFeePercent": 500
    }
}
```

A successful setMarketParams does not emit any error or event.

## create:
Initiate a new auction. A fee of 1 BEE is required to set up an auction.

* requires active key: yes
* can be called by: anyone (holding NFTs)
* parameters:
  * symbol (string): NFT symbol
  * nfts (array): NFT ids to auction
  * minBid (string): minimum quantity anyone is allowed to bid
  * finalPrice (string): final price of the NFTs, if bid auction is settled immediately
  * priceSymbol (string): symbol of the token NFTs are to be sold in (i.e SWAP.HIVE)
  * feePercent (integer): a whole number ranging from 0 to 10000 inclusive. Represents a percentage of the price that will be taken as a fee and sent to a designated market or agent account specified by bidders. These fees provide an incentive for construction of third-party marketplace apps by giving such apps a way to monetize themselves. To calculate the fee percentage, divide this number by 10000. For example, a fee value of 500 is 5% (0.05). A value of 1234 is 12.34% (0.1234). Must be >= minFeePercent, if minFeePercent has been set using the [setMarketParams action](#setmarketparams).
  * expiry (date): time auction is settled/expired (i.e 2021-03-25T00:00:00)

* examples:
```
{
    "contractName": "nftauction",
    "contractAction": "create",
    "contractPayload": {
        "symbol": "NFTSR",
        "nfts": ["759", "18602", "18603"],
        "minBid": "100",
        "finalPrice": "25000",
        "priceSymbol": "SWAP.HIVE",
        "feePercent": 500,
        "expiry": "2021-04-20T12:30:00"
    }
}
```

A successful action will emit a "create" event, e.g.
```
{
    "contract": "nftauction",
    "event": "create",
    "data": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11"
    }
}
```

## bid:
Bid in an auction. same action can be used to update a bid for a larger quantity

* requires active key: yes
* can be called by: anyone (having enough tokens)
* parameters:
  * auctionId (string): ID of the auction to bid in
  * bid (string): bid quantity
  * marketAccount (string): Hive account to receive the market fee percentage of the total sale price. Will be overridden by the officialMarket setting, if such has been set using the [setMarketParams action](#setmarketparams). This account will receive an agent fee instead, if agentFeePercent has been set using the [setMarketParams action](#setmarketparams). So this field can be either the market account or agent account depending on how settings for a particular market are configured, or ignored altogether if officialMarket is set but agentFeePercent is not set.

* examples:
```
{
    "contractName": "nftauction",
    "contractAction": "bid",
    "contractPayload": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "bid": "1652",
        "marketAccount": "ali-h"
    }
}
```

A successful action will emit a "bid" event, e.g.
```
{
    "contract": "nftauction",
    "event": "bid",
    "data": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "account": "saamya",
        "bid": "1652",
        "marketAccount": "ali-h"
    }
}
```

In case of a bid update an "updateBid" event is emitted, e.g.
```
{
    "contract": "nftauction",
    "event": "updateBid",
    "data": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "account": "saamya",
        "oldBid": "1325",
        "newBid": "1652",
        "oldTimestamp": 1616586507000,
        "newTimestamp": 1616586557620,
        "oldMarketAccount": "auction-mart"
        "marketAccount": "ali-h"
    }
}
```

## cancelBid:
cancel bid in an auction.

* requires active key: yes
* can be called by: bidder
* parameters:
  * auctionId (string): ID of the auction to remove bid from

* examples:
```
{
    "contractName": "nftauction",
    "contractAction": "cancelBid",
    "contractPayload": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
    }
}
```

A successful action will emit a "cancelBid" event, e.g.
```
{
    "contract": "nftauction",
    "event": "cancelBid",
    "data": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "account": "theguru",
        "bid": "196.5"
    }
}
```

## cancel:
Cancel an active auction.

* requires active key: yes
* can be called by: Auction owner
* parameters:
  * auctionId (string): ID of the auction to cancel

* examples:
```
{
    "contractName": "nftauction",
    "contractAction": "cancel",
    "contractPayload": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
    }
}
```

A successful action will emit a "cancelAuction" event, e.g.
```
{
    "contract": "nftauction",
    "event": "cancelAuction",
    "data": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11"
    }
}
```

## settle:
Settle an active auction for lead bidder or a specific bidder.

* requires active key: yes
* can be called by: Auction owner
* parameters:
  * auctionId (string): ID of the auction to settle
  * **(optional)** account (string): username of the bidder to settle with

* examples:
```
{
    "contractName": "nftauction",
    "contractAction": "settle",
    "contractPayload": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "account": "saamya"
    }
}
```

A successful action will emit a "settleAuction" event, e.g.
```
{
    "contract": "nftauction",
    "event": "settleAuction",
    "data": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "symbol": "NFTSR",
        "seller": "ali-h",
        "nftIds": ["759", "18602", "18603"],
        "bidder": "saamya",
        "price": "1652",
        "priceSymbol": "SWAP.HIVE",
        "marketAccount": "auction-mart",
        "fee": "13.216"
        "agentAccount": "ali-h",
        "agentFee": "3.304",
        "timestamp": 1616586807000
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

## params:
contains contract parameters such as the current fees, maximum expiry time, cancel lock time etc.
* fields
  * creationFee = fee required to set up an auction
  * minBidIncrementPercent = percent a bid needs to increase to take the lead
  * cancelLockTimeMillis = time remaining in the auction settling when cancel action is locked
  * expiryTimeMillis = time after the last lead bid it takes to settle the auction
  * maxExpiryTimeMillis = max time an auction can run
  * auctionsPerBlock = max auctions to settle per block

## marketParams:
contains market parameters set by the [setMarketParams action](#setmarketparams) will be stored in this table.
* fields
  * symbol = NFT symbol identifying which market these settings are for
  * officialMarket = official market account to receive purchase fees when NFT instances are sold
  * agentFeePercent (integer) = a whole number between 0 and 10000, inclusive, which represents the percentage of purchase fees that should go to the agent account
  * minFeePercent (integer) = a whole number between 0 and 10000, inclusive, which represents the minimum allowed fee for a sell order

## auctions:
contains information about active auction
* fields
  * auctionId = id of auction (unique)
  * symbol = symbol of NFT token being sold
  * seller = auction owner
  * nftIds = id of the NFT(s) being sold
  * priceSymbol = token symbol in which NFTs are being sold
  * minBid = minimum quantity anyone can bid
  * finalPrice = final price to buy NFT(s) immediately
  * feePercent = a whole number between 0 and 10000, inclusive, which represents the market fee percentage for this auction
  * expiryTimestamp = timestamp auction will be automatically settled/expired
  * bids = current active bids
  * currentLead = index of the current lead bid
  * lastLeadUpdate = timestamp of last lead bid
