# Getting Started

Welcome to the Forte Platform API! The Forte Platform API is the primary access point for interacting and integrating with Forte components. Its host of APIs provides tools to read and write transactions on the blockchain, and provides access to Wallet and Marketplace functionality. 

This section will get you started using the Platform API. You will create a game, create Game Coins, Stackable and Unique Assets for your game, and put them to use in a practical way.

<aside class="notice">
Check out the [glossary](#glossary) to help get you acquainted with some of the terms used in the API documentation.
 </aside>
 

## Overview

Platform-API is a restful API that runs on port 8088.

**Default URL**: `http://0.0.0.0:8088/`

- Installing the correct components
- Build Docker Image
- Starting the Cluster
- Startup Script
- First API Call
- Create a game, get API key
- Game Info
- Developer Wallet Balance
- Create a Game User
- Game all Game Users
- Get a User's Balance
- Transfer Game Coins
- User Transaction History
- Transfer Game Coins Between Users
- Transfer an Unique Assets
- Transfer Game Coins into a Unique Asset
- Check Unique Asset Balance (and retreive metadata)
- Create a Token Pool Wallet

## Installing the correct components

**Docker** with **docker-compose** must be installed on your machine.

[Install Docker for Mac](https://docs.docker.com/docker-for-mac/install/)

The docker cli is installed during the first run of the docker application.

## Build Docker Image

First, download the `build-image.sh` script:

`curl -O "http://forte-forte-platform.s3.us-east-2.amazonaws.com/<%= config[:version] %>/build-image.sh"`

Then, change the permissions to allow the script to execute:

`sudo chmod 775 build-image.sh`

Lastly, build the image by running the `build-image.sh` script and following the prompts. You can get the version number from your Forte representative.
This process downloads and builds the image from the AWS bucket, so it may take a while depending on your internet connection.

`./build-image.sh`

## Starting the Cluster

Now run the workstation `start.sh` script. This will spin up the Platform containers.

`./build/workstation/start.sh`

## Startup Script

To use the platform-api, you'll need to run some startup tasks. Open another terminal and navigate to the project root (where you were at on the previous step).
Then run the Platform `start.sh` script to get your environment set up:

`./build/start.sh`

## First API Call

```shell
curl http://0.0.0.0:8088/health
```

> The above command returns JSON structured like this:

```json
{ "status": "ok" }
```

Let's see if it's working...

### HTTP Request

`GET http://0.0.0.0:8088/health`

## Create a game, get API key

```shell
curl -X POST \
	http://0.0.0.0:8088/api/v1/game \
	-H 'Content-Type: application/json' \
	-H 'Authorization: Bearer tempKey' \
	-d '{
	"name": "newGame",
	"email": "test@platform.rules",
	"password": "YOUR_PASSWORD",
	"initial_tokens": 1000000,
	"blockchain_url": "http://game_client:8545",
  "bonding_curve_shape": "linear",
	"initial_slope": 1.0
	}'
```

> The above command returns JSON structured like this:

```json
{
  "id": "8229154f-ebd1-4448-9408-e62b5a5f6685",
  "name": "newGame",
  "api_key": "6625d19e-dd7a-4566-a590-0a84ad121b3f",
  "dev_user_id": "d03356a2-b262-4359-8b6a-21baf8455eb6",
  "contracts": [
    {
      "id": "c7276e75-a18d-46b5-b3b4-acbd02de9888",
      "game_id": "35ccc0fb-fd72-43f7-8e5c-7a7dafffc5ab",
      "kind": "erc20",
      "address": "0x4582378879a4759e23c9d311cc3d0a40ae3a2e01",
      "created_by": "13d86a47-abcf-457c-9531-05fc11c8d5ee",
      "created_at": "2019-06-03T16:51:58.051444Z",
      "updated_by": "13d86a47-abcf-457c-9531-05fc11c8d5ee",
      "updated_at": "2019-06-03T16:51:58.051444Z"
    },
    {
      "id": "0dbbdf0b-19ab-4aa0-b915-292b729774d7",
      "game_id": "35ccc0fb-fd72-43f7-8e5c-7a7dafffc5ab",
      "kind": "erc721",
      "address": "0x380f6fa5ac8b7e4e9ab7b0c0020889cd43d204cd",
      "created_by": "13d86a47-abcf-457c-9531-05fc11c8d5ee",
      "created_at": "2019-06-03T16:51:58.051444Z",
      "updated_by": "13d86a47-abcf-457c-9531-05fc11c8d5ee",
      "updated_at": "2019-06-03T16:51:58.051444Z"
    }
  ]
}
```

We'll start by creating a new game. The `/game` endpoint automatically creates a Developer Wallet and mints (creates) a given amount of Game Coins (ERC-20 tokens) for your game to use as the foundation currency. In addition, this endpoint creates the necessary contracts and Wallet services to execute transactions in your game.

Use the example to create a game called "newGame" with 1,000,000 Game Coins(`initial_tokens`). If you decide you want more Game Coins later you can use the `/token/mint` endpoint.

Take note of the `dev_user_id`. We will use this to transact with the newly created Developer Wallet and players.

Set the following Parameters:

- `bonding_curve_shape` = `linear` - This will use a linear pricing curve for when we stake Game Coins to Assets. 
- `initial_slope` = `1.0` - This is the slope of the linear pricing curve to change the Game Coin price per unit of supply. More on pricing curves and staking Game Coins later.


<aside class="notice">The <code>bonding_curve_shape</code> parameter can be either <code>'linear'</code>, which will use the <code>slope</code> parameter to create a linear pricing curve, or <code>'sigmoidal'</code>, which will deploy a pre-defined sigmoidal shaped pricing curve.</aside>

<aside class="notice">The <code>blockchain_url</code> parameter refers to the docker-compose parity instance. Unless you know what you're doing, you should use the default value provided in the example.</aside>

**Note: This command uses an 'Authorization' header with the value 'tempKey' because a valid api key does not exist prior to game creation. 'tempKey' matches the value found in the environment variable `APP_TOKEN_NAME`. You will need to include the 'Authorization' header with the api key value returned by this call when authenticating every call following this.**

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game`

### Body Parameters

| Parameter            | Description                                    |
| -------------------- | ---------------------------------------------- |
| name                 | Name of your game                              |
| email                | Your email address                             |
| password             | Your account password                          |
| initial_tokens       | Amount of tokens to start with                 |
| blockchain_url       | URL of blockchain to interface with (advanced) |
| bonding_curve_shape  | The shape of bonding curve                     |
| initial_slope        | The change in price for each unit of supply    |

> Set environment variables

```shell
export GAME_ID=8229154f-ebd1-4448-9408-e62b5a5f6685
export API_KEY=6625d19e-dd7a-4566-a590-0a84ad121b3f
```

For brevity, we set an environment variable for our `GAME_ID` and `API_KEY` based on the response of the previous request.

## Game Info

```shell
curl -X GET \
	http://0.0.0.0:8088/api/v1/game/$GAME_ID \
	-H "Authorization: Bearer $API_KEY"
```

> The above command returns JSON structured like this:

```json
{
  "id": "8229154f-ebd1-4448-9408-e62b5a5f6685",
  "name": "newGame",
  "api_key": "6625d19e-dd7a-4566-a590-0a84ad121b3f",
  "dev_user_id": "d03356a2-b262-4359-8b6a-21baf8455eb6",
  "contracts": [
    {
      "id": "c7276e75-a18d-46b5-b3b4-acbd02de9888",
      "game_id": "35ccc0fb-fd72-43f7-8e5c-7a7dafffc5ab",
      "kind": "erc20",
      "address": "0x4582378879a4759e23c9d311cc3d0a40ae3a2e01",
      "created_by": "13d86a47-abcf-457c-9531-05fc11c8d5ee",
      "created_at": "2019-06-03T16:51:58.051444Z",
      "updated_by": "13d86a47-abcf-457c-9531-05fc11c8d5ee",
      "updated_at": "2019-06-03T16:51:58.051444Z"
    },
    {
      "id": "0dbbdf0b-19ab-4aa0-b915-292b729774d7",
      "game_id": "35ccc0fb-fd72-43f7-8e5c-7a7dafffc5ab",
      "kind": "erc721",
      "address": "0x380f6fa5ac8b7e4e9ab7b0c0020889cd43d204cd",
      "created_by": "13d86a47-abcf-457c-9531-05fc11c8d5ee",
      "created_at": "2019-06-03T16:51:58.051444Z",
      "updated_by": "13d86a47-abcf-457c-9531-05fc11c8d5ee",
      "updated_at": "2019-06-03T16:51:58.051444Z"
    }
  ]
}
```

Now that we have created a game, we can use this endpoint to return detailed information about our game by passing the `GAME_ID`. You can view an example of the returned JSON structure. 

### HTTP Request

`GET http://0.0.0.0:8088/api/v1/game/<GAME_ID>`

### URL Parameters

| Parameter | Description                    |
| --------- | ------------------------------ |
| GAME_ID   | The ID of the game to retrieve |

## Developer Wallet Balance

```shell
curl -X GET \
	http://0.0.0.0:8088/api/v1/game/$GAME_ID/developer/wallet \
	-H "Authorization: Bearer $API_KEY"
```

> The above command returns JSON structured like this:

```json
{
  "tokens": [
    {
      "name": "TBC Token",
      "symbol": "TBC",
      "balance": 0
    },
    {
      "name": "Token",
      "symbol": "TOK",
      "balance": 1000000
    }
  ],
  "items": [
    {
      "name": "token_name",
      "balance": 190177113409627147017492480768293327364543271987
    }
  ],
  "stackables": [
    {
      "token_id": "1020847100762815390390123822295304634368",
      "balance": 3
    },
    {
      "token_id": "340282366920938463463374607431768211456",
      "balance": 2
    },
    {
      "token_id": "680564733841876926926749214863536422912",
      "balance": 1
    }
  ]
}
```

When we created our game, a Developer Wallet was created for us. The Developer Wallet account holds, manages and transfers Game Coins and Items between your developer account `dev_user_id` and player Wallet accounts. When you create a game, the Game Coins created are minted to your Developer Wallet account. This endpoint gets the balance of your Developer Wallet account.

The `balance` should reflect the number of Game Coins we set as our `initial_tokens`, "1000000" when we created our game. 

### HTTP Request

`GET http://0.0.0.0:8088/api/v1/game/<GAME_ID>/developer/wallet`

### URL Parameters

| Parameter | Description                                              |
| --------- | -------------------------------------------------------- |
| GAME_ID   | The ID of the game to retrieve the developer balance of. |

## Create a USer (Player)

```shell
curl -X POST \
	http://0.0.0.0:8088/api/v1/game/$GAME_ID/user \
	-H "Authorization: Bearer $API_KEY" \
	-H 'Content-Type: application/json' \
	-d '{
	"email": "brock@firstfoundry.co"
	}'
```

> The above command returns JSON structured like this:

```json
{
  "id": "6753d59d-9fa3-44f8-bfda-29d64ae7f152",
  "game_id": "8229154f-ebd1-4448-9408-e62b5a5f6685",
  "email": "brock@firstfoundry.co",
  "wallet_address": "0xe0DA6ab11545043eF141Df24A9211b7792356D0c",
  "wallet_user_uid": "f5cca3b4-788e-4fa7-be98-cf1f6ac5972f",
  "created_by": "badf8f8e-9690-4806-893c-176dc78470a3",
  "created_at": "2019-06-14T16:25:34.555304",
  "updated_by": "badf8f8e-9690-4806-893c-176dc78470a3",
  "updated_at": "2019-06-14T16:25:34.555304"
}
```

This endpoint creates a new player for your game and automatically creates a Wallet address to enable the player to transact and hold Game Coins, Stackable and Unique Assets. Creating a player will also generate an id used to pass as the player `USER_ID` to execute player actions including, initiating transactions and querying player details. 

The response includes the following: 

- `id` - This is the `USER_ID` for the individual player created. This ID can be used to initiate transactions and query player information. 
- `game_id` - The Game ID associated with the created player. 
- `email` - **OPTIONAL** This is an optional field and can be left empty. The email address of the created player.
- `wallet_address` - The blockchain wallet address. Note, the USER_ID is used to initiate transactions. 
- `wallet_user_uid` - This id is created by the Wallet and can be ignored and not to be confused with the “id” or USER_ID. 
- `created_by` - The Developer Wallet dev_user_id that created the User.
- `created_at` - UTC date of when the player was created. 
- `updated_by` - The Developer Wallet dev_user_id that last updated the User.
- `updated_at` - UTC date of when the player was last updated. 

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/user`

### URL Parameters

| Parameter | Description                             |
| --------- | --------------------------------------- |
| GAME_ID   | The ID of the game to create a user for |

### Body Parameters

| Parameter | Description          |
| --------- | -------------------- |
| name      | User's full name     |
| email     | (Optional) User's email address |

> Set environment variable

```shell
export USER_ID=6753d59d-9fa3-44f8-bfda-29d64ae7f152
```

Let's set another environment variable for our user ID.

## Get all Game Users

```shell
curl -X GET \
	http://0.0.0.0:8088/api/v1/game/$GAME_ID/user \
	-H "Authorization: Bearer $API_KEY" \
	-H 'Content-Type: application/json'
```

> The above command returns JSON structured like this:

```json
{
  "users": [
    {
      "user_id": "b9c78b96-d9da-41fc-8846-e2393c0b7376",
      "email": "dave@fakeemail.com",
      "wallet_address": "0x7E0135f53AB0A9D4F614913B0c200a0f97d627B9"
    },
    {
      "user_id": "3d697552-6e6d-41d8-ad5f-13229f3932d8",
      "email": "buster@totallyrealemail.com",
      "wallet_address": "0x76CF00FDe51c8FC44464BbEa220049c67F815FaE"
    }
  ]
}
```

To view all players that belong to a particular game, use the `/game/{GAME_ID}/user` endpoint.

### HTTP Request

`GET http://0.0.0.0:8088/api/v1/game/<GAME_ID>/user`

### URL Parameters

| Parameter | Description                               |
| --------- | ----------------------------------------- |
| GAME_ID   | The ID of the game to retrieve users from |

## Get a User's Balance

```shell
curl -X GET \
  http://0.0.0.0:8088/api/v1/game/$GAME_ID/user/$USER_ID \
  -H "Authorization: Bearer $API_KEY"
```

> The above command returns JSON structured like this:

```json
{
  "tokens": [
    {
      "name": "TBC Token",
      "symbol": "TBC",
      "balance": 0
    },
    {
      "name": "Token",
      "symbol": "TOK",
      "balance": 0
    }
  ],
  "items": [
    {
      "name": "token_name",
      "balance": "190177113409627147017492480768293327364543271987"
    }
  ],
  "stackables": [
    {
      "token_id": "1020847100762815390390123822295304634368",
      "balance": 3
    },
    {
      "token_id": "340282366920938463463374607431768211456",
      "balance": 2
    },
    {
      "token_id": "680564733841876926926749214863536422912",
      "balance": 1
    }
  ]
}
```

Returns the balance for all Game Coin, Stackables and Unique Assets for a given `USER_ID`.

### HTTP Request

`GET http://0.0.0.0:8088/api/v1/game/<GAME_ID>/user/<USER_ID>`

### URL Parameters

| Parameter | Description                        |
| --------- | ---------------------------------- |
| GAME_ID   | ID of the game                     |
| USER_ID   | ID of the user to check balance of |

> Set environment variable

```shell
export ITEM_ID=46340168683920357099280855628767929578552584533
```

Let's set an environment variable for our item ID as well.

## Transfer Game Coins From Dev to User

```shell
curl -X POST \
  http://0.0.0.0:8088/api/v1/game/$GAME_ID/developer/token/transfer \
  -H "Authorization: Bearer $API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
    "user_id_to": "'"$USER_ID"'",
	  "amount": 100
}'
```

> The above command returns 200 OK:

```shell
200 OK
```

Now that you have a user to transact with, you can send them some Game Coins from the Developer Wallet. This endpoint sends Game Coins from the "Developer Wallet" to players `user_id`. 

Let's send an `amount` of 100 Game Coins to the previously created player `user_id`. 

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/developer/token/transfer`

### URL Parameters

| Parameter | Description    |
| --------- | -------------- |
| GAME_ID   | ID of the game |

### Body Parameters

| Parameter  | Description                      |
| ---------- | -------------------------------- |
| user_id_to | ID of user to transfer tokens to |
| amount     | Number of tokens to transfer     |

## User Transaction History

```shell
curl -X POST \
  http://0.0.0.0:8088/api/v1/game/$GAME_ID/user/$USER_ID/transactions \
  -H "Authorization: Bearer $API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
	"start_date": "2019-04-30T22:32:45.192270Z",
	"end_date": "2019-05-07T22:32:45.192357Z",
	"page": 1,
	"per_page": 10
}'
```

> The above command returns JSON structured like this:

```json
{
  "transaction_history": [
    {
      "transaction_id": "388892a0-c843-4cc4-b94d-45a4679ad4c5",
      "wallet_from": "0x2bCf16681ce5a244b01A7E619Cce31577ec47089",
      "wallet_to": "0xE42CDaEd77c4E6DEDB613fBF9f0De823804EFf4c",
      "nonce": 0,
      "token_type": "ERC20",
      "transaction_data": "{\"amount\": 10}",
      "transaction_hash": "0x00eac78f5d8902fcaa6b1e8ddb8f580dfdc610468e21b98e403b06a705075053",
      "ip": "0.0.0.0"
    },
    ...
  ]
}
```

Let's get the transaction we just made by viewing the transaction history for our player using this endpoint to return a user's transaction history. Make sure to enter appropriate UTC-encoded date formats for the transaction range.

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/user/<USER_ID>/transactions`

### URL Parameters

| Parameter | Description                        |
| --------- | ---------------------------------- |
| GAME_ID   | ID of the game                     |
| USER_ID   | ID of user to get transactions for |

### Body Parameters

| Parameter  | Description                                               |
| ---------- | --------------------------------------------------------- |
| start_date | UTC-encoded timestamp indicating date/time to begin query |
| end_date   | UTC-encoded timestamp indicating date/time to end query   |
| page       | Pagination page, starts at 1                              |
| per_page   | Number of results to return per page                      |

## Transfer Tokens Between Users

```shell
curl -X POST \
  http://0.0.0.0:8088/api/v1/game/$GAME_ID/token/transfer \
  -H "Authorization: Bearer $API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
    "user_id_from": "'"$USER_ID"'",
    "user_id_to": "'"$USER_ID"'",
    "amount": 1
}'
```

> The above command returns 200 OK:

```shell
200 OK
```

Similar to transferring tokens between players and the Developer Wallet, you can transfer tokens between two players as a peer-to-peer transaction using this endpoint. 

To transact between users, first create another game user, make a note of the new users user_id and transfer a few tokens between the previously created user and the new user.

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/token/transfer`

### URL Parameters

| Parameter | Description    |
| --------- | -------------- |
| GAME_ID   | ID of the game |

### Body Parameters

| Parameter    | Description                    |
| ------------ | ------------------------------ |
| user_id_from | ID of user to send tokens from |
| user_id_to   | ID of user to receive tokens   |
| amount       | Number of tokens to send       |

## Mint a Unique Asset 

```shell
curl -X POST \
  http://0.0.0.0:8088/api/v1/game/$GAME_ID/item/mint \
  -H "Authorization: Bearer $API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
    "user_id_to": "'"$USER_ID"'",
    "creator_stake": 100,
    "metadata": "{fire_power: 1000, max_cooldown: 5}",
    "key":"item_key"
  }'
