Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

# Table of Contents

TODO: fill me in

# Introduction

## Market Maker Bot

The botcontroller smart contract is one half of the market maker bot system (the other half being the marketmaker smart contract). This is a bot that any user on Steem Engine or Hive Engine can register for. The contract provides a range of settings configurable by market which will control its trading strategies. Individual users can modify these settings at will. Here are the major behavioral characteristics of the bot:

* The bot is **not** designed to work with NFTs, only regular tokens.

* The market maker is not a trading bot in the sense of something that attempts to execute strategies based on technical analysis of charting patterns (i.e. buy low / sell high). Rather, it attempts to make money by **playing the order book spreads** (place orders on both buy & sell side, profit on the difference between buy / sell prices).

* The market maker is not a taker (i.e. will not hit open orders). Rather, it is a maker (i.e. will only place orders which others must hit).

* For now there are no performance analytics; individual users need to proactively monitor performance and adjust configuration or stop using the bot if they judge it is not performing optimally.

## Basic vs Premium Models

The market maker supports a basic mode and a premium mode that requires users to stake tokens. Differences between them are as follows (this list may be expanded over time):

Feature | Basic | Premium
---- | ---- | ----
Cost | 100 ENG/BEE | another 100 ENG/BEE fee + 1000 ENG/BEE staked
Duration* | can be used for a total time of 2 weeks, then requires a 2 week "cooldown" before being used again | no usage restrictions while premium is active
Markets | can only trade on one market pair at a time, must have 200 ENG/BEE staked to configure a market | can trade on unlimited markets, must have an additional 200 ENG/BEE staked on top of the 1000 base amount, per market pair
Order Strategy | tries to keep orders on top of the book | tries to keep orders on top of the book<br><br>OR<br><br>additional strategies TBD
Change Settings | 1 ENG/BEE to adjust settings (turning off & on is free) | can adjust settings any time free of charge
Tick Speed** | every 200 blocks (about 10 minutes) | every 100 blocks (about 5 minutes) 

&ast; **Special note for duration:** when basic service is used, duration starts at 2 weeks worth of blockchain blocks, and the block count is decremented whenever the smart contract updates state. This gradual usage of time stops if the user turns off bot service, and resumes when bot service is re-enabled. When the duration remaining hits 0 blocks, service will be suspended until the cooldown period elapses. At end of cooldown, duration is reset and the user is free to re-enable the service.

&ast;&ast; **Tick speed definition:** the rate at which the market maker bot updates orders in response to changing market conditions. When the bot ticks, it attempts to update as many orders as it can, but in some periods of high activity it may delay ticking some accounts so that blockchain processing is not slowed down. Thus the tick speed should be regarded as a "best attempt", not a guarantee, and your account may tick slower depending on overall system conditions at the time. In such cases, premium service accounts will be prioritized for faster ticking.

## Purpose of the Bot Controller

The botcontroller smart contract provides the account management scaffolding of the market maker, separating out configuration data from the actual logic used to place orders (which is contained in the marketmaker smart contract). The botcontroller smart contract provides actions for doing things such as registering a user for the market maker system, upgrading users to premium service, adding/removing/updating market trading configuration, and enabling/disabling the bot for users. In addition, this contract contains all the logic for managing fees & cooldowns as described above. 

# Actions available:
## Account Management
These actions affect the state of your account.
### register:
Registers a Steem or Hive account to use the market maker system. A registration fee of 100 ENG or BEE is required. Once registered, an account stays registered permanently and cannot be unregistered (although trading for an account can always be turned off if desired).

**Important Note:** registering an account gives implicit consent that the account owner authorizes the Engine platform to place market orders on the account owner's behalf, autonomously without requiring explicit approval of each order. Furthermore, this also represents acknowledgement that the account owner agrees not to hold the Engine platform responsible for any negative financial impact on the account owner that results directly or indirectly from use of the market maker system, for any reason including but not limited to bugs in the smart contract, improper trading configuration, or disadvantageous market movements.
* requires active key: yes

* can be called by: Steem or Hive account

* parameters: none, the calling account will be the one registered

* example:
```
{
    "contractName": "botcontroller",
    "contractAction": "register",
    "contractPayload": {}
}
```

A successful register action will emit a "register" event: ``account``
example:
```
{
    "contract": "botcontroller",
    "event": "register",
    "data": {
        "account": "myaccountname"
    }
}
```

