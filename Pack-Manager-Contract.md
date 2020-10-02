Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)
Note: this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

TODO: add content here

# Introduction

The Pack Manager is a smart contract designed for managing the issuance of NFT collectables. Consider a [Splinterlands](https://splinterlands.com/) style issuance mechanism: in Splinterlands you buy pack tokens (ALPHA, BETA, UNTAMED, ORB, SLDICE on Hive Engine). Then you open these packs in the game to generate Monster & Summoner card NFTs. Each pack generates 5 cards. The Pack Manager provides a similar issuance mechanism for your own projects. It's a generalized, more customizable version of what Splinterlands does.

As a user, you feed fungible pack tokens to the Pack Manager, and it "opens" them, sending generated NFT instances back to you.

As an app creator, you create your NFT and register pack -> NFT mappings with the Pack Manager. You set configuration that determines how many NFT instances are issued per pack, their edition, rarity, and various other properties.

# Creating an NFT Collectable

Not just any old NFT is compatible with the Pack Manager. In order for the Pack Manager to issue NFTs, an NFT symbol has to be **under management** with the Pack Manager. We may add a way later on for existing NFTs to be placed under management if they satisfy certain criteria, but currently the only way to have an NFT under management is to create it through the following action:

### createNft:
Creates a new NFT definition and places it under management of the packmanager smart contract. A creation fee of 100 BEE is required. Three data properties will be defined as follows:

Name | Type | Read Only | Authorized Editing Accounts | Authorized Editing Contracts | Description
--- | --- | --- | --- | --- | ---
edition | integer >= 0 | Yes | none | packmanager | A number that indicates which edition an NFT instance belongs to (analogous to Alpha, Beta, Untamed, etc in Splinterlands)
foil | integer >= 0 | Optional, up to creator | Hive account that creates the NFT | packmanager | A number that indicates the foil of an NFT instance (analogous to Gold or Regular in Splinterlands)
type | integer >= 0 | Optional, up to creator | Hive account that creates the NFT | packmanager | A number that indicates the specific type of an NFT instance (analogous to the type of card in Splinterlands, for example Goblin Shaman or Alric Stormbringer)

In addition, the NFT's groupBy will be set to ['edition', 'foil', 'type']. For more information about the purpose of groupBy, see [NFT contract documentation](https://github.com/hive-engine/steemsmartcontracts-wiki/blob/master/NFT-Contracts.md#setgroupby).
* requires active key: yes

* can be called by: Hive account

* parameters:
  * symbol (string): symbol of the new NFT
