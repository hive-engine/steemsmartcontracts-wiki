## 1. Environment setup
- Make sure you have a Linux server to run the node on. Low end specs are fine for now: 2 GB RAM, a dual core CPU, and at least 50 GB free disk space should work great. (For 2G RAM, you will need to add swap space, so recommend 4G RAM) Ubuntu is recommended, though other flavors of Linux will probably also work.
- Install NodeJS and NPM. Node version 12+ is recommended: https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/
- Install MongoDB. At least version 4.4.3 is required (that's what production nodes are running): https://docs.mongodb.com/v4.4/administration/install-community/, which needs to have replication enabled: https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/
  - To enable replication, you just need to add to the replication config in the mongo config:
    ```
    replication:
      replSetName: "rs0"
    ```
    and then after restarting mongo with this config, initiating replication in the `mongo` shell:
    `> rs.initiate()`

## 2. Install the Smart Contracts node
To install the app, simply follow these steps:
- get the files from the repository: 
	- via the git cli: ```git clone https://github.com/hive-engine/steemsmartcontracts.git```

- cd into the newly created project folder; for Hive Engine, make sure you are on the ```hive-engine``` branch (for Steem Engine use the witnesses branch). You may consider using a tagged release as well instead to match the primary nodes, in which case you should checkout a release tag e.g. `he_v1.7.2`
	- ```git checkout hive-engine```

- in your console type the following command in the folder that contains the files downloaded from the previous step:
	- ```npm install```

## 3. Configure the node
The ```config.json``` file has all the settings to make sure your node listens to the Hive blockchain and generates the proper genesis block. You shouldn't need to change any of the default config, it can just be used as is. If you are configuring a witness, then additionally set `witnessEnabled` to true, and copy `.env.example` to `.env` to set up your witness settings. Be careful as configuring asks for a witness private signing key in plain text.

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
    "witnessEnabled": false,
    "defaultLogLevel": "warn",
    "lightNode": false,
    "blocksToKeep": 864000,
    "domain" : ""
}
```

Optional config settings which you may wish to edit:

**domain** - if you have a domain name for your node, put the fully qualified domain name here. This information will be returned when status queries are made to your node's API.

**lightNode** - if set to true, your node will run in light configuration. A light node does not keep full transaction & block information, which reduces the disk space requirements for running the software (old blocks & transactions are cleaned up periodically). Note that you can't switch from a light node back to running as a full node later on (you would need to restore from a full node snapshot).

**blocksToKeep** - only used when lightNode = true. Specifies how many recent blocks worth of block data & transactions should be kept by light nodes. Defaults to 30 days worth, assuming a perfect 3 second block time.

For more details about light nodes, see the documentation on the pull request here: https://github.com/hive-engine/steemsmartcontracts/pull/144

## 4. Start the node
**Additional step needed for now** - Set your machine time zone to UTC:  `export TZ=UTC` for consistent contract behavior until the contract level bug is resolved.

You can easily start the node by typing the following command in the folder where the node was installed:

```npm run start```

To have it run as a daemon background process, allowing you to close your terminal window without stopping the node, you can use this command instead:

```nohup npm start &```

You may also use `pm2` as well, in which case you should run with the command

```pm2 start app.js --no-treekill --kill-timeout 10000 --no-autorestart```

Or else `pm2 stop` will kill all sub processes prematurely and not allow for a clean exit.

## 5a. Replay from a blocks.log file (DO NOT USE, CURRENTLY NOT WORKING)
When starting a node for the first time you can either replay the whole sidechain from the Hive blockchain (which can last very long) or replay from a blocks.log file.
The blocks.log file is actually the table called "chain" that you can find in your MongoDB database.

- Find a blocks.log file (ask someone to provide you a JSON version of their "chain" table)
- Start the tool via ```node app.js --replay file```

This command will basically read the file located under "blocksLogFilePath" from the "config.json" file and rebuild the sidechain from the blocks stored in this file.

## 5b. Restore a MongoDB dump (recommended approach)
The fastest way to fire up a node is by restoring a MongoDB dump.

~~The latest public DB snapshot is available here, file size is about 6 GB: hsc_05-22-2021_b54107973.archive. When using this snapshot, set ```startHiveBlock``` to **54107973** in your config file.~~

June 7, 2021 update: sorry, we no longer provide public snapshots due to server bandwidth considerations. We may make them available again in the future, but for now there are some Hive Engine witnesses that provide snapshots as a public service. The latest witness provided snapshots are available here:

https://snap.primersion.com/

https://jedigeiss.tech/

snapshots for light nodes:  https://snap.primersion.com/light/

- Make sure node is stopped
- Download a dump of the MongoDB database
- Restore it
	- open mongodb shell by running mongo command
	- show dbs
	- use hsc
	- db.dropDatabase()
	- show dbs    // to confirm db has been dropped
	- quit()
	- mongorestore --gzip --archive=hsc_05-22-2021_b54107973.archive
- Update the "config.json" file with the "startHiveBlock" that matches the dump you just restored (number after ```b``` e.g. 54107973)
- Start the node

## 5c. Restore from hive genesis block.

No extra work needed, the default setting will start syncing
from the hive genesis block.

## 6. Checking that your node works

To verify your node is running properly, you can query data from its API. A getStatus query will show you info on the running software version and latest block processed. Refer to [Querying the Engine API](https://github.com/hive-engine/steemsmartcontracts-wiki/blob/master/Smart-Contracts-Guide.md#querying-the-engine-api) for details. In the API URL you should replace ```https://api.hive-engine.com/rpc``` with ```http://<YOUR SERVER IP>:5000```

So for example, instead of ```https://api.hive-engine.com/rpc/blockchain``` you would use ```http://<YOUR SERVER IP>:5000/blockchain```

## 7. Enabling and Approving Witness

There is a script called `witness_action.js` that can help with a few quick actions. See `node witness_action.js --help` for a list of commands, noting it uses the settings in your .env file to perform the custom json broadcasts.

Witnesses dashboard where you can also approve witnesses is on tribaldex: https://tribaldex.com/witnesses 
