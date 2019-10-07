# API

## Authentication

```shell
curl \
-H "Authorization: Bearer API_KEY" \
-H "Content-Type: application/json" \
http://<endpoint>:8088
```

Most API calls require the following headers, where `API_KEY` is the key in the db associated with the appropriate game.

To ease the burden of testing, you can set a global `API_KEY` using the `APP_TOKEN_NAME` environment variable.

Throughout this document, these headers will be represented as `$headers`.

Where different headers are required (i.e. when creating a game or when retrieving all registered game data) the differences are documented.

## Async Routes

Every POST/PUT endpoint (with the exception of "create game" and "create forte game") has an async counterpart that returns a <code>202 OK</code> instead of a body. Async routes require an `order_id` (string) parameter in the request body, and their routes are prefixed with `/async`.

### Body Parameters

| Parameter       | Type   | Description                                    |
| --------------- | ------ | ---------------------------------------------- |
| existing params | ...    | ...                                            |
| order_id        | String | Unique user-defined ID to track request status |

Each of these routes is highlighted with a success tag containing the async route, like so:

<aside class="success">
  Async route: <code>/api/v1/async/{route}</code>
</aside>

## Create a Game

```shell
curl -d '
{
    "name": "First Game",
    "email": "dev@forte.io",
    "password": "jibberish",
    "initial_tokens": 1000000,
    "blockchain_url": "http://game_client:8545",
    "initial_slope": 1.0,
    "bonding_curve_shape": "linear"
}' \
  -H "Authorization: Bearer APP_TOKEN_NAME" \
  -H "Content-Type: application/json" \
  -X POST http://localhost:8088/api/v1/game
```

> Response:

```json
{
  "id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
  "name": "fba7ee59-9fa3-40de-bcd7-03d51855e978",
  "api_key": "7044ae8b-82ec-460d-b7ba-708ccd99d029",
  "dev_user_id": "badf8f8e-9690-4806-893c-176dc78470a3",
  "contracts": [
    {
      "id": "0def97ea-002f-4ad3-8745-51662cfd9beb",
      "game_id": "ad0f1e8f-1b59-43c7-aef1-56ccf00d4212",
      "kind": "erc20_tbc",
      "address": "0xd0b1bd8bbca5f2fa5c6c7a3dd6577a1d8f02f5dd",
      "created_by": "3eab5ae3-4f69-47c6-a659-ad7e55750346",
      "created_at": "2019-07-10T19:16:24.953187",
      "updated_by": "3eab5ae3-4f69-47c6-a659-ad7e55750346",
      "updated_at": "2019-07-10T19:16:24.953187"
    },
    {
      "id": "b82c0b66-7f20-4a64-b7c0-e21859c1d669",
      "game_id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
      "kind": "erc20",
      "address": "0x7974514b15b454146a34979b6e68b7e88dd00ed8",
      "created_by": "badf8f8e-9690-4806-893c-176dc78470a3",
      "created_at": "2019-06-14T16:20:26.749144",
      "updated_by": "badf8f8e-9690-4806-893c-176dc78470a3",
      "updated_at": "2019-06-14T16:20:26.749144"
    },
    {
      "id": "d593cd5b-20cf-4c0b-9882-55f537b6bc2d",
      "game_id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
      "kind": "erc721",
      "address": "0x65cfc77ee8adc4a76bfb08324c670d69c03be544",
      "created_by": "badf8f8e-9690-4806-893c-176dc78470a3",
      "created_at": "2019-06-14T16:20:26.768967",
      "updated_by": "badf8f8e-9690-4806-893c-176dc78470a3",
      "updated_at": "2019-06-14T16:20:26.768967"
    },
    {
      "id": "d2991f27-824b-4d67-851e-867e348e5837",
      "game_id": "ad0f1e8f-1b59-43c7-aef1-56ccf00d4212",
      "kind": "erc1155",
      "address": "0x43c3c41a8dbf7e79939a621537139ce4fc15b1c0",
      "created_by": "3eab5ae3-4f69-47c6-a659-ad7e55750346",
      "created_at": "2019-07-10T19:16:24.996795",
      "updated_by": "3eab5ae3-4f69-47c6-a659-ad7e55750346",
      "updated_at": "2019-07-10T19:16:24.996795"
    }
  ]
}
```

> Invalid Parameter Response:

```json
{
  "errors": [
    "Name must be at least 2 characters in length.",
    "Number must be between 0 and 1000."
  ]
}
```

The `/game` endpoint automatically creates a Developer Wallet and mints (creates) a given amount of Game Coins (ERC-20 tokens) for your game to use as the foundation currency. In addition, this endpoint creates the necessary contracts and Wallet services to execute transactions in your game. 

This request will execute the following:

- Generate a game object and store it in the Forte DB
- Register a game with the Wallet API service
- Register a new Developer Wallet and generate a `dev_user_id` used to to transact with and query the Developer Wallet
- Provisions an ERC-20 contract to be used as the game's Game Coin currency
- Provisions an ERC-20 contract with token bonding curve functionality to facilitate Game Coin transactions with the Automated Marketplace
- Provisions an ERC-721 contract to be used for managing Unique Assets
- Provisions an ERC-1155 contract to be used for managing Stackable Assets

**Note: This command uses authorization which differs from the standard used for most other commands. Since no valid api key exists prior to game creation, the value found in the environment variable APP_TOKEN_NAME must be used in the authorization header instead.**


### HTTP Request

`POST http://localhost:8088/api/v1/game`

### Body Parameters

