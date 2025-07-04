---
title: BitDa Spot API Documentation
---

# Introduction

## API Introduction

Welcome to use BitDa API！ You can use this API get market information, trading and managing your account.

The code will display on the right side of this document. Currently we only provide `shell` code examples.

It can be accessed using the following domain names: api.bitda.com

Welcome all professional maker strategies and organizations for long term market making program.

## Public API
Verification free for below portals:

| API                                          | Introduction            | Trading area |
| -------------------------------------------- | ----------------------- | ------------ |
| [GET /open/v1/tickers/market](#495cebdeec-2) | Get Market List         | Spot         |
| [GET /open/v1/depth/market](#a1128a972d-2)   | Get Market Depth        | Spot         |
| [GET /open/v1/trade/market](#775841b581-2)   | Get Market Transactions | Spot         |
| [GET /open/v1/kline/market](#k-3)            | Get Market K-line       | Spot         |

## Private API
Eligible to access portals:

| API                                           | Introduction                   | Trading area |
| --------------------------------------------- | ------------------------------ | ------------ |
| [GET /open/v1/tickers](#ebe64e52ff)           | Get All Market                 | Spot         |
| [GET /open/v1/balance](#870c0ab88b)           | Get Balance                    | Spot         |
| [GET /open/v1/timestamp](#fc5a31ea39)         | Get Sever Time                 | Spot         |
| [GET /open/v1/kline](#k-2)                    | Get Market K-line              | Spot         |
| [GET /open/v1/depth](#0f7bd4961a)             | Get Market Depth               | Spot         |
| [GET /open/v1/tickers/trade](#5)              | Get Latest Transaction Records | Spot         |
| [POST /open/v1/orders/place](#fd6ce2a756)     | Place Order                    | Spot         |
| [POST /open/v1/orders/cancel](#7742416be6)    | Cancel Order                   | Spot         |
| [POST /open/v1/orders/batcancel](#cedb99e805) | Batch Cancel Orders            | Spot         |
| [GET /open/v1/orders/last](#c2313ec9bf)       | Pending orders                 | Spot         |
| [GET /open/v1/orders](#cbbcc98be2)            | Get Completed Orders List      | Spot         |
| [GET /open/v1/orders/detail](#3fbc9cb788)     | Get Order Detail               | Spot         |
| [GET /open/v1/orders/detailmore](#d1baf83d74) | Get Order Detail List          | Spot         |
| [GET /open/v1/fee-rate](#6033256dc0)          | Get Fee Rate                   | Spot         |

# Connection Guide

## Restful host:
    https://api.bitda.com

## Websocket host:
    spot
    wss://api.bitda.com/ws

## Verification Notice


1. All portals need to conduct verification. Parameters are "client_id","ts","Nonce","sign". "client_id" is the api key. "client_key" is the secret key. Please be careful.

1. Ts is the current time stamp. Query with more than 5 seconds time difference will be rejected. "Nonce" is a random code that should Not be the same with last time query.

2. Signature method: link "client_id","ts","Nonce" in correct order, use hmac-sha256 to sign. e.g. the unsigned code: client_id=abc&Nonce=xyz&ts=1571293029

4. Signature: sign = hmac.New(client_key, sign_str, sha256)

5. Content-Type: application/x-www-form-urlencoded

6. Please put the parameters in the request body for the post interface request, and carry the get interface request in the url link


# WebSocket Guide

1. Need verification before subscription.

2. Verification method: {"method": "sign", "params": {"ts":"", "nonce":"", "client_id":"", "sign": ""}, "id": 1}, e.g:{"method": "sign", "params": {"ts":"1738944827", "nonce":"abcdef", "client_id":"xxxx", "sign": "xxxxx"}, "id": 1}

3. Client need to timely upload arbitrary code to check. Server will check status every 30 seconds. Links will be closed if No information received.{"method": "ping", "params": {}, "id": 1}

## Websocket Public API
    {"method":"sub", "params": {}, "id": 1}

### Market KLine
#### Request parameters

```json
{"method": "subscribe.kline", "params": {"period": "1Min", "market": "BTC-USDT"}, "id": 1}
```

| Parameter |  Type  | Description  |
| :-------: | :----: | :----------: |
|  period   | string | Kline period |
|  market   | string | Market name  |

> Responds:

```json
{
    "id": 1,
    "method": "update.kline",
    "result": {
        "data": {
            "symbol": "BTC-USDT",
            "ticks": [
                {
                    "amount": "1803565.8537891",
                    "close": "92613.10",
                    "high": "92613.18",
                    "low": "92613.01",
                    "open": "92613.01",
                    "timestamp": 1745474460,
                    "volume": "19.47420"
                }
            ],
            "timestamp": 1745474460,
            "topic": "kline:1Min:BTC-USDT",
            "tpp": 13,
            "type": "1Min"
        },
        "period": "1Min"
    },
    "error": null,
    "timestamp": 1745474480.5352995
}
```

#### Data refresh string list
| Parameter |  Type  |    Description     |
| :-------: | :----: | :----------------: |
|  symbol   | string |    Market name     |
|   ticks   | object | Kline information  |
|  amount   | string | Transaction amount |
|   close   | string |    Close price     |
|   high    | string |   Highest price    |
|    low    | string |    Lowest price    |
|   open    | string |     Open price     |
| timestamp | number | timestamp, second  |
|  volume   | string | Transaction volume |

### Market Transactions
#### Request parameters

```json
{"method": "subscribe.trade", "params": {"market": "BTC-USDT"}, "id": 1}
```

| Parameter |  Type  | Description |
| :-------: | :----: | :---------: |
|  market   | string | Market name |

> Responds:

```json
{
    "id": 1,
    "method": "update.trade",
    "result": {
        "amount": "35112.4213533",
        "price": "92618.03",
        "side": 2,
        "symbol": "BTC-USDT",
        "timestamp": 1745474636.571989,
        "topic": "trade:BTC-USDT",
        "tpp": 13,
        "volume": "0.37911"
    },
    "error": null,
    "timestamp": 1745474636.586593
}
```

#### Data refresh string list

| Parameter |  Type   |           Description           |
| :-------: | :-----: | :-----------------------------: |
|  amount   | string  |       Transaction amount        |
|   price   | string  |        Transaction price        |
|   side    | integer | Transaction side, 1 sell, 2 buy |
|  symbol   | string  |           Market name           |
| timestamp | number  |       Timestamp, sencond        |
|  volume   | string  |       Transaction volume        |

### Market Depth
#### Request parameters

```json
{"method": "subscribe.depth", "params": {"market": "BTC-USDT", "merge": "step1"}, "id": 1}
```

| Parameter |  Type  |                  Description                  |
| :-------: | :----: | :-------------------------------------------: |
|  market   | string |                  Market name                  |
|   merge   | string | Merge depth, supports 4 levels of merge depth |

> Responds:

```json
{
    "id": 1,
    "method": "update.depth",
    "result": {
        "data": {
            "asks": [
                {
                    "price": "92624.7",
                    "quantity": "0.24461"
                },
                {
                    "price": "92624.8",
                    "quantity": "0.22173"
                },
                {
                    "price": "92624.9",
                    "quantity": "0.21227"
                }
            ],
            "bids": [
                {
                    "price": "92607.8",
                    "quantity": "4.09932"
                },
                {
                    "price": "92602.3",
                    "quantity": "0.82077"
                },
                {
                    "price": "92592.4",
                    "quantity": "0.99661"
                }
            ],
            "symbol": "BTC-USDT",
            "timestamp": 1745474746.4458873,
            "topic": "depth:step1:BTC-USDT",
            "tpp": 13
        },
        "merge": "step1"
    },
    "error": null,
    "timestamp": 1745474746.4458911
}
```

#### Data refresh string list

| Parameter |  Type  |         Description          |
| :-------: | :----: | :--------------------------: |
|   bids    | object | bid depth[{price, quantity}] |
|   asks    | object | ask depth[{price, quantity}] |

### Market quotes
#### Request parameters

```json
{"method": "subscribe.quote", "params": {"market": "BTC-USDT"}, "id": 1}
```

| Parameter |  Type  | Description |
| :-------: | :----: | :---------: |
|  market   | string | Market name |

> Responds:

```json
{
    "id": 1,
    "method": "update.quote",
    "result": {
        "data": {
            "symbol": "BTC-USDT",
            "timestamp": 1745474844,
            "topic": "quote:BTC-USDT",
            "price": "92565.56",
            "volume": "232009.60329",
            "amount": "21672997623.5353922",
            "high": "94904.3",
            "low": "91942.82",
            "change": "-0.0097665217635025",
            "tpp": 13,
            "l_price": "92565.56"
        },
        "market": "BTC-USDT"
    },
    "error": null,
    "timestamp": 1745474845.0166624
}
```
#### Data refresh string list

| Parameter |  Type   |    Description     |
| :-------: | :-----: | :----------------: |
|  amount   | string  | Transaction amount |
|  change   | string  |   Rise and fall    |
|   price   | string  |       Price        |
|  symbol   | string  |    Market name     |
| timestamp | numbert | Timestamp, second  |
|   high    | string  |   Highest price    |
|    low    | string  |    Lowest price    |
|  l_price  | string  |     Last price     |
|  volume   | string  | Transaction volume |

### Account balance changes
#### Request parameters

```json
{"method": "subscribe.asset", "params": {}, "id": 1}
```

| Parameter | Type  | Description |
| :-------: | :---: | :---------: |
None

> Responds:

```json
{
    "id": 1,
    "method": "update.asset"
    "result": {
        "available": "10186.04408055", 
        "freeze": "0", 
        "symbol": "USDT", 
        "topic": "accounts", 
        "total": "10186.04408055"
    }
    "error": null, 
    "timestamp": 1745475263.2521095
}
```

#### Data refresh string list

| Parameter |  Type  |    Description    |
| :-------: | :----: | :---------------: |
| available | string | Available balance |
|  freeze   | string |  Freeze Balance   |
|  symbol   | string |     Coin name     |
|   total   | string |   Total balance   |

### Order changes
#### Request parameters

```json
{"method": "subscribe.orders", "params": {"market": "BTC-USDT"}, "id": 1}
```

| Parameter |  Type  | Description |
| :-------: | :----: | :---------: |
|  market   | string | Market name |

> Responds:

```json
{
    "id": 1,
    "method": "update.orders"
    "result": {
        "frm": "USDT",
        "left": "0.0439925710",
        "match_amt": "999.9560074290",
        "match_price": "0",
        "match_qty": "0.010807",
        "order_id": "16693603",
        "order_sub_type": 0,
        "order_type": 2,
        "price": "0",
        "quantity": "1000.000000",
        "real_order_id": "16693603",
        "side": 2,
        "status": 4,
        "stop_price": "0",
        "symbol": "BTC-USDT",
        "ticker_id": 13,
        "timestamp": 1745475263.240633,
        "to": "BTC",
        "topic": "orders",
        "tpp": 13,
        "trade_no": "553119111551218040041",
        "update_timestamp": 1745475263.240749
    }
    "error": null, 
    "timestamp": 1745475263.2521095
}
```

#### Response Content

|    Parameter     |  Type   |                                                                         Description                                                                         |
| :--------------: | :-----: | :---------------------------------------------------------------------------------------------------------------------------------------------------------: |
|       frm        | string  |                                                                      Money token name                                                                       |
|       left       | string  |                                                                     Remaining quantity                                                                      |
|    match_amt     | string  |                                                                        Match amount                                                                         |
|   match_price    | string  |                                                                         Match price                                                                         |
|    match_qty     | string  |                                                                       Match quantity                                                                        |
|     order_id     | string  |                                                                          Order ID                                                                           |
|    order_type    | string  |                                                              Order type,LIMIT(default),MARKET                                                               |
|      price       | string  |                                                                            Price                                                                            |
|     quantity     | string  |                                                                          Quantity                                                                           |
|       side       | integer |                                                                  Order side, 1 sell, 2 buy                                                                  |
|      status      | integer | Status, 2 pending，3 some transactions have not been completed ，4 all transactions completed，5 some transactions have been cancelled，6 cancel all orders |
|      symbol      | string  |                                                                         Market name                                                                         |
|    timestamp     | number  |                                                                         Create time                                                                         |
|        to        | string  |                                                                      Stock token name                                                                       |
|     trade_no     | string  |                                                                          Trade no                                                                           |
| update_timestamp | number  |                                                                         Update time                                                                         |

# Basic information

## Get All Market 
This api returns all or specified trading pairs supported by BitDa.

```shell
spot

"https://api.bitda.com/open/v1/tickers"

```

### HTTP Query
spot
- GET ` /open/v1/tickers`


<aside class="notice">limit 1r/s</aside>

### Request parameters
| Parameter |  Type  | Required |        Description         |
| :-------: | :----: | :------: | :------------------------: |
|  symbol   | string |    no    | Market name, e.g: BTC-USDT |

> Responds:

```json
{  
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": [
        {
            "amount": "1.586",
            "change": "-0.235462",
            "high": "3.05",
            "low": "3.05",
            "price": "98000",
            "l_price": "98000",
            "symbol": "BTC-USDT",
            "amt_num": 4,
            "qty_num": 2,
            "volume": "0.52"
        }, 
    ]
}
```

### Response Content

| Parameter |  Type   |         Description         |
| :-------: | :-----: | :-------------------------: |
|  amount   | string  | 24-hour transactions amount |
|  change   | string  |       24-hour change        |
|   high    | string  |    24-hour highest price    |
|    low    | string  |    24-hour lowest price     |
|   price   | string  |        Current price        |
|  l_price  | string  |         Last price          |
|  symbol   | string  |         Market name         |
|  amt_num  | integer |       Price precision       |
|  qty_num  | integer |      Volume precision       |
|  volume   | string  | 24-hour transactions volume |

## Get Balance

```shell
spot

"https://api.bitda.com/open/v1/balance"

```

### HTTP Query
spot
- GET ` /open/v1/balance`

### Request parameters
None

> Responds:

```json
{
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": [
        {
            "amount": "4317.6696678", 
            "symbol": "USDT", 
            "freeze": "71.609185"
        },
    ]
}
```

### Response Content

| Parameter |  Type  |    Description    |
| :-------: | :----: | :---------------: |
|  amount   | string | Available balance |
|  symbol   | string |     Coin name     |
|  freeze   | string |  Freeze balance   |

## Get Sever Time

```shell
spot

"https://api.bitda.com/open/v1/timestamp"
```

### HTTP Query
spot
- GET ` /open/v1/timestamp`


### Request parameters
| Parameter | Type  | Required | Description |
| :-------: | :---: | :------: | :---------: |
None
> Responds:

```json
{
    "code": 0,
    "msg": "ok",
    "data": "12354534",
    "time": 1745208892.421363,
}
```

### Response Content

| Parameter |  Type  |   Description    |
| :-------: | :----: | :--------------: |
|   data    | string | Server timestamp |


# Market data

## Get Market K-line

```shell
spot

"https://api.bitda.com/open/v1/kline"
```

### HTTP Query
spot
- GET ` /open/v1/kline`

<aside class="notice">limit 0.1r/s</aside>

### Request parameters
| Parameter |  Type  | Required |                               Description                                |
| :-------: | :----: | :------: | :----------------------------------------------------------------------: |
|  symbol   | string |   yes    |                        Market name, e.g: BTC-USDT                        |
|   type    | string |   yes    | Type: 1Min, 5Min, 15Min, 30Min,1Hour,2Hour,4Hour,6Hour,12Hour,1Day,1Week |

> Responds:

```json
{
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": [
        {
            "amount": "0",
            "close": "3.05",    
            "high": "3.05", 
            "low": "3.05", 
            "open": "3.05", 
            "time": 1571812440, 
            "volume": "0"
        }
    ]
}
```

### Response Content

| Parameter |  Type   |    Description     |
| :-------: | :-----: | :----------------: |
|  amount   | string  | Transaction amount |
|   close   | string  |    Close price     |
|   high    | string  |   Highest price    |
|    low    | string  |    Lowest price    |
|   open    | string  |     Open price     |
|   time    | integer | Timestamp, second  |
|  volume   | string  | Transaction volume |

## Get Market Depth

```shell
spot

"https://api.bitda.com/open/v1/depth"
```

### HTTP Query
spot
- GET ` /open/v1/depth`

<aside class="notice">limit 10r/s</aside>

### Request parameters
| Parameter |  Type  | Required | Description |
| :-------: | :----: | :------: | :---------: |
|  symbol   | string |   yes    | Market name |

> Responds:

```json
{
    "code": 0, 
    "time": 1745208892.421363,
    "data": {
        "bids": [
            {"price": "2.923", "quantity": "12"}, 
            {"price": "2.823", "quantity": "12"}, 
            {"price": "2.813", "quantity": "14"}
        ], 
        "asks": [
            {"price": "3.05", "quantity": "3.8"}, 
            {"price": "3.31", "quantity": "15"}, 
            {"price": "3.92", "quantity": "15"}
            ]
        }, 
    "msg": "Ok"
}
```

### Response Content

| Parameter |  Type  |       Description        |
| :-------: | :----: | :----------------------: |
|   bids    | object | Bids [{price, quantity}] |
|   asks    | object | Asks [{price, quantity}] |

## Get Latest Transaction Records

```shell
spot

"https://api.bitda.com/open/v1/tickers/trade"
```

### HTTP Query
spot
- GET ` /open/v1/tickers/trade`

<aside class="notice">limit 10r/s</aside>

### Request parameters
| Parameter |  Type  | Required | Description |
| :-------: | :----: | :------: | :---------: |
|  symbol   | string |   yes    | market name |

> Responds:

```json
{
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": [{
        "amount": "0.918",
        "price": "2.04",
        "side": 1,
        "time": 1574942822160,
        "volume": "0.45"
        }]
}
```

### Response Content

| Parameter |  Type   |     Description     |
| :-------: | :-----: | :-----------------: |
|  amount   | string  | Transaction amount  |
|   price   | string  |  Transaction price  |
|   side    | integer | side, 1 sell, 2 buy |
|   time    | integer | timestamp, sencond  |
|  volume   | string  | Transaction volume  |


# Spot

## Place Order 

```shell
spot

"https://api.bitda.com/open/v1/orders/place"
```

### HTTP Query
spot
- POST ` /open/v1/orders/place`

### Request parameters
| Parameter  |  Type  | Required |              Description               |
| :--------: | :----: | :------: | :------------------------------------: |
|   symbol   | string |   yes    |              Market name               |
|   price    | string |    no    | Price, required if it is a limit order |
|  quantity  | string |   yes    |                 Volume                 |
|    side    |  int   |   yes    |          Side, 1 sell, 2 buy           |
| order_type | string |    no    |   Order type LIMIT(default), MARKET    |

<aside class="warning">Whether buying or selling, quantity refers to the trading currency, such as BTC-USDT, quantity refers to the number of BTC</aside>
<aside class="warning">The first order for a new spot trading pair must be placed through this interface. Users can place orders only after the transaction is completed.</aside>

> Responds:

```json
{
    "code": 0,
    "msg": "ok",
    "time": 1745208892.421363,
    "data": {
        "create_at": 1738919982.01731,
        "frm": "USDT",
        "left": "0.000000",
        "match_amt": "9400.0000000000",
        "match_price": "94000",
        "match_qty": "0.100000",
        "order_id": "324",
        "order_sub_type": 0,
        "order_type": "LIMIT",
        "price": "8850.2100",
        "quantity": "0.100000",
        "side": 1,
        "status": 3,
        "stop_price": "0",
        "symbol": "BTC-USDT",
        "ticker": "BTC-USDT",
        "ticker_id": 13,
        "timestamp": 1738919982.01731,
        "to": "BTC",
        "trade_no": "40551041825640507172314",
        "update_timestamp": 1738919982.017397
    },
}
```

### Response Content

|    Parameter     |  Type   |                                                                         Description                                                                         |
| :--------------: | :-----: | :---------------------------------------------------------------------------------------------------------------------------------------------------------: |
|       frm        | string  |                                                                      Money token name                                                                       |
|       left       | string  |                                                                     Remaining quantity                                                                      |
|    match_amt     | string  |                                                                        Match amount                                                                         |
|   match_price    | string  |                                                                         Match price                                                                         |
|    match_qty     | string  |                                                                       Match quantity                                                                        |
|     order_id     | string  |                                                                          Order ID                                                                           |
|    order_type    | string  |                                                              Order type,LIMIT(default),MARKET                                                               |
|      price       | string  |                                                                            Price                                                                            |
|     quantity     | string  |                                                                          Quantity                                                                           |
|       side       | integer |                                                                  Order side, 1 sell, 2 buy                                                                  |
|      status      | integer | Status, 2 pending，3 some transactions have not been completed ，4 all transactions completed，5 some transactions have been cancelled，6 cancel all orders |
|      symbol      | string  |                                                                         Market name                                                                         |
|    timestamp     | number  |                                                                         Create time                                                                         |
|        to        | string  |                                                                      Stock token name                                                                       |
|     trade_no     | string  |                                                                          Trade no                                                                           |
| update_timestamp | number  |                                                                         Update time                                                                         |
|    create_at     | number  |                                                                         Create time                                                                         |

## Cancel Order

```shell
spot

"https://api.bitda.com/open/v1/orders/cancel"
```

### HTTP Query
spot
- POST ` /open/v1/orders/cancel`

### Request parameters
| Parameter |  Type  | Required |        Description         |
| :-------: | :----: | :------: | :------------------------: |
|  symbol   | string |   yes    | Market name, e.g: BTC-USDT |
| order_id  | string |   yes    |        Order number        |

> Responds:

```json
{
    "code": 0,
    "msg": "ok",
    "data": {
        "create_at": 1738923199.205091,
        "frm": "USDT",
        "left": "0.100000",
        "match_amt": "0",
        "match_price": "0",
        "match_qty": "0",
        "order_id": "327",
        "order_sub_type": 0,
        "order_type": "LIMIT",
        "price": "98850.2100",
        "quantity": "9885.021",
        "side": 1,
        "status": 6,
        "stop_price": "0",
        "symbol": "BTC-USDT",
        "ticker": "BTC-USDT",
        "ticker_id": 13,
        "timestamp": 1738923199.205091,
        "to": "BTC",
        "trade_no": "40551042845121219120919",
        "update_timestamp": 0
    },
    "time": 1745208892.421363,
}
```

### Response Content

|    Parameter     |  Type   |                                                                         Description                                                                         |
| :--------------: | :-----: | :---------------------------------------------------------------------------------------------------------------------------------------------------------: |
|    create_at     | number  |                                                                         Create time                                                                         |
|       frm        | string  |                                                                      Money token name                                                                       |
|       left       | string  |                                                                     Remaining quantity                                                                      |
|    match_amt     | string  |                                                                        Match amount                                                                         |
|   match_price    | string  |                                                                         Match price                                                                         |
|    match_qty     | string  |                                                                       Match quantity                                                                        |
|     order_id     | string  |                                                                          Order ID                                                                           |
|    order_type    | string  |                                                              Order type,LIMIT(default),MARKET                                                               |
|      price       | string  |                                                                            Price                                                                            |
|     quantity     | string  |                                                                          Quantity                                                                           |
|       side       | integer |                                                                  Order side, 1 sell, 2 buy                                                                  |
|      status      | integer | Status, 2 pending，3 some transactions have not been completed ，4 all transactions completed，5 some transactions have been cancelled，6 cancel all orders |
|    timestamp     | number  |                                                                         Create time                                                                         |
|        to        | string  |                                                                      Stock token name                                                                       |
|     trade_no     | string  |                                                                          Trade no                                                                           |
| update_timestamp | string  |                                                                         Update time                                                                         |


## Batch Cancel Orders 

```shell
spot

"https://api.bitda.com/open/v1/orders/batcancel"
```

### HTTP Query
spot
- POST ` /open/v1/orders/batcancel`

<aside class="notice">limit 1r/s</aside>

### Request parameters
| Parameter |  Type  | Required |                      Description                      |
| :-------: | :----: | :------: | :---------------------------------------------------: |
|  symbol   | string |   yes    |              Market name, e.g: BTC-USDT               |
| order_ids | string |    no    | Order id，1000,2000,3000， cancel all orders if empty |

> Responds:

```json
{
    "code": 0,
    "msg": "Ok",
    "data": {
        "success": [
        331,
        332
        ],
        "failed": []
    },
    "time": 1745208892.421363,
}
```

### Response Content

| Parameter | Type  |          Description           |
| :-------: | :---: | :----------------------------: |
|  success  | array | Cancel the successful order ID |
|  failed   | array |     Cancel failed order ID     |

## Pending orders

```shell
spot

"https://api.bitda.com/open/v1/orders/last"
```

### HTTP Query
spot
- GET ` /open/v1/orders/last`

### Request parameters
| Parameter |  Type   | Required |              Description              |
| :-------: | :-----: | :------: | :-----------------------------------: |
|  symbol   | string  |   yes    |      Market name, e.g: BTC-USDT       |
|  pagenum  | integer |    no    |              Page number              |
| pagesize  | integer |    no    | Page size,min 10, max 100, default 10 |

> Responds:

```json
{
    "code": 0, 
    "msg": "",
    "time": 1745208892.421363,
    "data": [
        {
            "create_at": 1738928701.630487,
            "frm": 80,
            "left": "0.100000",
            "match_amt": "0",
            "match_price": "0",
            "match_qty": "0",
            "order_id": "333",
            "order_sub_type": 0,
            "order_type": "LIMIT",
            "price": "98850.2100",
            "quantity": "0.100000",
            "side": 1,
            "status": 2,
            "stop_price": "",
            "symbol": "BTC-USDT",
            "ticker": "BTC-USDT",
            "ticker_id": 13,
            "timestamp": 1738928701.630487,
            "to": 81,
            "trade_no": "40551044588771203011001",
            "update_timestamp": 1738928701.630487
        }
    ], 
}
```

### Response Content

|    Parameter     |  Type   |                                                                         Description                                                                         |
| :--------------: | :-----: | :---------------------------------------------------------------------------------------------------------------------------------------------------------: |
|    create_at     | number  |                                                                         Create time                                                                         |
|       frm        | string  |                                                                      Money token name                                                                       |
|       left       | string  |                                                                     Remaining quantity                                                                      |
|    match_amt     | string  |                                                                        Match amount                                                                         |
|   match_price    | string  |                                                                         Match price                                                                         |
|    match_qty     | string  |                                                                       Match quantity                                                                        |
|     order_id     | string  |                                                                          Order ID                                                                           |
|    order_type    | string  |                                                              Order type,LIMIT(default),MARKET                                                               |
|      price       | string  |                                                                            Price                                                                            |
|     quantity     | string  |                                                                          Quantity                                                                           |
|       side       | integer |                                                                  Order side, 1 sell, 2 buy                                                                  |
|      status      | integer | Status, 2 pending，3 some transactions have not been completed ，4 all transactions completed，5 some transactions have been cancelled，6 cancel all orders |
|      symbol      | string  |                                                                         Market name                                                                         |
|    timestamp     | number  |                                                                         Create time                                                                         |
|        to        | string  |                                                                      Stock token name                                                                       |
|     trade_no     | string  |                                                                          Trade no                                                                           |
| update_timestamp | number  |                                                                         Update time                                                                         |

## Get Completed Orders List

```shell
spot

"https://api.bitda.com/open/v1/orders"
```

### HTTP Query
spot
- GET ` /open/v1/orders`

<aside class="notice">limit 0.5r/s</aside>

### Request parameters
| Parameter |  Type   | Required |             Description              |
| :-------: | :-----: | :------: | :----------------------------------: |
|  symbol   | string  |   yes    |      Market name, e.g: BTC-USDT      |
|  pagenum  | integer |    no    |             Page number              |
| pagesize  | integer |    no    | Page size,min 10, max 50, default 20 |
|   side    | integer |    no    |   Order side, 1 sell, 2 buy, 0 all   |
|   start   | integer |    否    |            Start time, s             |
|    end    | integer |    no    |             End time, s              |

> Responds:

```json
{
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": {
        "count": 4, 
        "orders": [
            {
                "create_at": 1738932323.230282,
                "match_amt": "9400",
                "match_price": "94000",
                "match_qty": "0.1",
                "order_id": "337",
                "order_type": "LIMIT",
                "price": "8850.21",
                "quantity": "0.1",
                "side": 1,
                "status": 4,
                "symbol": "BTC-USDT",
                "ticker": "BTC-USDT",
                "trade_no": "40551045736411111022504"
            }
        ]
    }, 
}
```

### Response Content

|  Parameter  |  Type   |                                                                         Description                                                                         |
| :---------: | :-----: | :---------------------------------------------------------------------------------------------------------------------------------------------------------: |
|  order_id   | string  |                                                                          Order ID                                                                           |
|  trade_no   | string  |                                                                          Trade no                                                                           |
|   symbol    | string  |                                                                         Market name                                                                         |
|    price    | string  |                                                                            price                                                                            |
|  quantity   | string  |                                                                          Quantity                                                                           |
|  match_amt  | string  |                                                                        Match amount                                                                         |
|  match_qty  | string  |                                                                       Match quantity                                                                        |
| match_price | string  |                                                                         Match price                                                                         |
|    side     | integer |                                                                  Order side, 1 sell, 2 buy                                                                  |
| order_type  | string  |                                                              Order type,LIMIT(default),MARKET                                                               |
|   status    | integer | Status, 2 pending，3 some transactions have not been completed ，4 all transactions completed，5 some transactions have been cancelled，6 cancel all orders |
|  create_at  | number  |                                                                         Create time                                                                         |

## Get Order Detail

```shell
spot

"https://api.bitda.com/open/v1/orders/detail"
```

### HTTP Query
spot
- GET ` /open/v1/orders/detail`

<aside class="notice">limit 6r/s</aside>

### Request parameters
| Parameter |  Type  | Required |        Description         |
| :-------: | :----: | :------: | :------------------------: |
|  symbol   | string |   yes    | Market name, e.g: BTC-USDT |
| order_id  | string |   yes    |          Order id          |

> Responds:

```json
{
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": {
        "create_at": 1738932323.230282,
        "fee": "0",
        "match_amt": "9400",
        "match_price": "94000",
        "match_qty": "0.1",
        "order_id": "337",
        "order_type": "LIMIT",
        "price": "8850.21",
        "quantity": "0.1",
        "side": 1,
        "status": 4,
        "symbol": "BTC-USDT",
        "ticker": "BTC-USDT",
        "trade_no": "40551045736411111022504",
        "trades": [
            {
                "amount": "9400",
                "fee": "0",
                "price": "94000",
                "quantity": "0.1",
                "time": 1738932323.230371,
                "trade_id": "208"
            }
        ]
    }
}
```

### Response Content

|  Parameter  |  Type   |                                                                         Description                                                                         |
| :---------: | :-----: | :---------------------------------------------------------------------------------------------------------------------------------------------------------: |
|  create_at  | number  |                                                                         Create time                                                                         |
|     fee     | string  |                                                                             Fee                                                                             |
|  match_amt  | string  |                                                                        Match amount                                                                         |
| match_price | string  |                                                                        成Match price                                                                        |
|  match_qty  | string  |                                                                      成Match quantity                                                                       |
|  order_id   | string  |                                                                          Order ID                                                                           |
| order_type  | string  |                                                              Order type,LIMIT(default),MARKET                                                               |
|    price    | string  |                                                                            Price                                                                            |
|  quantity   | string  |                                                                          Quantity                                                                           |
|    side     | integer |                                                                  Order side, 1 sell, 2 buy                                                                  |
|   status    | integer | Status, 2 pending，3 some transactions have not been completed ，4 all transactions completed，5 some transactions have been cancelled，6 cancel all orders |
|   symbol    | string  |                                                                         Market name                                                                         |
|  trade_no   | string  |                                                                          Trade no                                                                           |
|   trades    | object  |                                                                       Match orders[{                                                                        |
|   amount    | string  |                                                                        Match amount                                                                         |
|     fee     | string  |                                                                             Fee                                                                             |
|    price    | string  |                                                                            Price                                                                            |
|  quantity   | string  |                                                                          Quantity                                                                           |
|    time     | number  |                                                                         Match time                                                                          |
|  trade_id   | string  |                                                                          Deal ID}]                                                                          |

## Get Order Detail List

```shell
spot

"https://api.bitda.com/open/v1/orders/detailmore"
```

### HTTP Query
spot
- GET ` /open/v1/orders/detailmore`

<aside class="notice">limit 6r/s</aside>

### Request parameters
| Parameter |  Type   | Required |             Description             |
| :-------: | :-----: | :------: | :---------------------------------: |
|  symbol   | string  |   yes    |     Market name, e.g: BTC-USDT      |
| pagesize  | integer |    no    | Page size,min 10, max 50,default 10 |
|  pagenum  | integer |    no    |       Page number，default 1        |

> Responds:

```json
{
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": {
        "count": 10,
        "trades": [
            {
                "amount": "9400",
                "fee": "0",
                "price": "94000",
                "quantity": "0.1",
                "side": 1,
                "symbol": "BTC-USDT",
                "time": 1738932325.981797,
                "trade_id": "209"
            }
          ]
    }
}
```

### Response Content

| Parameter |  Type   |        Description        |
| :-------: | :-----: | :-----------------------: |
|   count   | integer |    Match orders count     |
|  trades   | object  |      match orders[{       |
|  symbol   | string  |        Market name        |
|   side    | integer | Order side, 1 sell, 2 buy |
| trade_id  | string  |          Deal id          |
|  amount   | string  |       Match amount        |
|   price   | string  |           Price           |
| quantity  | string  |         Quantity          |
|    fee    | string  |            Fee            |
|   time    | number  |       Match time}]        |

## Get Fee Rate

```shell
spot

"https://api.bitda.com/open/v1/fee-rate"
```

### HTTP Query
spot
- GET ` /open/v1/fee-rate`

### Request parameters
| Parameter |  Type  | Required |        Description         |
| :-------: | :----: | :------: | :------------------------: |
|  symbol   | string |   yes    | Market name, e.g: BTC-USDT |

> Responds:

```json
{
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": {
        "maker_fee": "0.0001",
        "taker_fee": "0.0002"
    }
}
```

### Response Content

| Parameter |  Type  | Description |
| :-------: | :----: | :---------: |
| maker_fee | string |  Maker fee  |
| taker_fee | string |  Taker fee  |


# Public API

## Get Market List

```shell
spot

"https://api.bitda.com/open/v1/tickers/market"
```

### HTTP Query
spot
- GET ` /open/v1/tickers/market`

<aside class="notice">limit 6r/s</aside>

### Request parameters
| Parameter | Type  | Required | Description |
| :-------: | :---: | :------: | :---------: |

> Responds:

```json
{  
    "code": 0,
    "msg": "Ok",
    "time": 1745208892.421363, 
    "data": [
        {
            "amount": "432142479.011539",
            "amt_num": 2,
            "change": "0.0089719236045157",
            "high": "88849.39",
            "low": "86328.39",
            "price": "88272.3",
            "l_price": "88272.3",
            "qty_num": 5,
            "symbol": "BTC-USDT",
            "volume": "4936.852"
        }
    ]
}
```

### Response Content

| Parameter |  Type   |    Description     |
| :-------: | :-----: | :----------------: |
|  amount   | string  | Transaction amount |
|  change   | string  |   Rise and fall    |
|   high    | string  |   Highest price    |
|    low    | string  |    Lowest price    |
|   price   | string  |       Price        |
|  l_price  | string  |     Last price     |
|  symbol   | string  |    Market name     |
|  amt_num  | integer |   Price decimal    |
|  qty_num  | integer |  Quantity decimal  |
|  volume   | string  | Transaction volume |

## Get Market Depth

```shell
spot

"https://api.bitda.com/open/v1/depth/market"
```

### HTTP Query
spot
- GET ` /open/v1/depth/market`

<aside class="notice">limit 5r/s</aside>

### Request parameters
| Parameter |  Type  | Required |        Description         |
| :-------: | :----: | :------: | :------------------------: |
|  symbol   | string |   yes    | Market name, e.g: BTC-USDT |

> Responds:

```json
{
    "code": 0, 
    "time": 1745208892.421363,
    "data": {
        "bids": [
            {"price": "2.923", "quantity": "12"}, 
            {"price": "2.823", "quantity": "12"}, 
            {"price": "2.813", "quantity": "14"}
        ], 
        "asks": [
            {"price": "3.05", "quantity": "31"}, 
            {"price": "3.31", "quantity": "15"}, 
            {"price": "3.92", "quantity": "15"}
            ]
        }, 
    "msg": "Ok"
}
```

### Response Content

| Parameter |  Type  |              Description               |
| :-------: | :----: | :------------------------------------: |
|   bids    | object |       Bids[{price , quantity }]        |
|   asks    | object | Asks[{price price, quantity quantity}] |

## Get Market Transactions

```shell
spot

"https://api.bitda.com/open/v1/trade/market"
```

### HTTP Query
spot
- GET ` /open/v1/trade/market`

<aside class="notice">limit 10r/s</aside>

### Request parameters
| Parameter |  Type  | Required |        Description         |
| :-------: | :----: | :------: | :------------------------: |
|  symbol   | string |   yes    | Market name, e.g: BTC-USDT |

> Responds:

```json
{
    "code": 0, 
    "msg": "Ok",
    "time": 1745208892.421363,
    "data": [{
        "amount": "0.918",
        "price": "2.04",
        "side": 1,
        "time": 1745307169.312,
        "volume": "0.45"
        }]
}
```

### Response Content

| Parameter |  Type   |        Description        |
| :-------: | :-----: | :-----------------------: |
|  amount   | string  |       Match amount        |
|   price   | string  |        Match price        |
|   side    | integer | Order side, 1 sell, 2 buy |
|   time    | integer |        Match time         |
|  volume   | string  |      Match quantity       |

## Get Market K-line

```shell
spot

"https://api.bitda.com/open/v1/kline/market"
```

### HTTP Query
spot
- GET ` /open/v1/kline/market`

<aside class="notice">limit 1r/s</aside>

### Request parameters
| Parameter |  Type  | Required |                               Description                               |
| :-------: | :----: | :------: | :---------------------------------------------------------------------: |
|  symbol   | string |    是    |                       Market name, e.g: BTC-USDT                        |
|   type    | string |    ye    | Type:1Min, 5Min, 15Min, 30Min,1Hour,2Hour,4Hour,6Hour,12Hour,1Day,1Week |

> Responds:

```json
{
    "code": 0, 
    "msg": "",
    "time": 1745208892.421363,
    "data": [
        {
            "timestamp": 1745298300,
            "open": "88065.43",
            "close": "88095.91",
            "high": "88095.91",
            "low": "88054.81",
            "volume": "2.54120",
            "amount": "223811.1117750"
        }
    ]
}
```

### Response Content

| Parameter |  Type   |    Description     |
| :-------: | :-----: | :----------------: |
|  amount   | string  | Transaction amount |
|   close   | string  |    Close price     |
|   high    | string  |   Highest price    |
|    low    | string  |    Lowest price    |
|   open    | string  |     Open price     |
| timestamp | integer | Timestamp, second  |
|  volume   | string  | Transaction volume |

# Examples

> Python:

```python
# -*- coding:utf-8 -*-

import requests
import time
import hmac
import hashlib
import ujson
import random

host = "https://api.bitda.com.io"
client_id = ""
client_key = ""

def gen_sign(client_id, client_key):
    ts = int(time.time())
    nonce = "abcdefg"
    obj = {"ts": ts, "nonce": nonce, "sign": "", "client_id": client_id}
    s = "client_id=%s&nonce=%s&ts=%s" % (client_id, nonce, ts) 
    v = hmac.new(client_key.encode(), s.encode(), digestmod=hashlib.sha256)
    obj["sign"] = v.hexdigest()
    return obj 

def get_open_orders():
    print("> Pending orders")
    # spot
    path = "/open/v1/orders/last"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_order_list():
    print("> Get Completed Orders List")
    # spot
    path = "/open/v1/orders"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT", "start": 1738886400, "end": 1738972800})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_order_detail():
    print("> Get Order Detail")
    # spot
    path = "/open/v1/orders/detail"
    obj = gen_sign(client_id, client_key)
    obj.update({"order_id": "337", "symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_order_detail_more():
    print("> Get Order Detail List")
    # spot
    path = "/open/v1/orders/detailmore"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT", "pagesize": 10, "page_num": 1})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_order_fee_rate():
    print("> Get Fee Rate")
    # spot
    path = "/open/v1/fee-rate"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_kline():
    print("> Get Market K-line")
    # spot
    path = "/open/v1/kline"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT", "type": "1Min"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_depth():
    print("> Get Market Depth")
    # spot
    path = "/open/v1/depth"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_balance():
    print("> Get Balance")
    # spot
    path = "/open/v1/balance"
    obj = gen_sign(client_id, client_key)
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_trade_tickers():
    print("> Get Latest Transaction Records")
    # spot
    path = "/open/v1/tickers/trade"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def place_order():
    print("> Place Order")
    # spot
    path = "/open/v1/orders/place"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "ETH-USDT", "price": "10", "quantity": "0.2", "side": "2", "order_type": "LIMIT"})
    res = requests.post(host + path, data=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def cancel_order():
    print("> Cancel Order")
    # spot
    path = "/open/v1/orders/cancel"
    obj = gen_sign(client_id, client_key)
    obj.update({"order_id": "327", "symbol": "BTC-USDT"})
    res = requests.post(host + path, data=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def bat_cancel_order():
    print("> Batch Cancel Orders")
    # spot
    path = "/open/v1/orders/batcancel"
    obj = gen_sign(client_id, client_key)
    ids = ["123", "124", "125"]
    obj.update({"symbol": "ETH-USDT", "order_ids": ",".join(ids)})
    res = requests.post(host + path, data=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_tickers():
    print("> Get All Market")
    # spot
    path = "/open/v1/tickers"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_server_time():
    print("> Get Sever Time")
    # spot
    path = "/open/v1/timestamp"
    obj = gen_sign(client_id, client_key)
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))
```

# Websocket examples

```python
# -*- coding:utf-8 -*-

import time
import hmac
import hashlib
import websockets
import json
import asyncio
import uvloop

asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())

# spot
host = "wss://api.bitda.com/ws"
client_id = ""
client_key = ""

def login():
    ts = int(time.time())
    nonce = "abcdefg"
    obj = {"method": "sign", "params": {"ts": ts, "nonce": nonce, "client_id": client_id, "sign": ""}, "id": 1}
    s = "client_id=%s&nonce=%s&ts=%s" % (client_id, nonce, ts)
    v = hmac.new(client_key.encode(), s.encode(), digestmod=hashlib.sha256)
    obj["sign"] = v.hexdigest()
    return obj

# Heartbeat
async def ping(ws):
    ping = {"method": "ping", "params": {}, "id": 1}
    await ws.send(json.dumps(ping))

# Depth
async def sub_topic_depth(ws):
    await ws.send(json.dumps({"method": "subscribe.depth", "params": {"market": "BTC-USDT", "merge": "step1"}, "id": 1}))

# Kline
async def sub_topic_kline(ws):
    await ws.send(json.dumps({"method": "subscribe.kline", "params": {"period": "1Min", "market": "BTC-USDT"}, "id": 1}))

# Orders
async def sub_topic_order(ws):
    await ws.send(json.dumps({"method": "subscribe.orders", "params": {"market": "BTC-USDT"}, "id": 1}))

# Trade
async def sub_topic_trade(ws):
    await ws.send(json.dumps({"method": "subscribe.trade", "params": {"market": "BTC-USDT"}, "id": 1}))

# Quote
async def sub_topic_quotes(ws):
    await ws.send(json.dumps({"method": "subscribe.quote", "params": {"market": "BTC-USDT"}, "id": 1}))

# Balance
async def sub_topic_asset(ws):
    await ws.send(json.dumps({"method": "subscribe.asset", "params": {}, "id": 1}))

async def startup():
    print("start to connect %s..." % host)
    ws = await websockets.connect(host)

    # Heartbeat
    await ping(ws)

    # Login
    obj = login()
    await ws.send(json.dumps(obj))

    # Depth
    await sub_topic_depth(ws)
    # Kline
    await sub_topic_kline(ws)
    # Orders
    await sub_topic_order(ws)
    # Trade
    await sub_topic_trade(ws)
    # Quote
    await sub_topic_quotes(ws)
    # Balance
    await sub_topic_asset(ws)

    while 1:
        try:
            data = await ws.recv()
            print(data)
        except websockets.exceptions.ConnectionClosed as e:
            print("connect closed...", e)
            return
        except:
            pass

if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(startup())
```

