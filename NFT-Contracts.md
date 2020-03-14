Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

# Table of Contents

* [Creating new NFTS](#creating-new-nfts)
  * actions:
  * [create](#create)
* [Managing NFT metadata](#managing-nft-metadata)
  * actions:
  * [updateUrl](#updateurl)
  * [updateMetadata](#updatemetadata)
  * [updateName](#updatename)
  * [updateOrgName](#updateorgname)
  * [updateProductName](#updateproductname)
  * [addAuthorizedIssuingAccounts](#addauthorizedissuingaccounts)
  * [addAuthorizedIssuingContracts](#addauthorizedissuingcontracts)
  * [removeAuthorizedIssuingAccounts](#removeauthorizedissuingaccounts)
  * [removeAuthorizedIssuingContracts](#removeauthorizedissuingcontracts)
  * [transferOwnership](#transferownership)
* [Configuring Data Properties](#configuring-data-properties)
  * actions:
  * [addProperty](#addproperty)
  * [setPropertyPermissions](#setpropertypermissions)
  * [setProperties](#setproperties)
  * [setGroupBy](#setgroupby)
  * [updatePropertyDefinition](#updatepropertydefinition)
* [Token issuance](#token-issuance)
  * [fees](#fees)
  * [locked tokens](#locked-tokens)
  * [locked token restrictions](#locked-token-restrictions)
  * [token IDs](#token-ids)
  * actions:
  * [issue](#issue)
  * [issueMultiple](#issuemultiple)
* [Delegation](#delegation)
  * actions:
  * [enableDelegation](#enabledelegation)
  * [delegate](#delegate)
  * [undelegate](#undelegate)
* [Other actions](#other-actions)
  * actions:
  * [transfer](#transfer)
  * [burn](#burn)
* [Tables available](#tables-available)
  * [params](#params)
  * [nfts](#nfts)
  * [pendingUndelegations](#pendingundelegations)
  * [SYMBOLinstances](#symbolinstances)
* [Example smart contract for token issuance](#example-smart-contract-for-token-issuance)

# Actions available:
## Creating new NFTs
### create:
Creates a new NFT. A creation fee of 100 ENG is required.
* requires active key: yes

* can be called by: Steem account

* parameters:
  * name (string): name of the token (letters, numbers, whitespace only, max length of 50)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * **(optional)** orgName (string): name of the company/organization that created this NFT (letters, numbers, whitespace only, max length of 50)
  * **(optional)** productName (string): product/brand that this NFT is associated with (letters, numbers, whitespace only, max length of 50)
  * **(optional)** url (string): url of the project (max length of 255)
  * **(optional)** maxSupply (string): maximum supply for the token (between 1 and 9,007,199,254,740,991). If maxSupply is not specified, then the supply will be unlimited.
  * **(optional)** authorizedIssuingAccounts (array of string): a list of Steem accounts which are authorized to issue new tokens on behalf of the NFT owner. If no list is provided, then the NFT owner (the account that calls create) will be the only such authorized account by default.
  * **(optional)** authorizedIssuingContracts (array of string): a list of smart contracts which are authorized to issue new tokens on behalf of the NFT owner. If no list is provided, then no smart contracts will be authorized as such.

* examples:
```
{
    "contractName": "nft",
    "contractAction": "create",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "My Test NFT"
    }
}

{
    "contractName": "nft",
    "contractAction": "create",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "My Test NFT",
        "url": "https://mynft.com"
        "maxSupply": "1000"
    }
}

{
    "contractName": "nft",
    "contractAction": "create",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "My Test NFT",
        "orgName": "My Company Inc",
        "productName": "Uber Cool Product",
        "url": "https://mynft.com"
        "maxSupply": "1000",
        "authorizedIssuingAccounts": [ "cryptomancer","aggroed","harpagon" ],
        "authorizedIssuingContracts": [ "mycontract","myothercontract" ]
    }
}
```
## Managing NFT metadata
### updateUrl:
Updates the url of the token's project web site.
* requires active key: no

* can be called by: Steem account that owns the NFT

* parameters:
  * url (string): new url for the token (0 <= length <= 255)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)

* example:
```
{
    "contractName": "nft",
    "contractAction": "updateUrl",
    "contractPayload": {
        "symbol": "TESTNFT",
        "url": "https://mynewurl.com"
    }
}
```

### updateMetadata:
Updates the metadata of a token.
* requires active key: no

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * metadata (JSON): the metadata for the token (the JSON when stringified can't exceed 1000 characters)

* example:
```
{
    "contractName": "nft",
    "contractAction": "updateMetadata",
    "contractPayload": {
        "symbol": "TESTNFT",
        "metadata": {
            "url": "https://mycoolnft.com",
            "icon": "https://mycoolnft.com/token.jpg",
            "desc": "This NFT will rock your world! It has features x, y, and z. So cool!"
        }
    }
}
```

### updateName:
Updates the user friendly name of an NFT.
* requires active key: no

* can be called by: Steem account that owns the NFT

* parameters:
  * name (string): new name of the token (letters, numbers, whitespace only, max length of 50)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)

* example:
```
{
    "contractName": "nft",
    "contractAction": "updateName",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "My Awesome NFT"
    }
}
```

### updateOrgName:
Updates the name of the company/organization that manages an NFT.
* requires active key: no

* can be called by: Steem account that owns the NFT

* parameters:
  * orgName (string): new name of the company/organization (letters, numbers, whitespace only, max length of 50)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)

* example:
```
{
    "contractName": "nft",
    "contractAction": "updateOrgName",
    "contractPayload": {
        "symbol": "TESTNFT",
        "orgName": "Nifty Company Inc"
    }
}
```

### updateProductName:
Updates the name of the product/brand that an NFT is associated with.
* requires active key: no

* can be called by: Steem account that owns the NFT

* parameters:
  * productName (string): new name of the product/brand (letters, numbers, whitespace only, max length of 50)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)

* example:
```
{
    "contractName": "nft",
    "contractAction": "updateProductName",
    "contractPayload": {
        "symbol": "TESTNFT",
        "productName": "Acme Exploding NFTs"
    }
}
```

### addAuthorizedIssuingAccounts:
Adds Steem accounts to the list of accounts that are authorized to issue new tokens on behalf of the NFT owner. This will NOT replace the existing list; it only adds to it. There is a maximum limit of 10 accounts that can be on this list.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * accounts (array of string): a list of Steem accounts to add to the authorized list

* example:
```
{
    "contractName": "nft",
    "contractAction": "addAuthorizedIssuingAccounts",
    "contractPayload": {
        "symbol": "TESTNFT",
        "accounts": [ "satoshi","aggroed","cryptomancer" ]
    }
}
```
If the current authorized account list is ``[ "marc" ]``, then after calling the above example action, the list will be updated to ``[ "marc","satoshi","aggroed","cryptomancer" ]``

### addAuthorizedIssuingContracts:
Adds smart contracts to the list of contracts that are authorized to issue new tokens on behalf of the NFT owner. This will NOT replace the existing list; it only adds to it. There is a maximum limit of 10 smart contracts that can be on this list.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * contracts (array of string): a list of smart contracts to add to the authorized list

* example:
```
{
    "contractName": "nft",
    "contractAction": "addAuthorizedIssuingContracts",
    "contractPayload": {
        "symbol": "TESTNFT",
        "contracts": [ "mycontract","anothercontract","mygamecontract" ]
    }
}
```
If the current authorized contract list is ``[ "splinterlands" ]``, then after calling the above example action, the list will be updated to ``[ "splinterlands","mycontract","anothercontract","mygamecontract" ]``

### removeAuthorizedIssuingAccounts:
Removes Steem accounts from the list of accounts that are authorized to issue new tokens on behalf of the NFT owner. This will NOT replace the existing list; it only removes from it.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * accounts (array of string): a list of Steem accounts to remove from the authorized list

* example:
```
{
    "contractName": "nft",
    "contractAction": "removeAuthorizedIssuingAccounts",
    "contractPayload": {
        "symbol": "TESTNFT",
        "accounts": [ "satoshi","aggroed" ]
    }
}
```
If the current authorized account list is ``[ "satoshi","aggroed","cryptomancer" ]``, then after calling the above example action, the list will be updated to ``[ "cryptomancer" ]``

### removeAuthorizedIssuingContracts:
Removes smart contracts from the list of contracts that are authorized to issue new tokens on behalf of the NFT owner. This will NOT replace the existing list; it only removes from it.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * contracts (array of string): a list of smart contracts to remove from the authorized list

* example:
```
{
    "contractName": "nft",
    "contractAction": "removeAuthorizedIssuingContracts",
    "contractPayload": {
        "symbol": "TESTNFT",
        "contracts": [ "mycontract","mygamecontract" ]
    }
}
```
If the current authorized contract list is ``[ "mycontract","anothercontract","mygamecontract" ]``, then after calling the above example action, the list will be updated to ``[ "anothercontract" ]``

### transferOwnership:
Transfers ownership of an NFT from the current owner to another Steem account.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * to (string): Steem account to become the new owner (3 <= length <= 16)

* example:
```
{
    "contractName": "nft",
    "contractAction": "transferOwnership",
    "contractPayload": {
        "symbol": "TESTNFT",
        "to": "aggroed"
    }
}
```

## Configuring Data Properties
Once you have created your new NFT, most likely the next thing you will want to do is define some data properties for it. Data properties are the details that make each NFT unique. You can have as many data properties as you want, but since on-chain storage space is precious, adding a lot of data properties will cost a lot in fees. Issuance fees also increase as the number of data properties increases. So you are encouraged to be concise & compact when designing your storage schema, and use links to off-chain storage whenever possible.

The first 3 data properties are free. For each data property added beyond the first three, you must pay a fee of 100 ENG.

Data properties are limited to the following types:

* **number** = represents any number, can be integer or floating point. The precision of a number is limited to what is supported by Javascript number primitives (since Javascript is what smart contracts are implemented in). If you require a greater degree of precision, or if rounding errors are of concern (for example with financial applications), then you should use the string data property type instead.
* **string** = can contain any text data, limited to a maximum of 100 characters
* **boolean** = represents a true / false value

To prevent abuse, JSON type is not directly supported. If you wish to format your data as JSON, you can stringify it and use a string data property. Do keep in mind the maximum length; if you have a lot of data to store, you may need to break it into sections and use multiple data properties.

In addition, data properties can be defined as **read only**. A read only property can be set exactly one time, and then never changed again. This is ideal for static data that needs to be set on an NFT during issuance and does not require later edits.

As with an NFT's list of authorized issuing accounts & contracts, similarly each individual data property also has a list of accounts & contracts that are authorized to edit that property.

### addProperty:
Adds a new data property schema to an existing NFT definition. For every data property added beyond the third, a 100 ENG fee is required.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * name (string): name of the new data property (letters & numbers only, max length of 25)
  * type (string): must be number, string, or boolean as explained above
  * **(optional)** isReadOnly (boolean): if true, then this data property can be set exactly one time and never changed again. The default value is false if this parameter is not specified.
  * **(optional)** authorizedEditingAccounts (array of string): a list of Steem accounts which are authorized to edit this data property on behalf of the NFT owner. If no list is provided, then the NFT owner (the account that calls create) will be the only such authorized account by default.
  * **(optional)** authorizedEditingContracts (array of string): a list of smart contracts which are authorized to edit this data property on behalf of the NFT owner. If no list is provided, then no smart contracts will be authorized as such.

* examples:
```
{
    "contractName": "nft",
    "contractAction": "addProperty",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "color",
        "type": "string"
    }
}

{
    "contractName": "nft",
    "contractAction": "addProperty",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "edition",
        "type": "number",
        "isReadOnly": true
    }
}

{
    "contractName": "nft",
    "contractAction": "addProperty",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "isPremium",
        "type": "boolean",
        "authorizedEditingAccounts": [ "cryptomancer" ],
        "authorizedEditingContracts": [ "mycontract","myothercontract" ]
    }
}
```
Note that once a data property name and type are set, they normally CANNOT be changed (however under some limited circumstances this is possible; see [updatePropertyDefinition](#updatepropertydefinition) below). Also, data property schemas cannot be deleted once added.

### setPropertyPermissions:
Can be used after calling the addProperty action to change the lists of authorized editing accounts & contracts for a given data property. There is a maximum limit of 10 accounts/contracts that can be on each list.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * name (string): name of the data property (letters & numbers only, max length of 25)
  * **(optional)** accounts (array of string): a list of Steem accounts which are authorized to edit this data property on behalf of the NFT owner. This list will completely replace the current one (not add to it).
  * **(optional)** contracts (array of string): a list of smart contracts which are authorized to edit this data property on behalf of the NFT owner. This list will completely replace the current one (not add to it).

* example:
```
{
    "contractName": "nft",
    "contractAction": "setPropertyPermissions",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "color",
        "accounts": [ "cryptomancer","marc" ],
        "contracts": []       // setting an empty array will clear the current list
    }
}
```

### setProperties:
Edits one or more data properties on one or more instances of an NFT.
* requires active key: no

* can be called by: Steem account or smart contract on the authorized list of editing accounts/contracts for the data properties in question

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * **(optional)** fromType (string): indicates whether this action is being called by a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user. Note that a smart contract can still set this to user in order to execute the action on behalf of a Steem account rather than the calling contract itself.
  * nfts (array of object): the data properties to set and their corresponding NFT instance ID. Should be formatted as follows: ``[ {"id":"1", "properties": {"name1":"value1","name2":"value2",...}}, {"id":"2", "properties": {"name1":"value1","name2":"value2",...}, ...} ]``

A maximum of 50 tokens can be edited in a single call of this action.

* examples:
```
{
    "contractName": "nft",
    "contractAction": "setProperties",
    "contractPayload": {
        "symbol": "TESTNFT",
        "nfts": [
            { "id":"573", "properties": {
                "color": "red",
                "level": "2"       // a number property can be set using either a string or number
               }
            },
            { "id":"301", "properties": {
                "level": 3         // a number property can be set using either a string or number
               }
            }
        ]
    }
}

{
    "contractName": "nft",
    "contractAction": "setProperties",
    "contractPayload": {
        "symbol": "TESTNFT",
        "fromType":"contract",
        "nfts": [
            { "id":"1284", "properties": {
                "isFrozen": true
               }
            }
        ]
    }
}
```
Data properties that are not included in the list of properties to edit will have their values left unchanged.

### setGroupBy:
After you have created some data properties via the addProperty action, you can call setGroupBy in order to define a list of data properties by which market orders for NFT instances should be grouped by. You can think of this grouping as an index used to organize orders on the market, similar to how PeakMonsters groups Splinterlands cards according to type & gold foil status. NFT instances which have the same values for the designated data properties are considered equivalent as far as the market is concerned.

Consider the following points carefully before calling this action:

* Data properties which never change once set (i.e. read-only properties) are the best ones to use for this grouping.
* Long text strings do not make ideal properties to group by. Integers and boolean types make the best grouping.
* Numbers with fractional parts (for example 3.1415926) should be avoided due to possible rounding issues. Integers without fractional parts are ideal for grouping.
* This grouping **can only be set once!** You can't change it later on, so don't call this action until you are completely ready to do so.
* Token holders will not be able to place market orders until you have defined a valid grouping via this action.

* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * properties (array of string): list of data property names to set as the grouping. The schema for each property must have been previously created via the addProperty action.

* example:
```
{
    "contractName": "nft",
    "contractAction": "setGroupBy",
    "contractPayload": {
        "symbol": "TESTNFT",
        "properties": [ "level","isFood" ]
    }
}
```

### updatePropertyDefinition:
Updates the schema of a data property. This action can be used to change a data property's name, type, or read only attribute, with a couple caveats.

*This action can only be called if no tokens for this NFT have been issued yet.* Once tokens have been issued, data property schema definitions cannot be changed, and any attempt to use this action will fail. This action is primarily intended as a safeguard to protect against mistakes during NFT creation time, not for changing token characteristics later on.

There are further restrictions on data property name changes (see newName parameter below).
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * name (string): name of the data property to change (letters & numbers only, max length of 25)
  * **(optional)** newName (string): new name of the data property (letters & numbers only, max length of 25). There must not be an existing data property with this new name, and the data property being changed must not be part of a grouping previously created by [setGroupBy](#setgroupby).
  * **(optional)** type (string): new type of the data property. Must be number, string, or boolean.
  * **(optional)** isReadOnly (boolean): new read only attribute of the data property. If true, then this data property can be set exactly one time and never changed again.
  
* examples:
```
{
    "contractName": "nft",
    "contractAction": "updatePropertyDefinition",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "color",
        "newName": "Color"
    }
}

{
    "contractName": "nft",
    "contractAction": "updatePropertyDefinition",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "edition",
        "type": "string"
    }
}

{
    "contractName": "nft",
    "contractAction": "updatePropertyDefinition",
    "contractPayload": {
        "symbol": "TESTNFT",
        "name": "frozen",
        "newName": "age",
        "type": "number",
        "isReadOnly": false
    }
}
```

A successful updatePropertyDefinition action will emit an "updatePropertyDefinition" event:
``symbol, originalName, originalType, originalIsReadOnly, newName (only if changed), newType (only if changed), newIsReadOnly (only if changed)``
example:
```
{
    "contract": "nft",
    "event": "updatePropertyDefinition",
    "data": {
        "symbol": "TSTNFT",
        "originalName": "frozen",
        "originalType": "boolean",
        "originalIsReadOnly": true,
        "newName": "age",
        "newType": "number",
        "newIsReadOnly": false
    }
}
```

## Token issuance
Once an NFT has been created, its data properties defined, and editing permissions set, it's time to issue some tokens!

### fees

There is a fee per each token issued, which the issuing account or smart contract must pay. The issuer can choose to pay the fee in one of several different regular Steem Engine token types. Initially, such fees will be payable in ENG or PAL. The issuance fee is calculated as follows:

``fee = base fee + ((base fee) x (number of data properties))``

The base fee when paying with ENG or PAL is 0.001. The fee reflects the fact that on-chain storage has a cost, so the more data properties an NFT has, the higher the issuance fee will be per token.

**Example:** you want to issue 3 tokens and pay the fees in ENG. Your NFT has 4 data properties. You will pay 0.005 ENG per token issued (the base fee of 0.001, plus an additional 0.001 for each data property). The total cost to issue 3 tokens will be 0.015 ENG. 

### locked tokens

When a token is issued, the issuer can optionally choose to lock up a basket of other tokens within the issued NFT token (or NFT instance, as we sometimes refer to them). These locked tokens will be transferred from the issuing account or smart contract, and considered "owned" by the newly issued token itself. The locked tokens will stay attached to the issued NFT instance for the duration of its lifetime. Locked tokens cannot be spent or interacted with in any way, they are effectively out of circulation.

The only way to retrieve locked tokens is by burning the NFT instance that contains them. When an NFT token is burned, any locked tokens are released & transferred back to the account or smart contract that held the burned token (they are NOT sent back to the issuing account!).

Thus locked tokens are a way to give intrinsic value to an NFT, above & beyond what the NFT is worth in its own right. Token holders are incentivized to burn their NFTs in order to get back the value of the tokens locked within.

Note that both regular Steem Engine tokens and other NFT instances can be locked within tokens in this manner. You can even have a mix of different token types.

### locked token restrictions

For performance reasons, the following rules apply when dealing with locked tokens. For the purposes of these rules, we use the term **container token** to refer to an NFT instance that contains other NFT instances locked within it (an NFT instance that only contains regular Steem Engine tokens is NOT considered to be a container token).

* A maximum of 10 different types of regular tokens can be locked up in a single NFT token (though the quantity of each has no limit).
* A maximum of 50 NFT instances can be locked up in a single NFT token. A mix of different symbols is permitted without restriction.
* Container tokens must be issued individually, you may not issue more than one of them at once using the issueMultiple action.
* Container tokens must be burned individually, you may not burn more than one at a time using the burn action.
* In the issueMultiple and burn actions, container tokens and non-container tokens cannot be mixed. You can burn up to EITHER 50 non-container tokens OR 1 container token at a time.

### token IDs

Each issued NFT token has its own unique ID number. This ID starts at 1 for each token symbol, and increments by 1 every time a new token is issued. Thus if you issue a new instance of a MYNFT, and the ID of that newly issued token is 501, you know it is the 501st MYNFT to be issued. Token ID numbers can be used to look up data properties through RPC queries, or to identify tokens to be the subject of transfer, burn, delegate, etc actions.

Token IDs are always represented as strings when used in arguments to contract actions. Note that ID numbers are not globally unique; they will only be unique for a given symbol (i.e. MYNFT can have a token with id 5, and OTHERNFT can also have a token with id 5). The combination of symbol and ID number uniquely identifies a token (or NFT instance, as we sometimes refer to them).

### issue:
Issues a new instance of an NFT to a Steem account or smart contract. Requires the issuance fee to be paid as described above. The ID of the newly issued token will be returned as part of an "issue" emission in transaction logs.
* requires active key: yes

* can be called by: Steem account or smart contract on the authorized list of issuing accounts/contracts for the NFT in question

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * to (string): Steem account or smart contract to issue the token to
  * feeSymbol (string): the symbol of the regular Steem Engine token to be used for paying issuance fees. Initially, ENG and PAL are the only valid fee symbols, though other types may be added later.
  * **(optional)** fromType (string): indicates whether this action is being called by a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user. Note that a smart contract can still set this to user in order to perform the issuance on behalf of a Steem account rather than the calling contract itself.
  * **(optional)** toType (string): indicates whether the destination specified by the "to" parameter is a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user.
  * **(optional)** lockTokens (dictionary object): if desired, specifies a basket of regular Steem Engine tokens to be locked within the newly issued NFT instance, as described above. Should be formatted as follows: ``{"SYMBOLONE": "quantity to lock", "SYMBOLTWO": "quantity to lock", ...}``.
  * **(optional)** lockNfts (array of object): if desired, specifies a basket of other NFT instances to be locked within the newly issued NFT instance, as described above. Should be formatted as follows: ``[ {"symbol":"SYMBOLONE", "ids":["1","2","3", ...]}, {"symbol":"SYMBOLTWO", "ids":["1","2","3", ...]}, ... ]``
  * **(optional)** properties (dictionary object): if desired, data properties can be set directly at issuance time using this parameter. Should be formatted as follows: ``{ "name1":"value1", "name2":"value2", ... }``

* examples:
```
// issue from user -> user
{
    "contractName": "nft",
    "contractAction": "issue",
    "contractPayload": {
        "symbol": "TESTNFT",
        "to": "aggroed",
        "feeSymbol": "PAL"
    }
}

// issue from user -> contract
{
    "contractName": "nft",
    "contractAction": "issue",
    "contractPayload": {
        "symbol": "TESTNFT",
        "to": "acontract",
        "toType": "contract",
        "feeSymbol": "ENG",
        "lockTokens": {
            "ENG": "3.5",
            "BETA": "10",
            "SCT": "1.123"
        }
    }
}

// issue from contract -> contract
{
    "contractName": "nft",
    "contractAction": "issue",
    "contractPayload": {
        "symbol": "TESTNFT",
        "to": "acontract",
        "toType": "contract",
        "fromType":"contract",
        "feeSymbol": "ENG",
        "properties": {
            "color": "gold",
            "xp": 0,
            "isPremium": false
        }
    }
}

// issue from contract -> user
{
    "contractName": "nft",
    "contractAction": "issue",
    "contractPayload": {
        "symbol": "DRAGON",
        "to": "cryptomancer",
        "toType": "user",      // could be omitted, but we include it here for clarity
        "fromType":"contract",
        "feeSymbol": "PAL",
        "lockTokens": {
            "SPT": "0.01"
        },
        "lockNfts": [
            {"symbol":"TSTNFT", "ids":["200"]},
            {"symbol":"DRAGON", "ids":["1","1000","10000","9999"]}
        ],
        "properties": {
            "type": "wyvern",
            "xp": 0,
            "hp": 200,
            "isFireBreathing": true
        }
    }
}
```
A successful issue action will emit an "issue" event for each token issued:
``from: issuing account, fromType: user or contract, to: destination account, toType: user or contract, symbol, lockedTokens: list of locked regular tokens, lockedNfts: list of locked NFT instances, properties: any data properties set, id: token ID of newly issued token as an integer``
example:
```
{
    "contract": "nft",
    "event": "issue",
    "data": {
        "from": "cryptomancer",
        "fromType": "user",
        "to": "aggroed",
        "toType": "user",
        "symbol": "TSTNFT",
        "lockedTokens": {"ENG": "10"},
        "lockedNfts": [{"symbol": "TSTNFT", "ids":["3"]}, {"symbol": "TEST", "ids": ["1","2"]}]
        "properties": {"color": "yellow"},
        "id": 4
    }
}
```

### issueMultiple:
Issues multiple NFT instances at once. A maximum of 10 tokens can be issued by calling this action (there are some caveats however, see [locked token restrictions](#locked-token-restrictions) above). Issuance fees & other issuing behavior is same as for the issue action above.
* requires active key: yes

* can be called by: Steem account or smart contract on the authorized list of issuing accounts/contracts for the NFT in question

* parameters:
  * instances (array of object): a list of parameters for each token to be issued, as described above for the issue action

* example:
```
// issue 3 tokens from contract -> a combination of users & contracts
{
    "contractName": "nft",
    "contractAction": "issueMultiple",
    "contractPayload": {
        "instances": [
            {
                "fromType": "contract",
                "symbol": "TSTNFT",
                "to": "aggroed",
                "feeSymbol": "ENG",
                "properties": {"level": 0}
            },
            {
                "fromType": "contract",
                "symbol": "TSTNFT",
                "to": "dice",
                "toType": "contract",
                "feeSymbol": "ENG",
                "lockTokens": {"ENG": "5.75"}
            },
            {
                "fromType": "contract",
                "symbol": "TSTNFT",
                "to": "marc",
                "feeSymbol": "PAL"
            }
        ]
    }
}
```
Under the hood, this action simply calls the issue action repeatedly, passing in the parameters given for each instance. Note that it is possible for some tokens to be issued, and for others to fail if parameters are incorrect (it's NOT all or nothing!).

## Delegation
### enableDelegation:
Enables the delegation feature for a token. A fee of 1000 ENG is required to perform this action.
* requires active key: yes

* can be called by: Steem account that owns the NFT

* parameters:
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * undelegationCooldown (integer): number of days that a user will have to wait until the tokens that were delegated are returned to them (between 1 and 18,250 days - 50 years)

* examples:
```
{
    "contractName": "nft",
    "contractAction": "enableDelegation",
    "contractPayload": {
        "symbol": "TESTNFT",
        "undelegationCooldown": 1, // 1 day to cooldown
    }
}

{
    "contractName": "nft",
    "contractAction": "enableDelegation",
    "contractPayload": {
        "symbol": "TESTNFT",
        "undelegationCooldown": 30, // 30 days to cooldown
    }
}

{
    "contractName": "nft",
    "contractAction": "enableDelegation",
    "contractPayload": {
        "symbol": "TESTNFT",
        "undelegationCooldown": 91, // 13 weeks to cooldown
    }
}
```
Once delegation is enabled, the undelegation cooldown time can NOT be changed, so choose thoughtfully before calling this action! One day is the minimum cooldown time.

### delegate:
Delegates one or more tokens to another account or smart contract. Can only be called if an NFT has previously had delegation enabled through the enableDelegation action. If a token is already delegated, it cannot be delegated again until it has been undelegated.
* requires active key: yes

* can be called by: Steem account or smart contract that holds the token(s) to be delegated

* parameters:
  * to (string): Steem account or smart contract to delegate the token(s) to
  * **(optional)** fromType (string): indicates whether this action is being called by a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user. Note that a smart contract can still set this to user in order to execute the delegation on behalf of a Steem account rather than the calling contract itself.
  * **(optional)** toType (string): indicates whether the target specified by the "to" parameter is a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user.
  * nfts (array of object): list of tokens to delegate. Should be formatted as follows: ``[ {"symbol":"SYMBOLONE", "ids":["1","2","3", ...]}, {"symbol":"SYMBOLTWO", "ids":["1","2","3", ...]}, ... ]``

A maximum of 50 tokens can be delegated in a single call of this action. Note that tokens cannot be delegated to the null account.

* examples:
```
// user -> user
{
    "contractName": "nft",
    "contractAction": "delegate",
    "contractPayload": {
        "to": "cryptomancer",
        "nfts": [
            {"symbol":"TSTNFT", "ids":["200"]},
            {"symbol":"DRAGON", "ids":["1","1000","10000","9999"]}
        ]
    }
}

// user -> contract
{
    "contractName": "nft",
    "contractAction": "delegate",
    "contractPayload": {
        "to": "splinterlands",
        "toType":"contract",
        "nfts": [
            {"symbol":"TSTNFT", "ids":["200"]},
            {"symbol":"DRAGON", "ids":["1","1000","10000","9999"]}
        ]
    }
}

// contract -> contract
{
    "contractName": "nft",
    "contractAction": "delegate",
    "contractPayload": {
        "to": "splinterlands",
        "toType":"contract",
        "fromType":"contract",
        "nfts": [
            {"symbol":"DRAGON", "ids":["111","112"]}
        ]
    }
}

// contract -> user
{
    "contractName": "nft",
    "contractAction": "delegate",
    "contractPayload": {
        "to": "aggroed",
        "toType":"user",      // could have omitted this, but including it here for clarity
        "fromType":"contract",
        "nfts": [
            {"symbol":"NFTNFT", "ids":["987654"]}
        ]
    }
}
```
A successful delegate action will emit a "delegate" event for each token delegated:
``from: token holder, fromType: u or c, to: account delegated to, toType: u or c, symbol, id``
example:
```
{
    "contract": "nft",
    "event": "delegate",
    "data": {
        "from": "testContract",
        "fromType": "c",
        "to": "harpagon",
        "toType": "u",
        "symbol": "TEST",
        "id": "4"
    }
}
```

### undelegate:
Undelegates one or more tokens that have previously been delegated. If an undelegation is currently pending, calling this action again will have no effect. Tokens being undelegated have to wait for the cooldown time (see enableDelegation action above) to elapse before the undelegation will complete.
* requires active key: yes

* can be called by: Steem account or smart contract that holds the token(s) to undelegate

* parameters:
  * **(optional)** fromType (string): indicates whether this action is being called by a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user. Note that a smart contract can still set this to user in order to execute the undelegation on behalf of a Steem account rather than the calling contract itself.
  * nfts (array of object): list of tokens to undelegate. Should be formatted as follows: ``[ {"symbol":"SYMBOLONE", "ids":["1","2","3", ...]}, {"symbol":"SYMBOLTWO", "ids":["1","2","3", ...]}, ... ]``

A maximum of 50 tokens can be undelegated in a single call of this action.

* examples:
```
{
    "contractName": "nft",
    "contractAction": "undelegate",
    "contractPayload": {
        "nfts": [
            {"symbol":"TSTNFT", "ids":["200"]},
            {"symbol":"DRAGON", "ids":["1","1000","10000","9999"]}
        ]
    }
}

{
    "contractName": "nft",
    "contractAction": "undelegate",
    "contractPayload": {
        "fromType":"contract",
        "nfts": [
            {"symbol":"TSTNFT", "ids":["200"]},
            {"symbol":"DRAGON", "ids":["1","1000","10000","9999"]}
        ]
    }
}
```
A successful undelegate action will emit an "undelegateStart" event for each token undelegated:
``from: account undelegated from, fromType: u or c, symbol, id``
example:
```
{
    "contract": "nft",
    "event": "undelegateStart",
    "data": {
        "from": "cryptomancer",
        "fromType": "u",
        "symbol": "TEST",
        "id": "999"
    }
}
```

When the undelegation cooldown is complete, an "undelegateDone" event will be emitted for each completed undelegation grouping:
``symbol, ids: list of token IDs grouped together for undelegation``
example:
```
{
    "contract": "nft",
    "event": "undelegateDone",
    "data": {
        "symbol": "TEST",
        "ids": [ 2, 3, 4 ]
    }
}
```

## Other actions
### transfer:
Transfers one or more tokens to another account or smart contract.
* requires active key: yes

* can be called by: Steem account or smart contract that holds the token(s) to be transferred

* parameters:
  * to (string): Steem account or smart contract to transfer the token(s) to
  * **(optional)** fromType (string): indicates whether this action is being called by a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user. Note that a smart contract can still set this to user in order to execute the transfer on behalf of a Steem account rather than the calling contract itself.
  * **(optional)** toType (string): indicates whether the destination specified by the "to" parameter is a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user.
  * nfts (array of object): list of tokens to transfer. Should be formatted as follows: ``[ {"symbol":"SYMBOLONE", "ids":["1","2","3", ...]}, {"symbol":"SYMBOLTWO", "ids":["1","2","3", ...]}, ... ]``

A maximum of 50 tokens can be transferred in a single call of this action. Note that tokens cannot be transferred if they are currently being delegated to another account. Also, tokens cannot be transferred to null; for that you need to use the burn action.

* examples:
```
// user -> user
{
    "contractName": "nft",
    "contractAction": "transfer",
    "contractPayload": {
        "to": "cryptomancer",
        "nfts": [
            {"symbol":"TSTNFT", "ids":["200"]},
            {"symbol":"DRAGON", "ids":["1","1000","10000","9999"]}
        ]
    }
}

// user -> contract
{
    "contractName": "nft",
    "contractAction": "transfer",
    "contractPayload": {
        "to": "splinterlands",
        "toType":"contract",
        "nfts": [
            {"symbol":"TSTNFT", "ids":["200"]},
            {"symbol":"DRAGON", "ids":["1","1000","10000","9999"]}
        ]
    }
}

// contract -> contract
{
    "contractName": "nft",
    "contractAction": "transfer",
    "contractPayload": {
        "to": "splinterlands",
        "toType":"contract",
        "fromType":"contract",
        "nfts": [
            {"symbol":"DRAGON", "ids":["111","112"]}
        ]
    }
}

// contract -> user
{
    "contractName": "nft",
    "contractAction": "transfer",
    "contractPayload": {
        "to": "aggroed",
        "toType":"user",      // could have omitted this, but including it here for clarity
        "fromType":"contract",
        "nfts": [
            {"symbol":"NFTNFT", "ids":["987654"]}
        ]
    }
}
```
A successful transfer action will emit a "transfer" event for each token transferred:
``from: original token holder, fromType: u or c, to: new token holder, toType: u or c, symbol, id``
example:
```
{
    "contract": "nft",
    "event": "transfer",
    "data": {
        "from": "aggroed",
        "fromType": "u",
        "to": "cryptomancer",
        "toType": "u",
        "symbol": "TEST",
        "id": "1000"
    }
}
```

### burn:
Burns one or more tokens. When a token is burned, it is sent to the null account and the circulating supply of that NFT is reduced by 1. In addition, any locked tokens contained within the burned token will be released and transferred back to the account that held the burned token.
* requires active key: yes

* can be called by: Steem account or smart contract that holds the token(s) to be burned

* parameters:
  * **(optional)** fromType (string): indicates whether this action is being called by a Steem account or a smart contract. Can be set to user or contract. If not specified, defaults to user. Note that a smart contract can still set this to user in order to execute the action on behalf of a Steem account rather than the calling contract itself.
  * nfts (array of object): list of tokens to burn. Should be formatted as follows: ``[ {"symbol":"SYMBOLONE", "ids":["1","2","3", ...]}, {"symbol":"SYMBOLTWO", "ids":["1","2","3", ...]}, ... ]``

A maximum of 50 tokens can be burned in a single call of this action. Note that tokens cannot be burned if they are currently being delegated to another account. There are some additional caveats, see [locked token restrictions](#locked-token-restrictions) above.

* examples:
```
{
    "contractName": "nft",
    "contractAction": "burn",
    "contractPayload": {
        "nfts": [ {"symbol":"TESTNFT", "ids":["666"]} ]
    }
}

{
    "contractName": "nft",
    "contractAction": "burn",
    "contractPayload": {
        "fromType": "contract",
        "nfts": [
            {"symbol":"TESTNFT", "ids":["12","45","882"]},
            {"symbol":"MYNFT", "ids":["672","33"]},
            {"symbol":"DRAGON", "ids":["123456"]}
        ]
    }
}
```
A successful burn action will emit a "burn" event for each token burned:
``account: who burned it, ownedBy: u or c, unlockedTokens: released regular token list, unlockedNfts: released NFT instance list, symbol, id``
example:
```
{
    "contract": "nft",
    "event": "burn",
    "data": {
        "account": "aggroed",
        "ownedBy": "u",
        "unlockedTokens": {"ENG":"15", "TKN":"0.75"},
        "unlockedNfts": [{"symbol": "TSTNFT", "ids": ["1","2"]}, {"symbol": "OTHERNFT", "ids": ["320"]}],
        "symbol": "TSTNFT",
        "id": "305"
    }
}
```

# Tables available:
Note: all tables below have an implicit _id field that provides a unique numeric identifier for each particular object in the database. Most of the time the _id field is not important, so we have omitted it from table descriptions. The one exception is the NFT instance table, as the _id serves as the token ID used to refer to that particular token in data queries and various smart contract actions.
## params:
contains contract parameters such as the current fees
* fields
  * nftCreationFee = the fee in ENG for creating a new NFT through the create action
  * nftIssuanceFee = a mapping of Steem Engine tokens that can be used to pay for new NFT issuance through the issue action. The issuer can choose which of these tokens to pay the fee in.
  * dataPropertyCreationFee = the fee in ENG for adding new data properties to an NFT's definition through the addProperty action. Note that the first 3 properties are free; this fee must be paid for each additional property added beyond the first three.
  * enableDelegationFee = the fee in ENG for enabling delegation support through the enableDelegation action

## nfts:
contains definitions of each NFT
* fields
  * issuer = the Steem account that owns the NFT (i.e. called the create action)
  * symbol = the token symbol
  * name = the human friendly name of the token
  * metadata = token metadata such as the URL and long text description
  * maxSupply = max supply of the token, or 0 if supply is unlimited
  * supply = the number of tokens which have been issued to date
  * circulatingSupply = the number of tokens in circulation (i.e. tokens that have not been burned)
  * delegationEnabled = is delegation enabled for this NFT? (true / false)
  * undelegationCooldown = if delegation is enabled, this will be the number of days of cooldown needed when a token is undelegated
  * authorizedIssuingAccounts = list of Steem accounts authorized to issue tokens on behalf of the NFT owner
  * authorizedIssuingContracts = list of smart contracts authorized to issue tokens on behalf of the NFT owner
  * properties = schema definition of any data properties belonging to this NFT
  * groupBy = list of data property names by which market orders for NFT instances should be grouped 

The properties field is a dictionary mapping property names to schema that have their own structure as follows:
* type = indicates the type of the data property. Can be string, number, or boolean.
* isReadOnly = if true, this data property can be set exactly once and then never changed again.
* authorizedEditingAccounts = list of Steem accounts that are authorized to edit this data property
* authorizedEditingContracts = list of smart contracts that are authorized to edit this data property

example of a typical NFT definition:
```
{
    _id: 1,
    issuer: 'cryptomancer',
    symbol: 'TSTNFT',
    name: 'test NFT',
    metadata: '{\"url\":\"https://my-test-nft.com\",\"icon\":\"https://my-test-nft.com/image.jpg\",\"desc\":\"My cool token will rule the world!\"}',
    maxSupply: 1000,
    supply: 456,
    circulatingSupply: 450,
    delegationEnabled: true,
    undelegationCooldown: 7,
    authorizedIssuingAccounts: [ 'cryptomancer' ],
    authorizedIssuingContracts: [],
    properties: { 
        color:
          {
            type: 'string',
            isReadOnly: false,
            authorizedEditingAccounts: [ 'cryptomancer' ],
            authorizedEditingContracts: []
          },
        level:
          {
            type: 'number',
            isReadOnly: false,
            authorizedEditingAccounts: [ 'cryptomancer' ],
            authorizedEditingContracts: []
          },
        frozen:
          {
            type: 'boolean',
            isReadOnly: true,
            authorizedEditingAccounts: [ 'cryptomancer' ],
            authorizedEditingContracts: []
          },
        isFood:
          {
            type: 'boolean',
            isReadOnly: false,
            authorizedEditingAccounts: [ 'cryptomancer' ],
            authorizedEditingContracts: []
          }
    },
    groupBy: [ 'level', 'isFood' ]
}
```

## pendingUndelegations
contains records of all undelegations which are waiting for cooldown to complete
* fields
  * symbol = the token symbol
  * ids = array of token IDs which are batched together in this undelegation
  * completeTimestamp = timestamp when the undelegation will be completed (in milliseconds)

## SYMBOLinstances
Every NFT symbol has its own separate table to store NFT instances (issued tokens). The instance table name for a particular symbol is formed by taking the symbol and adding "instances" to the end of it. Thus, if you have an NFT called MYNFT, the MYNFTinstances table will store all the issued MYNFT tokens.
* fields
  * _id = the token ID number
  * account = the Steem account or smart contract that holds this particular token
  * ownedBy = indicates if this token is held in a Steem account or smart contract. For a Steem account, the value will be "u".  For a smart contract, the value will be "c".
  * lockedTokens = describes all regular Steem Engine tokens which are locked in this particular NFT instance. If there are no locked tokens, the value will be {}
  * properties = values of all the data properties for this particular NFT instance. If there are no data properties set, the value will be {}
  * **(optional)** lockedNfts = describes all NFT instances which are locked in this particular NFT instance. If there are no locked NFT instances, this field will not exist (will be undefined). For burned tokens on the null account, this field *may* exist but with the value set to an empty array []
  * **(optional)** delegatedTo = if this token is delegated, will contain information about which account or contract the token is delegated to. If there is no delegation, this field will not exist (will be undefined).
  * **(optional)** previousAccount = the Steem account or smart contract that previously held this particular token. Will only be set if the token has been burned or transferred at least once. If a token was bought on the market, previousAccount will be the NFT market contract itself.
  * **(optional)** previousOwnedBy = same meaning as ownedBy, but for the Steem account or smart contract that previously held this particular token. Will only be set if previousAccount is set.

The delegatedTo field has its own structure as follows:
* account = the Steem account or smart contract that the token is delegated to
* ownedBy = indicates if the token is delegated to a Steem account or smart contract. For a Steem account, the value will be "u".  For a smart contract, the value will be "c".
* **(optional)** undelegateAt = if this token is pending undelegation, this will be the timestamp when the undelegation will be completed (in milliseconds). If there is no pending undelegation, this field will not exist (will be undefined)

examples of typical token data:
```
{
    _id: 4,
    account: 'marc',
    ownedBy: 'u',
    lockedTokens: {},
    properties: {}
}

{
    _id: 605,
    account: 'gamecontract',
    ownedBy: 'c',
    lockedTokens: { ENG: '10', SPT: '0.5' },
    properties: { color: 'red', frozen: true }
}

{
    _id: 222,
    account: 'aggroed',
    ownedBy: 'u',
    lockedTokens: { ENG: '10', TKN: '0.01' },
    properties: { color: 'orange' },
    delegatedTo: {
        account: 'cryptomancer',
        ownedBy: 'u'
    }
}

{
    _id: 12345,
    account: 'testContract',
    ownedBy: 'c',
    lockedTokens: {},
    lockedNfts: [ { symbol: 'TSTNFT', ids: [ '333' ] },
                  { symbol: 'TEST', ids: [ '101','202' ] } ],
    properties: { color: 'blue', edition: 1 },
    delegatedTo: {
        account: 'contract2',
        ownedBy: 'c',
        undelegateAt: 1528243200000
    },
    previousAccount: 'aggroed',
    previousOwnedBy: 'u'
}
```

# Example smart contract for token issuance

The crittermanager contract serves as a reference example of how to do Splinterlands style NFT pack issuance. The comments in the source code should give a good idea of how it works.

[Critter Manager smart contract source code](https://github.com/harpagon210/steemsmartcontracts/blob/witnesses/contracts/crittermanager.js)

Everything done in this example contract can be done just as well from Python or Javascript. It's worth repeating that all NFT interactions can be boiled down to simply broadcasting the appropriate custom json commands. It demonstrates the following features:

* allow NFT owner to configure different editions (think Splinterlands ALPHA, BETA, and UNTAMED)
* programmatically create the CRITTER NFT through a contract action
* open a "pack" to generate critters by randomly varying properties such as critter type, rarity, and whether the critter is a gold foil or not.
* allow different pack tokens for each edition - the contract gives users a way to exchange pack tokens for newly issued critters
* update data properties - a user can call a contract action to set a name for critters that he/she owns
