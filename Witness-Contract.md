Documentation written by [eonwarped](https://github.com/eonwarped)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* Actions:
  * [register](#register)
  * [approve](#approve)
  * [disapprove](#disapprove)
  * [proposeRound](#proposeround)
* [Tables available](#tables-available)
  * [params](#params)
  * [witnesses](#witnesses)
  * [accounts](#accounts)
  * [approvals](#approvals)
  * [schedules](#schedules)

# Introduction

The witness contract, in conjunction with the P2P plugin, is the
system that allows a network of hive engine nodes to validate
the sidechain blocks being produced, and validators are rewarded
with tokens. A stake-based approval system determines which nodes
participate in the validation of blocks.

The block validation process occurs in rounds of 11 blocks
(determined by the witness contract parameter), and in each
round, 11 witnesses are chosen to validate 11 blocks. Validation
is done by performing hashes, which just combines the hashes of
the individual block hashes of the round. In the schedule,
 the first witness is designated to propose a round to the
other witnesses in the schedule, who then respond with the
computed hash. When 9/11 responses come back with a matching hash,
this information is recorded to the sidechain as a witness contract
operation, which advances the schedule and rewards the witnesses
of that round.

There are 10 top witness spots and 1 backup witnness spot.
The top six participate in every round while the backup is
randomly rotated into the last slot in proportion with their
approval weight.

# Actions available:

### register:
Register (or unregister) a witness.

* requires active key: yes
* parameters:
  * IP (string): IP of witness.
  * RPCPort (integer): RPC port of witness.
  * P2PPort (integer): P2P port of witness.
  * signingKey (string): Public key of witness to use for signing messages.
  * enabled (boolean): Whether to enable (or disable) the witness.

* examples:
```
{
    "contractName": "witnesses",
    "contractAction": "register",
    "contractPayload": {
        "IP" "1.2.3.4",
        "RPCPort": 5000,
        "P2PPort": 5001,
        "signingKey": "STM...",
        "enabled": true,
    }
}
```

### approve
Approve a witness.

* requires active key: yes
* parameters:
  * witness (string): Witness to approve.

* examples:
```
{
    "contractName": "witnesses",
    "contractAction": "approve",
    "contractPayload": {
        "witness": "eonwarped",
    }
}
```

### disapprove
Dispprove a witness.

* requires active key: yes
* parameters:
  * witness (string): Witness to disapprove.

* examples:
```
{
    "contractName": "witnesses",
    "contractAction": "disapprove",
    "contractPayload": {
        "witness": "badwitness",
    }
}
```

### proposeRound
Called by the current witness of the round after collecting
signed hashes from other witnesses. The P2P plugin handles
this process when enabled.

* requires active key: yes
* parameters:
  * roundHash (string): Computed hash of the round, derived from the individual block hashes.
  * signatures (array of [witness, signature]): Array of witness signatures. Signatures should be from the witnesses in the schedule from that round and should have more than the minimum signatures required by the contract.

* examples:
```
{
    "contractName": "witnesses",
    "contractAction": "proposeRound",
    "contractPayload": {
        "roundHash": "<hash>",
        "signatures": [["eonwarped", "sig1"], ... ],
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions.

## params:
contains contract parameters such as the current round information.
* fields
  * totalApprovalWeight = sum of witness approval weights.
  * numberOfApprovedWitnesses = number of approved witnesses.
  * lastVerifiedBlockNumber = last verified block number.
  * round = current round number.
  * lastBlockRound = the number of the last block in the current round.
  * currentWitness = current witness that is expected to propose the current round,
  * blockNumberWitnessChange = block number where the witness will change if the current round has not yet been verified.
  * lastWitnesses = array of previously scheduled proposing witnesses. The last entry is set to the current witness.

## witnesses:
contains information about the registered witnesses.
* fields
  * account = account of witness.
  * approvalWeight.$numberDecimal = total approval weight for the witness.
  * signingKey = public key used for signing.
  * IP = IP address of witness.
  * RPCPort = RPC port of witness.
  * P2PPort = P2P port of witness.
  * enabled = whether the witness is enabled.
  * missedRounds = total number of missed rounds for the witness.
  * missedRoundsInARow = number of missed rounds in a row for the witness.
  * verifiedRounds = total number of verified rounds for thr witness.
  * lastRoundVerified = number of last round verified by the witness.
  * lastBlockVerified = last verified block number by this witness as part of a scheduled round.

## accounts:
contains information about approving accounts.
* fields
  * account = account name.
  * approvals = number of witnesses approved by this account.
  * approvalWeight.$numberDecimal = approval weight of the account. Includes delegated stake.

## approvals:
contains approval information.
* fields
  * from = aporoving account
  * to = approved witness.

## schedules:
contains schedule for the current round.
* fields
  * witness = scheduled witness.
  * blockNumber = assigned block number for witness. The last witness of the round schedule is the one designated for round proposal.
  * round = round number of schedule.

