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

&ast; **Special note for duration:** when basic service is used, duration starts at 2 weeks worth of blockchain blocks, and the block count is decremented whenever the smart contract updates state. This gradual usage of time stops if the user turns off bot service, and resumes when bot service is re-enabled. When the duration remaining hits 0 blocks, service will be suspended until the cooldown period elapses. At end of cooldown, duration is reset and the user is free to re-enable the service.

## Purpose of the Bot Controller

The botcontroller smart contract provides the account management scaffolding of the market maker, separating out configuration data from the actual logic used to place orders (which is contained in the marketmaker smart contract). The botcontroller smart contract provides actions for doing things such as registering a user for the market maker system, upgrading users to premium service, adding/removing/updating market trading configuration, and enabling/disabling the bot for users. In addition, this contract contains all the logic for managing fees & cooldowns as described above. 

# Actions available:
## Account Management
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