| Parameter           | Type   | Description                     |
| ------------------- | ------ | ------------------------------- |
| name                | String | Name of game                    |
| email               | String | Developer's email address       |
| password            | String | Developer password              |
| initial_tokens      | Number | Number of tokens to start with  |
| blockchain_url      | String | URL of blockchain to connect to |
| bonding_curve_shape | String | The shape of bonding curve      |
| initial_slope       | Number | Initial slope of bonding curve  |

<aside class="notice">The <code>blockchain_url</code> parameter refers to the docker-compose parity instance. Unless you know what you're doing, you should use the default value provided in the example.</aside>
<aside class="notice">The <code>bonding_curve_shape</code> parameter can be either <code>'linear'</code>, which will use the <code>slope</code> parameter to create a linear pricing curve, or <code>'sigmoidal'</code>, which will deploy a pre-defined sigmoidal shaped pricing curve.</aside>

## Get a Game

```shell
curl -H $headers http://localhost:8088/api/v1/game/d03356a2-b262-4359-8b6a-21baf8455eb
```

> Response:

```json
{
  "id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
  "name": "fba7ee59-9fa3-40de-bcd7-03d51855e978",
  "api_key": "7044ae8b-82ec-460d-b7ba-708ccd99d029",
  "dev_user_id": "badf8f8e-9690-4806-893c-176dc78470a3",
  "contracts": [
    {
      "id": "b82c0b66-7f20-4a64-b7c0-e21859c1d669",
      "game_id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
      "kind": "erc20",
      "address": "0x7974514b15b454146a34979b6e68b7e88dd00ed8",
      "created_by": "badf8f8e-9690-4806-893c-176dc78470a3",
      "created_at": "2019-06-14T16:20:26.749144",
      "updated_by": "badf8f8e-9690-4806-893c-176dc78470a3",
      "updated_at": "2019-06-14T16:20:26.749144"
    },
    {
      "id": "d593cd5b-20cf-4c0b-9882-55f537b6bc2d",
      "game_id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
      "kind": "erc721",
      "address": "0x65cfc77ee8adc4a76bfb08324c670d69c03be544",
      "created_by": "badf8f8e-9690-4806-893c-176dc78470a3",
      "created_at": "2019-06-14T16:20:26.768967",
      "updated_by": "badf8f8e-9690-4806-893c-176dc78470a3",
      "updated_at": "2019-06-14T16:20:26.768967"
    }
  ]
}
```

Retrieve details for a specific game by passing the `GAME_ID`.

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>`

### URL Parameters

| Parameter | Description                |
| --------- | -------------------------- |
| GAME_ID   | ID of the game to retrieve |

## Get All Games

```shell
curl -H "Authorization: Bearer APP_TOKEN_NAME" http://localhost:8088/api/v1/game
```

> Response:

```json
[
  {
    "id": "35ccc0fb-fd72-43f7-8e5c-7a7dafffc5ab",
    "name": "newGame",
    "api_key": "f3663cad-1bf2-4c63-93a0-5834b4cb2213",
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
  },
  {
    "id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
    "name": "fba7ee59-9fa3-40de-bcd7-03d51855e978",
    "api_key": "7044ae8b-82ec-460d-b7ba-708ccd99d029",
    "dev_user_id": "badf8f8e-9690-4806-893c-176dc78470a3",
    "contracts": [
      {
        "id": "b82c0b66-7f20-4a64-b7c0-e21859c1d669",
        "game_id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
        "kind": "erc20",
        "address": "0x7974514b15b454146a34979b6e68b7e88dd00ed8",
        "created_by": "badf8f8e-9690-4806-893c-176dc78470a3",
        "created_at": "2019-06-14T16:20:26.749144",
        "updated_by": "badf8f8e-9690-4806-893c-176dc78470a3",
        "updated_at": "2019-06-14T16:20:26.749144"
      },
      {
        "id": "d593cd5b-20cf-4c0b-9882-55f537b6bc2d",
        "game_id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
        "kind": "erc721",
        "address": "0x65cfc77ee8adc4a76bfb08324c670d69c03be544",
        "created_by": "badf8f8e-9690-4806-893c-176dc78470a3",
        "created_at": "2019-06-14T16:20:26.768967",
        "updated_by": "badf8f8e-9690-4806-893c-176dc78470a3",
        "updated_at": "2019-06-14T16:20:26.768967"
      }
    ]
  }
]
```

Retrieve all games created and registered for a given `APP_TOKEN_NAME`.

### HTTP Request

`GET http://localhost:8088/api/v1/game`

**Note: This command uses authorization which differs from the standard used for most other commands. Since this command retrieves information regarding all registered games, rather than for a specific game, the value found in the environment variable `APP_TOKEN_NAME` must be used in the authorization header instead of a specific api key.**


## Create a User (Player)

```shell
curl -d '{"email": "dev@test.io"}' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/user
```

> Response:

```json
{
  "id": "6753d59d-9fa3-44f8-bfda-29d64ae7f152",
  "game_id": "ab5b3d39-5324-402b-8276-9fcacdca6811",
  "email": "fakeemail@fake.com",
  "wallet_address": "0xe0DA6ab11545043eF141Df24A9211b7792356D0c",
  "wallet_user_uid": "f5cca3b4-788e-4fa7-be98-cf1f6ac5972f",
  "created_by": "badf8f8e-9690-4806-893c-176dc78470a3",
  "created_at": "2019-06-14T16:25:34.555304",
  "updated_by": "badf8f8e-9690-4806-893c-176dc78470a3",
  "updated_at": "2019-06-14T16:25:34.555304"
}
```

