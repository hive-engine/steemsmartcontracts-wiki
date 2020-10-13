Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

If you haven't already, skim through everything in [General System Design & Usage](https://github.com/hive-engine/steemsmartcontracts-wiki#general-system-design--usage) before proceeding any further.

For convenience, code links in this document are for Hive Engine, but path structure will be similar for Steem Engine versions.

# Table of Contents

* [Quickstart](#quickstart)

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

# Creating skeleton files

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

Test cases are expected to have both a negative and positive version for each publicly exposed smart contract action. For example, consider an NFT transfer operation. There is a [transfers tokens](https://github.com/hive-engine/steemsmartcontracts/blob/c11ce22b93d6160967c00560a79e83b67bfcb3e1/test/nft.js#L1281) test to verify tokens can be transferred OK from one account to another (that's the positive test); there is also a [does not transfer tokens](https://github.com/hive-engine/steemsmartcontracts/blob/c11ce22b93d6160967c00560a79e83b67bfcb3e1/test/nft.js#L1410) test that checks all the different ways a transfer action can fail (that's the negative test).
