## 1. Environment setup
- Make sure you have a Linux server to run the node on. Low end specs are fine for now: 2 GB RAM, a dual core CPU, and at least 10 GB free disk space should work great. Ubuntu is recommended, though other flavors of Linux will probably also work.
- Install NodeJS and NPM. Node version 12+ is recommended: https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/
- Install MongoDB. Version 4.2.3 is recommended (that's what production nodes are running): https://docs.mongodb.com/v4.2/administration/install-community/

## 2. Install the Smart Contracts node
To install the app, simply follow these steps:
- get the files from the repository: 
	- via the git cli: ```git clone https://github.com/hive-engine/steemsmartcontracts.git```

- cd into the newly created project folder; for Hive Engine, make sure you are on the ```hive-engine``` branch (for Steem Engine use the witnesses branch)
	- ```git checkout hive-engine```

- in your console type the following command in the folder that contains the files downloaded from the previous step:
	- ```npm install```

## 3. Configure the node
The ```config.json``` file has all the settings to make sure your node listens to the Hive blockchain and generates the proper genesis block. You shouldn't need to change any of the default config, it can just be used as is.

(the following file is a pre-configuration that will listen to the sidechain id "mainnet-hive" and will start up a HTTP JSON RPC server listening on port 5000)

```js
{
    "chainId": "mainnet-hive",   // the id of the sidechain that the node will listen to
    "rpcNodePort": 5000,   // port of the JSON RPC server that people will use to retrieve data from your node
    "p2pPort": 5001,
    "databaseURL": "mongodb://localhost:27017",   // url to your MongoDB server
    "databaseName": "hsc",   // name of the MongoDB database
    "blocksLogFilePath": "./blocks.log",   // path to a blocks log file (used with the replay function)
    "javascriptVMTimeout": 10000,   // the timeout that will be applied to the JavaScript virtual machine, needs to be the same on all the nodes of the sidechain
    // array of Hive full nodes used for failover
    "streamNodes": [
        "https://api.openhive.network",
        "https://api.hive.blog",
        "https://anyx.io",
        "https://api.hivekings.com"
    ],
    "startHiveBlock": 41967000,   // last Hive block parsed by the node
    "genesisHiveBlock": 41967000,   // first block that was parsed by the sidechain, needs to be the same on all nodes listening to the sidechain id previously defined
    "witnessEnabled": false
}
```

## 4. Start the node
You can easily start the node by typing the following command in the folder where the node was installed:

```npm run start```

To have it run as a daemon background process, allowing you to close your terminal window without stopping the node, you can use this command instead:

```nohup npm start &```

## 5a. ~~Replay from a blocks.log file~~ (NOT RECOMMENDED)
When starting a node for the first time you can either replay the whole sidechain from the Hive blockchain (which can last very long) or replay from a blocks.log file.
The blocks.log file is actually the table called "chain" that you can find in your MongoDB database.

- Find a blocks.log file (ask someone to provide you a JSON version of their "chain" table)
- Start the tool via ```node app.js --replay file```

This command will basically read the file located under "blocksLogFilePath" from the "config.json" file and rebuild the sidechain from the blocks stored in this file.

**Note: right now we don't recommended replaying this way as there is a replay bug with virtual transactions, you should restore from a MongoDB dump instead**

## 5b. Restore a MongoDB dump (recommended approach)
The fastest way to fire up a node is by restoring a MongoDB dump.

The latest public DB snapshot is available here, file size is about 2.4 GB:
https://api.hive-engine.com/hsc_12-19-2020.archive

When using this snapshot, set ```startHiveBlock``` to **49669398** in your config file.

- Make sure node is stopped
- Download a dump of the MongoDB database
- Restore it
	- open mongodb shell by running mongo command
	- show dbs
	- use hsc
	- db.dropDatabase()
	- show dbs    // to confirm db has been dropped
	- quit()
	- mongorestore --gzip --archive=hsc_12-19-2020.archive
- Update the "config.json" file with the "startHiveBlock" that matches the dump you just restored
- Start the node

## 6. Checking that your node works

To verify your node is running properly, you can query data from its API. A getStatus query will show you info on the running software version and latest block processed. Refer to [Querying the Engine API](https://github.com/hive-engine/steemsmartcontracts-wiki/blob/master/Smart-Contracts-Guide.md#querying-the-engine-api) for details. In the API URL you should replace ```https://api.hive-engine.com/rpc``` with ```http://<YOUR SERVER IP>:5000```

So for example, instead of ```https://api.hive-engine.com/rpc/blockchain``` you would use ```http://<YOUR SERVER IP>:5000/blockchain```