This endpoint creates a new player for your game and automatically creates a Wallet address to enable the player to transact and hold Game Coins, Stackable and Unique Assets. Creating a player will also generate an `id` used to pass as the player `USER_ID` to execute player actions including, initiating transactions and querying player details. 

The response includes the following: 

- `id` - This is the `USER_ID` for the individual player created. This ID can be used to initiate transactions and query player information. 
- `game_id` - The Game ID associated with the created player. 
- `email` - (**OPTIONAL**) This is an optional field and can be left empty. The email address of the created player.
- `wallet_address` - The blockchain wallet address. Note, the `USER_ID` is used to initiate transactions. 
- `wallet_user_uid` - Created by the Wallet and can be ignored. NOTE: This is not the “id” or `USER_ID` used to query and transact. 
- `created_by` - The Developer Wallet `dev_user_id` that created the User.
- `created_at` - UTC date of when the player was created. 
- `updated_by` - The Developer Wallet `dev_user_id` that last updated the User.
- `updated_at` - UTC date of when the player was last updated. 

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/user`

### URL Parameters

| Parameter | Description                   |
| --------- | ----------------------------- |
| GAME_ID   | ID of game to create user for |

### Body Parameters

| Parameter | Type   | Description                     |
| --------- | ------ | ------------------------------- |
| email     | String | (Optional) User's email address |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/user</code>
</aside>

## Get User Balance

```shell
curl -H $headers \
http://localhost:8088/api/v1/game/{id}/user/{user_id}
```

> Response:

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

Returns the balance for all Game Coin, Stackables and Unique Assets for a given `USER_ID`. This includes the `balance` of Game Coins staked to Stackable and Unique Assets. 

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/user/<USER_ID>`

### URL Parameters

| Parameter | Description                         |
| --------- | ----------------------------------- |
| GAME_ID   | ID of game to get user balance from |
| USER_ID   | ID of user to get balance of        |

## Get all Game Users

```shell
curl -H $headers \
http://localhost:8088/api/v1/game/{id}/user
```

> Response:

```json
{
  "users": [
    {
      "user_id": "b9c78b96-d9da-41fc-8846-e2393c0b7376",
      "email": null,
      "wallet_address": "0x7E0135f53AB0A9D4F614913B0c200a0f97d627B9"
    },
    {
      "user_id": "3d697552-6e6d-41d8-ad5f-13229f3932d8",
      "email": "frank@totallyrealemail.com",
      "wallet_address": "0x76CF00FDe51c8FC44464BbEa220049c67F815FaE"
    }
  ]
}
```

Returns all players for a given game by passing the `GAME_ID`.

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/user`

### URL Parameters

| Parameter | Description                    |
| --------- | ------------------------------ |
| GAME_ID   | ID of game to fetch users from |

## Get User Transactions

```shell
curl -d '{"start_date": "2019-04-24T17:15:36.801682Z", "end_date": "2019-04-28T17:15:36.801682Z", "page": 1, "per_page": 10}' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/user/{user_id}/transactions
```

> Response:

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

Pass a `USER_ID` to query a player’s transaction history by date.

To query Developer Wallet Transactions you can pass the `DEV_USER_ID` generated and provided in the response upon creating a game to this endpoint.

**NOTE: Make sure to enter appropriate UTC-encoded date formats for the transaction range.**

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/user/<USER_ID>/transactions`

### URL Parameters

| Parameter | Description                        |
| --------- | ---------------------------------- |
| GAME_ID   | ID of the game                     |
| USER_ID   | ID of user to get transactions for |

### Body Parameters

| Parameter  | Type   | Description                                               |
| ---------- | ------ | --------------------------------------------------------- |
| start_date | String | UTC-encoded timestamp indicating date/time to begin query |
| end_date   | String | UTC-encoded timestamp indicating date/time to end query   |
| page       | Number | Pagination page, starts on page 1                         |
| per_page   | Number | Number of results to return per page, for pagination      |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/user/{USER_ID}/transactions</code>
</aside>

## Get Developer Wallet Balance

```shell
curl -H $headers \
http://localhost:8088/api/v1/game/{id}/developer/wallet
```

> Response:

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

This endpoint returns the Game Coins, Unique and Stackable Asset balances of the developer Wallet. The Developer Wallets account holds, manages and transfers Game Coins and Items between your developer account (Developer Wallet) and player (end-user) Wallet accounts. When you create a game, the Game Coins created are minted to your Developer Wallet account. 

### HTTP Request

`GET http://0.0.0.0:8088/api/v1/game/<GAME_ID>/developer/wallet`

### URL Parameters

| Parameter | Description                                              |
| --------- | -------------------------------------------------------- |
| GAME_ID   | The ID of the game to retrieve the developer balance of. |

## Create Token Pool

```shell
curl -X POST -d '{
    "name": "My Token Pool",
    "max_tokens": 100,
    "interval": ""
}' -H $headers \
http://localhost:8088/api/v1/game/{id}/developer/wallet/pool
```

> Example Params for a pool without an interval

```json
{
  "name": "My Token Pool",
  "max_tokens": 100,
  "interval": ""
}
```

> Response:

```json
{
  "token_pool_id": "d03356a2-b262-4359-8b6a-21baf8455eb6",
  "name": "My Token Pool",
  "max_tokens": 100,
  "interval": "",
  "tokens": 100
}
```

The Token Pool Wallet is a secondary Wallet owned by the Developer Wallet account, designed to automatically refill and distribute tokens to players. Multiple Token Pool Wallets can be created to enable various game transactions and functions.