```

> The above command returns 200 OK:

```shell
200 OK
```

This endpoint allows you to create new Unique Assets. These Unique Assets (or Items) are minted as ERC-721 Tokens. You can define `metadata` to determine the Item’s attributes (e.g. fire power, cool down times, etc.). All game assets will require developers to stake Game Coins when the asset is minted. 

We will stake 100 Game Coins from the Developer Wallet in the `creator_stake` Parameter.

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/item/mint`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |
| ITEM_ID   | ID of item to transfer |

### Body Parameters

| Parameter     | Type   | Description                                                  |
| ------------- | ------ | ------------------------------------------------------------ |
| user_id_to    | String | ID of user to mint item to                                   |
| metadata      | String | JSON-encoded metadata                                        |
| key           | String | Optional key that can be used to fetch metadata for the item |
| creator_stake | Number | The inital amount of ERC-20s deposited into the item         |

## Transfer a Unique Asset

```shell
curl -X POST \
  http://0.0.0.0:8088/api/v1/game/$GAME_ID/item/$ITEM_ID/transfer \
  -H "Authorization: Bearer $API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
  "user_id_from": "'"$USER_ID"'",
  "user_id_to": "'"$USER_ID"'"
}'
```

> The above command returns 200 OK:

```shell
200 OK
```

