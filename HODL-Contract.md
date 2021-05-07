Documentation written by [dannychain](https://github.com/DannyChain)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

- [Introduction](#introduction)
- [Actions](#actions-available):
  - [deposit](#deposit)
  - [withdraw](#withdraw)
- [Tables available](#tables-available)
  - [params](#params)
  - [hodlers](#hodlers)
  - [tokens](#tokens)
  - [lockUps](#lockUps)

# Introduction

The `hodl` contract is a lock-up smart contract HODL protocol
that creates a pool with the amount deposited in it and then applies
a fee to withdraws whenever users withdraw from it before their set lock-up period,
distributing part of that fee to the users that kept their deposits locked.

The total fee charged when a user withdraws before their lock-up period is
10% of their total lock-up amount being withdrawn. From that total fee:
70% will be distribute evenly to all `hodlers` that remained with their
tokens locked; 20% will be distribute to the account that issued the token
being withdrawn; 10% will be paid to the 'agents' involved in the deposit and
the withdraw actions (evenly split between `deposit` and `withdraw` agents).

Agent fees were implemented so the community are incentivized for building
front-ends for the `hodl` procotol, however if the agent field are not used
in the action payload it will default to `null`, effectively burning that part
of the withdrawn token.

TL; DR; It’s a trustless protocol destined to motivate long-term investment.

# Actions available:

### deposit:

Deposit a token into the HODL contract.

- requires active key: yes
- parameters:

  - token (string): Token symbol.
  - amount (string): Amount to be deposit.
  - period (integer): Period of the lock-up, in months. (min: 1 / max: 120)
  - agent (string): Agent for the deposit. (defaults to null if not defined on payload)

Example:

```js
{
    "contractName": "hodl",
    "contractAction": "deposit",
    "contractPayload": {
        "token" "SWAP.HIVE",
        "amount": "100.0",
        "period": 12,
    }
}
```

### withdraw:

Withdraw a token from the HODL contract.

- requires active key: yes
- parameters:

  - token (string): Token symbol.
  - lockupId (string): ID for the lockup being withdrawn.
  - agent (string): Agent for the withdraw. (defaults to null if not defined on payload)

Example:

```js
{
    "contractName": "hodl",
    "contractAction": "withdraw",
    "contractPayload": {
        "token" "SWAP.HIVE",
        "lockupId": "TXID0001"
    }
}
```

# Tables available:

Note: all tables below have an implicit \_id field that provides a unique numeric identifier for each particular object in the database. Most of the time the \_id field is not important, so we have omitted it from table descriptions when not relevant.

## params:

contains contract parameters such as tokens and their locked-up amounts.

- fields
  - amountLockedUp = array of objects containing the tokens and its amount currently locked-up on the contract.
  - withdrawPenaltyFee = 7% withdraw fee distribute to other hodlers. (only charged on withdraws made before lock-up period expires)
  - withdrawTokenFee = 2% withdraw fee distribute to token issuer. (only charged on withdraws made before lock-up period expires)
  - withdrawAgentFee = 1% withdraw fee distribute to agents. (only charged on withdraws made before lock-up period expires)

## lockUps:

contains information about all the lock-ups made (both active and those already withdrawn).

- fields
  - \_id = relavant, as a hodlers and tokens could have multiple lock-ups.
  - account = account username.
  - agent = array with `deposit` and `withdraw` agents, if they are used.
  - amount = amount initially deposited on contract.
  - bonus = amount of bonus already earned, to be received when withdrawing lock-up.
  - duration = duration of the lockup, specified in months.
  - effectiveAmount = amount \* durationBoost (calculated per token).
  - status = array containing logs relevant to that lock-up.
    - type = type of the log (Enum: `deposit` | `withdraw` | `bonus`).
    - date = time the action was made.
    - amount = amount involved.
  - token = token symbol.
  - unlockedAt = when the lock-up period will expire.
