


## 1. Requirements
- have NodeJS and NPM installed on your server, see https://nodejs.org/en/download/
- have a MongoDB server installed on your server, see https://docs.mongodb.com/v3.2/administration/install-community/

## 2. Install the Steem Smart Contracts node
To "install" the app, simply follow these steps:
- get the files from the repository: 
	- via the git cli: ```git clone https://github.com/harpagon210/steemsmartcontracts.git```
	- by downloading the zip file: https://github.com/harpagon210/steemsmartcontracts/archive/master.zip

- in your console type the following command in the folder that contains the files downloaded from the previous step:
	- ```npm install```

## 3. Configure the node
You'll have to configure your node in order to make it listen to the Steem blockchain, simply edit the ```config.json``` file: 

(the following file is a pre-configuration that will listen to the sidechain id "mainnet1" and will start up a HTTP JSON RPC server listening on port 5000)

```js
{
    "chainId": "mainnet1", // the id of the sidechain that the node will listen to
    "rpcNodePort": 5000, // port of the JSON RPC server that people will use to retrieve data from your node
    "databaseURL": "mongodb://localhost:27017", // url to your MongoDB server
    "databaseName": "ssc", // name of the MongoDB database
    "blocksLogFilePath": "./blocks.log", // path to a blocks log file (used with the replay function)
    "autosaveInterval": 1800000, // interval for which the database will be saved, in milliseconds, if 0, the autosave will be deactivated
    "javascriptVMTimeout": 10000, // the timeout that will be applied to the JavaScript virtual machine, needs to be the same on all the nodes of the sidechain
    // array of Steem full nodes url for the failover
    "streamNodes": [
        "https://api.steemit.com",
        "https://rpc.buildteam.io",
        "https://rpc.steemviz.com",
        "https://steemd.minnowsupportproject.org"
    ],
    "startSteemBlock": 29862600, // last Steem block parsed by the node
    "genesisSteemBlock": 29862600, // first block that was parsed by the sidechain, needs to be the same on all nodes listening to the sidechain id previously defined
```

## 4. Start the node
You can easily start the node by typing the following command in the folder where the node was installed:

```npm run start```

## 5a. Replay from a blocks.log file
When starting a node for the first time you can either replay the whole sidechain from the Steem blockchain (which can last very long) or replay from a blocks.log file.
The blocks.log file is actually the table called "chain" that you can find in your MongoDB database.

- Find a blocks.log file (ask someone to provide you a JSON version of their "chain" table)
- Start the tool via ```node app.js --replay file```

This command will basically read the file located under "blocksLogFilePath" from the "config.json" file and rebuild the sidechain from the blocks stored in this file.

## 5b. Restore a MongoDB dump
The fastest way to fire up a node is by restoring a MongoDB dump.

- Find a dump of the MongoDB database (ie: http://api.steem-engine.com/ssc.archive)
- Restore it (mongorestore --gzip --archive=ssc.archive)
- Update the "config.json" file with the "startSteemBlock" that matches the dump you just restored
- Start the tool via ```npm run start```