Now that we have created a Unique Asset, we can transfer it, along with the 100 Game Coins set in the `creator_stake`, to another player.

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/item/<ITEM_ID>/transfer`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |
| ITEM_ID   | ID of item to transfer |

### Body Parameters

| Parameter    | Description                     |
| ------------ | ------------------------------- |
| user_id_from | ID of user to transfer NFT from |
| user_id_to   | ID of user to transfer NFT to   |

## Transfer Game Coins into a Unique Asset

When we created our game, we minted 1,000,000 tokens to our developer account. In [Mint an NFT](#mint-an-nft) we minted an NFT and gave it to a user account.

A key feature of our NFTs is that they can _hold their own tokens_. This allows NFTs to hold value.

```shell
curl -X POST \
  http://localhost:8088/api/v1/game/$GAME_ID/developer/token/deposit \
  -H "Authorization: Bearer $API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"token_id": "'"$ITEM_ID"'","amount": 1}'
```

> The above command returns JSON structured like this:

```shell
200 OK
```

A key feature of our Assets is that they can hold their own Game Coins. This allows Assets to hold value, creating a minimum value. 

Now, let's transfer some Game Coins from the Developer Wallet Account into the created Unique Asset by defining the `token_id` of the created Unique Asset and send an `amount` of "1" Game Coin to the Asset.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/developer/token/deposit`

