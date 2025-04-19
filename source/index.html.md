---
title: BitDa API 文档
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
    wss://api.bitda.com/ws

## 鉴权说明


1. 所有接口都需要进行鉴权，参数为client_id, ts, nonce, sign。client_id是api key, client_key为密钥，请妥善保管。

2. client_id为api key，ts为当前时间戳，与服务器时间差正负5秒会被拒绝，nonce为随机字符串，不能与上次请求所使用相同。

3. 签名方法, 将client_id, ts, nonce进行排序连接，使用hmac-sha256方法进行签名，例如待签名字符串为: client_id=abc&nonce=xyz&ts=1571293029

4. 签名: sign = hmac.New(client_key, sign_str, sha256)

5. Content-Type: application/x-www-form-urlencoded

6. 现货接口，post接口请求请将参数放在请求体里面，get接口请求携带在url链接中。


# WebSocket说明

1. 需要先进行鉴权，才可进行订阅。

2. 鉴权格式: {"op":"apilogin","sign":"","client_id":"","nonce":"","ts": int type},如:{"op":"apilogin","sign":"abc123","client_id":"abc123","nonce":"1","ts": 1576207749}

3. 心跳处理，客户端需定时上发心跳信息，任意字符串，服务端每30秒会检查心跳，超时没有收到自动关闭连接。{"op":"sub", "topic":"hb"}

## 订阅主题
    {"op":"sub", "topic": ""}

### K线数据
#### 请求参数

```json
{"op":"sub", "topic": "kline:1Min:BTC-USDT"}
```

|        参数         |        说明        |
| :-----------------: | :----------------: |
| kline:1Min:BTC-USDT | BTC-USDT的1分钟k线 |

> Responds:

```json
{
    "symbol":"BTC-USDT",
    "ticks":[
        {
            "close":"2.62",
            "high":"3.11",
            "low":"2.62",
            "open":"3.01",
            "timestamp":1572851100,
            "volume":"17.55"
        }
    ],
    "timestamp":1572851160917,
    "topic":"kline:1Min:BTC-USDT",
    "type":"60000"
}
```

#### 数据更新字段列表
|  参数名   | 参数类型 |     描述     |
| :-------: | :------: | :----------: |
|  symbol   |  string  |    交易对    |
|   ticks   |  object  |   ␈k线信息   |
|   close   |  string  | 本阶段收盘价 |
|   high    |  string  | 本阶段最高价 |
|    low    |  string  | 本阶段最低价 |
|   open    |  string  | 本阶段开盘价 |
| timestamp | integer  | 时间戳 毫秒  |
|  volume   |  string  |    成交量    |

### 逐笔成交
#### 请求参数

```json
{"op":"sub", "topic": "trade:BTC-USDT"}
```

|      参数      |         说明         |
| :------------: | :------------------: |
| trade:BTC-USDT | BTC-USDT逐笔成交记录 |

> Responds:

```json
{
    "amount":"7.473",
    "price":"2.82",
    "side":1,
    "symbol":"BTC-USDT",
    "timestamp":1572851197910,
    "topic":"trade:BTC-USDT",
    "volume":"2.65"
}
```

#### 数据更新字段列表

|  参数名   | 参数类型 |        描述         |
| :-------: | :------: | :-----------------: |
|  amount   |  string  |       成交额        |
|   price   |  string  |       成交价        |
|   side    | integer  | 成交方向，1买，-1卖 |
|  symbol   |  string  |       交易对        |
| timestamp | integer  |     时间戳 毫秒     |
|  volume   |  string  |       成交量        |

### 深度
#### 请求参数

```json
{"op":"sub", "topic": "depth:0:BTC-USDT"}
```

|       参数       | 说明  |
| :--------------: | :---: |
| depth:0:BTC-USDT | 深度  |

> Responds:

```json
{
    "bids":[
        {'price': '2.923', 'quantity': '12'},
        {'price': '2.823', 'quantity': '12'},
        {'price': '2.723', 'quantity': '16'}
    ], 
    "asks":[
        {'price': '3.05', 'quantity': '3.48'},
        {'price': '3.31', 'quantity': '5'},
        {'price': '3.55', 'quantity': '10'}
    ],
    "symbol":"BTC-USDT",
    "timestamp":1572851208935,
    "topic":"depth:0:BTC-USDT"
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
{"op":"sub", "topic": "quotes"}
```

