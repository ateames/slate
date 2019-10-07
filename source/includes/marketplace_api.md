# Marketplace API

The Forte Automated Marketplace (AMM) is a dedicated API service for executing buys, sells and trades between players and the Automated Marketplace. The Automated Marketplace API is configured to run on a different port than the Platform API. If you get a 404 response, double check the hostname and port.

The default URL for interacting with the Marketplace API is `http://0.0.0.0:8090/`

Executing the Trade Option - If the player agrees to the Asset Price and has the required Game Coin balance, they can execute the Trade Option before it expires. This will record the transaction on-chain and generate a transaction hash. 
Transaction Success - Once successfully executed, the AMM service will record the transaction on the blockchain and return the transaction_hash for future reference. 

### Trade Options
Executing a request to sell to the AMM will not exchange immediately, but instead will generate a Trade Option. A Trade Option is an agreement between the Automated Marketplace and the Player that gives the player an option to sell (or buy) an asset for a predetermined price, defined by the Automated Marketplace. This allows the player to preview and accept or deny the AMM price before agreeing to the exchange.


## Deposit Game Coins to the AMM

Before using the AMM, you will first need to transfer Game Coins from the Developer Wallet to be sure you have enough Game Coins (liquidity) available to initiate and execute transactions. This allows players to sell assets directly to the AMM, providing maximum tradeability (liquidity) for player owned assets.

```shell
curl -X POST \
  "http://localhost:8090/api/v1/game/$GAME_ID/deposit" \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer $API_KEY' \
     -d $'{ "amount": 1000 }'
```

> The above command returns the current balance for the AMM:

```json
{
  "amount": 1000
}
```

### HTTP Request

`POST http://0.0.0.0:8090/api/v1/game/<GAME_ID>/deposit`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |

### Body Parameters

| Parameter    | Description                     |
| ------------ | ------------------------------- |
| amount       | Amount of tokens to deposit     |


## Getting the current balance for the AMM

Get the current available Game Coin balance in the AMM.

```shell
curl -X GET \
  "http://localhost:8090/api/v1/game/$GAME_ID/balance" \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer $API_KEY' \
```

> The above command returns the current balance for the AMM:

```json
{
  "amount": 1000
}
```

### HTTP Request

`GET http://0.0.0.0:8090/api/v1/game/<GAME_ID>/balance`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |


## Selling a Unique Item to the AMM

You can request to sell a\ Unique Item to the AMM. Executing a sell will generate a Sell Trade Option and allow the player to execute and complete the transaction. 

Trade Option Attributes: 

- `Trade_type` - Determined by executing a buy, sell or trade
- `user_wallet_address` - The selling or buying player Wallet address for a specific item 
- `item_id` - The Unique Item ID
- `price` - the Game Coin value determined by the AMM’s price curve used to display a guaranteed price the AMM is willing to pay the player for the Item. This allows the player to know the sell or buy price for individual items in the respective Game Coins before agreeing to the exchange.
- `valid_until` - An expiration date and time that determines how long the player has to execute the buy or sell before the “offer” expires and the transaction is terminated. 
- `tag` - Tags can be ignored as they will likely be removed in a future update.


```shell
curl -X POST \
  "http://localhost:8090/api/v1/game/$GAME_ID/item/$ITEM_ID/sell" \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer $API_KEY' \
     -d $'{ "user_id": "$USER_ID" }'
```

> The above command returns the Trade Option:

```json
{
  "id": "8db0177f-bb92-4f39-91b4-a8b52481733b",
  "trade_type": "Sell",
  "user_wallet_address": "0x881a618ed68546006878cee835cf0e9d5e25ff7e",
  "item_id": "1182899485466738358818149947060468042005224297909",
  "price": 10,
  "valid_until": "2019-09-14T23:24:16.920304Z",
  "tag": ""
}
```

### HTTP Request

`POST http://0.0.0.0:8090/api/v1/game/<GAME_ID>/item/<ITEM_ID>/sell`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |
| ITEM_ID   | ID of item to sell |

### Body Parameters

| Parameter    | Description                     |
| ------------ | ------------------------------- |
| user_id      | ID of the user that is selling  |

## Executing the Trade Option for Unique Item