### URL Parameters

| Parameter | Description    |
| --------- | -------------- |
| GAME_ID   | ID of the game |

### Body Parameters

| Parameter | Description                          |
| --------- | ------------------------------------ |
| token_id  | ID of NFT to deposit tokens into     |
| amount    | Number of tokens to deposit into NFT |

## Get Unique Asset Balance and metadata

We can check the balance and retrieve metadata for our Unique Asset by passing the `GAME_ID` and `ITEM_ID` parameters in the URL.

```shell
curl -X GET \
  http://0.0.0.0:8088/api/v1/game/$GAME_ID/item/$ITEM_ID \
  -H "Authorization: Bearer $API_KEY"
```

> The above command returns JSON structured like this:

```json
{
  "token_id": "874619939929244439599189760038633387039161848844",
  "owner_id": "9b10fa3d-cf6c-4a22-9c4a-f0356cc3de29",
  "tokens": [
    "ERC20BalanceCondensed" {
      "name": "Token",
      "symbol": "TOK",
      "balance": 0
    },
    "ERC20BalanceCondensed" {
      "name": "TBC Token",
      "symbol": "TBC",
      "balance": 1
    }
  ],
  "metadata": {
    "fire_power": 1000,
    "max_cooldown": 5
  }
}
```

### URL Parameters