Working with a limited amount of Game Coins, the main Token Pool Wallet will not run out. In the `interval` field, a developer may specify a cron expression to set how often a pool will be refilled. Transaction requests will be rejected if the Token Pool Wallet token supply is depleted. The `max_tokens` field sets the maximum number of tokens able to transfer to a Token Pool Wallet. Once this `max_tokens` is reached tokens will no longer automatically transfer to the Token Pool Wallet until this field is below the set max.

For more information about cron expressions check out [cronmaker](http://www.cronmaker.com/). 

<aside class="notice">
    The `interval` field can expect either a valid cron expression as a string or an empty string to indicate that no cron interval will be set.
</aside>

### HTTP Request

`POST http://0.0.0.0:8088/api/v1/game/<GAME_ID>/developer/wallet/pool`

### URL Parameters

| Parameter | Description                                |
| --------- | ------------------------------------------ |
| GAME_ID   | ID of game to create token pool wallet for |

### Body Parameters

| Parameter  | Type   | Description                                          |
| ---------- | ------ | ---------------------------------------------------- |
| name       | String | Name of token pool wallet                            |
| max_tokens | Number | Maximum number of tokens to transfer into token pool |
| interval   | String | cron expression; how often the pool gets refilled.   |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/developer/wallet/pool</code>
</aside>

## Get Token Pool Details

```shell
curl -H $headers \
http://localhost:8088/api/v1/game/{id}/developer/wallet/pool/{token_pool_id}
```

> Response:

```json
{
  "token_pool_id": "d03356a2-b262-4359-8b6a-21baf8455eb6",
  "name": "My Token Pool",
  "max_tokens": 100,
  "interval": "0 0 0 1/1 * ? *",
  "tokens": 100
}
```

Retrieve details of a Token Pool Wallet including the current balance of `tokens` available, the set `max_tokens` and the Token Pool Wallet metadata. 

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/developer/wallet/pool/<TOKEN_POOL_ID>`

### URL Parameters

| Parameter     | Description                                      |
| ------------- | ------------------------------------------------ |
| GAME_ID       | ID of game with token pool wallet                |
| TOKEN_POOL_ID | ID of token wallet to retrieve information about |

## Transfer From Token Pool

```shell
curl -d '{
    "user_id_to": "d03356a2-b262-4359-8b6a-21baf8455eb6",
    "amount": 100
}' \
-H $headers \
http://localhost:8088/api/v1/game/{id}/developer/wallet/pool/{token_pool_id}/transfer
```

> Response:

```json
{
  "token_pool_id": "d03356a2-b262-4359-8b6a-21baf8455eb6",
  "name": "My Token Pool",
  "max_tokens": 100,
  "interval": "0 0 0 1/1 * ? *",
  "tokens": 100
}
```

Transfer Game Coins from a Token Pool Wallet to a User Wallet.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/developer/wallet/pool/<TOKEN_POOL_ID>/transfer`

### URL Parameters

| Parameter     | Description                              |
| ------------- | ---------------------------------------- |
| GAME_ID       | ID of game with token pool wallet        |
| TOKEN_POOL_ID | ID of token pool wallet to transfer from |

### Body Parameters

| Parameter  | Type   | Description                      |
| ---------- | ------ | -------------------------------- |
| user_id_to | String | Recipient user id                |
| amount     | Number | The amount of tokens to transfer |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/developer/wallet/pool/{TOKEN_POOL_ID}/transfer</code>
</aside>

## Refill Token Pool

```shell
curl -H $headers \
http://localhost:8088/api/v1/game/{id}/developer/wallet/pool/{token_pool_id}/refill
```

> Response:

```json
{
  "token_pool_id": "d03356a2-b262-4359-8b6a-21baf8455eb6",
  "name": "My Token Pool",
  "max_tokens": 100,
  "interval": "0 0 0 1/1 * ? *",
  "tokens": 100
}
```

Transfers tokens from the Developer Wallet and refills a Token Pool Wallet to the level set in `max_tokens`. Example: If your Token Pool Wallet currently has 75 Game Coins and your `max_tokens` is set to 100, executing `/refill` will transfer 25 Game Coins from your Developer Wallet to the Token Pool Wallet.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/developer/wallet/pool/<TOKEN_POOL_ID>/refill`

### URL Parameters

| Parameter     | Description                       |
| ------------- | --------------------------------- |
| GAME_ID       | ID of game with token pool wallet |
| TOKEN_POOL_ID | ID of token pool to refill        |

## Delete Token Pool

```shell
curl -H $headers \
-X DELETE http://localhost:8088/api/v1/game/{id}/developer/wallet/pool/{token_pool_id}
```


Executing the DELETE function for a specified `TOKEN_POOL_ID` will delete the Token Pool Wallet and return any remaining Game Coins to the Developer Wallet.


### HTTP Request

`DELETE http://localhost:8088/api/v1/game/<GAME_ID>/developer/wallet/pool/<TOKEN_POOL_ID>`

### URL Parameters

| Parameter     | Description                       |
| ------------- | --------------------------------- |
| GAME_ID       | ID of game with token pool wallet |
| TOKEN_POOL_ID | ID of token wallet to delete      |

## Mint Game Coins

```shell
curl -d '{
    "user_id_to": "d03356a2-b262-4359-8b6a-21baf8455eb6",
    "value": 10
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/token/mint
```

> Response:

```shell
200 OK
```