On the returned Trade Option, we can see the `price` the AMM is willing to pay for the Item and `valid_until` date and time. If the player agrees within the expiration date, we need to execute and complete the transaction. Once successfully executed, the AMM service will record the transaction on the blockchain and return the `transaction_hash` for future reference. 

The trade will fail if the AMM or the player does not have the required Item or Game Coins available. 

If the `valid_until` date and time expires prior to executing the Trade Option, the transaction will be terminated and all Game Coins and Item will be returned to the user and AMM (developer) respectively. 

```shell
curl -X POST \
  "http://localhost:8090/api/v1/game/$GAME_ID/item/$ITEM_ID/sell" \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer $API_KEY' \
     -d $'{
  "id": "8db0177f-bb92-4f39-91b4-a8b52481733b",
  "price": 10,
  "user_wallet_address": "0x881a618ed68546006878cee835cf0e9d5e25ff7e",
  "tag": "",
  "valid_until": "2019-09-14T23:24:16.920304Z",
  "item_id": "1182899485466738358818149947060468042005224297909",
  "trade_type": "Sell"
}'
```

> The above command returns whether the trade was successful,
the transaction hash for it in the chain and the trade option.

```json
{
  "success": true,
  "transaction_hash": "0xde333960224622f99d660a58350621dda30d0f36368c59b691184a1834fb5269",
  "trade_option": {
    "id": "8db0177f-bb92-4f39-91b4-a8b52481733b",
    "trade_type": "Sell",
    "user_wallet_address": "0x881a618ed68546006878cee835cf0e9d5e25ff7e",
    "item_id": "1182899485466738358818149947060468042005224297909",
    "price": 10,
    "valid_until": "2019-09-14T23:24:16.920304Z",
    "tag": ""
  }
}
```

### HTTP Request

`POST http://0.0.0.0:8090/api/v1/game/<GAME_ID>/item/execute`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |

### Body Parameters

