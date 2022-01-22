Documentation written by [eonwarped](https://github.com/eonwarped)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* [Voting System](#voting-system)
* Actions:
  * [createRewardPool](#createrewardpool)
  * [updateRewardPool](#updaterewardpool)
  * [setActive](#setactive)
  * [setMute](#setmute)
  * [resetPool](#resetpool)
  * [comment](#comment)
  * [commentOptions](#commentoptions)
  * [vote](#vote)
  * [tokenMaintenance](#tokenmaintenance)
* [Tables available](#tables-available)
  * [params](#params)
  * [rewardPools](#rewardpools)
  * [posts](#posts)
  * [votes](#votes)
  * [votingPower](#votingpower)

# Introduction

The comments contract allows a token issuer to set up
a post reward system
that allows stakers to assign value to posts on the
main chain. The voting mechanism mirrors that of 
Steem/Hive, with a few notable differences:

* Separate upvote/downvote voting power. How much a
  full power vote affects the value as well as how
  fast the power is consumed, and the regeneration
  time are all configurable.
* Does not use mana system where powered up stake is
  immediately usable.

# Voting System

The voting system uses a user's staked and delegated token amounts, along with voting power, to determine how much value a vote assigns to a given post.

Voting power is a numeric value between 0 and 10000, and each vote consumes an amount proportional to the vote weight. Roughly, a full 100% vote consumes (`votePowerConsumption / 10000`) fraction of the current voting power. Voting power replenishes at a linear rate configured by `voteRegenerationDays`.

Downvoting has a separate independently configured voting power.

Votes assign a certain quantity called rshares to the post which is determined from multiplying the consumed voting power by the user's stake (+delegated).

To determine the post value, we apply the reward curve to the rshares total, divide by pendingClaims (accumulated curved rshare total of posts in a (`cashoutWindowDays * 2`) window), and multiply by the rewardPool.

Every vote operation triggers a maintenance action that adds to the reward pool in accordance with the configured schedule, and pays out any posts that have cashoutTime in the past.


# Actions available:

### createRewardPool:
Create a new reward pool. A creation fee of 1000 BEE is required.
For now, only one reward pool per token is allowed.

* requires active key: yes
* can be called by: token issuer
* parameters:
  * symbol (string): Token symbol
  * config (object): Reward pool configuration
    * config.postRewardCurve (string): Post reward curve. Options are: "power" (r^a)
    * config.postRewardCurveParameter (BigNumber): Post reward curve parameter. For "power" curve, this is the exponent, "a" of "r^a". Use "1" for linear. The decimal precision of this parameter can be at most 2.
    * config.curationRewardCurve (string): Post reward curve. Options are: "power" (r^a)
    * config.curationRewardCurveParameter (BigNumber): Post reward curve parameter. For "power" curve, this is the exponent, "a" of "r^a". Use "1" for linear. The decimal precision of this parameter can be at most 2.
    * config.curationRewardPercentage (integer): When a post pays out, what percentage is allocated to curators. Must be between 0 and 100, inclusive.
    * config.cashoutWindowDays (integer): How long, in days, until a post is scheduled to pay out. Must be between 1 and 30, inclusive.
    * config.rewardPerInterval (BigNumber): How much to add to the reward pool every reward interval.
    * config.rewardIntervalSeconds (integer): How often to add to the reward pool.
    * config.voteRegenerationDays (integer): How long it takes to fully regenerate voting power from 0 to 10000.
    * config.votePowerConsumption (integer): How much vote power is consumed at full voting power for a 100% vote.
    * config.downvoteRegenerationDays (integer): How long it takes to fully regenerate downvoting power from 0 to 10000.
    * config.downvotePowerConsumption (integer): How much downvote power is consumed at full downvoting power for a 100% downvote.
    * config.stakedRewardPercentage (integer): What percentage of rewards should be given as staked. Should be between 0 and 100, inclusive.
    * config.tags (array of strings): Which tags should be looked at to index a post for this reward pool. This will also look at the community (stored in parent_permlink for the root post).
    * config.disableDownvote (boolean): Whether to disable downvotes.
    * config.ignoreDeclinePayout (boolean): Whether to ignore decline payout in comment options.
    * config.appTaxConfig (object): Configure an app tax for posts not using a designated app.
      * config.appTaxConfig.app (string): App to compare. Matches to a comment's `jsonMetadata.app` field.
      * config.appTaxConfig.percent (integer): Percent to deduct from non-curation portion of rewards. This portion is deducted first, then author beneficiary split is processed on the remaining amount. Must be between 1 and 100, inclusive.
      * config.appTaxConfig.beneficiary (string): Account to send deducted rewards to. Will be paid out fully liquid.
    * config.excludeTags (array of strings): Which tags should be ignored when indexing a post for this reward pool.
      


* examples:
```
{
    "contractName": "comments",
    "contractAction": "createRewardPool",
    "contractPayload": {
        "symbol": "TKN",
        "config": {
            "postRewardCurve": "power",
            "postRewardCurveParameter": "1",
            "curationRewardCurve": "power",
            "curationRewardCurveParameter": "0.5",
            "curationRewardPercentage": 50,
            "cashoutWindowDays": 7,
            "rewardPerInterval": "1.5",
            "rewardIntervalSeconds": 3,
            "voteRegenerationDays": 14,
            "downvoteRegenerationDays": 14,
            "stakedRewardPercentage": 50,
            "votePowerConsumption": 200,
            "downvotePowerConsumption": 2000,
            "tags": ["scottest"],
            "disableDownvote": true,
            "ignoreDeclinePayout": true,
            "appTaxConfig": {
                "app": "leofinance",
                "percent": 20,
                "beneficiary": "null"
            },
            "excludeTags": ["spam"]
        }
    }
}
```

A successful action will emit a "createRewardPool" event, e.g.:
```
{
    "contract": "comments",
    "event": "createRewardPool",
    "data": {
        "_id": 1,
    }
}
```

### updateRewardPool
Update a reward pool. An update fee of 100 BEE is required.
* requires active key: yes
* can be called by: token issuer
* parameters:
  * rewardPoolId (integer): ID of pool to modify.
  * config (object): Reward pool configuration
    * config.postRewardCurve (string): Post reward curve. Options are: "power" (r^a)
    * config.postRewardCurveParameter (BigNumber): Post reward curve parameter. For "power" curve, this is the exponent, "a" of "r^a". Use "1" for linear. The decimal precision of this parameter can be at most 2.
    * config.curationRewardCurve (string): Post reward curve. Options are: "power" (r^a)
    * config.curationRewardCurveParameter (BigNumber): Post reward curve parameter. For "power" curve, this is the exponent, "a" of "r^a". Use "1" for linear. The decimal precision of this parameter can be at most 2.
    * config.curationRewardPercentage (integer): When a post pays out, what percentage is allocated to curators. Must be between 0 and 100, inclusive.
    * config.cashoutWindowDays (integer): How long, in days, until a post is scheduled to pay out. Must be between 1 and 30, inclusive.
    * config.rewardPerInterval (BigNumber): How much to add to the reward pool every reward interval.
    * config.rewardIntervalSeconds (integer): How often to add to the reward pool.
    * config.voteRegenerationDays (integer): How long it takes to fully regenerate voting power from 0 to 10000.
    * config.votePowerConsumption (integer): How much vote power is consumed at full voting power for a 100% vote.
    * config.downvoteRegenerationDays (integer): How long it takes to fully regenerate downvoting power from 0 to 10000.
    * config.downvotePowerConsumption (integer): How much downvote power is consumed at full downvoting power for a 100% downvote.
    * config.stakedRewardPercentage (integer): What percentage of rewards should be given as staked. Should be between 0 and 100, inclusive.
    * config.tags (array of strings): Which tags should be looked at to index a post for this reward pool. This will also look at the community (stored in parent_permlink for the root post).
    * config.disableDownvote (boolean): Whether to disable downvotes.
    * config.ignoreDeclinePayout (boolean): Whether to ignore decline payout in comment options.
    * config.appTaxConfig (object): Configure an app tax for posts not using a designated app.
      * config.appTaxConfig.app (string): App to compare. Matches to a comment's `jsonMetadata.app` field.
      * config.appTaxConfig.percent (integer): Percent to deduct from non-curation portion of rewards. This portion is deducted first, then author beneficiary split is processed on the remaining amount. Must be between 1 and 100, inclusive.
      * config.appTaxConfig.beneficiary (string): Account to send deducted rewards to. Will be paid out fully liquid.
    * config.excludeTags (array of strings): Which tags should be ignored when indexing a post for this reward pool.
      

* examples:
```
{
    "contractName": "comments",
    "contractAction": "updateRewardPool",
    "contractPayload": {
        "rewardPoolId": 1,
        "config": {
            "postRewardCurve": "power",
            "postRewardCurveParameter": "1",
            "curationRewardCurve": "power",
            "curationRewardCurveParameter": "0.5",
            "curationRewardPercentage": 50,
            "cashoutWindowDays": 7,
            "rewardPerInterval": "1.5",
            "rewardIntervalSeconds": 3,
            "voteRegenerationDays": 14,
            "downvoteRegenerationDays": 14,
            "stakedRewardPercentage": 50,
            "votePowerConsumption": 200,
            "downvotePowerConsumption": 2000,
            "tags": ["scottest"],
            "disableDownvote": true,
            "ignoreDeclinePayout": true,
            "appTaxConfig": {
                "app": "leofinance",
                "percent": 20,
                "beneficiary": "null"
            },
            "excludeTags": ["spam"]
        }
    }
}
```

### setActive
Disable or enable a reward pool. Posts and votes will stop being processed while inactive. Posts pending payout will be processed on reactivation, but posts and votes that occured while inactive will not be reprocessed.
* requires active key: yes
* can be called by: token issuer
* parameters:
  * rewardPoolId (id): ID of pool to modify.
  * active (boolean): Whether the pool is active.

* examples:
```
{
    "contractName": "comments",
    "contractAction": "setActive",
    "contractPayload": {
        "rewardPoolId": 1,
        "active": true
    }
}
```

### setMute
Sets mute status for an account in a reward pool. 
Votes from a muted account will count for 0 rshares
and any payouts to the muted account will not pay out,
but a log will still be emitted for how much would
have paid out.
* requires active key: yes
* can be called by: token issuer
* parameters:
  * rewardPoolId (id): ID of pool to modify.
  * account (string): Account to set mute for.
  * mute (boolean): Whether account should be muted.

* examples:
```
{
    "contractName": "comments",
    "contractAction": "setMute",
    "contractPayload": {
        "rewardPoolId": 1,
        "account": "spammer",
        "mute": true
    }
}
```

### resetPool
Resets a reward pool. Reward pool is set to 0, 
timestamps are set to current, as if the pool
was just created. Note pending payouts will
pay out at the next reward interval.

* requires active key: yes
* can be called by: token issuer
* parameters:
  * rewardPoolId (id): ID of pool to modify.

* examples:
```
{
    "contractName": "comments",
    "contractAction": "resetPool",
    "contractPayload": {
        "rewardPoolId": 1
    }
}
```

### comment
Automatic action derived from comment on main chain.
This action cannot be called with custom json or by other
means.
* parameters:
  * author (string): Author of comment, derived from comment op sender.
  * permlink (string): Permlink of comment, derived from comment op.
  * rewardPools (array of integer): List of reward poolIDs. Can be specified if attaching using a hive engine transaction to the comment op via json_metadata.ssc.transactions[]. Will only accept the first params.maxPoolsPerPost reward pools.
  * parentAuthor (string): Parent author of comment, derived from comment op parent_author. Note that replies to indexed posts are automaticaly indexed as well to the same reward pools.
  * parentPermlink (string): Parent permlink of comment, derived from comment op parent_permlink. Used by main chain to specify community category.
  * jsonMetadata (object): Metadata of comment, derived from comment op json_metadata, used for deciding whether to index the comment. Specifically, we look up reward pools corresponding to json_metadata.tags.

* examples:
```
{
    "contractName": "comments",
    "contractAction": "comment",
    "contractPayload": {
         "author": "author",
         "permlink": "permlink",
         "rewardPoolIds": [1,2],
    }
}
```

A successful action will emit a "newComment" event for each reward pool, e.g.:
```
{
    "contract": "comments",
    "event": "newComment",
    "data": {
        "rewardPoolId": 1,
        "symbol": "TKN",
    }
}
```



### commentOptions
Automatic action derived from comment options op on main chain.
This action cannot be called with custom json or by other
means.
* parameters:
  * author (string): Author of comment to set options for, derived from comment_options op.
  * permlink (string): Permlink of comment to set options for, derived from comment_options op.
  * maxAcceptedPayout (string): Max accepted payout, derived from comment_options op, only parsed to determine if payout is being declined.
  * beneficiaries (array): Beneficiaries, derived from comment_options 
    * beneficiaries[].account: Account of beneficiary.
    * beneficiaries[].weight: Determines what percentage of the author reward to go to the beneficiary account. Must be an integer from 1 to 10000

### vote
Automatic action derived from vote op on main chain.
This action cannot be called with custom json or by other means.
* parameters
  * voter (string): Voter of vote op.
  * author (string): Author of comment to vote.
  * permlink (string): Permlink of comment to vote.
  * weight (integer): Weight of vote, out of 10000.

A successful action will emit a "newVote" or an "updateVote" event for each reward pool with the rshares added to the post, e.g.:
```
{
    "contract": "comments",
    "event": "newVote",
    "data": {
        "rewardPoolId": 1,
        "symbol": "TKN",
        "rshares": "2.1084201919",
    }
}
```

### tokenMaintenance
Not an action, but automatically performed before processing a comment action, once per block. Processes posts that are pending payout at a rate determined by `maintenanceTokensPerBlock` and `maxPostsProcessedPerRound`.

Note that after payout, the post and votes objects are deleted, but they emit events for how much authors, curators, and beneficiaries are paid:

```
{
    "contract": "comments",
    "event": "authorReward",
    "data": {
        "rewardPoolId": 1,
        "authorperm": "@author/perm"
```

## params:
contains contract parameters such as the current fees
* fields
  * setupFee: the cost in BEE to create a pool
  * updateFee: the cost in BEE to update a pool
  * maxPoolsPerPost: max number of reward pools a comment can be part of.
  * maxTagsPerPool: max number of tags a reward pool config can specify.
  * maintenenaceTokensPerBlock: number of tokens that can do reward maintenance per block that contains a comment op.
  * lastMaintenanceBlock: last sidechain block where maintenance occurred.
  * maxPostsProcessedPerRound: max number of posts to pay out each maintenance round.
  * voteQueryLimit: number of votes to process at a time (e.g. the limit parameter in fetching votes to pay out for a post). Defaults to 1000.

## rewardPools:
contains information about the pool
* fields
  * `_id`: numeric ID of reward pool.
  * symbol: symbol of reward pool.
  * rewardPool: amount currently in the reward pool.
  * lastRewardTimestamp: timestamp of when reward pool was last increased.
  * lastPostRewardTimestamp: timestamp of when post rewards were last computed.
  * createdTimestamp: timestamp of when the reward pool was created.
  * config: config of reward pool, see createRewardPool.
  * pendingClaims: accumulated post claims, roughly over (2 times the cashoutWindowDays) days. Decays and grows to match reward pool usage. A post's payout is determined by applying the reward curve to the rshares of a post, dividing by pendingClaims, and multiplying by the reward pool.
  * active: whether the reward pool is active.

## posts:
contains information about reward pool posts
* fields
  * rewardPoolId: reward pool ID
  * symbol: symbol of reward pool
  * authorperm: author and permlink of post
  * author: author of post
  * created: creation timestamp
  * cashoutTime: scheduled cashout timestamp
  * votePositiveRshareSum: sum of positive rshares assigned to post, for curation payout calculation.
  * voteRshareSum: sum of rshares, both upvote amd downvote.
  * declinePayout: whether payout was declined
  * lastPayout: last payout timestamp
  * totalPayoutValue: total value of post
  * curatorPayoutValue: curator portion of payout
  * beneficiariesPayoutValue: beneficiary portion of payout

## votes:
contains information about reward pool votes
* fields
  * rewardPoolId: reward pool ID
  * symbol: symbol of reward pool
  * authorperm: author and permlink of voted post
  * weight: weight of vote, max 10000
  * rshares: rshares assigned by this vote to the post
  * curationWeight: curation weight of vote
  * timestamp: timestamp of vote
  * voter: account of voter.
 
## votingPower:
contains information about an account's voting power
* fields
  * rewardPoolId: reward pool ID
  * account: account name
  * lastVoteTimestamp: timestamp of last vote or downvote
  * votingPower: voting power at last vote timestamp
  * downvotingPower: downvoting power at last vote timestamp
  * mute: whether account is muted