|  参数  | 说明  |
| :----: | :---: |
| quotes | 行情  |

> Responds:

```json
{
    "amount":"52080.1255",
    "change":"0.00949367",
    "price":"3.19",
    "symbol":"BTC-USDT",
    "timestamp":1572851216950,
    "topic":"quotes",
    "volume":"17965.65"
}
```
#### 数据更新字段列表

|  参数名   | 参数类型 |     描述     |
| :-------: | :------: | :----------: |
|  amount   |  string  |    成交额    |
|  change   |  string  |    涨跌幅    |
|   price   |  string  |    当前价    |
|  symbol   |  string  |    交易对    |
| timestamp | integer  | 时间戳 毫秒  |
|  volume   |  string  | 24小时成交量 |

### 账户余额变化
#### 请求参数

```json
{"op":"sub", "topic": "accounts"}
```

|   参数   |     说明     |
| :------: | :----------: |
| accounts | 账户余额变化 |

> Responds:

```json
{
    "available":"4194.3466678",
    "freeze":"71.609185",
    "symbol":"USDT",
    "topic":"accounts",
    "total":"4265.9558528"
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
{"op":"sub", "topic": "orders:BTC-USDT"}
```

|      参数       |   说明   |
| :-------------: | :------: |
| orders:BTC-USDT | 委托变化 |

> Responds:

```json
// 下单
{
    "left":"1",
    "order_id":"11574948935833473",
    "order_type":1,
    "price":"80000",
    "quantity":"1",
    "side":-1,
    "status":2,
    "symbol":"BTC-USDT",
    "timestamp":1574949805841,
    "topic":"orders:BTC-USDT",
    "trade_no":"499081745280826070655",
    "match_qty":"0"
}

// 撤单
{
    "left":"0",
    "order_id":"11574948935833473",
    "order_type":1,
    "price":"80000",
    "quantity":"1",
    "side":-1,
    "status":6,
    "symbol":"BTC-USDT",
    "timestamp":1574949805841,
    "topic":"orders:BTC-USDT",
    "trade_no":"499081745280826070655",
    "match_qty":"0",
    "match_price":"0"
}
```

#### 返回字段

|   参数名    | 参数类型 |                              描述                               |
| :---------: | :------: | :-------------------------------------------------------------: |
|    left     |  string  |                            剩余数量                             |
|  order_id   |  string  |                             订单id                              |
| order_type  |   int    |                      订单类型,1限价，3市价                      |
|    price    |  string  |                             委托价                              |
|  quantity   |  string  |                            委托数量                             |
|    side     |   int    |                         方向，1买，-1卖                         |
|   status    |   int    | 状态 2 委托中，3部分成交，4全部成交，5部分成交后撤消，6全部撤消 |
|   symbol    |  string  |                             交易对                              |
|  timestamp  |   int    |                          创建时间 毫秒                          |
|  trade_no   |  string  |                           订单流水号                            |
|  match_qty  |  string  |                           已成交数量                            |
| match_price |  string  |                            成交均价                             |


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
| 参数名 | 参数类型 | 是否必须 |         描述         |
| :----: | :------: | :------: | :------------------: |
| symbol |  string  |    否    | 交易对，如: BTC-USDT |

> Responds:

