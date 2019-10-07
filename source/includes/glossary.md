# Glossary

### Asset / Item 
An Asset or Item is a general term for both Unique Assets (or NFT) and Stackable Assets. 

### NFT 
A [Non-Fungible-Token (NFT)](http://erc721.org/) is a unique representation of an Asset or Item on the blockchain. We use the term Unique Asset to represent NFT’s on the Forte Platform. See, Unique Asset

### Unique Asset 
Unique Assets (Also referred to as “Non-Fungible Token” - NFT) are based on the [ERC-721 specification](http://erc721.org/). Each Unique Asset created is truly unique, including assets that appear identical, and have a unique ID per asset. Examples include a unique character, weapon, vehicle or skin. 

### Stackable Asset 
Stackable Assets are a “asset class” or “type” (based on the [ERC-1155 specification](https://github.com/ethereum/eips/issues/1155)) of assets that do not require a record of state (e.g. “evolved” or “level-up”) or transaction history (e.g. A Unique item once owned by Sir Patrick Stewart). As an example, Stackable Assets can be used when players collect multiples of crafting materials (e.g. leather scraps, wood, brick) or multiples of the same cards in a deck. Once a Stackable Asset type has been defined, developers can mint and transfer any number of the specified Stackable Asset to players. As an example, imagine a player has to collect wood as a material used for building. As a Developer, you would define a Stackable Asset type as “Wood” with specific defining metadata to be created and transferred to users as needed. More info on Stackable Assets.

### Game Coins 
Based on the [ERC-20 specification](https://github.com/ethereum/eips/issues/20), Game Coins are the foundation currency used to buy and sell assets in games. Game Coins are considered fungible tokens, meaning  each individual coin is essentially interchangeable, 1 Game Coin will always be exchangeable for 1 Game Coin. 

### Token(s) 
In the blockchain world, a Token is any coin, asset or item that can be held in a wallet and transacted on-chain. We use the term “Token” as a general term for all Unique Assets, Stackable Assets and Game Coins.

### Player 
The end-user of a game, also referred to as a “user”.

### Developer 
A Developer is a 3rd party (external from Forte) game developer and customer of the Forte Application Platform API. 

### Forte Platform 
The Forte Platform (or Platform) refers to all services, products and functionality built by Forte and provided to Developers to integrate and enable blockchain games. 

### Automated Marketplace (AMM) 
Also referred to as the Automated Market Maker (AMM), the Automated Marketplace is a dedicated service that simplifies the execution of buys, sells and trades. The AMM is a complex system that, amongst other things, allows for instant asset tradeability (liquidity), price discovery and marketplace trading systems and functionality. The Automated Marketplace API is a dedicated API used by Developers to facilitate transactions. 

### Mint 
Minting simply creates a Token (Game Coin, Unique Asset and Stackable Asset) on the blockchain and associates the newly created Token with a Wallet address. 