### upgrade:
Upgrades a previously registered account to premium service. An upgrade fee of 100 ENG or BEE is required and the account must have at least 1000 ENG or BEE staked. Some things to note:

1. ENG or BEE that is in the process of being unstaked will **not** count toward the staking requirement of 1000 tokens.
2. If an account is on cooldown, upgrading it will remove the cooldown.
3. It is possible for an account to lose premium status later on, for example if the amount of staked ENG or BEE falls below the minimumed required threshold of 1000 tokens.
4. If an account previously had premium status (but lost it), the upgrade fee doesn't need to be paid again. However the staking requirement must still be met.

If an account loses premium status later on, there may also be secondary side effects. For instance, premium users are allowed to have unlimited markets configured for bot trading, but basic service allows only one. So if you lose premium status but have multiple markets configured, they will all be disabled until you have removed the extra markets or upgraded to premium again.

* requires active key: yes

* can be called by: previously registered Steem or Hive account

* parameters: none, the calling account will be the one upgraded

* example:
```
{
    "contractName": "botcontroller",
    "contractAction": "upgrade",
    "contractPayload": {}
}
```

A successful upgrade action will emit an "upgrade" event: ``account``
example:
```
{
    "contract": "botcontroller",
    "event": "upgrade",
    "data": {
        "account": "myaccountname"
    }
}
```

