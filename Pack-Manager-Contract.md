Documentation written by [bt-cryptomancer](https://github.com/bt-cryptomancer)

# Table of Contents

TODO: add content here

# Introduction

The Pack Manager is a smart contract designed for managing the issuance of NFT collectables. Consider a [Splinterlands](https://splinterlands.com/) style issuance mechanism: in Splinterlands you buy pack tokens (ALPHA, BETA, UNTAMED, ORB, SLDICE on Hive Engine). Then you open these packs in the game to generate Monster & Summoner card NFTs. Each pack generates 5 cards. The Pack Manager provides a similar issuance mechanism for your own projects. It's a generalized, more customizable version of what Splinterlands does.

As a user, you feed fungible pack tokens to the Pack Manager, and it "opens" them, sending generated NFT instances back to you.

As an app creator, you create your NFT and register pack -> NFT mappings with the Pack Manager. You set configuration that determines how many NFT instances are issued per pack, their edition, rarity, and various other properties.

# Creating an NFT Collectable

