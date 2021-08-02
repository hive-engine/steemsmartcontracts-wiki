Documentation written by [donchate](https://github.com/donchate)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
  * [DTF Naming](#dtf-naming)
* [Fund Actions](#fund-actions)
  * [createFund](#createFund)
  * [updateFund](#updateFund)
  * [setDtfActive](#setDtfActive)
* [Proposal Actions](#proposal-actions)
  * [createProposal](#createProposal)
  * [updateProposal](#updateProposal)
  * [disableProposal](#disableProposal)
  * [approveProposal](#approveProposal)
  * [disapproveProposal](#disapproveProposal)
* [Contract Integration](#contract-integration)
* [Tables available](#tables-available)
  * [params](#params)
  * [funds](#funds)
  * [proposals](#proposals)
  * [approvals](#approvals)
  * [accounts](#accounts)

# Introduction

This contract provides a decentralized token fund (DTF) that uses newly generated token supply to fund community supported proposals.
Configured by the token issuer, it allows flexibility in deployment by setting factors such as: amount of new token supply and what token is used for rewarding proposals, which token is used for stake-weighted voting, proposal duration, and optional proposal fees.

Every day, the newly created token supply is distributed to any number of active proposals which meet the on-going criteria of: proposal requested amount vs. available supply, and proposal stake-weighted ranking. Proposals that meet both criteria for the day will be funded. Anyone may submit a proposal, provided they pay the fee amount if one is required by the DTF. If a proposal has ranked to qualify for payment but there is not enough daily supply available, it will receive a partial payment consisting of the remaining supply for that period.

Token stakers are able to vote for as many proposals as they wish.

## DTF Naming

Within the contract, a DTF fund is identified by the payment token first, and voting token second, colon-separated. Example: ```TOKEN1:TOKEN2```

# Fund Actions

### createFund
This action creates and configures the token issuer's token fund.
Only one fund can be created for a given payment token / staking token combination.
A fee of 1000 BEE is required.

* requires active key: yes
* can be called by: token issuer
* parameters:
  * payToken (string): Token that will be paid out to ranked proposals
  * voteToken (string): Token that will be used for determining stake-weighted rank
  * voteThreshold (string): Minimum amount of stake required for proposal consideration
  * maxDays (string): Max duration of any proposal (between 1 and 730 days)
  * maxAmountPerDay (string): Max amount of payToken any proposal can request (greater than 0)
  * proposalFee (object) (optional): Configuration for proposal fees; may be omitted for no fee proposals
    * method (string): one of ```burn``` or ```issuer```, determining whether fee will be sent to the ```proposalFee.symbol``` issuer or burned
    * symbol (string): Token that will be used for fee
    * amount (string): Amount of fee

* example:
```
{
  "payToken": "GLD",
  "voteToken": "SLV",
  "voteThreshold": "1000",
  "maxDays": "365",
  "maxAmountPerDay": "10000",
  "proposalFee": {
    "method": "issuer",
    "symbol": "GLD",
    "amount": "100"
  },
  "isSignedWithActiveKey": true
}
```

### updateFund
This action updates the token issuer's token fund configuration.
Existing proposals continue to operate under the original parameters, changes affect updating or creating new proposals.
A fee of 300 BEE is required.

* requires active key: yes
* can be called by: token issuer
* parameters:
  * fundId (string): payment token first, and voting token second, colon-separated. Example: ```TOKEN1:TOKEN2```
  * voteThreshold (string): Minimum amount of stake required for proposal consideration
  * maxDays (string): Max duration of any proposal (between 1 and 730 days)
  * maxAmountPerDay (string): Max amount of payToken any proposal can request (greater than 0)
  * proposalFee (object) (optional): Configuration for proposal fees; may be omitted for no fee proposals
    * method (string): one of ```burn``` or ```issuer```, determining whether fee will be sent to the ```proposalFee.symbol``` issuer or burned
    * symbol (string): Token that will be used for fee
    * amount (string): Amount of fee

* example:
```
{
  "fundId": "GLD:SLV",
  "voteThreshold": "500",
  "maxDays": "90",
  "maxAmountPerDay": "10000",
  "proposalFee": {
    "method": "burn",
    "symbol": "GLD",
    "amount": "1"
  },
  "isSignedWithActiveKey": true
}
```

### setDtfActive
This action allows the token fund owner to suspend or activate the operation of the fund. While disabled, no proposals can be changed and no payouts occur.
Upon reactivation, the fund will not do any catch up calculations and will simply resume evaluating the proposals on the next tick.
Newly created funds are not automatically activated upon creation.
No fee is required.

* requires active key: yes
* can be called by: token issuer
* parameters:
  * fundId (string): payment token first, and voting token second, colon-separated. Example: ```TOKEN1:TOKEN2```
  * active (boolean): whether fund is active or not

* example:
```
{
  "fundId": "GLD:SLV",
  "active": true,
  "isSignedWithActiveKey": true
}
```

# Proposal Actions

### createProposal
This action creates a proposal in a given token fund.
A fee may be required, depending on the configuration of the token fund.

Proposal funding may be sent to a regular user account, or to another smart contract.
Proposals are automatically activated upon creation, and can only be disabled by ```disableProposal```.
In the case of a smart contract recipient, that contract may receive a payload via special action ```receiveDtfTokens```.
> _Other contracts may not be designed to receive tokens from the DTF, ensure the contract is compatible with the DTF before attempting to use it._


* requires active key: yes
* can be called by: anyone
* parameters:
  * fundId (string): payment token first, and voting token second, colon-separated. Example: ```TOKEN1:TOKEN2```
  * title (string): Brief headline describing the proposal between 1 and 80 characters
  * startDate (string YYYY-MM-DDThh:mm:ss.sssZ): Proposal start date at least 1 day in the future
  * endDate (string YYYY-MM-DDThh:mm:ss.sssZ): Proposal end date, not exceeding the max duration of the token fund
  * amountPerDay (string): Requested funding per day
  * authorPermlink (string): identifier to a Hive post/comment in the form of ``@author/permlink``
  * payout (object): Configuration for payout of the proposal amount
    * type (string): one of ```user``` or ```contract```
    * contractPayload (object) (optional): Custom object that is passed to the target contract's ```receiveDtfTokens``` action
    * name (string): name of user or contract

* example (user payment):
```
{
  "fundId": "GLD:SLV",
  "title": "A Big Community Project",
  "startDate": "2021-03-30T00:00:00.000Z",
  "endDate": "2021-04-30T00:00:00.000Z",
  "amountPerDay": "1000",
  "authorPermlink": "@abc123/test",
  "payout": {
    "type": "user",
    "name": "projectfund"
  },
  "isSignedWithActiveKey": true
}
```

* example (contract payment):
```
{
  "fundId": "GLD:SLV",
  "title": "A Big Community Project",
  "startDate": "2021-03-30T00:00:00.000Z",
  "endDate": "2021-04-30T00:00:00.000Z",
  "amountPerDay": "1000",
  "authorPermlink": "@abc123/test",
  "payout": {
    "type": "contract",
    "contractPayload": {
      "id": "1"
    },
    "name": "distribution"
  },
  "isSignedWithActiveKey": true
}
```

### updateProposal
This action allows limited updates to be made a proposal.

* requires active key: yes
* can be called by: proposal creator
* parameters:
  * id (string): proposal ID to update
  * title (string): Brief headline describing the proposal between 1 and 80 characters
  * endDate (string YYYY-MM-DDThh:mm:ss.sssZ): Proposal end date, not exceeding the max duration of the token fund, duration cannot be increased
  * amountPerDay (string): Requested funding per day, cannot be increased
  * authorPermlink (string): identifier to a Hive post/comment in the form of ``@author/permlink``

* example:
```
{
  "id": "1",
  "title": "The Biggest Community Project",
  "endDate": "2021-04-29T00:00:00.000Z",
  "amountPerDay": "1000",
  "authorPermlink": "@abc123/testers",
  "isSignedWithActiveKey": true
}
```

### disableProposal
This action allows a proposal creator to permanently disable a running proposal. Once disabled, it cannot be restarted.
No fee is required.

* requires active key: yes
* can be called by: proposal creator
* parameters:
  * id (string): proposal ID to disable

* example:
```
{
  "id": "1",
  "isSignedWithActiveKey": true
}
```

### approveProposal
This action allows anyone to approve a proposal.

* requires active key: no
* can be called by: anyone
* parameters:
  * id (string): proposal ID to approve

* example:
```
{
  "id": "1",
}
```

### disapproveProposal
This action allows anyone to disapprove a previously approved proposal.

* requires active key: no
* can be called by: anyone
* parameters:
  * id (string): proposal ID to disapprove

* example:
```
{
  "id": "1",
}
```
# Contract Integration
To configure your contract to handle payments from the DTF, add an action called ```receiveDtfTokens```.
Upon making a token transfer to a contract's balance, the DTF will also call this action and pass a payload configured by the proposal, as well as the symbol and quantity of tokens transferred.
This action should be configured to only accept calls from the ```tokenfunds``` contract. It should also verify the receipt of tokens, and then perform contract logic accordingly.

* example custom_json action:
```
{
    "contractName": "mycontract",
    "contractAction": "receiveDtfTokens",
    "contractPayload": {
        "data": { "id": "100" },
        "symbol": "TKN",
        "quantity": "1.00000000",
    }
}
```

* example contract action:
```
actions.receiveDtfTokens = async (payload) => {
  const {
    data, symbol, quantity,
    callingContractInfo,
  } = payload;

  if (!api.assert(callingContractInfo && callingContractInfo.name === 'tokenfunds', 'not authorized')) return;
  ...
}
```

# Tables available

## params:
contains internal contract parameters
* fields
  * dtfCreationFee = 1000 BEE
  * dtfUpdateFee = 300 BEE
  * dtfTickHours = 24 hours
  * maxDtfsPerBlock = 40
  * processQueryLimit = 1000

## funds:
contains the instance configurations
* fields
  * *_id = MongoDB internal primary key
  * payToken = token for funding proposals
  * voteToken = token for stake weighting
  * voteThreshold = minimum weight to process a proposal
  * maxDays = max duration of a proposal
  * maxAmountPerDay = max token amount a proposal can request per day
  * proposalFee = object describing the proposal creation fee
  * active = fund active or disabled
  * creator = fund creator account
  * lastTickTime = last time this fund was processed

## proposals:
* fields
  * _id = MongoDB internal primary key
  * fundId = fund identifier ```TOKEN1:TOKEN2```
  * title = Brief headline describing the proposal
  * startDate
  * endDate
  * amountPerDay = amount of funding requested per day
  * authorPermlink = identifier to a Hive post/comment in the form of ``@author/permlink``
  * payout = object describing how to pay out funding to user or contract
  * creator = proposal creator account
  * approvalWeight = current cumulative stake weight
  * active = proposal active or disabled

## approvals
* fields
  * _id = MongoDB internal primary key
  * from = from user account
  * to = approved proposal ID

## accounts
* fields
  * _id = MongoDB internal primary key
  * account = hive account name
  * weights = object describing staked weight for relevant tokens