```json
{  
    'code': 0, 
    'data': [
            {'amount': '1.586',
            'change': '-0.235462',
            'high': '3.05',
            'low': '3.05',
            'price': '0',
            'symbol': 'BTC-USDT',
            'amt_num': 4,
            'qty_num': 2,
            'volume': '0.52'
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
| symbol  |  string  |    交易对    |
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
    'code': 0, 
    'data': [
        {
            'amount': '4317.6696678', 
            'symbol': "USDT", 
            'freeze': '71.609185'
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
| 参数名 | 参数类型 | 是否必须 |              描述              |
| :----: | :------: | :------: | :----------------------------: |
| symbol |  string  |    是    |      交易对，如: BTC-USDT      |
|  type  |  string  |    是    | 类型1Min, 5Min, 15Min, 30Min等 |

> Responds:

```json
{
    'code': 0, 
    'data': [
        {
            'amount': '0',
            'close': '3.05',    
            'high': '3.05', 
            'low': '3.05', 
            'open': '3.05', 
            'time': 1571812440, 
            'volume': '0'
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
| volume |  string  |    成交量    |

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
| 参数名 | 参数类型 | 是否必须 |  描述  |
| :----: | :------: | :------: | :----: |
| symbol |  string  |    是    | 交易对 |

> Responds:

```json
{
    'code': 0, 
    'data': {
        'bids': [
            {'price': '2.923', 'quantity': '12'}, 
            {'price': '2.823', 'quantity': '12'}, 
            {'price': '2.813', 'quantity': '14'}
        ], 
        'asks': [
            {'price': '3.05', 'quantity': '3.48'}, 
            {'price': '3.31', 'quantity': '15'}, 
            {'price': '3.923', 'quantity': '15'}
            ]
        }, 
    'msg': 'ok'
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
| 参数名 | 参数类型 | 是否必须 |  描述  |
| :----: | :------: | :------: | :----: |
| symbol |  string  |    是    | 交易对 |

> Responds:

```json
{
    'code': 0, 
    'data': [{
        'amount': '0.918',
        'price': '2.04',
        'side': -1,
        'time': 1574942822160,
        'volume': '0.45'
        }]
}
```

### 返回字段

| 参数名 | 参数类型 |        描述         |
| :----: | :------: | :-----------------: |
| amount |  string  |       成交额        |
| price  |  string  |       成交价        |
|  side  | integer  | 成交方向，1买，-1卖 |
|  time  | integer  |        时间         |
| volume |  string  |       成交量        |


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
|   symbol   |  string  |    是    |                   交易对                    |
|   price    |  string  |    否    |          价格，如果是限价单，必填           |
|  quantity  |  string  |    是    |                    数量                     |
|    side    |   int    |    是    |               方向,1买，-1卖                |
| order_type |  string  |    否    | 买卖单类型,LIMIT限价单（默认）,MARKET市价单 |

<aside class="warning">无论买或卖，quantity都表示交易币，如BTC-USDT，quantity都代表eos的数量</aside>
<aside class="warning">现货新上交易对第一笔订单必须通过此接口下单，成交后用户才能下单</aside>

> Responds:

```json
{
    "code": 0,
    "msg": "ok",
    "data": {
        "order_id": "xxx",
        "trade_no": "xxx",
    },
}
```

### 返回字段

|  参数名  | 参数类型 |  描述  |
| :------: | :------: | :----: |
| order_id |  string  | 委托号 |
| trade_no |  string  | 流水号 |

## 撤销单个订单

```shell
现货交易

"https://api.bitda.com/open/v1/orders/cancel"
```

### HTTP请求
现货交易
- POST ` /open/v1/orders/cancel`

### 请求参数
|  参数名  | 参数类型 | 是否必须 |  描述  |
| :------: | :------: | :------: | :----: |
|  symbol  |  string  |    是    | 交易对 |
| order_id |  string  |    是    | 委托号 |
| trade_no |  string  |    是    | 流水号 |

> Responds:

```json
{
    "code": 0,
    "msg": "ok",
}
```

### 返回字段

| 参数名 | 参数类型 | 描述  |
| :----: | :------: | :---: |
无

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
    "msg": "ok",
}
```

### 返回字段

| 参数名 | 参数类型 | 描述  |
| :----: | :------: | :---: |
无

## 委托中列表

```shell
现货交易

"https://api.bitda.com/open/v1/orders/last"
```

### HTTP请求
现货交易
- GET ` /open/v1/orders/last`

### 请求参数
| 参数名 | 参数类型 | 是否必须 |  描述  |
| :----: | :------: | :------: | :----: |
| symbol |  string  |    是    | 交易对 |

> Responds:

```json
{
    'code': 0, 
    'data': [
        {
            'symbol': 'BTC-USDT',
            'order_id': '11574744030837944',
            'trade_no': '499016576021202015341',
            'price': '7900',
            'quantity': '1',
            'match_amt': '0',
            'match_qty': '0',
            'match_price': '',
            'side': -1,
            'order_type': 1,
            'create_at': 1574744151836
        }, 
    ], 
}
```

### 返回字段

|   参数名    | 参数类型 |         描述          |
| :---------: | :------: | :-------------------: |
|   symbol    |  string  |        交易对         |
|  order_id   |  string  |        订单ID         |
|  trade_no   |  string  |      订单流水号       |
|    price    |  string  |        委托价         |
|  quantity   |  string  |       委托数量        |
|  match_amt  |  string  |      已成交金额       |
|  match_qty  |  string  |      已成交数量       |
| match_price |  string  |       成交均价        |
|    side     |   int    |    方向，1买，-1卖    |
| order_type  |   int    | 订单类型,1限价，3市价 |
|  create_at  |   int    |       创建时间        |

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
|  symbol  |  string  |    是    |      交易对,如BTC-USDT       |
| pagenum  |   int    |    否    |             页码             |
| pagesize |   int    |    否    | 页大小,最小10, 最大50,默认20 |
|   side   |   int    |    否    |    方向，1买，-1卖，0所有    |
|  start   |   int    |    否    |         时间，时间戳         |
|   end    |   int    |    否    |       结束时间，时间戳       |

> Responds:

```json
{
    'code': 0, 
    'msg': 'ok',
    'data': {
        'count': 4, 
        'orders': [
            {
                'order_id': '11574744030837944',
                'trade_no': '499016576021202015341',
                'symbol': 'BTC-USDT',
                'price': '7900',
                'quantity': '1',
                'match_amt': '0',
                'match_qty': '0',
                'match_price': '',
                'side': -1,
                'order_type': 1,
                'status': 6,
                'create_at': 1574744151836
            }, 
        ]
    }, 
}
```

### 返回字段

|   参数名    | 参数类型 |                              描述                               |
| :---------: | :------: | :-------------------------------------------------------------: |
|  order_id   |  string  |                             订单id                              |
|  trade_no   |  string  |                           订单流水号                            |
|   symbol    |  string  |                             交易对                              |
|    price    |  string  |                             委托价                              |
|  quantity   |  string  |                            委托数量                             |
|  match_amt  |  string  |                           已成交金额                            |
|  match_qty  |  string  |                           已成交数量                            |
| match_price |  string  |                            成交均价                             |
|    side     |   int    |                         方向，1买，-1卖                         |
| order_type  |   int    |                      订单类型,1限价，3市价                      |
|   status    |   int    | 状态 2 委托中，3部分成交，4全部成交，5部分成交后撤消，6全部撤消 |
|  create_at  |   int    |                            创建时间                             |

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
|  参数名  | 参数类型 | 是否必须 |       描述        |
| :------: | :------: | :------: | :---------------: |
|  symbol  |  string  |    是    | 交易对,如BTC-USDT |
| order_id |  string  |    是    |    委托订单id     |

> Responds:

```json
{
    'code': 0, 
    'data': {
        'order_id': '11574751725833010',
        'trade_no': '499073202290421221116', 
        'symbol': 'BTC-USDT', 
        'price': '70000', 
        'quantity': '0.0001', 
        'match_amt': '7', 
        'match_qty': '0.0001',
        'match_price': '70000',  
        'fee': '0.0112',
        'side': -1, 
        'order_type': 1,
        'status': 4,
        'create_at': 1574922846832,
        'trades': [{
            'trade_id': "1",
            'amount': '7', 
            'price': '70000', 
            'quantity': '0.0001',
            'fee': '0.0112',  
            'time': 1574922846833
            }]
    }
}
```

### 返回字段

|   参数名    | 参数类型 |                              描述                               |
| :---------: | :------: | :-------------------------------------------------------------: |
|  order_id   |  string  |                             订单id                              |
|  trade_no   |  string  |                           订单流水号                            |
|   symbol    |  string  |                             交易对                              |
|    price    |  string  |                             委托价                              |
|  quantity   |  string  |                            委托数量                             |
|  match_amt  |  string  |                           已成交金额                            |
|  match_qty  |  string  |                           已成交数量                            |
| match_price |  string  |                            成交均价                             |
|     fee     |  string  |                             手续费                              |
|    side     |   int    |                         方向，1买，-1卖                         |
| order_type  |   int    |                      订单类型,1限价，3市价                      |
|   status    |   int    | 状态 2 委托中，3部分成交，4全部成交，5部分成交后撤消，6全部撤消 |
|  create_at  |   int    |                         委托单创建时间                          |
|   trades    |  object  |                          已成交数据[{                           |
|  trade_id   |  string  |                           成交记录id                            |
|   amount    |  string  |                      每条成交记录的成交额                       |
|    price    |  string  |                      每条成交记录的成交价                       |
|  quantity   |  string  |                      每条成交记录的成交量                       |
|     fee     |  string  |                      每条成交记录的手续费                       |
|    time     |   int    |                    每条成交记录的成交时间}]                     |

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
|  symbol  |  string  |    是    |      交易对,如BTC-USDT       |
| pagesize |   int    |    否    | 页大小,最小10, 最大50,默认10 |
| pagenum  |   int    |    否    |        页码，默认为1         |

> Responds:

```json
{
    'code': 0, 
    'data': {
        'count': 10,
        'trades': [
            {
              'amount': '11574751725833010',
              'fee': '499073202290421221116',
              'symbol': 'BTC-USDT',
              'price': '70000',
              'quantity': '0.0001',
              'side': '7',
              'time': 1574922846833,
              'trade_id': 1,
            }
          ]
    }
}
```

### 返回字段

|  参数名  | 参数类型 |           描述           |
| :------: | :------: | :----------------------: |
|  count   |   int    |       成交订单数量       |
|  trades  |  object  |       已成交数据[{       |
|  symbol  |  string  |          交易对          |
|   side   |   int    |     方向，1买，-1卖      |
| trade_id |   int    |        成交记录id        |
|  amount  |  string  |   每条成交记录的成交额   |
|  price   |  string  |   每条成交记录的成交价   |
| quantity |  string  |   每条成交记录的成交量   |
|   fee    |  string  |   每条成交记录的手续费   |
|   time   |   int    | 每条成交记录的成交时间}] |

## 获取用户某个交易对手续费

```shell
现货市场

"https://api.bitda.com/open/v1/fee-rate"
```

### HTTP请求
现货交易
- GET ` /open/v1/fee-rate`

### 请求参数
| 参数名 | 参数类型 | 是否必须 |  描述  |
| :----: | :------: | :------: | :----: |
| symbol |  string  |    是    | 交易对 |

> Responds:

```json
{
    'code': 0, 
    'data': {
        'maker_fee': '0.0001',
        'taker_fee': "0.0002"
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
    'code': 0, 
    'data': [
            {'amount': '1.586',
            'change': '-0.235462',
            'high': '3.05',
            'low': '3.05',
            'price': '0',
            'symbol': 'BTC-USDT',
            'amt_num': 4,
            'qty_num': 2,
            'volume': '0.52'
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
| symbol  |  string  |    交易对    |
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
| symbol |  string  |    是    | 交易对名称,如BTC-USDT |

> Responds:

```json
{
    'code': 0, 
    'data': {
        'bids': [
            {'price': '2.923', 'quantity': '12'}, 
            {'price': '2.823', 'quantity': '12'}, 
            {'price': '2.813', 'quantity': '14'}
        ], 
        'asks': [
            {'price': '3.05', 'quantity': '3.48'}, 
            {'price': '3.31', 'quantity': '15'}, 
            {'price': '3.923', 'quantity': '15'}
            ]
        }, 
    'msg': 'ok'
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
| symbol |  string  |    是    | 交易对名称,如BTC-USDT |

> Responds:

```json
{
    'code': 0, 
    'data': [{
        'amount': '0.918',
        'price': '2.04',
        'side': -1,
        'time': 1574942822160,
        'volume': '0.45'
        }]
}
```

### 返回字段

| 参数名 | 参数类型 |        描述         |
| :----: | :------: | :-----------------: |
| amount |  string  |       成交额        |
| price  |  string  |       成交价        |
|  side  | integer  | 成交方向，1买，-1卖 |
|  time  | integer  |        时间         |
| volume |  string  |       成交量        |

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
| 参数名 | 参数类型 | 是否必须 |              描述              |
| :----: | :------: | :------: | :----------------------------: |
| symbol |  string  |    是    |      交易对，如: BTC-USDT      |
|  type  |  string  |    是    | 类型1Min, 5Min, 15Min, 30Min等 |

> Responds:

```json
{
    'code': 0, 
    'data': [
        {
            'amount': '0',
            'close': '3.05',    
            'high': '3.05', 
            'low': '3.05', 
            'open': '3.05', 
            'time': 1571812440, 
            'volume': '0'
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
| volume |  string  |    成交量    |

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

print("> 获取open委托中")
# 现货交易
path = "/open/v1/orders/last"
obj = gen_sign(client_id, client_key)
obj.update({"symbol": "BTC-USDT"})
res = requests.get(host + path, params=obj)
print(ujson.loads(res.content))

print("> 获取单个订单成交明细")
# 现货交易
path = "/open/v1/orders/detail"
obj = gen_sign(client_id, client_key)
obj.update({"order_id": "11574751725833010", "symbol": "BTC-USDT"})
res = requests.get(host + path, params=obj)
print(ujson.loads(res.content))

print("> 分页获取成交明细")
# 现货交易
path = "/open/v1/orders/detailmore"
obj = gen_sign(client_id, client_key)
obj.update({"symbol": "BTC-USDT", "pagesize": 10, "pagenum": 1"})
res = requests.get(host + path, params=obj)
print(ujson.loads(res.content))

print("> 获取用户某个交易对手续费")
# 现货交易
path = "/open/v1/orders/fee-rate"
obj = gen_sign(client_id, client_key)
obj.update({"symbol": "BTC-USDT"})
res = requests.get(host + path, params=obj)
print(ujson.loads(res.content))

print("> 获取kline")
# 现货交易
path = "/open/v1/kline"
obj = gen_sign(client_id, client_key)
obj.update({"symbol": "BTC-USDT", "type": "1Min"})
res = requests.get(host + path, params=obj)
print(ujson.loads(res.content))

print("> 获取余额")
# 现货交易
path = "/open/v1/balance"
obj = gen_sign(client_id, client_key)
res = requests.get(host + path, params=obj)
print(ujson.loads(res.content))

print("> 获取最近成交记录")
# 现货交易
path = "/open/v1/tickers/trade"
obj = gen_sign(client_id, client_key)
obj.update({"symbol": "BTC-USDT"})
res = requests.get(host + path, params=obj)
print(ujson.loads(res.content))

print("> 下单")
# 现货交易
path = "/open/v1/orders/place"
obj = gen_sign(client_id, client_key)
obj.update({"symbol": "BTC-USDT", "price": "8850.21", "quantity": "0.1", "side": "1", "order_type": "LIMIT"})
res = requests.post(host + path, data=obj)
print(ujson.loads(res.content))
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
host = "wss://api.bitda.com/ws"
client_id = ""
client_key = ""

def login():
    ts = int(time.time())
    nonce = "abcdefg"
    obj = {"ts": ts, "nonce": nonce, "sign": "", "client_id": client_id, "op": "apilogin"}
    s = "client_id=%s&nonce=%s&ts=%s" % (client_id, nonce, ts)
    v = hmac.new(client_key.encode(), s.encode(), digestmod=hashlib.sha256)
    obj["sign"] = v.hexdigest()
    return obj

async def sub_topic(ws):
    sub = "depth:0:BTC-USDT"
    await ws.send(json.dumps({"op": "sub", "topic": sub}))

async def startup():
    print("start to connect %s..." % host)
    ws = await websockets.connect(host)

    obj = login()
    await ws.send(json.dumps(obj))
    await sub_topic(ws)

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

