---
title: BitDa Spot API 文档
---

# 简介

## API 简介

欢迎使用BitDa API！ 你可以使用此 API 获得市场行情数据，进行交易，并且管理你的账户。

在文档的右侧是代码，目前我们仅提供针对 `shell` 的代码示例。

可以使用以下域名访问：api.bitda.com

欢迎有优秀 maker 策略且交易量大的机构参与长期做市商项目。

## 公共接口
不需要鉴权可访问接口如下：

| 接口                                         | 说明       | 市场 |
| -------------------------------------------- | ---------- | ---- |
| [GET /open/v1/tickers/market](#495cebdeec-2) | 所有交易对 | 现货 |
| [GET /open/v1/depth/market](#a1128a972d-2)   | 深度       | 现货 |
| [GET /open/v1/trade/market](#775841b581-2)   | 逐笔成交   | 现货 |
| [GET /open/v1/kline/market](#k-3)            | k线数据    | 现货 |

## 鉴权接口
可以访问的接口如下：

| 接口                                          | 说明                     | 市场 |
| --------------------------------------------- | ------------------------ | ---- |
| [GET /open/v1/tickers](#ebe64e52ff)           | 全部或指定交易对         | 现货 |
| [GET /open/v1/balance](#870c0ab88b)           | 获取余额                 | 现货 |
| [GET /open/v1/timestamp](#fc5a31ea39)         | 服务器时间戳             | 现货 |
| [GET /open/v1/kline](#k-2)                    | 市场k线数据              | 现货 |
| [GET /open/v1/depth](#0f7bd4961a)             | 市场深度数据             | 现货 |
| [GET /open/v1/tickers/trade](#5)              | 获取最近5条成交记录      | 现货 |
| [POST /open/v1/orders/place](#fd6ce2a756)     | 下单                     | 现货 |
| [POST /open/v1/orders/cancel](#7742416be6)    | 撤销单个订单             | 现货 |
| [POST /open/v1/orders/batcancel](#cedb99e805) | 撤销全部或部分委托中订单 | 现货 |
| [GET /open/v1/orders/last](#c2313ec9bf)       | 委托中列表               | 现货 |
| [GET /open/v1/orders](#cbbcc98be2)            | 订单列表                 | 现货 |
| [GET /open/v1/orders/detail](#3fbc9cb788)     | 单个订单成交明细         | 现货 |
| [GET /open/v1/orders/detailmore](#d1baf83d74) | 分页获取成交明细         | 现货 |
| [GET /open/v1/orders/fee-rate](#6033256dc0)   | 获取用户某个交易对手续费 | 现货 |

# 接入说明

## Restful host:
    https://api.bitda.com

## Websocket host:
    现货交易
    wss://ws.bitda.com/ws

## 鉴权说明


1. 所有鉴权接口都需要进行鉴权，参数为client_id, ts, nonce, sign。client_id是api key, client_key为密钥，请妥善保管。

2. client_id为api key，ts为当前时间戳，与服务器时间差正负5秒会被拒绝，nonce为随机字符串，不能与上次请求所使用相同。

3. 签名方法, 将client_id, ts, nonce进行排序连接，使用hmac-sha256方法进行签名，例如待签名字符串为: client_id=abc&nonce=xyz&ts=1571293029

4. 签名: sign = hmac.New(client_key, sign_str, sha256)

5. Content-Type: application/x-www-form-urlencoded

6. 现货接口，post接口请求请将参数放在请求体里面，get接口请求携带在url链接中。


# WebSocket说明

1. 需要先进行鉴权，才可进行订阅。

2. 鉴权格式: {"method": "sign", "params": {"ts":"", "nonce":"", "client_id":"", "sign": ""}, "id": 1},如:{"method": "sign", "params": {"ts":"1738944827", "nonce":"abcdef", "client_id":"xxxx", "sign": "xxxxx"}, "id": 1}

3. 心跳处理，客户端需定时上发心跳信息，任意字符串，服务端每30秒会检查心跳，超时没有收到自动关闭连接。{"method": "ping", "params": {}, "id": 1}

## 订阅主题
    {"method":"sub", "params": {}, "id": 1}

### K线数据
#### 请求参数

```json
{"method": "subscribe.kline", "params": {"period": "1Min", "market": "BTC-USDT"}, "id": 1}
```

| 参数名 | 参数类型 |    描述    |
| :----: | :------: | :--------: |
| period |  string  | Kline周期  |
| market |  string  | 市场交易对 |

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

#### 数据更新字段列表
|  参数名   | 参数类型 |     描述     |
| :-------: | :------: | :----------: |
|  symbol   |  string  |  市场交易对  |
|   ticks   |  object  |   k线信息    |
|  amount   |  string  | 本阶段交易额 |
|   close   |  string  | 本阶段收盘价 |
|   high    |  string  | 本阶段最高价 |
|    low    |  string  | 本阶段最低价 |
|   open    |  string  | 本阶段开盘价 |
| timestamp |  number  |  时间戳 秒   |
|  volume   |  string  | 本阶段成交量 |

### 逐笔成交
#### 请求参数

```json
{"method": "subscribe.trade", "params": {"market": "BTC-USDT"}, "id": 1}
```

| 参数名 | 参数类型 |    描述    |
| :----: | :------: | :--------: |
| market |  string  | 市场交易对 |

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

#### 数据更新字段列表

|  参数名   | 参数类型 |        描述        |
| :-------: | :------: | :----------------: |
|  amount   |  string  |       成交额       |
|   price   |  string  |       成交价       |
|   side    | integer  | 成交方向，1卖，2买 |
|  symbol   |  string  |       交易对       |
| timestamp |  number  |     时间戳 秒      |
|  volume   |  string  |       成交量       |

### 深度
#### 请求参数

```json
{"method": "subscribe.depth", "params": {"market": "BTC-USDT", "merge": "step1"}, "id": 1}
```

| 参数名 | 参数类型 |           描述            |
| :----: | :------: | :-----------------------: |
| market |  string  |        市场交易对         |
| merge  |  string  | 合并深度，支持4级合并深度 |

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

#### 数据更新字段列表

| 参数名 | 参数类型 |                   描述                    |
| :----: | :------: | :---------------------------------------: |
|  bids  |  object  | 当前所有买单[{price 价格, quantity 数量}] |
|  asks  |  object  | 当前所有卖单[{price 价格, quantity 数量}] |

### 行情
#### 请求参数

```json
{"method": "subscribe.quote", "params": {"market": "BTC-USDT"}, "id": 1}
```

| 参数名 | 参数类型 |    描述    |
| :----: | :------: | :--------: |
| market |  string  | 市场交易对 |

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
#### 数据更新字段列表

|  参数名   | 参数类型 |    描述    |
| :-------: | :------: | :--------: |
|  amount   |  string  |   成交额   |
|  change   |  string  |   涨跌幅   |
|   price   |  string  |   当前价   |
|  symbol   |  string  |   交易对   |
| timestamp | numbert  | 时间戳 秒  |
|   high    |  string  |   最高价   |
|    low    |  string  |   最低价   |
|  l_price  |  string  | 上一次价格 |
|  volume   |  string  |   成交量   |

### 账户余额变化
#### 请求参数

```json
{"method": "subscribe.asset", "params": {}, "id": 1}
```

| 参数  | 说明  |
| :---: | :---: |
无

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

#### 数据更新字段列表

|  参数名   | 参数类型 |   描述   |
| :-------: | :------: | :------: |
| available |  string  | 可用余额 |
|  freeze   |  string  | 冻结余额 |
|  symbol   |  string  |   币种   |
|   total   |  string  |  总余额  |

### 委托变化
#### 请求参数

```json
{"method": "subscribe.orders", "params": {"market": "BTC-USDT"}, "id": 1}
```

| 参数名 | 参数类型 |    描述    |
| :----: | :------: | :--------: |
| market |  string  | 市场交易对 |

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

#### 返回字段

|      参数名      | 参数类型 |                                 描述                                  |
| :--------------: | :------: | :-------------------------------------------------------------------: |
|       frm        |  string  |                               价格币种                                |
|       left       |  string  |                               剩余数量                                |
|    match_amt     |  string  |                               成交金额                                |
|   match_price    |  string  |                               成交价格                                |
|    match_qty     |  string  |                               成交数量                                |
|     order_id     |  string  |                               委托单ID                                |
|    order_type    |  string  |              委托单类型,LIMIT限价单（默认）,MARKET市价单              |
|      price       |  string  |                                 价格                                  |
|     quantity     |  string  |                                 数量                                  |
|       side       | integer  |                         委托单方向，1卖，2买                          |
|      status      | integer  | 状态，2委托中，3部分成交未完成，4全部成交，5部分成交已撤单，6全部撤单 |
|      symbol      |  string  |                              市场交易对                               |
|    timestamp     |  number  |                               下单时间                                |
|        to        |  string  |                               数量币种                                |
|     trade_no     |  string  |                              委托单流水                               |
| update_timestamp |  number  |                               更新时间                                |

# 基础信息

## 所有交易对
此接口返回全部或指定BitDa支持的交易对。

```shell
现货市场

"https://api.bitda.com/open/v1/tickers"

```

### HTTP请求
现货市场
- GET ` /open/v1/tickers`


<aside class="notice">限速1r/s</aside>

### 请求参数
| 参数名 | 参数类型 | 是否必须 |           描述           |
| :----: | :------: | :------: | :----------------------: |
| symbol |  string  |    否    | 市场交易对，如: BTC-USDT |

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

### 返回字段

| 参数名  | 参数类型 |     描述     |
| :-----: | :------: | :----------: |
| amount  |  string  | 24小时成交额 |
| change  |  string  | 24小时涨跌幅 |
|  high   |  string  |  24小时最高  |
|   low   |  string  |  24小时最低  |
|  price  |  string  |    当前价    |
| l_price |  string  |   上次价格   |
| symbol  |  string  |  市场交易对  |
| amt_num | integer  |   价格精度   |
| qty_num | integer  |   数量精度   |
| volume  |  string  | 24小时成交量 |

## 账户余额

```shell
现货市场

"https://api.bitda.com/open/v1/balance"

```

### HTTP请求
现货交易
- GET ` /open/v1/balance`

### 请求参数
无

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

### 返回字段

| 参数名 | 参数类型 |   描述   |
| :----: | :------: | :------: |
| amount |  string  | 可用余额 |
| symbol |  string  |   币种   |
| freeze |  string  | 冻结余额 |

## 服务器时间戳

```shell
现货交易

"https://api.bitda.com/open/v1/timestamp"
```

### HTTP请求
现货交易
- GET ` /open/v1/timestamp`


### 请求参数
| 参数名 | 参数类型 | 是否必须 | 描述  |
| :----: | :------: | :------: | :---: |
无
> Responds:

```json
{
    "code": 0,
    "msg": "ok",
    "data": "12354534",
    "time": 1745208892.421363,
}
```

### 返回字段

| 参数名 | 参数类型 |  描述  |
| :----: | :------: | :----: |
|  data  |  string  | 时间戳 |


# 行情数据

## 市场k线数据

```shell
现货交易

"https://api.bitda.com/open/v1/kline"
```

### HTTP请求
现货交易
- GET ` /open/v1/kline`

<aside class="notice">限速0.1r/s</aside>

### 请求参数
| 参数名 | 参数类型 | 是否必须 |                                   描述                                   |
| :----: | :------: | :------: | :----------------------------------------------------------------------: |
| symbol |  string  |    是    |                         市场交易对，如: BTC-USDT                         |
|  type  |  string  |    是    | 类型1Min, 5Min, 15Min, 30Min,1Hour,2Hour,4Hour,6Hour,12Hour,1Day,1Week等 |

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

### 返回字段

| 参数名 | 参数类型 |     描述     |
| :----: | :------: | :----------: |
| amount |  string  |    成交额    |
| close  |  string  | 本阶段收盘价 |
|  high  |  string  | 本阶段最高价 |
|  low   |  string  | 本阶段最低价 |
|  open  |  string  | 本阶段开盘价 |
|  time  | integer  |     时间     |
| volume |  string  | 本阶段成交量 |

## 市场深度数据

```shell
现货交易

"https://api.bitda.com/open/v1/depth"
```

### HTTP请求
现货交易
- GET ` /open/v1/depth`

<aside class="notice">限速10r/s</aside>

### 请求参数
| 参数名 | 参数类型 | 是否必须 |    描述    |
| :----: | :------: | :------: | :--------: |
| symbol |  string  |    是    | 市场交易对 |

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

### 返回字段

| 参数名 | 参数类型 |                   描述                    |
| :----: | :------: | :---------------------------------------: |
|  bids  |  object  | 当前所有买单[{price 价格, quantity 数量}] |
|  asks  |  object  | 当前所有卖单[{price 价格, quantity 数量}] |

## 获取最近5条成交

```shell
现货交易

"https://api.bitda.com/open/v1/tickers/trade"
```

### HTTP请求
现货交易
- GET ` /open/v1/tickers/trade`

<aside class="notice">限速10r/s</aside>

### 请求参数
| 参数名 | 参数类型 | 是否必须 |    描述    |
| :----: | :------: | :------: | :--------: |
| symbol |  string  |    是    | 市场交易对 |

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

### 返回字段

| 参数名 | 参数类型 |        描述        |
| :----: | :------: | :----------------: |
| amount |  string  |       成交额       |
| price  |  string  |       成交价       |
|  side  | integer  | 成交方向，1卖，2买 |
|  time  | integer  |        时间        |
| volume |  string  |       成交量       |


# 现货

## 下单

```shell
现货交易

"https://api.bitda.com/open/v1/orders/place"
```

### HTTP请求
现货交易
- POST ` /open/v1/orders/place`

### 请求参数
|   参数名   | 参数类型 | 是否必须 |                    描述                     |
| :--------: | :------: | :------: | :-----------------------------------------: |
|   symbol   |  string  |    是    |                 市场交易对                  |
|   price    |  string  |    否    |          价格，如果是限价单，必填           |
|  quantity  |  string  |    是    |                    数量                     |
|    side    |   int    |    是    |                方向,1卖，2买                |
| order_type |  string  |    否    | 买卖单类型,LIMIT限价单（默认）,MARKET市价单 |

<aside class="warning">无论买或卖，quantity都表示交易币，如BTC-USDT，quantity都代表BTC的数量</aside>
<aside class="warning">现货新上交易对第一笔订单必须通过此接口下单，成交后用户才能下单</aside>

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

### 返回字段

|      参数名      | 参数类型 |                                 描述                                  |
| :--------------: | :------: | :-------------------------------------------------------------------: |
|       frm        |  string  |                               价格币种                                |
|       left       |  string  |                               剩余数量                                |
|    match_amt     |  string  |                               成交金额                                |
|   match_price    |  string  |                               成交价格                                |
|    match_qty     |  string  |                               成交数量                                |
|     order_id     |  string  |                               委托单ID                                |
|    order_type    |  string  |              委托单类型,LIMIT限价单（默认）,MARKET市价单              |
|      price       |  string  |                                 价格                                  |
|     quantity     |  string  |                                 数量                                  |
|       side       | integer  |                         委托单方向，1卖，2买                          |
|      status      | integer  | 状态，2委托中，3部分成交未完成，4全部成交，5部分成交已撤单，6全部撤单 |
|      symbol      |  string  |                                交易对                                 |
|    timestamp     |  number  |                               下单时间                                |
|        to        |  string  |                               数量币种                                |
|     trade_no     |  string  |                              委托单流水                               |
| update_timestamp |  number  |                               更新时间                                |
|    create_at     |  number  |                               下单时间                                |

## 撤销单个订单

```shell
现货交易

"https://api.bitda.com/open/v1/orders/cancel"
```

### HTTP请求
现货交易
- POST ` /open/v1/orders/cancel`

### 请求参数
|  参数名  | 参数类型 | 是否必须 |    描述    |
| :------: | :------: | :------: | :--------: |
|  symbol  |  string  |    是    | 市场交易对 |
| order_id |  string  |    是    |   委托号   |

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

### 返回字段

|      参数名      | 参数类型 |                                 描述                                  |
| :--------------: | :------: | :-------------------------------------------------------------------: |
|    create_at     |  number  |                               创建时间                                |
|       frm        |  string  |                               价格币种                                |
|       left       |  string  |                               剩余数量                                |
|    match_amt     |  string  |                               成交金额                                |
|   match_price    |  string  |                               成交价格                                |
|    match_qty     |  string  |                               成交数量                                |
|     order_id     |  string  |                               委托单ID                                |
|    order_type    |  string  |              委托单类型,LIMIT限价单（默认）,MARKET市价单              |
|      price       |  string  |                                 价格                                  |
|     quantity     |  string  |                                 数量                                  |
|       side       | integer  |                               1卖，2买                                |
|      status      | integer  | 状态，2委托中，3部分成交未完成，4全部成交，5部分成交已撤单，6全部撤单 |
|    timestamp     |  number  |                               下单时间                                |
|        to        |  string  |                               数量币种                                |
|     trade_no     |  string  |                                流水号                                 |
| update_timestamp |  string  |                               更新时间                                |


## 撤销部分或所有委托中订单

```shell
现货交易

"https://api.bitda.com/open/v1/orders/batcancel"
```

### HTTP请求
现货交易
- POST ` /open/v1/orders/batcancel`

<aside class="notice">限速1r/s</aside>

### 请求参数
|  参数名   | 参数类型 | 是否必须 |                            描述                             |
| :-------: | :------: | :------: | :---------------------------------------------------------: |
|  symbol   |  string  |    是    |                           交易对                            |
| order_ids |  string  |    否    | 交易对id，1000,2000,3000， 英文逗号分隔订单id，为空全部撤单 |

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

### 返回字段

| 参数名  | 参数类型 |     描述     |
| :-----: | :------: | :----------: |
| success |  array   | 成功的订单ID |
| failed  |  array   | 失败的订单ID |

## 委托中列表

```shell
现货交易

"https://api.bitda.com/open/v1/orders/last"
```

### HTTP请求
现货交易
- GET ` /open/v1/orders/last`

### 请求参数
|  参数名  | 参数类型 | 是否必须 |             描述              |
| :------: | :------: | :------: | :---------------------------: |
|  symbol  |  string  |    是    |          市场交易对           |
| pagenum  | integer  |    否    |             页码              |
| pagesize | integer  |    否    | 页大小,最小10, 最大100,默认10 |

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

### 返回字段

|      参数名      | 参数类型 |                                 描述                                  |
| :--------------: | :------: | :-------------------------------------------------------------------: |
|    create_at     |  number  |                               下单时间                                |
|       frm        |  string  |                               价格币种                                |
|       left       |  string  |                               剩余数量                                |
|    match_amt     |  string  |                               成交金额                                |
|   match_price    |  string  |                               成交价格                                |
|    match_qty     |  string  |                               成交数量                                |
|     order_id     |  string  |                               委托单ID                                |
|    order_type    |  string  |              委托单类型,LIMIT限价单（默认）,MARKET市价单              |
|      price       |  string  |                                 价格                                  |
|     quantity     |  string  |                                 数量                                  |
|       side       | integer  |                               1卖，2买                                |
|      status      | integer  | 状态，2委托中，3部分成交未完成，4全部成交，5部分成交已撤单，6全部撤单 |
|      symbol      |  string  |                                交易对                                 |
|    timestamp     |  number  |                               创建时间                                |
|        to        |  string  |                               数量币种                                |
|     trade_no     |  string  |                               委托流水                                |
| update_timestamp |  number  |                               更新时间                                |

## 订单列表

```shell
现货交易

"https://api.bitda.com/open/v1/orders"
```

### HTTP请求
现货交易
- GET ` /open/v1/orders`

<aside class="notice">限速0.5r/s</aside>

### 请求参数
|  参数名  | 参数类型 | 是否必须 |             描述             |
| :------: | :------: | :------: | :--------------------------: |
|  symbol  |  string  |    是    |    市场交易对,如BTC-USDT     |
| pagenum  | integer  |    否    |             页码             |
| pagesize | integer  |    否    | 页大小,最小10, 最大50,默认20 |
|   side   | integer  |    否    |    方向，1卖，2买，0所有     |
|  start   | integer  |    否    |       时间，时间戳，秒       |
|   end    | integer  |    否    |     结束时间，时间戳，秒     |

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

### 返回字段

|   参数名    | 参数类型 |                                 描述                                  |
| :---------: | :------: | :-------------------------------------------------------------------: |
|  order_id   |  string  |                               委托单ID                                |
|  trade_no   |  string  |                               委托流水                                |
|   symbol    |  string  |                                交易对                                 |
|    price    |  string  |                                 价格                                  |
|  quantity   |  string  |                                 数量                                  |
|  match_amt  |  string  |                                成交额                                 |
|  match_qty  |  string  |                               成交数量                                |
| match_price |  string  |                               成交价格                                |
|    side     | integer  |                            方向，1卖，2买                             |
| order_type  |  string  |              委托单类型,LIMIT限价单（默认）,MARKET市价单              |
|   status    | integer  | 状态，2委托中，3部分成交未完成，4全部成交，5部分成交已撤单，6全部撤单 |
|  create_at  |  number  |                               创建时间                                |

## 单个订单成交明细

```shell
现货交易

"https://api.bitda.com/open/v1/orders/detail"
```

### HTTP请求
现货交易
- GET ` /open/v1/orders/detail`

<aside class="notice">限速6r/s</aside>

### 请求参数
|  参数名  | 参数类型 | 是否必须 |         描述          |
| :------: | :------: | :------: | :-------------------: |
|  symbol  |  string  |    是    | 市场交易对,如BTC-USDT |
| order_id |  string  |    是    |      委托订单id       |

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

### 返回字段

|   参数名    | 参数类型 |                                 描述                                  |
| :---------: | :------: | :-------------------------------------------------------------------: |
|  create_at  |  number  |                               下单时间                                |
|     fee     |  string  |                                手续费                                 |
|  match_amt  |  string  |                                成交额                                 |
| match_price |  string  |                               成交价格                                |
|  match_qty  |  string  |                               成交数量                                |
|  order_id   |  string  |                               委托单ID                                |
| order_type  |  string  |              委托单类型,LIMIT限价单（默认）,MARKET市价单              |
|    price    |  string  |                                 价格                                  |
|  quantity   |  string  |                                 数量                                  |
|    side     | integer  |                            方向，1卖，2买                             |
|   status    | integer  | 状态，2委托中，3部分成交未完成，4全部成交，5部分成交已撤单，6全部撤单 |
|   symbol    |  string  |                                交易对                                 |
|  trade_no   |  string  |                              委托单流水                               |
|   trades    |  object  |                              成交订单[{                               |
|   amount    |  string  |                               成交金额                                |
|     fee     |  string  |                                手续费                                 |
|    price    |  string  |                                 价格                                  |
|  quantity   |  string  |                                 数量                                  |
|    time     |  number  |                                 时间                                  |
|  trade_id   |  string  |                              成交单ID}]                               |

## 分页获取订单成交明细

```shell
现货交易

"https://api.bitda.com/open/v1/orders/detailmore"
```

### HTTP请求
现货交易
- GET ` /open/v1/orders/detailmore`

<aside class="notice">限速6r/s</aside>

### 请求参数
|  参数名  | 参数类型 | 是否必须 |             描述             |
| :------: | :------: | :------: | :--------------------------: |
|  symbol  |  string  |    是    |    市场交易对,如BTC-USDT     |
| pagesize | integer  |    否    | 页大小,最小10, 最大50,默认10 |
| pagenum  | integer  |    否    |        页码，默认为1         |

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

### 返回字段

|  参数名  | 参数类型 |      描述      |
| :------: | :------: | :------------: |
|  count   | integer  |  成交订单数量  |
|  trades  |  object  |   成交订单[{   |
|  symbol  |  string  |   市场交易对   |
|   side   | integer  | 方向，1卖，2买 |
| trade_id |  string  |    成交单ID    |
|  amount  |  string  |      金额      |
|  price   |  string  |      价格      |
| quantity |  string  |      数量      |
|   fee    |  string  |     手续费     |
|   time   |  number  |     时间}]     |

## 获取用户某个交易对手续费

```shell
现货市场

"https://api.bitda.com/open/v1/fee-rate"
```

### HTTP请求
现货交易
- GET ` /open/v1/fee-rate`

### 请求参数
| 参数名 | 参数类型 | 是否必须 |    描述    |
| :----: | :------: | :------: | :--------: |
| symbol |  string  |    是    | 市场交易对 |

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

### 返回字段

|  参数名   | 参数类型 |    描述    |
| :-------: | :------: | :--------: |
| maker_fee |  string  | 挂单手续费 |
| taker_fee |  string  | 吃单手续费 |


# 公共接口

## 所有交易对

```shell
现货交易

"https://api.bitda.com/open/v1/tickers/market"
```

### HTTP请求
现货交易
- GET ` /open/v1/tickers/market`

<aside class="notice">限速6r/s</aside>

### 请求参数
| 参数名 | 参数类型 | 是否必须 | 描述  |
| :----: | :------: | :------: | :---: |

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

### 返回字段

| 参数名  | 参数类型 |     描述     |
| :-----: | :------: | :----------: |
| amount  |  string  | 24小时成交额 |
| change  |  string  | 24小时涨跌幅 |
|  high   |  string  |  24小时最高  |
|   low   |  string  |  24小时最低  |
|  price  |  string  |    当前价    |
| l_price |  string  |   上次价格   |
| symbol  |  string  |  市场交易对  |
| amt_num | integer  |   价格精度   |
| qty_num | integer  |   数量精度   |
| volume  |  string  | 24小时成交量 |

## 深度

```shell
现货交易

"https://api.bitda.com/open/v1/depth/market"
```

### HTTP请求
现货交易
- GET ` /open/v1/depth/market`

<aside class="notice">限速5r/s</aside>

### 请求参数
| 参数名 | 参数类型 | 是否必须 |         描述          |
| :----: | :------: | :------: | :-------------------: |
| symbol |  string  |    是    | 市场交易对,如BTC-USDT |

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

### 返回字段

| 参数名 | 参数类型 |                   描述                    |
| :----: | :------: | :---------------------------------------: |
|  bids  |  object  | 当前所有买单[{price 价格, quantity 数量}] |
|  asks  |  object  | 当前所有卖单[{price 价格, quantity 数量}] |

## 逐笔成交

```shell
现货交易

"https://api.bitda.com/open/v1/trade/market"
```

### HTTP请求
现货交易
- GET ` /open/v1/trade/market`

<aside class="notice">限速10r/s</aside>

### 请求参数
| 参数名 | 参数类型 | 是否必须 |         描述          |
| :----: | :------: | :------: | :-------------------: |
| symbol |  string  |    是    | 市场交易对,如BTC-USDT |

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

### 返回字段

| 参数名 | 参数类型 |        描述        |
| :----: | :------: | :----------------: |
| amount |  string  |       成交额       |
| price  |  string  |       成交价       |
|  side  | integer  | 成交方向，1卖，2买 |
|  time  | integer  |        时间        |
| volume |  string  |       成交量       |

## k线数据

```shell
现货交易

"https://api.bitda.com/open/v1/kline/market"
```

### HTTP请求
现货交易
- GET ` /open/v1/kline/market`

<aside class="notice">限速1r/s</aside>

### 请求参数
| 参数名 | 参数类型 | 是否必须 |                                   描述                                   |
| :----: | :------: | :------: | :----------------------------------------------------------------------: |
| symbol |  string  |    是    |                           市场交易对: BTC-USDT                           |
|  type  |  string  |    是    | 类型1Min, 5Min, 15Min, 30Min,1Hour,2Hour,4Hour,6Hour,12Hour,1Day,1Week等 |

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

### 返回字段

|  参数名   | 参数类型 |     描述     |
| :-------: | :------: | :----------: |
|  amount   |  string  |    成交额    |
|   close   |  string  | 本阶段收盘价 |
|   high    |  string  | 本阶段最高价 |
|    low    |  string  | 本阶段最低价 |
|   open    |  string  | 本阶段开盘价 |
| timestamp | integer  |     时间     |
|  volume   |  string  | 本阶段成交量 |

# 现货API调用示例

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
    print("> 获取open委托中")
    # 现货交易
    path = "/open/v1/orders/last"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_order_list():
    print("> 获取委托单列表")
    # 现货交易
    path = "/open/v1/orders"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT", "start": 1738886400, "end": 1738972800})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_order_detail():
    print("> 获取单个订单成交明细")
    # 现货交易
    path = "/open/v1/orders/detail"
    obj = gen_sign(client_id, client_key)
    obj.update({"order_id": "337", "symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_order_detail_more():
    print("> 分页获取成交明细")
    # 现货交易
    path = "/open/v1/orders/detailmore"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT", "pagesize": 10, "page_num": 1})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_order_fee_rate():
    print("> 获取用户某个交易对手续费")
    # 现货交易
    path = "/open/v1/orders/fee-rate"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_kline():
    print("> 获取kline")
    # 现货交易
    path = "/open/v1/kline"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT", "type": "1Min"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

# 获取深度
def get_depth():
    print("> 获取深度")
    # 现货交易
    path = "/open/v1/depth"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_balance():
    print("> 获取余额")
    # 现货交易
    path = "/open/v1/balance"
    obj = gen_sign(client_id, client_key)
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_trade_tickers():
    print("> 获取最近成交记录")
    # 现货交易
    path = "/open/v1/tickers/trade"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def place_order():
    print("> 下单")
    # 现货交易
    path = "/open/v1/orders/place"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "ETH-USDT", "price": "10", "quantity": "0.2", "side": "2", "order_type": "LIMIT"})
    res = requests.post(host + path, data=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def cancel_order():
    print("> 撤单")
    # 现货交易
    path = "/open/v1/orders/cancel"
    obj = gen_sign(client_id, client_key)
    obj.update({"order_id": "327", "symbol": "BTC-USDT"})
    res = requests.post(host + path, data=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

# 批量撤单
def bat_cancel_order():
    print("> 批量撤单")
    # 现货交易
    path = "/open/v1/orders/batcancel"
    obj = gen_sign(client_id, client_key)
    ids = ["123", "124", "125"]
    obj.update({"symbol": "ETH-USDT", "order_ids": ",".join(ids)})
    res = requests.post(host + path, data=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

def get_tickers():
    print("> 获取ticker")
    # 现货交易
    path = "/open/v1/tickers"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

# 获取服务器时间
def get_server_time():
    print("> 获取服务器时间")
    # 现货交易
    path = "/open/v1/timestamp"
    obj = gen_sign(client_id, client_key)
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

# 获取最新的成交
def get_trade():
    print("> 获取最新的成交")
    # 现货交易
    path = "/open/v1/tickers/trade"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))

# 获取用户手续费
def get_fee_rate():
    print("> 获取用户手续费")
    # 现货交易
    path = "/open/v1/fee-rate"
    obj = gen_sign(client_id, client_key)
    obj.update({"symbol": "BTC-USDT"})
    res = requests.get(host + path, params=obj)
    print(ujson.dumps(ujson.loads(res.content), ensure_ascii=False))
```

# Websocket示例

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

# 现货交易
host = "wss://ws.bitda.com/ws"
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

# 心跳
async def ping(ws):
    ping = {"method": "ping", "params": {}, "id": 1}
    await ws.send(json.dumps(ping))

# 订阅深度
async def sub_topic_depth(ws):
    await ws.send(json.dumps({"method": "subscribe.depth", "params": {"market": "BTC-USDT", "merge": "step1"}, "id": 1}))

# 订阅K线
async def sub_topic_kline(ws):
    await ws.send(json.dumps({"method": "subscribe.kline", "params": {"period": "1Min", "market": "BTC-USDT"}, "id": 1}))

# 订阅订单
async def sub_topic_order(ws):
    await ws.send(json.dumps({"method": "subscribe.orders", "params": {"market": "BTC-USDT"}, "id": 1}))

# 订阅成交
async def sub_topic_trade(ws):
    await ws.send(json.dumps({"method": "subscribe.trade", "params": {"market": "BTC-USDT"}, "id": 1}))

# 行情
async def sub_topic_quotes(ws):
    await ws.send(json.dumps({"method": "subscribe.quote", "params": {"market": "BTC-USDT"}, "id": 1}))

# 资产
async def sub_topic_asset(ws):
    await ws.send(json.dumps({"method": "subscribe.asset", "params": {}, "id": 1}))

async def startup():
    print("start to connect %s..." % host)
    ws = await websockets.connect(host)

    # 心跳
    await ping(ws)

    # 登录
    obj = login()
    await ws.send(json.dumps(obj))

    # 订阅深度
    await sub_topic_depth(ws)
    # 订阅K线
    await sub_topic_kline(ws)
    # 订阅订单
    await sub_topic_order(ws)
    # 订阅成交
    await sub_topic_trade(ws)
    # 行情
    await sub_topic_quotes(ws)
    # 资产
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

