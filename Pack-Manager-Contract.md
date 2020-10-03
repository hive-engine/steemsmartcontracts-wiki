Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)
Note: this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

TODO: add content here

# Introduction

The Pack Manager is a smart contract designed for managing the issuance of NFT collectables. Consider a [Splinterlands](https://splinterlands.com/) style issuance mechanism: in Splinterlands you buy pack tokens (ALPHA, BETA, UNTAMED, ORB, SLDICE on Hive Engine). Then you open these packs in the game to generate Monster & Summoner card NFTs. Each pack generates 5 cards. The Pack Manager provides a similar issuance mechanism for your own projects. It's a generalized, more customizable version of what Splinterlands does.

As a user, you feed fungible pack tokens to the Pack Manager, and it "opens" them, sending generated NFT instances back to you.

As an app creator, you create your NFT and register pack -> NFT mappings with the Pack Manager. You set configuration that determines how many NFT instances are issued per pack, their edition, rarity, and various other properties.

# Actions available:
## Creating an NFT Collectable

Not just any old NFT is compatible with the Pack Manager. In order for the Pack Manager to issue NFTs, an NFT symbol has to be **under management** with the Pack Manager. We may add a way later on for existing NFTs to be placed under management if they satisfy certain criteria, but currently the only way to have an NFT under management is to create it through the following action:

### createNft:
Creates a new NFT definition and places it under management of the packmanager smart contract. A creation fee of 100 BEE is required. Three data properties will be defined as follows:

Name | Type | Read Only | Authorized Editing Accounts | Authorized Editing Contracts | Description
--- | --- | --- | --- | --- | ---
edition | integer >= 0 | Yes | none | packmanager | A number that indicates which edition an NFT instance belongs to (analogous to Alpha, Beta, Untamed, etc in Splinterlands)
foil | integer >= 0 | Optional, up to creator | Hive account that creates the NFT | packmanager | A number that indicates the foil of an NFT instance (analogous to Gold or Regular in Splinterlands)
type | integer >= 0 | Optional, up to creator | Hive account that creates the NFT | packmanager | A number that indicates the specific type of an NFT instance (analogous to the type of card in Splinterlands, for example Goblin Shaman or Alric Stormbringer)

Note that the max supply of the NFT will be technically unlimited. Instead of capping supply with a specific number, supply is determined indirectly based on the number of associated pack tokens in existence.

In addition, the NFT's groupBy will be set to ['edition', 'foil', 'type']. For more information about the purpose of groupBy, see [NFT contract documentation](https://github.com/hive-engine/steemsmartcontracts-wiki/blob/master/NFT-Contracts.md#setgroupby).
* requires active key: yes

* can be called by: Hive account

* parameters:
  * name (string): name of the token (letters, numbers, whitespace only, max length of 50)
  * symbol (string): symbol of the token (uppercase letters only, max length of 10)
  * **(optional)** orgName (string): name of the company/organization that created this NFT (letters, numbers, whitespace only, max length of 50)
  * **(optional)** productName (string): product/brand that this NFT is associated with (letters, numbers, whitespace only, max length of 50)
  * **(optional)** url (string): url of the project (max length of 255)
  * **(optional)** isFoilReadOnly (boolean): if true, then the foil data property will be created as read-only. The default value is true if this parameter is not specified.
  * **(optional)** isTypeReadOnly (boolean): if true, then the type data property will be created as read-only. The default value is true if this parameter is not specified.

* examples:
```
{
    "contractName": "packmanager",
    "contractAction": "createNft",
    "contractPayload": {
        "symbol": "WAR",
        "name": "War Game Military Units"
    }
}

{
    "contractName": "packmanager",
    "contractAction": "createNft",
    "contractPayload": {
        "symbol": "WAR",
        "name": "War Game Military Units",
        "orgName": "Wars R Us Inc",
        "productName": "War Game Name Here",
        "url": "https://mywargame.com"
    }
}

{
    "contractName": "packmanager",
    "contractAction": "createNft",
    "contractPayload": {
        "symbol": "WAR",
        "name": "War Game Military Units",
        "orgName": "Wars R Us Inc",
        "productName": "War Game Name Here",
        "url": "https://mywargame.com",
        "isFoilReadOnly": false,
        "isTypeReadOnly": false
    }
}
```

A successful action will emit a "createNft" event: ``symbol``
example:
```
{
    "contract": "packmanager",
    "event": "createNft",
    "data": {
        "symbol": "WAR"
    }
}
```

## Registering Packs

Once you have an NFT under management, the next step is to register pack tokens & corresponding settings that will be associated with the NFT. This is a many-to-many mapping: you can have multiple pack tokens that map to multiple NFTs. For example, you could have small chest, medium chest, and large chest packs that all map to the same NFT and issue 5, 10, and 15 NFT instances per pack, respectively. You could also have a chest token that maps to 2 or more NFTs, so the player can choose which type of NFT it opens as.

Any already existing fungible token can be used as a pack token; it doesn't have to be a new token that was created specifically for this purpose (although for most new projects making a new token is sensible). For example, you could give Splinterlands ALPHA tokens new utility by having them used to generate military units for your new War Game. Or have dCity SIM used as a pack token for your own city simulator.

The following actions are available to manage pack registrations:

### registerPack:

TODO: add content here

### updateSettings:

TODO: add content here

## Defining NFT Instance Types

The third step, after placing an NFT under management and setting up packs, is to define all the different types of NFT instances that may be generated from the packs. Using Splinterlands as an example again, that game has hundreds of different cards you can mix & match to form teams for battle. There's Yodin Zaku, Xander Foxwood, Pirate Captain, and Naga Warrior, to name a few. This is what we mean when we talk about **types** in the context of the Pack Manager. All of these different cards are encompassed by the same NFT definition; they are differentiated internally by a number that represents the type of the card.

To give another example, if you used the Pack Manager to create an NFT for a war game, your types might be the military units used by your game: soldier, general, destroyer, tank, cruise missile, marine, cannon, howitzer, etc.

All types have the following properties, which must be integers >= 0 (the text name of the property is stored elsewhere and linked to the numeric value):

* **category**: what category of thing is this type? ex: boat (types can be canoe, sailboat, yacht, tanker), plane (types can be Boeing 747, F-22A Raptor, Japanese Zero), land (types can be swamp, mountain, forest). Splinterlands cards can be Monsters or Summoners.
* **rarity**: how rare is this type? ex: common, uncommon, epic, legendary
* **team**: what grouping, if any, should this type be part of? ex: Splinterlands has cards grouped into Fire, Water, Earth, Life, Death, Dragon, and Neutrals. For sports cards, you might use the names of various sports teams.

New types can be added at any time, even after NFTs have been issued (maybe you want to add a new type to be airdropped once a certain number of packs have been opened, for example). Existing types can also be edited. However, types can only be deleted if no NFT instances have been issued yet (i.e. if the NFT creator is still working on getting everything setup). Once NFTs have been issued, it wouldn't make sense to be able to delete types (you don't want that expensive Dragon you bought to suddenly be removed from the game and made worthless, right?).

