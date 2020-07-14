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

# Actions available:

TODO: fill me in