| Parameter | Description                  |
| --------- | ---------------------------- |
| GAME_ID   | ID of the game               |
| ITEM_ID   | ID of item to get balance of |

## Create a Token Pool Wallet

```shell
curl -X POST \
  http://0.0.0.0:8088/api/v1/game/$GAME_ID/developer/wallet/pool \
  -H "Authorization: Bearer $API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"name": "Token Pool Wallet Name","max_tokens": 100, "interval": "0 0 0 1/1 * ? *"}'
```

> The above command returns JSON structured like this:

```json
{
  "token_pool_id": "0x84ca43b1836ef70175d0974e70094ab5d4f4d4ba",
  "name": "30f3c17c-b3c3-40b5-b45e-e0d246f4a25f",
  "max_tokens": 100,
  "interval": "0 0 0 1/1 * ? *",
  "tokens": 100
}
```

The Token Pool Wallet is a secondary Wallet owned by the Developer Wallet account, designed to automatically refill and distribute tokens to users. Multiple Token Pool Wallets can be created to enable various game transactions and functions.  

Working with a limited amount of Game Coins, the main Token Pool Wallet will not run out. In the `interval` field, a developer may specify a cron expression to set how often a pool will be refilled. Transaction requests will be rejected if the Token Pool Wallet token supply is depleted. The `max_tokens` field sets the maximum number of tokens able to transfer to a Token Pool Wallet. Once this `max_tokens` is reached tokens will no longer automatically transfer to the Token Pool Wallet until this field is below the set max.

For more information about cron expressions check out
[cronmaker](http://www.cronmaker.com/).

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/developer/wallet/pool`

### URL Parameters

| Parameter | Description    |
| --------- | -------------- |
| GAME_ID   | ID of the game |

### Body Parameters

| Parameter  | Description                                          |
| ---------- | ---------------------------------------------------- |
| name       | Name of token pool wallet                            |
| max_tokens | Maximum number of tokens to transfer into token pool |
| interval   | cron expression; how often the pool gets refilled    |