The following actions are available to manage types:

### addType:
Adds a new type for an NFT currently under management. A 1 BEE fee is required every time this contract action is used. When added, new types are assigned an ID number that starts at 0 and goes up by 1 for each new type added. So the first type for a given NFT and edition will have ID 0, the 2nd will have ID 1, the 3rd will have ID 2, and so on.
* requires active key: yes

* can be called by: Hive account that created/owns the NFT in question

* parameters:
  * nftSymbol (string): symbol of the NFT (uppercase letters only, max length of 10)
  * edition (integer >= 0): what edition does this type belong to (In Splinterlands there is Alpha, Beta, Untamed; other projects might have a 1st Edition, 2nd Edition, etc)? Note that type values are distinct per NFT per edition, and the edition must have been previously registered with the registerPack action.
  * category (integer >= 0): what is the category of the new type?
  * rarity (integer >= 0): what is the rarity of the new type?
  * team (integer >= 0): what is the team of the new type?
  * name: what is the name of the type? Names must consist of letters, numbers, and whitespaces only, with a maximum length of 100 characters.

* example:
```
{
    "contractName": "packmanager",
    "contractAction": "addType",
    "contractPayload": {
        "nftSymbol": "WAR",
        "edition": 0,
        "category": 1,
        "rarity": 2,
        "team": 0,
        "name": "F22A Raptor"
    }
}
```

A successful action will emit an "addType" event: ``nft, edition, typeId (ID number of the newly added type), rowId (internal db ID number from the implicit _id field)``
example:
```
{
    "contract": "packmanager",
    "event": "addType",
    "data": {
        "nft": "WAR",
        "edition": 0,
        "typeId": 0,
        "rowId": 567
    }
}
```

Any given type can be uniquely identified by the combination of nft, edition, and typeId. For direct db queries, the rowId also uniquely identifies the type.
