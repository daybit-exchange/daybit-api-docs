---
title: DAYBIT API Reference
includes:
  - pydaybit
search: true
toc_footers:
  - <a href='https://www.daybit.com'>DAYBIT</a>
  - <a href='/kr'>DAYBIT API Document - Korean</a>
  - <a href='https://www.daybit.com/policy/terms-api'>Terms of Service for DAYBIT API</a>
  - <br/>
  - <a href="https://github.com/daybit-exchange/daybit-api-docs"><img src="https://img.shields.io/badge/Github-daybit--api--docs-181717.svg?logo=github" /></a>
---
# Introduction

Target audience of this document is those who are capable of writing proper program source code. This document contains examples that might put your assets in danger, even losing your assets. You need to fully understand descriptions and functionalities of the source code before run them. Use this API at your own risk and all kinds of outcomes of using this API is your responsibility. Please refer [Terms of Service for DAYBIT API](https://www.daybit.com/policy/terms-api) for details.

You need to generate an API key pair from [DAYBIT](https://www.daybit.com) before using DAYBIT API. An API key pair has multiple types of authorization - receiving [Candle data](https://en.wikipedia.org/wiki/Candlestick_chart) or [Order book](https://en.wikipedia.org/wiki/Order_book_(trading)), checking personal assets, trading personal assets, and withdrawal authorizations. Please refer [Authorization](#authorization) for details of authorization.

There are two types of DAYBIT APIs. First one is an [API Call](#api-calls) - client sends a request and server responds accordingly. Usually this type of APIs is used for asset trading, deposit or withdraw. Second type is a [API Subscription](#subscriptions) which allows you to subscribe to an API and keep getting a notification from DAYBIT API server. There are API subscriptions which includes price change of coins, information of one's wallet, result of one's order and so on.

DAYBIT APIs are implemented based on websocket connection and follow the format defined in [Phoenix framework](https://phoenixframework.org/). DAYBIT also provides [Pydaybit](#pydaybit) written in Python which allows users to easily use DAYBIT API.

If you are not familiar with programming, it would be better to take a look at examples of [Pydaybit](#pydaybit) first.

# Authorization

Before you are using DAYBIT APIs, you first need to generate API key pair with proper authorization. An API key pair has multiple types of authorization - receiving [candle data](https://en.wikipedia.org/wiki/Candlestick_chart) or a [order book](https://en.wikipedia.org/wiki/Order_book_(trading)), checking personal assets, trading personal assets, and withdrawal authorizations.

The usage of APIs is restricted by given right to each API key pair. You would get `unauthenticated` response error_code if you called an API that is not accessible from your API key pair. Please look above for types of API key pair and details of it.

| Type         | Description                                                                                                                                                                                  |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| public_data  | Authorized to access public data (ex, Market Summary, Order Book and so on). To get this authorization level from an API key, please include `Market Inquiry` at the API key pair creation.  |
| private_data | Authorized to access private data (ex, Asset and so on). To get this authorization level from an API key, please include `Private Asset Inquiry` at the API key pair creation.               |
| trade        | Authorized to call trade related APIs (ex, Order, Trade and so on). To get this authorization level from an API key, please include `Trading API` at the API key pair creation.              |
| transaction  | Authorized to call transaction related APIs (ex, Deposit, Withdrawal and so on). To get this authorization level from API key, please include `Withdrawal API` at the API key pair creation. |

You can generate maximum 5 API key pairs per account in [API Keys](https://www.daybit.com/mypage/api-managements) tab in My Page. Each API Key pair can have following authorizations:

* `public_data`
* `public_data`, `private_data`, `trade`
* `public_data`, `private_data`, `transaction`
* `public_data`, `private_data`, `trade`, `transaction`

# Host address

* DAYBIT API endpoint : [wss://api.daybit.com/v1/user_api_socket/websocket/](wss://api.daybit.com/v1/user_api_socket/websocket/)

# APIs

There are two types of DAYBIT's APIs. First one is an [API Call](#api-calls) - client sends a request and server responds accordingly. Usually this type of a call is used for asset trading, deposit or withdraw. Second type is a [Subscription](#subscriptions) which allows you to subscribe to an API and keep getting a notification from the server. Based on type of the notification, it includes price change of coins, information of one's wallet, result of one's order and so on.

## Channels

Channels are based on a simple idea of [Phoenix](https://phoenixframework.org/) that sending and receiving messages. Senders broadcast messages about [topics](#topics). Receivers subscribe to a channel which has a specific [topics](#topics) so that they can get those messages. Senders and receivers can switch roles on the same topic at any time. For details, refer to [Phoenix Documents](https://hexdocs.pm/phoenix/channels.html).

### Topics

Topics are string identifiers of [channels](#channels) that the various layers use in order to make sure messages end up in the right place. DAYBIT APIs are using following types of topics.

* [`/api`](#api-calls)
* [`/subscription:coins`](#coins)
* [`/subscription:market_summaries;<market_summary_intvl>`](#market_summaries)

### Events

Event is a `string` representing specific actions of the channel. `"phx_join"` is for joinning the channel and `"phx_leave"` is for leaving the channel. DAYBIT [API Calls](#api-calls) include types of request in event.

### Messages

> Example of `create_wdrl`

```python
import asyncio
import logging

from pydaybit import Daybit

logger = logging.getLogger('pydaybit')
logger.setLevel(logging.DEBUG)

stream_handler = logging.StreamHandler()
stream_handler.setFormatter(logging.Formatter('%(message)s'))
logger.addHandler(stream_handler)


async def daybit_create_wdrl():
    async with Daybit() as daybit:
        await daybit.create_wdrl(coin='BTC', to_addr='ABTCADDRESS', amount='0.05')


asyncio.get_event_loop().run_until_complete(daybit_create_wdrl())
```

> Output of the example

```console
> {"join_ref": "1", "ref": "1", "topic": "/api", "event": "phx_join", "payload": {"timestamp": 1538739991045, "timeout": 3000}}
< {"topic":"/api","ref":"1","payload":{"status":"ok","response":{}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "2", "topic": "/api", "event": "create_wdrl", "payload": {"to_addr": "ABTCADDRESS", "timestamp": 1538739991059, "coin": "BTC", "amount": "0.05", "timeout": 3000}}
< {"topic":"/api","ref":"2","payload":{"status":"ok","response":{"data":{"wdrl_to_tag":null,"wdrl_to_org":null,"wdrl_to_addr":"ABTCADDRESS","wdrl_status":"queued","type":"wdrl","txid":null,"tx_link_url":"https://live.blockcypher.com/btc/tx/","req_confirm":2,"id":5882,"deposit_status":null,"created_at":1538739991072,"confirm":0,"completed_at":null,"coin":"BTC","amount":"0.050000000000000000"}}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "3", "topic": "/api", "event": "phx_leave", "payload": {"timestamp": 1538739991124, "timeout": 3000}}
< {"topic":"/api","ref":"3","payload":{"status":"ok","response":{}},"event":"phx_reply"}
< {"topic":"/api","ref":"1","payload":{},"event":"phx_close"}
```

Following properties is delivered in the format of [JSON](https://en.wikipedia.org/wiki/JSON) object. The example shows messages on the websocket between a client and DAYBIT API server when the client calls `create_wdrl`.

* `topic` - The string identifier of a channel. for example [`"/api"`](#api-calls) or [`"/subscription:coins"`](#coins)
* `event` - The string event name. for example [`"create_order"`](#create_order), [`"cancel_order"`](#cancel_order), `"phx_join"`, `"phx_leave"`, and so on
* `payload` - The message payload
* `ref` - The unique string ref
* `join_ref` - ref of joinning the channel

## API Calls

You may connect to [DAYBIT API endpoint](#host-address) through a websocket and join `/api` channel. Then you can call a API by sending message with proper `event` among followings.

### Channel of API Calls

All API calls should use `/api` channel.

### API Call List

In `/api` channel, set `event` property with a API name . Available API call list are following.

* [`create_order`](#create_order)
* [`cancel_order`](#cancel_order)
* [`cancel_orders`](#cancel_orders)
* [`cancel_all_my_orders`](#cancel_all_my_orders)
* [`create_wdrl`](#create_wdrl)
* [`get_server_time`](#get_server_time)

<aside class="notice">
It is recommended to retrieve data from <code>notification</code> of <code>subscription:&ltsub_topic&gt</code> channel, not response from <code>/api</code> channel.
</aside>

> Request `create_wdrl`

```console
{"join_ref": "1", "ref": "2", "topic": "/api", "event": "create_wdrl", "payload": {"to_addr": "ABTCADDRESS", "timestamp": 1538739991059, "coin": "BTC", "amount": "0.05", "timeout": 3000}}
```

> Response

```console
{"topic":"/api","ref":"2","payload":{"status":"ok","response":{"data":{"wdrl_to_tag":null,"wdrl_to_org":null,"wdrl_to_addr":"ABTCADDRESS","wdrl_status":"queued","type":"wdrl","txid":null,"tx_link_url":"https://live.blockcypher.com/btc/tx/","req_confirm":2,"id":5882,"deposit_status":null,"created_at":1538739991072,"confirm":0,"completed_at":null,"coin":"BTC","amount":"0.050000000000000000"}}},"event":"phx_reply"}
```

## Subscriptions

> Example of `coins`

```python
import asyncio
import logging

from pydaybit import Daybit

logger = logging.getLogger('pydaybit')
logger.setLevel(logging.DEBUG)

stream_handler = logging.StreamHandler()
stream_handler.setFormatter(logging.Formatter('%(message)s'))
logger.addHandler(stream_handler)


async def daybit_coins():
    async with Daybit() as daybit:
        await daybit.coins()


asyncio.get_event_loop().run_until_complete(daybit_coins())
```

> Output

```console
> {"join_ref": "1", "ref": "1", "topic": "/subscription:coins", "event": "phx_join", "payload": {"timestamp": 1538740890363, "timeout": 3000}}
< {"topic":"/subscription:coins","ref":"1","payload":{"status":"ok","response":{}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "2", "topic": "/subscription:coins", "event": "request", "payload": {"timestamp": 1538740890374, "timeout": 3000}}
< {"topic":"/subscription:coins","ref":"2","payload":{"status":"ok","response":{"data":[{"data":[{"wdrl_fee":"5.00000000","wdrl_enabled":true,"wdrl_confirm":2,"tradable":true,"tick_amount":"0.10000000","sym":"USDT","native_decimal_places":2,"name":"Tether","min_wdrl":"10.00000000","min_deposit":"10.00000000","has_tag":false,"has_org":false,"deposit_enabled":true,"deposit_confirm":2} ... ],"action":"init"}]}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "3", "topic": "/subscription:coins", "event": "phx_leave", "payload": {"timestamp": 1538740890386, "timeout": 3000}}
< {"topic":"/subscription:coins","ref":"3","payload":{"status":"ok","response":{}},"event":"phx_reply"}
< {"topic":"/subscription:coins","ref":"1","payload":{},"event":"phx_close"}
```

For using API Subscriptions, first you need to join `/subscription:<subtopic>` channel and send `request` event. Once you are successfully subscribed by joinning a channel, DAYBIT API server sends `notification` for any kind of updated data.

### API Subscription Channels

Subscription Channels are following. The topic of each channels has `/subscription:<subtopic>` format. For example, In case of the topic of `coins` channel is `/subscription:coins`.

* [`coins`](#coins)
* [`coin_prices`](#coin_prices)
* [`quote_coins`](#quote_coins)
* [`markets`](#markets)
* [`market_summary_intvls`](#market_summary_intvls)
* [`market_summaries`](#market_summaries)
* [`order_books`](#order_books)
* [`price_history_intvls`](#price_history_intvls)
* [`price_histories`](#price_histories)
* [`trades`](#trades), [`my_users`](#my_users)
* [`my_assets`](#my_assets)
* [`my_orders`](#my_orders)
* [`my_trades`](#my_trades)
* [`my_tx_summaries`](#my_tx_summaries)
* [`my_airdrop_histories`](#my_airdrop_histories)
* [`trade_vols`](#trade_vols)
* [`day_avgs`](#day_avgs)
* [`div_plan`](#div_plan)
* [`my_trade_vols`](#my_trade_vols)
* [`my_day_avgs`](#my_day_avgs)
* [`my_divs`](#my_divs)

### Event of API Subscriptions

`request` is a push event and `notification` is a pull event. [Message](https://hexdocs.pm/phoenix/Phoenix.Socket.Message.html) transported from client and server have `request` and `notification` events, respectively. When you subscribe to an API, i.e., joining the channel related to the API and sending a message with `request` event, you will get either `init` or `upsert` action response from the channel. After that, you would get one of `insert`, `update`, `upsert`, or `delete` from the channel with `notification` event. For more information of actions, please look following [Action](#action).

### Rate Limit of API Subscriptions

Limit of messages which have `request` event for every second.

> Example of `coins`

```python
import asyncio
import logging

from pydaybit import Daybit

logger = logging.getLogger('pydaybit')
logger.setLevel(logging.DEBUG)

stream_handler = logging.StreamHandler()
stream_handler.setFormatter(logging.Formatter('%(message)s'))
logger.addHandler(stream_handler)


async def daybit_coins():
    async with Daybit() as daybit:
        await daybit.coins()


asyncio.get_event_loop().run_until_complete(daybit_coins())
```

> Output

```console
> {"join_ref": "1", "ref": "1", "topic": "/subscription:coins", "event": "phx_join", "payload": {"timestamp": 1538740890363, "timeout": 3000}}
< {"topic":"/subscription:coins","ref":"1","payload":{"status":"ok","response":{}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "2", "topic": "/subscription:coins", "event": "request", "payload": {"timestamp": 1538740890374, "timeout": 3000}}
< {"topic":"/subscription:coins","ref":"2","payload":{"status":"ok","response":{"data":[{"data":[{"wdrl_fee":"5.00000000","wdrl_enabled":true,"wdrl_confirm":2,"tradable":true,"tick_amount":"0.10000000","sym":"USDT","native_decimal_places":2,"name":"Tether","min_wdrl":"10.00000000","min_deposit":"10.00000000","has_tag":false,"has_org":false,"deposit_enabled":true,"deposit_confirm":2} ... ],"action":"init"}]}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "3", "topic": "/subscription:coins", "event": "phx_leave", "payload": {"timestamp": 1538740890386, "timeout": 3000}}
< {"topic":"/subscription:coins","ref":"3","payload":{"status":"ok","response":{}},"event":"phx_reply"}
< {"topic":"/subscription:coins","ref":"1","payload":{},"event":"phx_close"}
```

## Rate limit

Each API has limit of calls for every second. You will get `api_exceeded_rate_limit` error_code in response if you exceeded the limit.

## Timestamp

All request takes `timestamp` and `timeout` to prevent unexpected calls because of network delay and so on.

* `timestamp`: `unix_timestamp`
* `timeout`(**optional**): `ms` unit time in `integer`. Default value is `3000` which means 3 seconds.

A request will be rejected in next condition: `server time` - `timestamp` > `timeout`.

If there was a problem, below error_code will be returned in response.

* `api_invalid_timestamp_or_timeout`: `timestamp` and/or `timeout` are not existed or they are not `integer` (unix timestamp in millisecond).
* `api_timeout`: Rejected because of request time out.

## Response format

There are two types of response formats. Based on the result of both API calls and subscriptions, you would get one of success or fail models. Please see success and fail response examples on right column. You can also find [Error List](#error-list) in below.

> Success

```python
{
  # plain json object with proper data
}
```

> Fail

```python
{
  "error_code": 'string',
  "message": 'string'
}
```

# Libraries

DAYBIT is using [Elixir](https://elixir-lang.org/) language and built on [Phoenix framework](https://phoenixframework.org/). DAYBIT API can be accessed from Phoenix client, which makes it easy to connect to Phoenix sockets. For now, DAYBIT officially provides a wrapper for Python language.

* PyDaybit(Python): [https://github.com/daybit-exchange/pydaybit](https://github.com/daybit-exchange/pydaybit/)

For other languages, please refer below libraries to implement the features of DAYBIT API.

* Phoenix.js(Javascript): [https://github.com/phoenixframework/phoenix](https://github.com/phoenixframework/phoenix/)
* SwiftPhoenixClient(Swift): [https://github.com/davidstump/SwiftPhoenixClient](https://github.com/davidstump/SwiftPhoenixClient/)
* PhoenixSharp(C#): [https://github.com/Mazyod/PhoenixSharp](https://github.com/Mazyod/PhoenixSharp/)

# Error list

### General

* `unauthenticated`: Unauthenticated user or action
* `invalid_arguments`: Invalid arguments in request
* `resource_not_found`: Resource not found

### Api

* `api_invalid_timestamp_or_timeout`: `timestamp` and/or `timeout` of request is not valid
* `api_timeout`: Timeout happens by requested `timestamp` and/or `timeout`
* `api_exceeded_rate_limit`: Rate limit exceeded
* `api_invalid_param_types`: Invalid request parameter type
* `api_required_params_not_provided`: Missing required parameter
* `api_invalid_event`: Invalid request event

### Order

* `order_invalid_market`: Invalid market(`quote`, `base`)
* `order_not_tradable_coin`: Coin trade suspended
* `order_not_sellable_market`: Selling is suspended in the market
* `order_not_buyable_market`: Buying is suspended in the market
* `order_invalid_price`: Invalid price
* `order_invalid_amount`: Invalid amount
* `order_only_both_role_can_be_cond`: Conditional order is available only when `role` is `both`
* `order_out_of_price_range`: Out of price range (Selling: 20% ~ 200%, Buying: 50% ~ 500%)
* `order_exceeded_max_tstops`: Exceeded maximum Trailing Stop order count (currently 2 per market)
* `order_suspended_due_to_frequent_canceling`: Order suspended due to frequent canceling
* `order_exceeded_asset_values`: Order exceeded my asset values
* `order_already_closed`: Order already closed
* `order_exceeded_void_rate`: Order suspended for 10 minutes because invalid order rate exceeded specific value (currently 80%)
* `order_exceeded_max_orders`: Order exceeded maximum number of outstanding orders (currently 100)
* `order_violates_min_usd`: Order violates minimum of total USD exchanged amount (currently $10)
* `order_unplaceable_maker_only`: Order didn't meet maker only conditions
* `order_unplaceable_taker_only`: Order didn't meet taker only conditions

### Wdrl

* `wdrl_suspended_coin`: Coin was Suspended to withdraw
* `wdrl_precision_error`: Decimal place accuracy error
* `wdrl_under_min_amount`: One time withdrawal amount is less than minimum
* `wdrl_over_daily_wdrl_limit`: Withdrawal amount exceeded daily limit
* `wdrl_exceeds_my_asset_values`: Withdrawal amount exceeded my asset
* `wdrl_needs_to_tag`: Missing `to_tag` parameter
* `wdrl_invalid_addr`: Invalid address

# Action

When you subscribe an API, a message from the server holds `action` which helps you to understand how to handle the data.

* `init` : Dump all previous local data and initialize everything with data reponsed
* `insert` : Add responsed data to local data set
* `update` : Search in local data set and replace if it was found.
* `upsert` : Search in data set and replace if it was found, or insert if there's no matching data.
* `delete` : Search in data set and remove if it was found.

# Types

### Integer

`integer` type.

* `123`
* `20181010`

### Decimal

Decimal number. This is `string` data type to precisely express the exact amount of number which can't be expressed in ordinary decimal number types.

* `"880.524"`
* `"59.55000000000000000"`

### String

`string`.

* `"string"`
* `"bitcoin"`
* `"DAYBIT"`

### Boolean

`boolean`.

* `true`
* `false`

### Unix Timestamp

`millisecond` unit Unix timestamp.

* `1528269989516`

### CSV

comma separated values expressed in `string`.

* `"1, 2, 3"`

# Models

Below are generic data models.

### Coin

identifier: `sym`

| Name                   | Type    | Description                                           |
| ---------------------- | ------- | ----------------------------------------------------- |
| sym                    | string  | Coin symbol                                           |
| native_decimal_point | integer | Withdrawal amount decimal restriction                 |
| amount_decimal_point | integer | Order amount decimal point restriction                |
| tick_amount            | decimal | Order amount unit                                     |
| deposit_confirm        | integer | Number of confirms required for deposit completion    |
| wdrl_confirm           | integer | Number of confirms required for withdrawal completion |
| public                 | boolean | Listed on the exchange (`true`) or not (`false`)      |
| name                   | string  | Name of coin (locale applied)                         |
| tradable               | boolean | Tradable or not                                       |
| deposit_enabled        | boolean | Deposit enabled or not                                |
| wdrl_enabled           | boolean | Withdrawal enabled or not                             |
| wdrl_fee               | decimal | Withdrawal fee                                        |
| min_deposit            | decimal | Minimum deposit amount                                |
| min_wdrl               | decimal | Minimum withdrawal amount                             |
| has_tag                | boolean | to_tag required or not for deposit/withdrawal         |
| has_org                | boolean | to_org required or not for deposit/withdrawal         |

### Coin price

identifier: `sym`

| Name      | Type    | Description         |
| --------- | ------- | ------------------- |
| sym       | string  | Coin symbol         |
| usd_price | decimal | USD exchanged price |

### Quote coin

identifier: `sym`

| Name | Type   | Description |
| ---- | ------ | ----------- |
| sym  | string | Coin symbol |

### Market

identifier: `quote`, `base`

| Name        | Type    | Description                                                                                                                                                                   |
| ----------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| quote       | string  | Quote coin                                                                                                                                                                    |
| base        | string  | Base coin                                                                                                                                                                     |
| tick_price  | decimal | Order price unit                                                                                                                                                              |
| sellable    | boolean | Sellable or not                                                                                                                                                               |
| buyable     | boolean | Buyable or not                                                                                                                                                                |
| tick_levels | integer | Number of levels for order book existence per `tick_price`. Order book is increased ten times for each level. ex) tick_price = 0.01, order book intvl = 0.01, 0.1, 1, 10, 100 |

### Market summary interval

identifier: `seconds`

| Name    | Type    | Description                                        |
| ------- | ------- | -------------------------------------------------- |
| seconds | integer | Interval unit of [Market Summary](#market-summary) |

### Market summary

identifier: `quote`, `base`, `seconds`

| Name       | Type            | Description                   |
| ---------- | --------------- | ----------------------------- |
| quote      | string          | Quote coin                    |
| base       | string          | Base coin                     |
| seconds    | integer         | Time unit                     |
| open       | decimal         | Opening price                 |
| close      | decimal         | Closing price                 |
| high       | decimal or null | Highest price                 |
| low        | decimal or null | Lowest price                  |
| volatility | decimal         | Price volatility              |
| quote_vol  | decimal or null | Trading volume per quote coin |
| base_vol   | decimal or null | Trading volume per base coin  |

### Order book

identifier: `quote`, `base`, `intvl`, `min_price`

| Name      | Type    | Description         |
| --------- | ------- | ------------------- |
| quote     | string  | Quote coin          |
| base      | string  | Base coin           |
| intvl     | decimal | Price interval unit |
| min_price | decimal | Minimum price       |
| max_price | decimal | Maximum price       |
| sell_vol  | decimal | Selling volume      |
| buy_vol   | decimal | Buying volume       |

### Price history interval

identifier: `seconds`

| Name    | Type    | Description                                      |
| ------- | ------- | ------------------------------------------------ |
| seconds | integer | Interval unit of [Price history](#price-history) |

### Price history

identifier: `quote`, `base`, `intvl`, `start_time`

| Name       | Type           | Description                   |
| ---------- | -------------- | ----------------------------- |
| quote      | string         | Quote coin                    |
| base       | string         | Base coin                     |
| intvl      | integer        | Time interval                 |
| start_time | unix_timestamp | Start time of price history   |
| end_time   | unix_timestamp | End time of price history     |
| open       | decimal        | Opening price                 |
| close      | decimal        | Closing price                 |
| high       | decimal        | Highest price                 |
| low        | decimal        | Lowest price                  |
| base_vol   | decimal        | Trading volume per base coin  |
| quote_vol  | decimal        | Trading volume per quote coin |

### User

identifier: -

| Name                       | Type    | Description                          |
| -------------------------- | ------- | ------------------------------------ |
| maker_fee_rate           | decimal | Maker fee rate                       |
| taker_fee_rate           | decimal | Taker fee rate                       |
| one_day_wdrl_usd_limit | decimal | Withdrawal limit for 24 hours in USD |

### Asset

identifier: `coin`

| Name           | Type    | Description                         |
| -------------- | ------- | ----------------------------------- |
| coin           | string  | Coin symbol                         |
| total          | decimal | Total amount                        |
| reserved       | decimal | Reserved amount for order and so on |
| available      | decimal | Availalbe amount for order          |
| investment_usd | decimal | Total investment in USD             |

### Order

identifier: `id`

| Name          | Type                   | Description                                                                                                                                           |
| ------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| id            | integer                | id                                                                                                                                                    |
| sell          | boolean                | Selling (`true`) or buying (`false`)                                                                                                                  |
| quote         | string                 | Quote coin                                                                                                                                            |
| base          | string                 | Base coin                                                                                                                                             |
| price         | decimal                | Price                                                                                                                                                 |
| role          | string                 | Order role. `"both"`, `"maker_only"`, `"taker_only"`                                                                                                  |
| cond_type     | string                 | Conditional order type. `"none"`, `"le"`, `"ge"`, `"fall_from_top"`, `"rise_from_bottom"`                                                             |
| cond_arg1     | decimal or null        | First conditional order value                                                                                                                         |
| cond_arg2     | decimal or null        | Second conditional order value                                                                                                                        |
| coin_fee      | decimal                | Fee                                                                                                                                                   |
| amount        | decimal                | Order amount                                                                                                                                          |
| filled        | decimal                | Order filled (per base at selling)                                                                                                                    |
| filled_quote  | decimal                | Order filled (per quote at buying)                                                                                                                    |
| unfilled      | decimal                | Order unfilled                                                                                                                                        |
| received_at   | unix_timestamp         | Order received time                                                                                                                                   |
| placed_at     | unix_timestamp or null | Order placed time (to order book)                                                                                                                     |
| closed_at     | unix_timestamp or null | Order closed time                                                                                                                                     |
| status        | string                 | Order status (`"received"`, `"placed"`, `"closed"`)                                                                                                   |
| close_type    | string                 | Closing type (`"rejected"`, `"filled"`, `"canceled"`)                                                                                                 |
| cancel_reason | string                 | Cancel reason (`"user"`, `"conflicting_self_order"`, `"expired"`, `"no_placed_amount"`, `"partially_filled_taker_only"`, `"maintenance"`, `"admin"`,) |

### Trade

identifier: `id`

| Name                                                  | Type           | Description                                                                                                                                                       |
| ----------------------------------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                                                    | integer        | id                                                                                                                                                                |
| quote                                                 | string         | Quote coin                                                                                                                                                        |
| base                                                  | string         | Base coin                                                                                                                                                         |
| quote_amount                                          | decimal        | Quote coin trade amount                                                                                                                                           |
| base_amount                                           | decimal        | Base coin trade amount                                                                                                                                            |
| price                                                 | decimal        | Trade closing price                                                                                                                                               |
| taker_sold                                            | boolean        | Taker is seller (`true`) or buyer (`false`)                                                                                                                       |
| exec_at                                               | unix_timestamp | Execution time                                                                                                                                                    |
| Below are only available from [my_trades](#my_trades) |                |                                                                                                                                                                   |
| order_id                                              | integer        | Trade order id                                                                                                                                                    |
| fee                                                   | decimal        | Fee                                                                                                                                                               |
| sell                                                  | boolean        | Selling (`true`) or buying (`false`)                                                                                                                              |
| counterpart                                           | string         | (`"user"`, `"daybit"`, `"project"`), The counterparty of the trade. `"daybit"` and `"project"` means DAYBIT exchange and another coin project team, respectively. |

### Transaction Summary

identifier: `id`

| Name           | Type                   | Description                                     |
| -------------- | ---------------------- | ----------------------------------------------- |
| id             | integer                | id                                              |
| coin           | string                 | Transaction coin                                |
| type           | string                 | Type of the transaction (`"wdrl"`, `"deposit"`) |
| amount         | decimal                | Transaction amount                              |
| txid           | string or null         | Withdrawal transaction id                       |
| confirm        | integer                | Confirm count                                   |
| req_confirm    | integer                |                                                 |
| deposit_status | string                 | Deposit status                                  |
| wdrl_status    | string or null         | Withdrawal status                               |
| wdrl_to_addr | string or null         | Withdrawal receiving address                    |
| wdrl_to_tag  | string or null         | Withdrawal tag                                  |
| wdrl_to_org  | string or null         | Withdrawal organization                         |
| created_at     | unix_timestamp         | Transaction created time                        |
| completed_at   | unix_timestamp or null | Transaction completed time                      |
| tx_link_url  | string                 |                                                 |

### Airdrop

identifier: `id`

| Name          | Type           | Description         |
| ------------- | -------------- | ------------------- |
| id            | integer        | id                  |
| coin          | string         | coin symbol         |
| amount        | decimal        | Airdrop amount      |
| description   | string         | Airdrop description |
| airdropped_at | unix_timestamp | Airdrop time        |

### Trade Volume

identifier: `start_time`

| Name       | Type           | Description                                           |
| ---------- | -------------- | ----------------------------------------------------- |
| start_time | unix_timestamp | Start Time                                            |
| end_time   | unix_timestamp | End Time                                              |
| usd_amount | decimal        | Aggregated volume between `start_time` and `end_time` |

### Day Average

identifier: `start_time`

| Name       | Type           | Description                                                                               |
| ---------- | -------------- | ----------------------------------------------------------------------------------------- |
| start_time | unix_timestamp | Start Time                                                                                |
| end_time   | unix_timestamp | End Time                                                                                  |
| avg        | decimal        | Average of total volume of distributed DAY as rewards between `start_time` and `end_time` |

### Div Plan

identifier: `end_time`

| Name       | Type           | Description                        |
| ---------- | -------------- | ---------------------------------- |
| start_time | unix_timestamp | Start Time                         |
| end_time   | unix_timestamp | End Time                           |
| div_count  | integer        | The number of BTC reward receivers |
| div_btc    | decimal        | Total BTC rewards                  |