Mint a deployed Game Coin (ERC-20 Token) and add to the existing supply to be used as the primary currency in your game. 

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/token/mint`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter  | Type   | Description                  |
| ---------- | ------ | ---------------------------- |
| user_id_to | String | ID of user to mint tokens to |
| value      | Number | Number of tokens to mint     |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/token/mint</code>
</aside>

## Transfer Game Coins

```shell
curl -d '{
    "user_id_from": "d03356a2-b262-4359-8b6a-21baf8455eb6",
    "user_id_to": "2bfac8f4-04a0-4fd4-b282-2b11f108889b",
    "amount": 1
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/token/transfer
```

> Response:

```shell
200 OK
```

This endpoint transfers Game Coins (ERC-20 Tokens) from one player to another player by passing the `user_id` in `user_id_from` and `user_id_to` respectively.

Other Transctions include:

[Transfer Game Coins from the Developer Wallet](#transfer-tokens-from-dev-wallet)

[Transfer Unique Assets](#transfer-item)

[Transfer Stackable Assets](#transfer-stackable-item)


### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/token/transfer`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter    | Type   | Description                        |
| ------------ | ------ | ---------------------------------- |
| user_id_from | String | ID of user to transfer tokens from |
| user_id_to   | String | ID of user to transfer tokens to   |
| amount       | Number | Number of tokens to transfer       |

<aside class="notice">
    Both wallets must be registered with the game.
</aside>

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/token/transfer</code>
</aside>

## Transfer Game Coins from Dev Wallet

```shell
curl -d '{
    "user_id_to": "2bfac8f4-04a0-4fd4-b282-2b11f108889b",
    "amount": 1
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/developer/token/transfer
```

> Response:

```shell
200 OK
```

Transfers a deployed Game Coin (ERC20 token) from the Developer Wallet to a player `user_to` Wallet.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/developer/token/transfer`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter  | Type   | Description                     |
| ---------- | ------ | ------------------------------- |
| user_id_to | String | ID of user to transfer token to |
| amount     | Number | Number of tokens to transfer    |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/developer/token/transfer</code>
</aside>


## Mint a Unique Item (Asset)

```shell
curl -d '{
    "user_id_to": "d03356a2-b262-4359-8b6a-21baf8455eb6",
    "creator_stake": 100,
    "metadata": "{fire_power: 1000, max_cooldown: 5}",
    "key": "item_key",
    "pricing_curve_contract_id": "d03356a2-b262-4359-8b6a-21baf8455eb4"
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/item/mint
```

> Response:

```json
{
  "id": "1080760074800369048132675024553509235066712411322"
}
```

This endpoint allows you to create new Unique Assets. These Unique Assets (or Items) are minted as ERC-721 Tokens. You can define `metadata` to determine the Item’s attributes (e.g. fire power, cool down times, etc.). All game assets will require developers to stake Game Coins from the Developer Wallet when the asset is minted.

Developers can choose one of two options when minting assets: 

- `creator_stake` - Developer defined number of Game Coins to be staked with asset upon minting.

- `bonding_curve_shape`- **(OPTIONAL)** This field is optional and can be left empty. Game Coins will be staked to the asset based on a `pricing_curve_contract_id` from a previously [created pricing curve](#create-pricing-curve).

<aside class="notice">
   Only Game Coins allocated via the `pricing_curve_contract_id` will reflect in an asset’s ‘balance’. As such, if a pricing curve is not specified, the `balance` of the Asset may reflect as “0” even if Game Coins were allocated in the `creator_stake`. In this case, the creator stake Game Coins are not held directly in the Asset.
</aside>

<aside class="notice">
   If and when an Asset is burned (destroyed), the current owner of the Asset will receive the Game Coins set by the `pricing_curve_contract_id` back to their Wallet.
</aside>


### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/item/mint`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter                 | Type   | Description                                                    |
| ------------------------- | ------ | -------------------------------------------------------------- |
| user_id_to                | String | ID of user to mint item to                                     |
| metadata                  | String | JSON-encoded metadata                                          |
| creator_stake             | Number | The inital amount of ERC-20s deposited into the item           |
| key                       | String | (Optional) Key that can be used to fetch metadata for the item |
| pricing_curve_contract_id | String | (Optional) ID of the pricing curve                             |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/item/mint</code>
</aside>

## Transfer Unique Item Between Users

```shell
curl -d '{
    "user_id_from": "d03356a2-b262-4359-8b6a-21baf8455eb6",
    "user_id_to": "2bfac8f4-04a0-4fd4-b282-2b11f108889b",
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/item/{item_id}/transfer
```

> Response:

```shell
200 OK
```

Transfer a minted Unique Asset (ERC-721 token) from one player (‘user_id_from’) to another player (‘user_id_to’).

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/item/<ITEM_ID>/transfer`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | GAME_ID     |
| ITEM_ID   | String | ITEM_ID     |

### Body Parameters

| Parameter    | Type   | Description                      |
| ------------ | ------ | -------------------------------- |
| user_id_from | String | ID of user to transfer item from |
| user_id_to   | String | ID of user to transfer item to   |

<aside class="notice">
    Both wallets must be registered with the game.
</aside>

<aside class="success">
  Async route: <code>/api/v1/async/{GAME_ID}/item/{ITEM_ID}/transfer</code>
</aside>

## Transfer Unique Item From Developer to User

```shell
curl -d '{
    "user_id_to": "2bfac8f4-04a0-4fd4-b282-2b11f108889b",
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/developer/item/{item_id}/transfer
```

> Response:

```shell
200 OK
```

Transfer a minted Unique Asset from the Developer Wallet to a player (`user_id_to`) Wallet. You do not need to provide the Developer Wallet, executing this will automatically use the Developer Wallet associated with the given `GAME_ID`.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/developer/item/<ITEM_ID>/transfer`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |
| ITEM_ID   | String | Item ID     |

### Body Parameters

