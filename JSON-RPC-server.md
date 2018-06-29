When running the application, a JSON RPC server is made available to query the sidechain and its database on port 5000.

 ## 1. The "blockchain" endpoint (http://localhost:5000/blockchain)
Available methods:

**getLatestBlockInfo()**: get the latest block of the sidechain

Command:
```
{
    "jsonrpc": "2.0",
    "method": "getLatestBlockInfo",
    "id": 1
}
```

Result:

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "blockNumber": 2,
        "previousHash": "2f0467273993608f2fb08311ae6c1f8a94577427f395331051f92c5def94cda6",
        "timestamp": "2018-06-28T17:26:54",
        "transactions": [
            {
                "refBlockNumber": 23723607,
                "transactionId": "b42fed149252a83ddda03d1be7f02080ebb67492",
                "sender": "harpagon",
                "contract": "users_contract",
                "action": "addUser",
                "payload": "{ \"username\": \"AwesomeUsername\" }",
                "hash": "a90155eab610384d027fbc1ce706c24acb5a1151d91483e76e39e61f59ed65cb",
                "logs": "{\"events\":[{\"event\":\"newUserCreated\",\"data\":{\"id\":\"harpagon\",\"username\":\"AwesomeUsername\",\"verified\":false,\"meta\":{\"revision\":0,\"created\":1530230516595,\"version\":0},\"$loki\":1}}]}"
            }
        ],
        "hash": "48bd97fbc0b98a764d8f8d87e4469e21a3ab9f1c3eaa5d5d46d342b419f1b2f4",
        "merkleRoot": "1bd248954fa35533b2c9a637fad97f04406564f45366a6e4a7eb12d4f24f5b85"
    }
}
```

**getBlockInfo(blockNumber)**: get the block with the specified block number of the sidechain

Command:
```
{
    "jsonrpc": "2.0",
    "method": "getBlockInfo",
    "params": {
        "blockNumber": BLOCK_NUMBER
    },
    "id": 1
}
```

Result:

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "blockNumber": 1,
        "previousHash": "643fdd62e4c8e99049db8f19d6604f38dcf97aaf17d994741d8fee9925856c70",
        "timestamp": "2018-06-28T17:25:51",
        "transactions": [
            {
                "refBlockNumber": 23723586,
                "transactionId": "73fbb10207533ba32bb50a899ec64322a8b04a79",
                "sender": "harpagon",
                "contract": "contract",
                "action": "deploy",
                "payload": "{\"name\": \"users_contract\", \"code\": \"..."}",
                "hash": "e27d43e9c68fea1c65bc40bc09d5f34509373fc1945fd313d4ca1ffb4f652c2a",
                "logs": "{}"
            }
        ],
        "hash": "2f0467273993608f2fb08311ae6c1f8a94577427f395331051f92c5def94cda6",
        "merkleRoot": "46c0ca94ec8323e3594318adbe0b79ab813837bf854e5a4b6becf670849c5d9d"
    }
}
```

## 3. The "contracts" endpoint (http://localhost:5000/contracts)
Available methods:

**getContract(contract)**: get the contract specified from the database

Command:

```
{
    "jsonrpc": "2.0",
    "method": "getContract",
    "params": {
        "contract": "CONTRAC_NAME"
    },
    "id": 1
}
```

Result:

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "name": "users_contract",
        "owner": "harpagon",
        "code": {
            "code": "...",
            "filename": "vm.js",
            "_compiled": {}
        },
        "tables": [
            "users_contract_users"
        ],
        "meta": {
            "revision": 0,
            "created": 1530230481158,
            "version": 0
        },
        "$loki": 1
    }
}
```

**findOneInTable(query)**: get the object that matches the query from the table of the specified contract

Command:

```
{
    "jsonrpc": "2.0",
    "method": "findOneInTable",
    "params": {
        "contract": "CONTRACT_NAME",
        "table": "TABLE_NAME",
        "query": {}
    },
    "id": 1
}
```

Result:

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "id": "harpagon",
        "username": "AwesomeUsername",
        "verified": false,
        "meta": {
            "revision": 0,
            "created": 1530230516595,
            "version": 0
        },
        "$loki": 1
    }
}
```

**findInTable(query)**: get an array of objects that match the query from the table of the specified contract

Command:

```
{
    "jsonrpc": "2.0",
    "method": "findInTable",
    "params": {
        "contract": "CONTRACT_NAME",
        "table": "TABLE_NAME",
        "query": {}
    },
    "id": 1
}
```

Result:

```
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
        {
            "id": "harpagon",
            "username": "AwesomeUsername",
            "verified": false,
            "meta": {
                "revision": 0,
                "created": 1530230516595,
                "version": 0
            },
            "$loki": 1
        }
    ]
}
```