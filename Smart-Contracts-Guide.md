Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

If you haven't already, skim through everything in [General System Design & Usage](https://github.com/hive-engine/steemsmartcontracts-wiki#general-system-design--usage) before proceeding any further.

For convenience, code links in this document are for Hive Engine, but path structure will be similar for Steem Engine versions. In order to pass code review, you will be expected to strictly follow ALL [dev best practices](#dev-best-practices) given below.

# Table of Contents

* [Quickstart](#quickstart)
* [Parts of a smart contract](#parts-of-a-smart-contract)
  * [Actions](#actions)
  * [Storage](#storage)
* [Development pipeline](#development-pipeline)
  * [Creating skeleton files](#creating-skeleton-files)
  * [Write contract code](#write-contract-code)
  * [Testing your contract](#testing-your-contract)
  * [Running lint](#running-lint)
  * [Submitting your code for review](#submitting-your-code-for-review)
  * [Post-release checks](#post-release-checks)
    * [Broadcasting custom json with Beem](#broadcasting-custom-json-with-beem)
    * [Querying the Engine API](#querying-the-engine-api)
* [Dev Best Practices](#dev-best-practices)
  * [Contract storage initialization](#contract-storage-initialization)
  * [Contract configuration](#contract-configuration)
  * [Contract libraries & module usage](#contract-libraries--module-usage)
  * [Working with numbers](#working-with-numbers)
  * [Input validation](#input-validation)
    * [isSignedWithActiveKey](#issignedwithactivekey)
    * [callingContractInfo](#callingcontractinfo)
    * [Verify user authorization](#verify-user-authorization)
    * [Checking account balance & paying fees](#checking-account-balance--paying-fees)
  * [Avoiding non-determinism](#avoiding-non-determinism)
  * [Data size limits](#data-size-limits)
  * [Performance-considerations](#performance-considerations)
  * [Emitting events](#emitting-events)
  * [Ticking actions in every block](#ticking-actions-in-every-block)
  * [Calling smart contracts from other smart contracts](#calling-smart-contracts-from-other-smart-contracts)

# Quickstart

1. Make sure you have a Linux server to develop on. Low end specs are fine: 2 GB RAM, a dual core CPU, and at least 10 GB free disk space should work great. Ubuntu is recommended, though other flavors of Linux will probably also work.

2. Install NodeJS and NPM. Node version 12+ is recommended: https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/

3. Install MongoDB. Version 4.2.3 is recommended (that's what production nodes are running): https://docs.mongodb.com/v4.2/administration/install-community/

4. Fork this repository: https://github.com/hive-engine/steemsmartcontracts

5. Create a branch for yourself. The base branch depends on what Engine you will be developing for:
    * For Hive Engine: create a branch off the **hive-engine** branch.
    * For Steem Engine: create a branch off the **witnesses** branch.
    * Normally, you should NOT use master!
  
    Here is the purpose of various important branches:
    Branch Name | Used For | Platform
    --- | --- | ---
    master | core node software releases for Steem Engine | Steem
    hive-engine | core node software releases for Hive Engine + smart contracts for Hive Engine | Hive
    witnesses | smart contracts for Steem Engine | Steem

6. In your branch, do `npm install`

7. Verify everything is working by running some unit tests. Do `npm run test1` to run tests for the nft smart contract. All tests should pass.

8. That's it, you're ready for smart contract dev work!

# Parts of a smart contract

Engine smart contracts are bits of Javascript code that run in a sandbox VM (Virtual Machine) environment. You can think of them as living on the Engine sidechain. They can only access on-chain data storage and have no access to outside data sources (so you can't query external web sites, REST APIs, etc). Everything you do on the Engine web site is powered by smart contracts behind the scenes: token transfers, market orders, staking, delegating, and more.

## Actions

A smart contract provides a public interface in the form of actions, which can be called by users (which will be either a Hive account or another smart contract). This concept should be familiar to anyone who knows object oriented programming; it's analogous to a class in most major programming languages.

Actions are called by broadcasting custom json transactions to the Hive blockchain (they can also be triggered through posts, but this is less conventional and typically only used to deploy or update smart contracts). The smart contracts node software detects the custom json transaction and executes the appropriate smart contract action, passing in any parameters contained within the custom json. Results of the action are recorded in the transaction logs for the next sidechain block.

It's important to keep in mind that all smart contract actions are treated as atomic in nature (they are single threaded and cannot be interrupted by other actions), and must finish executing within one sidechain block. Thus they must execute quickly enough not to cause delays in block production.

Here is an example of a typical smart contract action:

<img src="https://i.imgur.com/J0Ee6XC.png">

This is a **marketBuy** action for the **market** smart contract. The **ssc-mainnet-hive** id indicates this custom json should be processed by the Hive Engine mainnet sidechain.

## Storage

Smart contracts have their own on-chain database storage in the form of what we call tables (these are really MongoDB collections). Tables can be created as part of a contract's initialization action, and then are permanently associated with the contract that created them. Only the owner contract can update data in its tables, but any contract can read them. Contract tables can also be queried externally through the node's API.

Refer to the [database API section](https://github.com/hive-engine/steemsmartcontracts-wiki/blob/master/Database-API.md) for more details.

# Development pipeline

The following sections describe the overall steps in the smart contract development process, from start to finish.

## Creating skeleton files

Each smart contract consists of two files: a .js file containing the contract code, and a .js file containing unit test code. Both the test file and contract file should have the same name. Also, the file name should be the same as the contract name itself which by convention should be all lowercase with no spaces or dashes between words. Keep names short and to-the-point.

Smart contract code goes here: **steemsmartcontracts/contracts**

Unit tests go here: **steemsmartcontracts/test**

Do NOT mix test code up with contract code, or vice versa! They must be strictly separated in the above two directories.

To get started, you can copy one of the existing test & contract files and then modify it as needed. These files are small ones so good to copy for this purpose:

https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/test/crittermanager.js
<br>
https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/crittermanager.js

The mining test cases are also good to take a look at, as they use a shortcut style for assertions which you may prefer when writing large test cases:

https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/test/mining.js

## Write contract code

When writing your smart contract & test cases, use the existing contracts as a model. Similar code structure and style should be maintained for all contracts, and if you violate existing style guidelines your code may be rejected during the review process.

Refer to the [dev best practices section](#dev-best-practices) below for details you should be aware of.

## Testing your contract

Generally speaking, I write test cases and smart contract code in parallel as I go. You don't want to wait until your contract is finished before testing it, as you might find the contract doesn't work and then it's harder to debug what went wrong. Typically I write a small bit of smart contract code, just a single callable action or piece of an action, then I write a test case for it. I go back and forth, writing small pieces of the contract then thoroughly testing it before moving on.

There is no need to run your own node to test against. Unit tests are sufficient to be confident your smart contract will work once deployed. The unit test environment closely simulates a real node environment. 100% test coverage is required for all smart contracts. 90% just won't cut it. Yes, writing test cases will often take longer than writing the contract itself. That is expected. You simply can't be too careful when dealing with this stuff; bugs in production can have far more disastrous consequences than for normal desktop programs.

Test cases are expected to have both a negative and positive version for each publicly exposed smart contract action. For example, consider an NFT transfer operation. There is a [transfers tokens](https://github.com/hive-engine/steemsmartcontracts/blob/c11ce22b93d6160967c00560a79e83b67bfcb3e1/test/nft.js#L1281) test to verify tokens can be transferred OK from one account to another (that's the positive test); there is also a [does not transfer tokens](https://github.com/hive-engine/steemsmartcontracts/blob/c11ce22b93d6160967c00560a79e83b67bfcb3e1/test/nft.js#L1410) test that checks all the different ways a transfer action can fail (that's the negative test). It's OK to combine both positive and negative tests into one test case if that is more convenient for a particular testing scenario.

To run a test, add a line for it in **package.json**:

```
"name": "steemsmartcontracts",
  "version": "0.1.23",
  "description": "",
  "main": "app.js",
  "scripts": {
    "start": "node --max-old-space-size=8192 app.js",
    "lint": "eslint .gitignore .",
    "test": "./node_modules/mocha/bin/mocha  --recursive",
    "test0": "./node_modules/mocha/bin/mocha ./test/hivesmartcontracts.js",
    "test1": "./node_modules/mocha/bin/mocha ./test/nft.js",
    "test2": "./node_modules/mocha/bin/mocha ./test/tokens.js",
    "test3": "./node_modules/mocha/bin/mocha ./test/nftmarket.js",
    "test4": "./node_modules/mocha/bin/mocha ./test/smarttokens.js",
    "test5": "./node_modules/mocha/bin/mocha ./test/botcontroller.js",
    "test6": "./node_modules/mocha/bin/mocha ./test/crittermanager.js",
    "test7": "./node_modules/mocha/bin/mocha ./test/market.js",
    "test8": "./node_modules/mocha/bin/mocha ./test/packmanager.js"
  },
```

Above, test8 is the last test. So you could add yours as test9, putting the path to your new test file. Then you'd run the tests with `npm run test9`.

## Running lint

Once your contract and tests are finished, you should run lint to verify your contract code obeys established coding style standards. Do ```npm run lint > lint_output.log```
This will take a few minutes as it will run lint checks for ALL contracts, not just your new one. Check the output file and fix any indicated problems with your contract. All lint checks must pass in order for your code to be accepted. It is OK to disable lint tests for certain lines of code if you have a good reason for doing so. There are plenty of examples of this scattered around the contracts, for example this snippet in the nft contract:

```
if (Object.keys(properties).length === 0) {
  // eslint-disable-next-line no-continue
  continue; // don't bother processing empty properties
}
```

However, try not to do that too much.

## Submitting your code for review

When you are finished, check in all code & tests to your branch and raise a PR (to merge into the **hive-engine** branch if you are developing for Hive Engine). Then notify @cryptomancer on Discord that you have submitted your PR. A code review will be done, to ensure you are following smart contract best practices & all test cases are passing. Note that github itself will automatically run unit tests & lint checks to verify there are no issues with your code.

Feedback will be given during the code review process; you may be required to make changes if there are any potential areas of concern.

Once the code review is passed, your PR will be merged into the main repository and your smart contract scheduled for production deployment.

## Post-release checks

Once your smart contract is deployed, you can perform post-release checks to verify it's working as intended. There are numerous ways to do this; I prefer using Python with Beem to broadcast custom json transactions, then verify contract data state using Postman.

### Broadcasting custom json with Beem

Follow the directions here to install Beem: https://github.com/holgern/beem

Here's a snippet of Python code for broadcasting an example custom json:

```
#!/usr/bin/python3

from beem import Hive
from beem.transactionbuilder import TransactionBuilder
from beembase import operations
from beem.instance import set_shared_hive_instance

hiveNode = 'https://api.hive.blog'
chainId = 'ssc-mainnet-hive'

if __name__ == '__main__':
    stm = Hive(hiveNode)
    set_shared_hive_instance(stm)

    # uncomment this to initialize Beem wallet
    #stm.wallet.wipe(True)
    #stm.wallet.create("my password")
    #stm.wallet.addPrivateKey("put private key here")

    stm.wallet.unlock("my password")

    src = "myaccountname"
    tx = TransactionBuilder()
    op = operations.Custom_json(**{"required_auths": [ src ],
                        "required_posting_auths": [],
                        "id": chainId,
                        "json": {
                           "contractName": "botcontroller",
                           "contractAction": "enableMarket",
                           "contractPayload": {
                               "symbol": "LEO"
                           }
                        } })
    tx.appendOps(op)
    #tx.appendSigner(src, 'posting')
    tx.appendSigner(src, 'active')

    print("broadcasting custom json transaction for", src, ":", tx)
    try:
        tx.sign()
        tx.broadcast()
        print("SUCCESS")
    except Exception as e:
        print("FAILED:", e)
```

### Querying the Engine API

For quick testing, I recommend using Postman: https://www.postman.com/downloads/

Postman allows you to easily broadcast POST queries to the Engine web API so you can examine the state of the sidechain and check smart contract activity. For details of all the various API calls available, refer to the [JSON RPC server section](https://github.com/hive-engine/steemsmartcontracts-wiki/blob/master/JSON-RPC-server.md).

Let's look at an example of how to query for DEC market orders, by fetching data from the market contract's sellBook table. First, in Postman make sure the query type is POST. The API endpoint is:

* Hive Engine: **https://api.hive-engine.com/rpc/contracts/**
* Steem Engine: **https://api.steem-engine.com/rpc/contracts/**

Body to send:

```
{
    "jsonrpc": "2.0",
    "method": "find",
    "params": {
       "contract": "market",
       "table": "sellBook",
       "query": {"symbol": "DEC"},
       "limit": 100,
	    "offset": 0,
	    "indexes": [{ "index": "priceDec", "descending": false }, { "index": "_id", "descending": false }]
    },
    "id": 1
}
```

Hit Send and if all goes well, you should get back some JSON results:

<img src="https://i.imgur.com/cOPlJs9.png">

You can find a detailed description of all the data tables you can query for each smart contract in the individual contract documentation here:

https://github.com/hive-engine/steemsmartcontracts-wiki#contract-documentation

# Dev Best Practices

Here are some things to be aware of when writing smart contracts:

## Contract storage initialization

When your contract is deployed or upgraded, the special **createSSC** action will be called by the core node software. This is the ONLY place where you can setup database storage for your contract! The createSSC action should always have code similar to the following example:

```
actions.createSSC = async () => {
  const tableExists = await api.db.tableExists('users');
  if (tableExists === false) {
    await api.db.createTable('users', ['account', 'lastTickBlock']);
    await api.db.createTable('markets', ['account', 'symbol']);
    await api.db.createTable('params');

    const params = {};
    params.basicFee = '100';
    params.basicSettingsFee = '1';
    await api.db.insert('params', params);
  } else {
    // do something else here if necessary, such as
    // any data modifications for new features
  }
};
```

As in the above example, you should:

1. Check to see if tables have already been created (the ```if (tableExists === false)``` part).
2. Create your tables. Indexes are the only way DB query results can be sorted, so typically you want indexes on fields that will be important to sort by, such as primary keys. Try to keep indexes to a minimum, as having too many can negatively impact performance. Note that there will always be an implicit _id index for the _id column that MongoDB adds to every collection object as a unique identifier. In this example, ```await api.db.createTable('users', ['account', 'lastTickBlock']);``` means create a table called users, with indexes on the account and lastTickBlock fields.
3. Set initial contract params. More on this below.
4. If tables have already been created (meaning this action is being called for an upgrade rather than first deployment), then perform any necessary data updates. That's the ```// do something else here if necessary``` part of the example.

## Contract configuration

By convention, global contract configuration settings should be stored in the **params** table, with initial settings inserted in the table in the createSSC action as demonstrated above. You should provide an **updateParams** action that can be used ONLY by the contract owner (account that deployed the contract) to update settings. Here's an example of such an action:

```
actions.updateParams = async (payload) => {
  if (api.sender !== api.owner) return;

  const {
    registerFee,
    typeAddFee,
  } = payload;

  const params = await api.db.findOne('params', {});

  if (registerFee && typeof registerFee === 'string' && !api.BigNumber(registerFee).isNaN() && api.BigNumber(registerFee).gte(0)) {
    params.registerFee = registerFee;
  }
  if (typeAddFee && typeof typeAddFee === 'string' && !api.BigNumber(typeAddFee).isNaN() && api.BigNumber(typeAddFee).gte(0)) {
    params.typeAddFee = typeAddFee;
  }

  await api.db.update('params', params);
};
```

This does the following:

1. Verify that the action's caller is the contract owner, and reject the transaction if not: ```if (api.sender !== api.owner) return;```
2. Retrieve current params from the database: ```const params = await api.db.findOne('params', {});```
3. Perform input validation and update fields on the params object. Note that all fields should be optional, don't require that a field value be passed into the action if the contract owner doesn't want to update it.
4. Save the updated object back to the database: ```await api.db.update('params', params);```

## Contract libraries & module usage

Smart contract code executes in a constrained sandbox environment. As such, you are not allowed to use arbitrary third party libraries. Some constants and built-in library functions are available for you to use via the ```api``` object (things such as ```api.sender``` and ```api.BigNumber```). As a general rule, If it's not part of the api object, and not a built-in part of the Javascript language, then you may NOT use it! You are NOT allowed to import any modules with the Node.js require function. All smart contracts must be self-contained in a single source file.

Refer to the [smart contracts API](https://github.com/hive-engine/steemsmartcontracts-wiki/blob/master/Smart-Contracts-API.md) section for more information on what is available to you through the api object.

## Working with numbers

Floating point arithmetic within a smart contract is a big no-no, as that can lead to non-deterministic results and unexpected rounding. For such math, you are required to use ```api.BigNumber```. For typical usage examples, refer to other smart contracts. The tokens & nft contracts have some good examples of working with BigNumbers, such as this utility function for performing addition and subtraction of token balances:

```
const calculateBalance = (balance, quantity, precision, add) => (add
  ? api.BigNumber(balance).plus(quantity).toFixed(precision)
  : api.BigNumber(balance).minus(quantity).toFixed(precision));
```

For contract action input parameters, floating point numbers should be represented as strings, which are then turned into BigNumber objects internally.

Full documentation on the BigNumber library is [available here](https://mikemcl.github.io/bignumber.js/#).

## Input validation

All user input to contract actions should be carefully validated using ```api.assert```. Never trust ANYTHING users send to the contract! You'll find that a very large amount of your coding time is spent fine tuning validations and making sure input is properly formatted. Refer to existing smart contracts for examples of how to do this; your validations should be written in a similar style. The [nft contract](https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/nft.js) has many good examples of a wide variety of input validations, particularly for the create and issue actions.

Some types of validation are notable and deserve further mention:

### isSignedWithActiveKey

isSignedWithActiveKey is a special input parameter that is supplied by the smart contract system and will always be available to every contract action. Users cannot use this parameter, it's reserved for system use. If true, then the contract action was called through a custom json operation signed by the user's Hive active key. If false, then the transaction was signed with the user's posting key. Usually contract actions should require active key signature, but the choice is up to you. There may be some situations where posting key is appropriate. Contract actions that require active key signature should always have this as one of their first validations:

```
const {
  isSignedWithActiveKey,
} = payload;

if (api.assert(isSignedWithActiveKey === true, 'you must use a custom_json signed with your active key')) {
  // validation passed, safe to do some stuff here
}
```

### callingContractInfo

callingContractInfo is another special input parameter, similar to isSignedWithActiveKey. If this smart contract action is being called from another contract, instead of a Hive account, then callingContractInfo will have information about the calling contract. This allows you to design actions that can only be called from another contract, or have different behaviors depending on if they are called from a contract or an account. For usage examples, see the [nft contract](https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/nft.js).

### Verify user authorization

Always check that the account calling this action is actually authorized to do so! For example, when transferring tokens, the account doing the transfer should own the tokens to be transferred. If a market order is being canceled, the account doing the cancellation should own the order. If your action is updating the properties of some object, such as changing an NFT data property, then the calling account should own the object being updated. Many contracts have examples of this that you can model your own code on.

This sort of check is just commonsense, but can be easy to forget about if you're not careful.

### Checking account balance & paying fees

You may wish to require a fee for executing smart contract operations. In general, this is a good idea any time your action could add new stuff to the database. On-chain storage is limited and thus paying fees to use it will prevent spammy data from bloating the sidechain. Fees are typically burned by sending to the null account. Usually BEE is used as a fee currency, but you may use other tokens if desired.

As part of an action's validation checks, you should make sure the calling account has enough balance to pay your fee. Once all validation checks have passed, you then transfer the fee from the user's account (contracts implicitly have permission to perform token transfers on behalf of the calling account, so long as the active key is used). After the fee is sent, you should do a sanity check to make sure the transfer actually succeeded. A typical code pattern for doing this is:

```
// utility function to verify balance requirement is met
const verifyTokenBalance = async (amount, symbol, account) => {
  if (api.BigNumber(amount).lte(0)) {
    return true;
  }
  const tokenBalance = await api.db.findOneInTable('tokens', 'balances', { account, symbol });
  if (tokenBalance && api.BigNumber(tokenBalance.balance).gte(amount)) {
    return true;
  }
  return false;
};

// utility function to confirm token transfers from action logs
const isTokenTransferVerified = (result, from, to, symbol, quantity, eventStr) => {
  if (result.errors === undefined
    && result.events && result.events.find(el => el.contract === 'tokens' && el.event === eventStr
    && el.data.from === from && el.data.to === to && el.data.quantity === quantity && el.data.symbol === symbol) !== undefined) {
    return true;
  }
  return false;
};

// utility function to do fee transfer to either a regular account or this contract itself
const transferFee = async (amount, feeSymbol, dest, isSignedWithActiveKey) => {
  const actionStr = (dest === CONTRACT_NAME) ? 'transferToContract' : 'transfer';
  if (api.BigNumber(amount).gt(0)) {
    const res = await api.executeSmartContract('tokens', actionStr, {
      to: dest, symbol: feeSymbol, quantity: amount, isSignedWithActiveKey,
    });
    // check if the tokens were sent
    if (!isTokenTransferVerified(res, api.sender, dest, feeSymbol, amount, actionStr)) {
      return false;
    }
  }
  return true;
};

actions.myaction = async (payload) => {
  const {
    isSignedWithActiveKey,
  } = payload;
  ..
  ..
  // inside contract action do something like this
  const params = await api.db.findOne('params', {});
  const hasEnoughBalance = await verifyTokenBalance(params.typeAddFee, UTILITY_TOKEN_SYMBOL, api.sender);
  if (api.assert(hasEnoughBalance, 'you must have enough tokens to cover the registration fee')) {
    // burn the fee
    if (!(await transferFee(params.typeAddFee, UTILITY_TOKEN_SYMBOL, 'null', isSignedWithActiveKey))) {
      return false;
    }
    // success, now we can do some stuff
    ..
    ..
  }
  ..
  ..
}
```

## Avoiding non-determinism

It's crucially important to avoid writing contract actions that can have non-deterministic outcomes. In other words, if all the custom json transactions are replayed from the Hive blockchain, and the sidechain database is empty, it should result in exactly the same contract actions being executed in exactly the same order, with exactly the same results. This is very important for maintaining consensus between multiple sidechain nodes, as each node will have its own copy of the MongoDB database, and all the copies should contain the same data. Non-deterministic outcomes could result in chain forks and other bad behavior.

One way in which non-determinism can accidentally creep in is through database queries. MongoDB query results are not deterministic unless you use an index to sort the results. Hence when order of query results is important, you should ALWAYS use indexes, or else sort the results yourself. The [botcontroller contract](https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/botcontroller.js) has good examples of using indexes to ensure deterministic query results:

```
// get some basic accounts that are ready to be ticked
const pendingBasicTicks = await api.db.find(
  'users',
  {
    isEnabled: true,
    isPremium: false,
    lastTickTimestamp: {
      $lte: cutoffBasic,
    },
  },
  params.basicMaxTicksPerBlock,
  0,
  [{ index: 'lastTickBlock', descending: false }, { index: '_id', descending: false }],
);
await tickUsers(params, pendingBasicTicks, currentTimestamp, 'MM-B');
```

## Data size limits

On-chain database storage is extremely limited, and relatively expensive. Thus you should ensure your contract does not provide any way for a user to fill the database with unlimited amounts of information at no cost. Fees should be charged for operations that add data to the database, and usage should be kept sensible and limited. As an example of this, user inputs for data that will be stored in the database, such as text strings, should have maximum allowed lengths. Limits should be clearly expressed within the contract code, and defined using constants that can be easily edited later if necessary. The [nft contract](https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/nft.js) has some good examples:

```
const MAX_NUM_AUTHORIZED_ISSUERS = 10;
const MAX_NUM_LOCKED_TOKEN_TYPES = 10;
const MAX_SYMBOL_LENGTH = 10;
const MAX_DATA_PROPERTY_LENGTH = 100;

// cannot issue more than this number of NFT instances in one action
const MAX_NUM_NFTS_ISSUABLE = 10;

// cannot set properties on more than this number of NFT instances in one action
const MAX_NUM_NFTS_EDITABLE = 50;

// cannot burn, transfer, delegate, or undelegate more than
// this number of NFT instances in one action
const MAX_NUM_NFTS_OPERABLE = 50;

// cannot issue or burn more than this number of NFT
// instances in one action, when the list of NFT instances
// to act on includes a token with locked NFT instances
// contained within it
const MAX_NUM_CONTAINER_NFTS_OPERABLE = 1;
```

Also, take care to design your database tables in an efficient manner. Don't use long text strings when numbers will do. Avoid duplicating data unless necessary. Just be aware that the sidechain operates under resource constraints and take care to ensure your smart contract is a good blockchain citizen and doesn't use more resources than it really needs.

## Performance considerations

In addition to being careful about data storage, you should also be careful about writing contract actions that take too long to execute. The sidechain is relatively slow when it comes to executing smart contracts (it's not a complete turtle, but neither is it a blazing fast lambo), and all contract actions need to be performed quickly enough not to slow down block production. The system itself will kill any smart contracts that take too long to execute (I think the cutoff time is around 10 seconds per action). To check performance, you can time your contract actions in unit test cases.

Some things to keep in mind:

* keep database queries to a minimum, and fetch objects in batches instead of one-at-a-time when possible
* avoid any loops that could have potentially unbounded execution times
* define maximum limits on numbers of objects that can be operated on at once; the [nft contract](https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/nft.js) is once again a good example of this
* if you do need to perform a costly operation, such as iterating over thousands of database records, break the action up across multiple blocks. The checkPendingLotteries action of the [mining contract](https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/mining.js]) is an example of a multi-block action.

## Emitting events

If an action completes successfully, consider emitting an event:

```
api.emit('disableMarket', {
  account: api.sender,
  symbol,
});
```

This is not required, but is a good way to log state changes which will show up on block explorers for people who want to track event history. Emitted events get recorded in the transaction logs for the block that included the action containing the emit call. Apps that trigger contract actions can read these logs to determine if the action succeeded or failed. If an ```api.assert``` fails, there will be an error recorded in the logs with the assertion failure message. Refer to unit test cases for numerous examples of how transaction logs are formatted.

For example, if you want to look at disableMarket events for the botcontroller contract, as in the above example, you can search on the hive-engine.rocks block explorer like this:

https://hive-engine.rocks/transactions?contract=botcontroller&contract_action=disableMarket

<img src="https://i.imgur.com/Rlz2jqY.png">

The block logs for this transaction look like this: https://hive-engine.rocks/tx/3c6dca629ce04ce2781c6cd376f660bac19f04d0

<img src="https://i.imgur.com/zYysZK2.png">

## Ticking actions in every block

When a smart contract action is required to execute automatically every sidechain block, we say the action "ticks every block". Examples of such actions include the tick action in the [botcontroller contract](https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/botcontroller.js) and checkPendingLotteries in the [mining contract](https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/mining.js). These actions begin like this:

```
actions.tick = async () => {
  if (api.assert(api.sender === 'null', 'not authorized')) {
    // do some stuff
    ...
    ...
```

The action first makes sure that only the null account can call it (effectively this means no normal Hive account can ever use this action; it's a priviledged system action). When ```api.sender === 'null'```, that indicates the caller is the smart contract core node software itself.

In the future we might introduce a general mechanism that can be used to register such priviledged system actions, but for now the only way to get the node software to call such an action is by adding some code to this file: https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/libs/Block.js

Specifically:

```
if (this.refHiveBlockNumber >= 47746850) {
  virtualTransactions.push(new Transaction(0, '', 'null', 'mining', 'checkPendingLotteries', ''));
}
...
...
} else if (transaction.contract === 'mining'
  && transaction.action === 'checkPendingLotteries'
  && transaction.logs === '{"errors":["contract doesn\'t exist"]}') {
  // don't save logs
} else {
...
...
```

A few important points about the above code:

* These special actions are executed as virtual transactions (i.e. actions built into the sidechain, not triggered by custom json) and will show up as such in block logs.
* ```refHiveBlockNumber >= 47746850``` indicates not to start ticking this action until Hive block 47746850. In general, you should pick a starting block about 1 week in the future from when you expect this code to be deployed in production. That gives node operators some advance notice to upgrade, and ensures block replayability if the sidechain database ever needs to be rebuilt.
* The ```else if``` block ensures that if the virtual transaction is executed before the smart contract in question is deployed, then block logs won't fill up with useless error messages.

Keep in mind that adding ticking in this manner requires a version increment and release of the core node software, which entails administrative overhead and will require witness consensus later on, once node operation is decentralized. So don't do this sort of thing for trivial reasons; new ticking actions should only be added if absolutely necessary. In general, such actions should remain extremely limited as adding too many of them could slow down block processing and negatively impact sidechain performance.

## Calling smart contracts from other smart contracts

Often you will need to have your smart contract trigger an action in another smart contract. The [checking account balance & paying fees](#checking-account-balance--paying-fees) code snippet above has an example of doing this, using the ```api.executeSmartContract``` function to call a transfer action in the tokens contract. Keep in mind the following points:

* The smart contract action being called will have ```api.sender``` set to the same account that originally called your smart contract's action. In this way a chain of contract actions can be nested together and traced all the way back to a single custom json transaction that touches off the whole sequence of subsequent events.
* The ```callingContractInfo``` object will be available to the other smart contract action, containing data on your calling smart contract.
* You will need to explicitly pass the ```isSignedWithActiveKey``` parameter to the other smart contract.
* Be aware of performance considerations; calling another smart contract action is a relatively expensive operation, especially if you need to do it in a loop.
