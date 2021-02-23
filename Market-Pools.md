Documentation written by [donchate](https://github.com/donchate)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
  * [Token Pair Naming](#token-pair-naming)
* [Administrative Actions](#administrative-actions)
  * [createPool](#createPool)
* [Swapping Actions](#swapping-actions)
  * [swapTokens](#swapTokens)
* [Liquidity Actions](#liquidity-actions)
  * [addLiquidity](#addLiquidity)
  * [removeLiquidity](#removeLiquidity)
* [Interface Integration](#interface-integration)
  * [Determining minimum amounts](#determining-minimum-amounts)
  * [Adding Liquidity](#adding-liquidity)
  * [Removing Liquidity](#removing-liquidity)
* [Tables available](#tables-available)
  * [params](#params)
  * [pools](#pools)
  * [liquidityPositions](#liquidityPositions)

# Introduction

This contract implements a constant product formula (CPF) automated market maker (AMM).
The AMM holds quantities of Hive Engine tokens whose relative price is measured according to its reserves. A constant product pricing function maps the quantities of the assets in reserves to their marginal price.
Hive Engine token holders may trade using this smart contract at the price specified by the pricing function, which is continually updated as reserves change after each trade.
They may also provide liquidity to allow for a certain pair of tokens to be traded.

Simplified pricing function: ```k = x * y```
```
where k is a constant
x, y are the available token reserves at time of trade
```

## Token Pair Naming

A price quote is defined as the price of one token in terms of another. Throughout the contract we show two tokens, the base token, which appears first and the quote token, which appears last. The price of the first token is always reflected in units of the second token.

# Actions available

### createPool
A trading pair must be created before liquidity can be added and trades made.
Only one pair may exist for any two tokens, whether it is defined as ```TOKEN1:TOKEN2``` or ```TOKEN2:TOKEN1``` is set by the initial creator.
A fee of 1000 BEE is required.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```

* example:
```
{
  "tokenPair": "GLD:SLV",
  "isSignedWithActiveKey": true
}
```
## Swapping Actions

### swapTokens
* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```
  * tokenSymbol (string): Valid and configured token symbol that is being traded per the ```tradeType```
  * tokenOut (string): Amount of tokenSymbol that is being traded per the ```tradeType```
  * tradeType (string): Either ```exactInput``` or ```exactOutput``` indicating the type of trade
  * maxSlippage (string): Amount of slippage tolerance (must be greater than 0% and less than 50%). The transaction will be cancelled if the price changes unfavorably by more than this percentage. A good default value is 0.5%-1%.

#### Trade Types
A trader may wish to receive automatically traded tokens for an exact amount of input tokens. In the example below: 1 GLD is requested out of the swap and the amount required will be automatically calculated and deducted from the holder's token balance.
* ```exactOutput``` example:
```
{
  "tokenPair": "GLD:SLV",
  "tokenSymbol": "GLD",
  "tokenAmount": "1",
  "tradeType": "exactOutput",
  "maxSlippage": "1",
  "isSignedWithActiveKey": true
}
```

A trader may wish to send an exact amount of tokens to the swap contract, and receive an automatically traded amount of tokens back. In the example below: 1 GLD is the amount being sent to the contract and a relative amount of SLV will be returned.
* ```exactInput``` example:
```
{
  "tokenPair": "GLD:SLV",
  "tokenSymbol": "GLD",
  "tokenAmount": "1",
  "tradeType": "exactInput",
  "maxSlippage": "1",
  "isSignedWithActiveKey": true
}
```

## Liquidity Actions
To provide the resources for trading to occur, liquidity must be sent and stored by the smart contract.

### addLiquidity
This action will send both tokens in a given pair to the smart contract to provide liquidity for traders to swap tokens.

If this is being called on a newly created market pool, the initial liquidity provider sets the starting price for the pair. If the pair tokens have orders in the HE order book, the initial price will be compared to the last price on the order book market for deviation beyond 1%. This is to avoid an unintentional arbitrage situation.

For existing pools, liquidity can only be added in such a way that the trading price is maintained. As such, the contract may adjust either base or quote quantity by a small factor to meet these requirements. If the adjustment exceeds the default or specified ```maxSlippage```, the liquidity will not be added.

If the provider is seeding a new pool, the number of shares they will receive will equal ```sqrt(x * y)```, where x and y represent the amount of each token provided.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```
  * baseQuantity (string): Amount to deposit into the base token reserve (first token in pair)
  * quoteQuantity (string): Amount to deposit into the quote token reserve (second token in pair)
  * maxSlippage (string) (optional): Amount of tolerance to price changes before the liquidity is added (must be greater than 0% and less than 50%). The transaction will be cancelled if the price changes unfavorably by more than this percentage. If unspecified, the default value is 1%.

* example:
```
{
  "tokenPair": "GLD:SLV",
  "baseQuantity": "1000",
  "quoteQuantity": "16000",
  "maxSlippage": 1,
  "isSignedWithActiveKey": true
}
```

### removeLiquidity
This action allows a liquidity provider to withdraw their tokens from the market pool.

* requires active key: yes
* can be called by: anyone
* parameters:
  * tokenPair (string): Trading pair name describing the two tokens that will be paired in the format ```TOKEN1:TOKEN2```
  * sharesOut (string): Percentage > 0 <= 100 - amount of liquidity shares to convert into tokens. Must be a whole number.

* example:
```
{
  "tokenPair": "GLD:SLV",
  "sharesOut": "50",
  "isSignedWithActiveKey": true
}
```

# Interface Integration
## Determining minimum amounts
To estimate the minimum token amount required to get an exact number of tokens out of a swap (```exactOutput```):
```
  const num = api.BigNumber(liquidityIn).times(amountOut);
  const den = api.BigNumber(liquidityOut).minus(amountOut);
  const amountIn = num.dividedBy(den);
```
To estimate the minimum token amount received when sending an exact number of tokens into a swap (```exactInput```):
```
  const num = api.BigNumber(amountIn).times(liquidityOut);
  const den = api.BigNumber(liquidityIn).plus(amountIn);
  const amountOut = num.dividedBy(den);
```
## Adding liquidity
To ensure that the addition of new liquidity doesn't move the price, a user can only add to both tokens in the pair in quantities that maintain the same price.
_Note: This does not apply to the initial liquidity provider, who sets the price if none is available from the internal oracle_

```
  const price = api.BigNumber(pool.quoteQuantity).dividedBy(pool.baseQuantity);
```

## Removing Liquidity
Liquidity is removed by specifying a % of shares to remove. The contract calculates the amount returned of each token based on the position shares relative to the pool ```totalShares```.
The amount of tokens returned can be estimated by the following formulas:
```
  const baseOut = api.BigNumber(sharesDelta).times(pool.baseQuantity).dividedBy(pool.totalShares).toFixed(baseToken.precision, api.BigNumber.ROUND_DOWN);
  const quoteOut = api.BigNumber(sharesDelta).times(pool.quoteQuantity).dividedBy(pool.totalShares).toFixed(quoteToken.precision, api.BigNumber.ROUND_DOWN);
```

# Tables available

## params:
contains internal contract parameters
* fields
  * poolCreationFee = the cost in BEE to start a market pool for a token pair

## pools:
contains the instance configurations
* fields
  * *_id = MongoDB internal primary key
  * *tokenPair = market pool token pair
  * baseQuantity = base token reserve
  * quoteQuantity = quote token reserve
  * basePrice = trading price base per quote (usual way of reading a pair)
  * quotePrice = trading price quote per base (reverse price)
  * baseVolume = total base tokens traded
  * quoteVolume = total quote tokens traded
  * totalShares = total shares issued to liquidity providers
  * precision = market pool token precision (maximum of either pair)
  * creator = account name of initial pool creator

## liquidityPositions:
* fields
  * _id = MongoDB internal primary key
  * account = account name of the LP
  * tokenPair = token pair in which there is a LP
  * shares = number of shares of the liquidity pool