### turnOff:
Completely disables an account. While disabled, the market maker will not place any orders on behalf of this account. However any existing orders will stay in place and need to be managed manually by the user. Also, time spent disabled does not count toward the usage time of a basic service account (see "special note for duration" in [Basic vs Premium Models](#basic-vs-premium-models) section above).

* requires active key: yes

* can be called by: previously registered Steem or Hive account

* parameters: none, the calling account will be the one turned off

* example:
```
{
    "contractName": "botcontroller",
    "contractAction": "turnOff",
    "contractPayload": {}
}
```

A successful action will emit a "turnOff" event: ``account``
example:
```
{
    "contract": "botcontroller",
    "event": "turnOff",
    "data": {
        "account": "myaccountname"
    }
}
```

### turnOn:
Re-enables a disabled account. Note that basic service accounts on cooldown cannot be turned on again until after the cooldown period expires. Once cooldown expires, accounts will not be automatically re-enabled. This action must be used to re-enable such an account (see "special note for duration" in [Basic vs Premium Models](#basic-vs-premium-models) section above).

* requires active key: yes

* can be called by: previously registered Steem or Hive account

* parameters: none, the calling account will be the one turned on

* example:
```
{
    "contractName": "botcontroller",
    "contractAction": "turnOn",
    "contractPayload": {}
}
```

A successful action will emit a "turnOn" event: ``account``
example:
```
{
    "contract": "botcontroller",
    "event": "turnOn",
    "data": {
        "account": "myaccountname"
    }
}
```

## Market Management
These actions control how the market maker bot places orders on individual markets configured for your account. They differ from the above [Account Management](#account-management) actions in that they don't have an account-wide effect, but are rather targeted at specific markets.

Note that none of these actions have an explicit account parameter, because they act only on configuration for the account that calls (triggers) the action. So the account is always implied.
### addMarket:
Instructs the market maker bot to begin placing orders on a new market for your account and, optionally, configure the trading parameters for the market. Accounts must have 200 ENG or BEE staked per market added. Basic service accounts are only allowed to add 1 market. Premium accounts can add unlimited markets.

Examples of staking requirements:
Markets | Requirement
---- | ----
Basic service, 1 market | 200 ENG or BEE staked
Premium service, 1 market | 1200 ENG or BEE staked (1000 base for premium + 200 for the market)
Premium service, 4 markets | 1800 ENG or BEE staked (1000 base for premium + 4x200 for the markets)

ENG or BEE that is in the process of being unstaked will **not** count toward the staking requirement. If the number of staked tokens falls below the total requirement for all markets, then all markets will be disabled until the staking requirement is once again met.
* requires active key: yes

* can be called by: previously registered Steem or Hive account

* parameters:
  * symbol (string): symbol of the token identifying the market to trade on (cannot be SWAP.HIVE or STEEMP)
  * **(optional)** maxBidPrice (string): the maximum price you’re willing to buy the token for, in SWAP.HIVE or STEEMP. The market maker bot will not place buy orders above this price.
  * **(optional)** minSellPrice (string): the minimum price you’re willing to sell the token for, in SWAP.HIVE or STEEMP. The market maker bot will not place sell orders below this price.
  * **(optional)** maxBaseToSpend (string): the maximum amount of SWAP.HIVE or STEEMP you’re willing to buy with in a single order. The market maker bot will not place buy orders for larger than this amount.
  * **(optional)** minBaseToSpend (string): the smallest amount of SWAP.HIVE or STEEMP you’re willing to buy with in a single order. The market maker bot will not place buy orders for less than this amount.
  * **(optional)** maxTokensToSell (string): the maximum amount of tokens you’re willing to sell in a single order. The market maker bot will not place sell orders for larger than this amount.
  * **(optional)** minTokensToSell (string): the smallest amount of tokens you’re willing to sell in a single order. The market maker bot will not place sell orders for less than this amount.
  * **(optional)** priceIncrement (string): the amount you want to increase/decrease the price by when placing new orders, in SWAP.HIVE or STEEMP. For example, if priceIncrement is 0.001 and the top-of-the-book buy price is 5.2, the bot will place a buy order for you at 5.201. Likewise if the top-of-the-book sell price is 5.5, the bot will place a sell order for you at 5.499.
  * **(optional)** minSpread (string): the minimum spread you desire to maintain between top-of-the-book bid and ask prices, in SWAP.HIVE or STEEMP. If the current spread is less than this amount, the market maker bot will not place orders.

  If any optional parameters are not specified, then sensible defaults will be set as follows:

  Parameter | Default
  ---- | ----
  maxBidPrice | 1000
  minSellPrice | 0.00000001
  maxBaseToSpend | 100
  minBaseToSpend | 1
  maxTokensToSell | 100
  minTokensToSell | 1
  priceIncrement | 0.00001
  minSpread | 0.00000001

* examples:
```
// add a market with default configuration
{
    "contractName": "botcontroller",
    "contractAction": "addMarket",
    "contractPayload": {
        "symbol": "EGG"
    }
}

// add a market and explicitly set some parameters, accepting defaults for the rest
{
    "contractName": "botcontroller",
    "contractAction": "addMarket",
    "contractPayload": {
        "symbol": "EGG",
        "maxBaseToSpend": "120",
        "priceIncrement": "0.005"
    }
}
```

A successful action will emit an "addMarket" event: ``account, symbol``
example:
```
{
    "contract": "botcontroller",
    "event": "addMarket",
    "data": {
        "account": "myaccountname",
        "symbol": "EGG"
    }
}
```

If any default trading configuration is overridden, there will also be an "updateMarket" event emitted as per below.

### updateMarket:
Updates the configuration for a previously added market. If your account is not premium, a 1 ENG or BEE service fee is required. Premium users can make unlimited updates for free.
* requires active key: yes

* can be called by: previously registered Steem or Hive account

* parameters:
  * symbol (string): symbol of the token identifying the market to trade on (cannot be SWAP.HIVE or STEEMP)
  * **(optional)** maxBidPrice (string): the maximum price you’re willing to buy the token for, in SWAP.HIVE or STEEMP. The market maker bot will not place buy orders above this price.
  * **(optional)** minSellPrice (string): the minimum price you’re willing to sell the token for, in SWAP.HIVE or STEEMP. The market maker bot will not place sell orders below this price.
  * **(optional)** maxBaseToSpend (string): the maximum amount of SWAP.HIVE or STEEMP you’re willing to buy with in a single order. The market maker bot will not place buy orders for larger than this amount.
  * **(optional)** minBaseToSpend (string): the smallest amount of SWAP.HIVE or STEEMP you’re willing to buy with in a single order. The market maker bot will not place buy orders for less than this amount.
  * **(optional)** maxTokensToSell (string): the maximum amount of tokens you’re willing to sell in a single order. The market maker bot will not place sell orders for larger than this amount.
  * **(optional)** minTokensToSell (string): the smallest amount of tokens you’re willing to sell in a single order. The market maker bot will not place sell orders for less than this amount.
  * **(optional)** priceIncrement (string): the amount you want to increase/decrease the price by when placing new orders, in SWAP.HIVE or STEEMP. For example, if priceIncrement is 0.001 and the top-of-the-book buy price is 5.2, the bot will place a buy order for you at 5.201. Likewise if the top-of-the-book sell price is 5.5, the bot will place a sell order for you at 5.499.
  * **(optional)** minSpread (string): the minimum spread you desire to maintain between top-of-the-book bid and ask prices, in SWAP.HIVE or STEEMP. If the current spread is less than this amount, the market maker bot will not place orders.

* examples:
```
{
    "contractName": "botcontroller",
    "contractAction": "updateMarket",
    "contractPayload": {
        "symbol": "EGG",
        "minBaseToSpend": "120",
        "priceIncrement": "0.005"
    }
}

{
    "contractName": "botcontroller",
    "contractAction": "updateMarket",
    "contractPayload": {
        "symbol": "DEC",
        "maxBidPrice": "100",        // effectively unlimited for this market (i.e. we don't care)
        "minSellPrice": "0.00001",
        "maxBaseToSpend": "200",
        "minBaseToSpend": "10",
        "maxTokensToSell": "60000",
        "minTokensToSell": "100",
        "priceIncrement": "0.00001",
        "minSpread": "0.00001"
    }
}

{
    "contractName": "botcontroller",
    "contractAction": "updateMarket",
    "contractPayload": {
        "symbol": "BETA",
        "minSellPrice": "10000"      // useful if you just want to slowly accumulate tokens without selling
    }
}
```

A successful action will emit an "updateMarket" event: ``account, symbol, old field value #1, new field value #1, old field value #2, new field value #2, ...``
example:
```
{
    "contract": "botcontroller",
    "event": "updateMarket",
    "data": {
        "account": "myaccountname",
        "symbol": "EGG",
        "oldMinBaseToSpend": "100",
        "newMinBaseToSpend": "120",
        "oldPriceIncrement": "0.001",
        "newPriceIncrement": "0.005"
    }
}
```

### removeMarket:
Completely removes all configuration for a market. The market maker bot will no longer place new orders on this market until you add it back using the [addMarket action](#addmarket). However any existing orders will stay in place and need to be managed manually by the user. Removing markets can be useful if your account loses premium status and you need to delete extra markets. In other circumstances, when you need to only temporarily stop the bot (for example in periods of extreme market volatility), it may be better to use the [disableMarket action](#disablemarket) instead.
* requires active key: yes

* can be called by: previously registered Steem or Hive account

* parameters:
  * symbol (string): symbol of the token identifying the market configuration to delete (cannot be SWAP.HIVE or STEEMP)

* example:
```
{
    "contractName": "botcontroller",
    "contractAction": "removeMarket",
    "contractPayload": {
        "symbol": "ALPHA"
    }
}
```

A successful action will emit a "removeMarket" event: ``account, symbol``
example:
```
{
    "contract": "botcontroller",
    "event": "removeMarket",
    "data": {
        "account": "myaccountname",
        "symbol": "ALPHA"
    }
}
```

### disableMarket:
Disables a market without removing its configuration. The market maker bot will no longer place new orders on this market until you re-enable it via the [enableMarket action](#enablemarket). However any existing orders will stay in place and need to be managed manually by the user. Note that disabled markets still count towards your maximum allowed number of markets (basic service accounts can have 1 market configured; premium service accounts can have unlimited markets).
* requires active key: yes

* can be called by: previously registered Steem or Hive account

* parameters:
  * symbol (string): symbol of the token identifying the market configuration to disable (cannot be SWAP.HIVE or STEEMP)

* example:
```
{
    "contractName": "botcontroller",
    "contractAction": "disableMarket",
    "contractPayload": {
        "symbol": "BETA"
    }
}
```

A successful action will emit a "disableMarket" event: ``account, symbol``
example:
```
{
    "contract": "botcontroller",
    "event": "disableMarket",
    "data": {
        "account": "myaccountname",
        "symbol": "BETA"
    }
}
```

### enableMarket:
Re-enables a previously disabled market, so that the market maker bot will begin placing orders for it again. To enable a market, the following 2 requirements must be met:

1. The number of markets configured for your account must not be greater than the maximum allowed (basic service accounts can have 1 market configured; premium service accounts can have unlimited markets).
2. You must have enough ENG or BEE staked to cover the staking requirements of all your markets, regardless of whether some are disabled or not.

For more info on staking requirements, refer to the [addMarket action section](#addmarket).

* requires active key: yes

* can be called by: previously registered Steem or Hive account

* parameters:
  * symbol (string): symbol of the token identifying the market configuration to enable (cannot be SWAP.HIVE or STEEMP)

* example:
```
{
    "contractName": "botcontroller",
    "contractAction": "enableMarket",
    "contractPayload": {
        "symbol": "EGG"
    }
}
```

A successful action will emit an "enableMarket" event: ``account, symbol``
example:
```
{
    "contract": "botcontroller",
    "event": "enableMarket",
    "data": {
        "account": "myaccountname",
        "symbol": "EGG"
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.
## params:
contains contract parameters such as the current fees
* fields
  * basicFee = the cost in ENG or BEE to register an account
  * basicSettingsFee = the cost in ENG or BEE to update market configuration
  * premiumFee = the cost in ENG or BEE to upgrade a basic service account to premium
  * premiumBaseStake = the amount of ENG or BEE that must be staked to maintain premium service
  * stakePerMarket = the amount of ENG or BEE that must be staked for each market an account has configured
  * basicDurationBlocks = the number of sidechain blocks that a basic service account is allowed to remain active before going into cooldown mode
  * basicCooldownBlocks = the number of sidechain blocks that an account in cooldown must wait before being able to turn itself back on
  * basicMinTickIntervalBlocks = the tick interval for a basic service account, measured in sidechain blocks (accounts are not guaranteed to tick at exactly this rate)
  * premiumMinTickIntervalBlocks = the tick interval for a premium service account, measured in sidechain blocks (accounts are not guaranteed to tick at exactly this rate)
  * basicMaxTicksPerBlock = the maximum number of basic service accounts that are allowed by the system to tick in each block
  * premiumMaxTicksPerBlock = the maximum number of premium service accounts that are allowed by the system to tick in each block

## users
contains top level configuration for each account registered with the market maker system
* fields
  * account = Steem or Hive account name
  * isPremium = is the account upgraded to premium service?
  * isPremiumFeePaid = has the upgrade fee for premium service been paid?
  * isOnCooldown = is the account currently on cooldown?
  * isEnabled = is the account currently turned on for market maker activity?
  * markets = the number of markets configured for this account
  * enabledMarkets = the number of configured markets that are currently enabled for this account
  * timeLimitBlocks = for basic service accounts, indicates the number of sidechain blocks remaining before the account will go into cooldown
  * lastTickBlock = the last sidechain block at which the account was ticked (i.e. at which the market maker bot updated order status for current market conditions)
  * creationTimestamp = date & time the account was registered, in Unix epoch milliseconds
  * creationBlock = the sidechain block at which the account was registered

There are database indexes on the account and lastTickBlock fields.

## markets
contains top level configuration for each account registered with the market maker system
* fields
  * account = Steem or Hive account name
  * symbol = symbol of the token identifying the market to trade on
  * precision = precision of this market's token (how many decimal places are allowed for fractional amounts)
  * strategy = indicates the trading strategy the market maker should follow for this market. Currently not used.
  * maxBidPrice = the maximum price you’re willing to buy the token for, in SWAP.HIVE or STEEMP. The market maker bot will not place buy orders above this price.
  * minSellPrice = the minimum price you’re willing to sell the token for, in SWAP.HIVE or STEEMP. The market maker bot will not place sell orders below this price.
  * maxBaseToSpend = the maximum amount of SWAP.HIVE or STEEMP you’re willing to buy with in a single order. The market maker bot will not place buy orders for larger than this amount.
  * minBaseToSpend = the smallest amount of SWAP.HIVE or STEEMP you’re willing to buy with in a single order. The market maker bot will not place buy orders for less than this amount.
  * maxTokensToSell = the maximum amount of tokens you’re willing to sell in a single order. The market maker bot will not place sell orders for larger than this amount.
  * minTokensToSell = the smallest amount of tokens you’re willing to sell in a single order. The market maker bot will not place sell orders for less than this amount.
  * priceIncrement = the amount you want to increase/decrease the price by when placing new orders, in SWAP.HIVE or STEEMP. For example, if priceIncrement is 0.001 and the top-of-the-book buy price is 5.2, the bot will place a buy order for you at 5.201. Likewise if the top-of-the-book sell price is 5.5, the bot will place a sell order for you at 5.499.
  * minSpread = the minimum spread you desire to maintain between top-of-the-book bid and ask prices, in SWAP.HIVE or STEEMP. If the current spread is less than this amount, the market maker bot will not place orders.
  * isEnabled = is the market currently enabled for market maker activity?
  * creationTimestamp = date & time the account was registered, in Unix epoch milliseconds
  * creationBlock = the sidechain block at which the account was registered

There are database indexes on the account and symbol fields.