| Parameter  | Type   | Description                      |
| ---------- | ------ | -------------------------------- |
| user_id_to | String | ID of user to transfer tokens to |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/developer/item/{ITEM_ID}/transfer</code>
</aside>

## Burn Unique Item

```shell
curl -H $headers -d '{"user_id_from": "9b10fa3d-cf6c-4a22-9c4a-f0356cc3de29"}' \
  -X POST http://localhost:8088/api/v1/game/{id}/item/{item_id}/burn
```

> Response:

```shell
200 OK
```

Burns (destroys) a specified Unique Asset. When an Item is burned (destroyed), the current owner of the Item will receive the Game Coins set in the `creator_stake` back to their Wallet.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/item/<ITEM_ID>/burn`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |
| ITEM_ID   | String | Item ID     |

### Body Parameters

| Parameter    | Type   | Description                   |
| ------------ | ------ | ----------------------------- |
| user_id_from | String | User ID of current item owner |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/item/{ITEM_ID}/burn</code>
</aside>

## Get Unique Item Details
```shell
curl \
  -H $headers
  http://localhost:8088/api/v1/game/{id}/item/{item_id}
```

> Response:

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

View a Unique Asset’s Game Coin balance, metadata, and owner id.

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/item/<ITEM_ID>`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |
| ITEM_ID   | String | Item ID     |

## Deposit Game Coins from User to Item

```shell
curl -d '{
    user_id_from: "e082e288-3cae-4795-8be0-60b5def154b2",
    token_id: "190177113409627147017492480768293327364543271987",
    amount: 1,
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/token/deposit
```

> Response:

```shell
200 OK
```

Transfer deployed Game Coins (ERC-20 tokens) from a player's Wallet to a Unique or Stackable Asset. This will add to the `balance` for the specified Item.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/token/deposit`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter    | Type   | Description                        |
| ------------ | ------ | ---------------------------------- |
| user_id_from | String | ID of user to transfer tokens from |
| token_id     | String | ID of item to deposit tokens into  |
| amount       | Number | Number of tokens to deposit        |

<aside class="notice">The originating wallet must exist for the game and the item must already be minted.</aside>

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/token/deposit</code>
</aside>

## Deposit Game Coins From Developer to Item

```shell
curl -d '{
    token_id: "190177113409627147017492480768293327364543271987",
    amount: 1,
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/developer/token/deposit
```

> Response:

```shell
200 OK
```

Transfer Game Coins (ERC-20) tokens from the Developer Wallet to a Unique or Stackable Asset. This will add to the `balance` for the specified Item.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/developer/token/deposit`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter | Type   | Description                       |
| --------- | ------ | --------------------------------- |
| token_id  | String | ID of item to deposit tokens into |
| amount    | Number | Number of tokens to deposit       |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/developer/token/deposit</code>
</aside>

## Create Stackable Asset Type

```shell
curl -d '{
    "metadata": {"name": "Strip of leather"},
    "key": "stackable_key"
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/stackable
```

> Response:

```json
{
  "metadata": { "name": "Strip of leather" },
  "id": "2381976568446569244243622252022377480192"
}
```

Stackable Assets are a “asset class” or “type” (based on the ERC-1155 specification) of assets that do not require a record of state (e.g. “evolved” or “level-up”) or transaction history (e.g. A Unique Item once owned by Sir Patrick Stewart).  As an example, Stackable Assets can be used when players collect multiples of crafting materials (e.g. leather scraps, wood, brick) or multiples of the same cards in a deck. Multiple Stackable Asset types can be created.

This endpoint creates a Stackable Asset type and returns an `id` for the Stackable Asset. The `id` can be used to mint new Stackable Assets and transfer to players as needed. 

You can use the `metadata` parameter to assign a unique identifier and metadata such as Name, Description and image (e.g. leather scraps, wood, brick) for each Stackable Asset Type.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/stackable`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter | Type   | Description                                                    |
| --------- | ------ | -------------------------------------------------------------- |
| metadata  | String | JSON metadata about the stackable item                         |
| key       | String | Optional key that can be used to fetch metadata about the item |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/stackable</code>
</aside>

## Mint Stackable Assets

```shell
curl -d '{
    	"user_id_to" : "637b02f2-6a21-40b8-bf95-cf37e7fb66be",
    	"amount" : 100,
    	"stake_value": 5000
  		}' \
  	-X  POST http://localhost:8088/api/v1/game/{id}/stackable/{id}/mint \
  	-H $headers
```

> Response:

```json
{
  "metadata": { "name": "Scraps of leather" },
  "uri": "e082e288-3cae-4795-8be0-60b5def154b2",
  "id": "2381976568446569244243622252022377480192"
}
```

Before minting Stackable Assets, it is required to first create the Stackable Asset Type. You can mint (create) one or many Stackable Assets from a previously created Stackable Asset Type and transfer to a single player.

- `user_id_to` - The `USER_ID` of the player you would like to transfer the Stackable Assets to.
- `amount` The number of Stackable Assets you would like to send to the player.
- `stake_value` - The amount (if different from the pricing curve) to stake to the newly minted assets. 

