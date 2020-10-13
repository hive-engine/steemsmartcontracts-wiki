Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

If you haven't already, skim through everything in [General System Design & Usage](https://github.com/hive-engine/steemsmartcontracts-wiki#general-system-design--usage) before proceeding any further.

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
    
