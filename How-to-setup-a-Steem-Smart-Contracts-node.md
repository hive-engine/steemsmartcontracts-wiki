


## 1. Requirements
The only requirement is to have NodeJS and NPM installed on your server, see https://nodejs.org/en/download/

## 2. Install the Steem Smart Contracts node
To "install" the app, simply follow these steps:
- get the files from the repository: 
	- via the git cli: ```git clone https://github.com/harpagon210/steemsmartcontracts.git```
	- by downloading the zip file: https://github.com/harpagon210/steemsmartcontracts/archive/master.zip

- in your console type the following command in the folder that contains the files downloaded from the previous step:
	- ```npm install```

## 3. Configure the node
You'll have to configure your node in order to make it listen to the Steem blockchain, simply edit the ```config.json``` file: 

(the following file is a pre-configuration that will listen to the sidechain id b8923f70-c1f7-497f-961j and will start up a HTTP JSON RPC server listening on port 5000)

```js
{
    "chainId": "b8923f70-c1f7-497f-961j", // the id of the sidechain that the node will listen to
    "rpcNodePort": 5000, // port of the JSON RPC server that people will use to retrieve data from your node
    "dataDirectory": "./data/", // directory where is stored the database
    "databaseFileName": "database.db", // name of the file for the database
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
    "startSteemBlock": 25647669, // last Steem block parsed by the node
    "genesisSteemBlock": 25647669, // first block that was parsed by the sidechain, needs to be the same on all nodes listening to the sidechain id previously defined
    // the folowing keys need to be populated in order to run an HTTPS node
    "keyCertificate": "", // path to your key.pem file
    "certificate": "", // path to your certificate.pem file
    "chainCertificate": "" // path to your chain.pem file
}
```
You'll also have to create a .env file in the root folder with the following entries:

```
NODE_ENV=production
ACTIVE_SIGNING_KEY=... // active signing key of a Steem account, required to sign the blocks
```
## 4. Start the node
You can easily start the node by typing the following command in the folder where the node was installed:

```node app.js```

It is recommended to run the node via [PM2](http://pm2.keymetrics.io/) which is a tool that manage NodeJS apps:

Install PM2: ```npm install pm2 -g```

Run the node: ```pm2 start app.pm2.js```

Check the status of the node (CPU/RAM usage): ```pm2 monit```

## 5. Replay from file blocks.log
When starting a node for the first time you can either replay the whole sidechain from the Steem blockchain (which can last very long) or replay from a file blocks.log.

- Find a blocks.log file
- Start the tool via ```node app.js -replay file```

This command will basically read the file located under "blocksLogFilePath" from the "config.json" file and rebuild the sidechain from the blocks stored in this file.

## 6. Run a test node on a Steem testnet
Simply create (or update) the .env file with the following information:

```
NODE_ENV=test
```
This will plug the tool to the testnet https://testnet.steem.vc/ ([more info on how to create test accounts on this testnet](https://testnet.steem.vc/))