<aside class="notice">
 If `stake_value` is less than the amount required by the pricing curve for that class of stackable the operation will fail.
 </aside>

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/stackable/<STACKABLE_ID>/mint`

### URL Parameters

| Parameter    | Type   | Description       |
| ------------ | ------ | ----------------- |
| GAME_ID      | String | Game ID           |
| STACKABLE_ID | String | Stackable Item ID |

### Body Parameters

| Parameter   | Type   | Description                               |
| ----------- | ------ | ----------------------------------------- |
| user_id_to  | String | ID of user to mint stackable items to     |
| amount      | Number | Number of stackable items to mint         |
| stake_value | Number | Amount of game coins to stake for minting |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/stackable/{STACKABLE_ID}/mint</code>
</aside>

## Burn Stackable Asset

```shell
curl -d '{
    "amount" : 8,
    "user_id_from": "e082e288-3cae-4795-8be0-60b5def154b2"
  }' \
  -X  POST http://localhost:8088/api/v1/game/{id}/stackable/{id}/burn \
  -H $headers
```

> Response:

```json
{
  "user_id": "e082e288-3cae-4795-8be0-60b5def154b2",
  "amount": 8
}
```

Removes (burns) a number of Stackable Assets from a single user.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/stackable/<STACKABLE_ID>/burn`

### URL Parameters

| Parameter    | Type   | Description       |
| ------------ | ------ | ----------------- |
| GAME_ID      | String | Game ID           |
| STACKABLE_ID | String | Stackable Item ID |

### Body Parameters

| Parameter    | Type   | Description                             |
| ------------ | ------ | --------------------------------------- |
| user_id_from | String | ID of user to burn stackable items from |
| amount       | Number | Number of stackable items to burn       |

### Response Parameters

| Parameter | Type   | Description                             |
| --------- | ------ | --------------------------------------- |
| user_id   | String | ID of user to burn stackable items from |
| amount    | Number | Number of stackable items burned        |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/stackable/{STACKABLE_ID}/burn</code>
</aside>

## Transfer Stackable Asset

```shell
curl -d '{
    "user_id_from": "637b02f2-6a21-40b8-bf95-cf37e7fb66be",
    "user_id_to": "637b02f2-6a21-40b8-bf95-cf37e7fb66be",
    "amount": 1
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/stackable/{stackable_id}/transfer
```

> Response:

```json
{
  "status": "TransactionConfirmed",
  "user_id_from": "9d103588-0630-42c0-97ce-a96c0f70a2ef",
  "user_id_to": "9d103588-0630-42c0-97ce-a96c0f70a2ef",
  "token_id": "340282366920938463463374607431768211456",
  "amount": 1
}
```

Users can transfer Stackable Assets as a peer-to-peer transaction. This endpoint enables the transfer of a single specified Stackable Asset type between two players; `user_id_from` to `user_id_to`.

Use the `amount` parameter to define the number of Stackable Assets you would like to transfer  from `user_id_from` to `user_id_to`. If `user_id_from` does not have enough Stackable Assets available in their Wallet, the transfer will fail.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/stackable/<STACKABLE_ID>/transfer`

### URL Parameters

| Parameter    | Type   | Description       |
| ------------ | ------ | ----------------- |
| GAME_ID      | String | Game ID           |
| STACKABLE_ID | String | Stackable Item ID |

### Body Parameters

| Parameter    | Type   | Description                             |
| ------------ | ------ | --------------------------------------- |
| user_id_from | String | ID of user to send stackable items from |
| user_id_to   | String | ID of user to send stackable items to   |
| amount       | u64    | Amount of stackable items to send       |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/stackable/{STACKABLE_ID}/transfer</code>
</aside>

## Transfer Batch of Stackable Assets

```shell
curl -d '{
    "user_id_from": "637b02f2-6a21-40b8-bf95-cf37e7fb66be",
    "user_id_to": "637b02f2-6a21-40b8-bf95-cf37e7fb66be",
    "token_ids": ["4083388403051261561560495289181218537472", "340282366920938463463374607431768211456"],
    "amounts": [1, 5]
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/stackable/transfer
```

> Response:

```json
{
  "status": "TransactionConfirmed",
  "user_id_from": "9d103588-0630-42c0-97ce-a96c0f70a2ef",
  "user_id_to": "9d103588-0630-42c0-97ce-a96c0f70a2ef",
  "amounts": [1, 5],
  "token_ids": [
    "4083388403051261561560495289181218537472",
    "340282366920938463463374607431768211456"
  ]
}
```

Most games will utilize multiple Stackable Asset Types. You can batch transfer multiple Stackable Asset Types by passing an array of multiple, comma-delimited Stackable Asset `token_ids` and respective `amounts` for each token id. 

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/stackable/transfer`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter    | Type     | Description                             |
| ------------ | -------- | --------------------------------------- |
| user_id_from | String   | User ID to transfer stackable item from |
| user_id_to   | String   | User ID to transfer stackable item to   |
| amounts      | [u64]    | Numbers of stackable items to send      |
| token_ids    | [String] | IDs of stackable item classes to send   |

<aside class="notice">The <code>amounts</code> and <code>token_ids</code> fields must be the same length.</aside>

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/stackable/transfer</code>
</aside>

## Get all Stackable Asset Types for a Game

```shell
curl \
  -H $headers
  http://localhost:8088/api/v1/game/{id}/stackable
```

> Response:

```json
[
  {
    "id": "2381976568446569244243622252022377480192",
    "metadata": {
      "name": "Scraps of Leather"
    }
  },
  {
    "id": "2041694201525630780780247644590609268736",
    "metadata": {
      "name": "Bundles of Wood"
    }
  },
  {
    "id": "1701411834604692317316873037158841057280",
    "metadata": {
      "name": "Rocks"
    }
  }
]
```

You can retrieve a list of all Stackable Asset Types for a given `GAME_ID` including details for each individual Stackable Assets.

The response will return: 

- `id` - The Stackable Asset ID generated when you created the asset