The body should be exactly what was received from [Selling an item to the AMM](#selling-an-item-to-the-amm)

## Buying Unique Item from the AMM

Buying Unique Items from the AMM is a similar process as selling. By executing the Marketplace /buy service, a Trade Option is generated and the user can choose to execute and complete the transaction using [same endpoint to execute it](#executing-the-trade-option). Once approved by the user, the AMM service will record the transaction on the blockchain and return the `transaction_hash`.


```shell
curl -X POST \
  "http://localhost:8090/api/v1/game/$GAME_ID/item/$ITEM_ID/buy" \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer $API_KEY' \
     -d $'{ "user_id": "$USER_ID" }'
```

> The above command returns the Trade Option:

```json
{
  "id": "c3da9fc0-3961-49c4-86b6-f91b6024992b",
  "trade_type": "Buy",
  "user_wallet_address": "0x881a618ed68546006878cee835cf0e9d5e25ff7e",
  "item_id": "1182899485466738358818149947060468042005224297909",
  "price": 11,
  "valid_until": "2019-09-14T23:27:54.267816Z",
  "tag": ""
}
```

### HTTP Request

`POST http://0.0.0.0:8090/api/v1/game/<GAME_ID>/item/<ITEM_ID>/buy`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |
| ITEM_ID   | ID of item to buy |

### Body Parameters

| Parameter    | Description                     |
| ------------ | ------------------------------- |
| user_id      | ID of the user that is buying   |


## Get Unique Item Listing

Returns a listing of all current Unique Items being sold by the AMM. 

```shell
curl -X GET \
  "http://localhost:8090/api/v1/game/$GAME_ID/item" \
     -H 'Authorization: Bearer $API_KEY'
```

> The above return a list of item IDs together with its estimated price

```json
[
  {
    "item": "1039306322283532498113112889386888021881223103599",
    "price": "10"
  }
]
```

### HTTP Request

`GET http://0.0.0.0:8090/api/v1/game/<GAME_ID>/item`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |

### HTTP Request

`GET http://0.0.0.0:8090/api/v1/game/<GAME_ID>/item`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |

## Sell Stackable Assets to the AMM

Creates a Sell Trade Option for Stackable Assets. Once the Trade Option has been created, you can execute the [Trade Order for Stackables](#execute-trade-order-for-stackables)

```shell
curl -X POST \
  "http://localhost:8090/api/v1/game/{game_id}/stackables/{stackable_id}/sell" \
     -H $headers \
     -d $'{ "user_id": "{user_id}", "amount": 10 }'
```

> The above command returns the Trade Option:

```json
{
  "id": "7216d83b-6139-4e19-a47a-b1136d329f15",
  "trade_type": "Sell",
  "user_wallet_address": "0x881a618ed68546006878cee835cf0e9d5e25ff7e",
  "token_id": "0x700000000000000000000000000000000",
  "amount": 10,
  "price": "1",
  "valid_until": "2019-09-15T20:37:37.729311Z",
  "tag": ""
}
```

### HTTP Request

`POST http://0.0.0.0:8090/api/v1/game/<GAME_ID>/stackables/<STACKABLE_ID>/sell`

### URL Parameters

| Parameter    | Description     |
| ------------ | --------------- |
| GAME_ID      | ID of the game  |
| STACKABLE_ID | ID of stackable |

### Body Parameters

| Parameter | Description                    |
| --------- | ------------------------------ |
| user_id   | ID of the user that is selling |
| amount    | how many items to sell         |

## Buy Stackable Assets from the AMM

Creates a Buy Trade Option for Stackable Assets. 

```shell
curl -X POST \
  "http://localhost:8090/api/v1/game/{game_id}/stackables/{stackable_id}/buy" \
     -H $headers \
     -d $'{ "user_id": "{user_id}", "amount": 10 }'
```

> The above command returns the Trade Option:

```json
{
  "id": "6ec60a81-86c0-4158-a7e0-8d5ebe698817",
  "trade_type": "Buy",
  "user_wallet_address": "0x881a618ed68546006878cee835cf0e9d5e25ff7e",
  "token_id": "0x700000000000000000000000000000000",
  "amount": 10,
  "price": "1",
  "valid_until": "2019-09-15T20:41:00.945611Z",
  "tag": ""
}
```

### HTTP Request

`POST http://0.0.0.0:8090/api/v1/game/<GAME_ID>/stackables/<STACKABLE_ID>/buy`

### URL Parameters

| Parameter    | Description     |
| ------------ | --------------- |
| GAME_ID      | ID of the game  |
| STACKABLE_ID | ID of stackable |

### Body Parameters

| Parameter | Description                    |
| --------- | ------------------------------ |
| user_id   | ID of the user that is selling |
| amount    | how many items to sell         |

## Execute Trade Order for Stackable Assets

**Trading Stackable Assets are still in development and executing an order will not work for now.**

```shell
curl -X POST \
  "http://localhost:8090/api/v1/game/{game_id}/stackables/execute" \
     -H $headers \
     -d $'{
  "amount": 10,
  "id": "6ec60a81-86c0-4158-a7e0-8d5ebe698817",
  "price": "1",
  "user_wallet_address": "0x881a618ed68546006878cee835cf0e9d5e25ff7e",
  "tag": "",
  "valid_until": "2019-09-15T20:41:00.945611Z",
  "token_id": "0x700000000000000000000000000000000",
  "trade_type": "Buy"
}'
```

> The above command returns the Trade Option:

```json
{
  "success": true,
  "transaction_hash": "",
  "trade_option": {
    "id": "6ec60a81-86c0-4158-a7e0-8d5ebe698817",
    "trade_type": "Buy",
    "user_wallet_address": "0x881a618ed68546006878cee835cf0e9d5e25ff7e",
    "token_id": "0x700000000000000000000000000000000",
    "amount": 10,
    "price": "1",
    "valid_until": "2019-09-15T20:41:00.945611Z",
    "tag": ""
  }
}
```

## Get Stackable Listing

Returns a listing of all current Stackable Assets being sold by the AMM. 

```shell
curl -X GET \
  "http://localhost:8090/api/v1/game/$GAME_ID/stackables" \
     -H 'Authorization: Bearer $API_KEY'
```

> The above return a list of stackable IDs together with the estimated price
for one item in the stack

```json
[
  {
    "token_id": "2381976568446569244243622252022377480192",
    "price": "10"
    "amount": "200"
  }
]
```

### HTTP Request

`GET http://0.0.0.0:8090/api/v1/game/<GAME_ID>/stackables`

### URL Parameters

| Parameter | Description            |
| --------- | ---------------------- |
| GAME_ID   | ID of the game         |
