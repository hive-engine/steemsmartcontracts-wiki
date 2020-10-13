Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

If you haven't already, skim through everything in [General System Design & Usage](https://github.com/hive-engine/steemsmartcontracts-wiki#general-system-design--usage) before proceeding any further.

For convenience, code links in this document are for Hive Engine, but path structure will be similar for Steem Engine versions.

# Table of Contents

* [Quickstart](#quickstart)

**TODO**: finish rest of ToC

# Quickstart

1. Make sure you have a Linux server to develop on. Low end specs are fine: 2 GB RAM, a dual core CPU, and at least 10 GB free disk space should work great. Ubuntu is recommended, though other flavors of Linux will probably also work.

2. Install NodeJS and NPM. Node version 12+ is recommended: https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/

3. Install MongoDB. Version 4.2 is recommended (that's what production nodes are running): https://docs.mongodb.com/v4.2/administration/install-community/

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

# Development pipeline

The following sections describe the overall steps in the smart contract development process, from start to finish.

## Creating skeleton files

Each smart contract consists of two files: a .js file containing the contract code, and a .js file containing unit test code. Both the test file and contract file should have the same name. Also, the file name should be the same as the contract name itself which by convention should be all lowercase with no spaces or dashes between words.

Smart contract code goes here: **steemsmartcontracts/contracts**

Unit tests go here: **steemsmartcontracts/test**

Do NOT mix test code up with contract code, or vice versa! They must be strictly separated in the above two directories.

To get started, you can copy one of the existing test & contract files and then modify it as needed. These files are small ones so good to copy for this purpose:

https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/test/crittermanager.js
https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/contracts/crittermanager.js

The mining test cases are also good to take a look at, as they use a shortcut style for assertions which you may prefer when writing large test cases:

https://github.com/hive-engine/steemsmartcontracts/blob/hive-engine/test/mining.js

When writing your smart contract & test cases, use the existing contracts as a model. Similar code structure and style should be maintained for all contracts, and if you violate existing style guidelines your code may be rejected during the review process.

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

@cryptomancer will give feedback during the code review process; you may be required to make changes if there are any potential areas of concern.

Once the code review is passed, @cryptomancer will merge your PR into the main repository and schedule your smart contract for production deployment.

## Post-release checks

Once your smart contract is deployed, you can perform post-release checks to verify it's working as intended. The best way to do this is using Python with Beem to broadcast custom json transactions, then verify contract data state using Postman.

TODO: add details of how to do this