- `metadata` - Developer defined metadata (e.g. asset name, description, images) associated with the Stackable Asset. 

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/stackable`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

## Get metadata for Stackable Asset Type

```shell
curl \
  -H $headers
  http://localhost:8088/api/v1/game/{id}/stackable/{id}
```

> Response:

```json
{
  "id": "2381976568446569244243622252022377480192",
  "metadata": {
    "name": "Scraps of Leather"
  }
}
```

You can retrieve the developer defined metadata for a specific Stackable Asset type by passing the `TOKEN_ID`.

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/stackable/<TOKEN_ID>`

### URL Parameters

| Parameter | Type   | Description       |
| --------- | ------ | ----------------- |
| GAME_ID   | String | Game ID           |
| TOKEN_ID  | String | Stackable Item Id |

## Get Staked Amount for Stackable Asset Class

```shell
curl \
  -H $headers
  http://localhost:8088/api/v1/game/{id}/stackable/{id}/stake
```

> Response:

```json
{
  "liquidity_manager": "0x...",
  "staking_currency": "0x...",
  "balance": 1000
}
```

Retrieves the total amount of Game Coins staked for a given Stackable Asset Type.

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/stackable/<TOKEN_ID>/stake`

### URL Parameters

| Parameter | Type   | Description       |
| --------- | ------ | ----------------- |
| GAME_ID   | String | Game ID           |
| TOKEN_ID  | String | Stackable Item Id |


## Create Pricing Curve

```shell
curl -H $headers \
  -d '{
    "slope": 1.0
  }' \
  -X POST http://localhost:8088/api/v1/game/{id}/pricing-curve
```

> Response:

```json
{
  "pricing_curve_contract_id": "2ac77871-6daf-413b-a556-5068a3d4ebe5"
}
```

Create a standalone pricing curve to be applied when minting Stackable and Unique Assets. Only linear slopes are supported at this time. Slope is expressed as a decimal value that specifies the rate at which price increases relative to supply. Pricing curves can be referenced from other contracts like Stackable and Unique Assets. 

Executing this will return a `pricing_curve_contract_id`. This pricing curve ID can be passed upon minting new assets to stake Game Coins per the defined slope.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/pricing-curve`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

### Body Parameters

| Parameter | Type   | Description                                              |
| --------- | ------ | -------------------------------------------------------- |
| slope     | Number | Linear slope at which price increases relative to supply |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/pricing-curve</code>
</aside>

## Push a Partition onto a Pricing Curve

```shell
curl -d '{
    "start_point": 10,
    "slope": 2.0
  }' \
  -H $headers
  -X POST http://localhost:8088/api/v1/game/{id}/pricing-curve/{pricing_curve_contract_id}/push
```

> Response:

```shell
200 OK
```

Push a new linear partition onto a pricing curve that will start at supply of `start_point`.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/pricing-curve/<PRICING_CURVE_CONTRACT_ID>/push`

### URL Parameters

| Parameter                 | Type   | Description                  |
| ------------------------- | ------ | ---------------------------- |
| GAME_ID                   | String | Game ID                      |
| PRICING_CURVE_CONTRACT_ID | String | Contract ID of Pricing Curve |

### Body Parameters

| Parameter   | Type   | Description                                              |
| ----------- | ------ | -------------------------------------------------------- |
| start_point | Number | Supply at which to start the pricing curve partition     |
| slope       | Number | Linear slope at which price increases relative to supply |

<aside class="success">
  Async route: <code>/api/v1/async/game/{GAME_ID}/pricing-curve/{PRICING_CURVE_CONTRACT_ID}/push</code>
</aside>

## Remove the last Pricing Curve Partition

```shell
curl -H $headers \
  -X POST http://localhost:8088/api/v1/game/{id}/pricing-curve/{pricing_curve_contract_id}/pop
```

> Response:

```shell
200 OK
```

Remove (pop) the last partition off a given pricing curve contract.

### HTTP Request

`POST http://localhost:8088/api/v1/game/<GAME_ID>/pricing-curve/<PRICING_CURVE_CONTRACT_ID>/pop`

### URL Parameters

| Parameter                 | Type   | Description                  |
| ------------------------- | ------ | ---------------------------- |
| GAME_ID                   | String | Game ID                      |
| PRICING_CURVE_CONTRACT_ID | String | Contract ID of Pricing Curve |

## Get Block Number

```shell
curl -H $headers http://localhost:8088/api/v1/game/{id}/chain/block
```

> Response:

```json
{ "id": 42 }
```

This endpoint returns the current number of validated blocks created for your games blockchain instance.

All transactions are written to a “block” on your individual game blockchain. Once all transactions in a block are validated, the block is closed and a new one is created along with a reference to the previous block. You can think of each “block” as a link in a chain.   

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/chain/block`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |

## Get Order Status

```shell
curl -H $headers \
  -X GET http://localhost:8088/api/v1/game/{id}/order/{order_id}
```

### The returned status indictes the state of the request

The returned status indicates the state of the request with respect to the on-chain status. 

- Pending: The transaction has not yet been confirmed on the chain.

- Completed: The transaction has successfully completed and is “recorded” on-chain.

- Error: Some internal error has happened and the transaction data was not recorded on-chain.

A 404 response is returned when the `order_id` is not found in the system for a given game id.

Fetches the order status based on the `order_id` and `game_id`.

> Response:

```json
{
  "status": "Pending",
  "data": {}
}
```

Fetches the order status based on the order id and game id

### HTTP Request

`GET http://localhost:8088/api/v1/game/<GAME_ID>/order/<ORDER_ID>/`

### URL Parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| GAME_ID   | String | Game ID     |
| ORDER_ID  | String | ORDER ID    |




