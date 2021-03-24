Documentation written by [ali-h](https://github.com/ali-h)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* [Actions available](#actions-available)
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

* examples:
```
{
    "contractName": "nftauction",
    "contractAction": "bid",
    "contractPayload": {
        "auctionId": "9a21888421bb6520fc6e5e33bec966af72212a11",
        "bid": "1652"
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
        "bid": "1652"
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
  * expiryTimestamp = timestamp auction will be automatically settled/expired
  * bids = current active bids
  * currentLead = index of the current lead bid
  * lastLeadUpdate = timestamp of last lead bid
