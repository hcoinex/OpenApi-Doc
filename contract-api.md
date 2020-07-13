# 合约交易

Broker Open API的地址请见 [这里](endpoint.md)

## 关键参数解释说明:

### `side`

交易的方向

`BUY_OPEN`: 开多仓

`SELL_CLOSE`: 平多仓

`SELL_OPEN`: 开空仓

`BUY_CLOSE`: 平空仓

### `priceType`

价格类型

`INPUT`: 系统将会用你输入的价格来撮合订单。

`OPPONENT`: 订单会以对手盘最优价格撮合。

假设你开多10张合约，盘口最佳买价为10最佳卖价为11，你将会下10张价格为11的合约订单。如果盘口数量不足成交10张，剩下的将留在盘口。

`QUEUE`: 订单会以相同方向的最优价格撮合。

假设你开多10张合约，盘口最佳买价为10最佳卖价为11，你将会下10张价格为10的合约订单。假设盘口原来有5张10的买单，加上你下的单现在一共有15张10的买单。

`OVER`: 订单会以对手盘的最优价格 + 超价（浮动）撮合

假设你开多10张合约，盘口最佳买价为10最佳卖价为11（超价现在为3），你将会下10张（11+3=）14的买单。如果盘口数量不足成交10张，剩下的将留在盘口。

`MARKET`: 订单会以 最新成交价 * (1 ± 5%) 撮合

假设你开多10张合约，最新成交价为10，你将会下10张（10*1.05=)10.5的买单。

### `timeInForce`

时效单类型。

`GTC`: 一直有效直到撤销。订单会一直有效除非撤销。

`IOC`: 马上成交或者撤销。订单会在一个最佳可成交价执行尽量多的交易量, 此订单可能被部份执行,剩余的部份将会自动撤销。

`FOK`: 全部成交或者撤销。订单要么在一个最佳可成交价上全部成交，要么就会直接撤销。

`LIMIT_MAKER`: 如果订单会马上成交，订单会被撤销。

### `orderType`

订单类型

`LIMIT`: 订单会以一个给定（或者更好）的价格成交

`STOP`: 一旦价格到达triggerPrice（触发价），订单会被触发

### `频率限制类型(rateLimitType)`

*  REQUESTS_WEIGHT
*  ORDERS

### `频率限制区间`

* SECOND
* MINUTE
* DAY

比如:
```javascript
{
  "rateLimitType": "ORDERS",
  "interval": "SECOND",
  "limit": 20
}
```

## 公共接口

### `brokerInfo`

获取当前broker的交易规则和合约symbol的信息（精度单位等信息），包括合约的风险限额和乘数等信息。

#### **Request Weight:**

0

#### **Request Url:**

```bash
GET /openapi/v1/brokerInfo
```

#### **Parameters：**

| 名称 | 类型 | 是否强制 | 默认 | 描述 |
| :--- | :--- | :--- | :--- | :--- |
| `type` | string | `NO` |  | 交易类型，支持的类型现为token（币币）、options（期权）、contracts（合约）。如果没有发送此参数，所有交易类型的symbol信息都会被返回。 |

#### **Response:**

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `timezone` | string | `UTC` | 时间戳的时区。 |
| `serverTime` | long | `1554887652929` | 返回现在的服务器时间戳（毫秒级） |

在rateLimits信息组里: 下单api的请求限制将会被展示。

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `rateLimitType` | string | `ORDERS` | 速度限制类型 |
| `interval` | string | `SECOND` | 速度限制区间 |
| `limit` | string | `1500` | 速度限制区间价值 |

在contracts信息组里: 所有当前券商正在交易的合约的信息将会被返回：

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `symbol` | string | `BTCUSD` | 合约名称 |
| `status` | string | `TRADING` | 合约状态 |
| `baseAsset` | string | `BTCUSD` | 基础资产。对于合约来说，合约本身就是基础资产。 |
| `baseAssetPrecision` | float | `0.001` | 基础资产（合约数量）的精度 |
| `quoteAsset` | string | `USDT` | 定价资产。对于合约来说，这个是合约是以什么来结算的。 |
| `quoteAssetPrecision` | float | `0.001` | 定价资产（合约价格）的精度。 |
| `inverse` | bool | `true` | 合约是否为反向合约（true=是反向合约，false=是正向合约）。 |
| `index` | string | `BTCUSDT` | 标的指数的名称。标的指数实时价格可在index端点访问得到。比如BTC-PERP-REV使用BTCUSDT为标的指数，那么可以在index端点寻找BTCUSDT的实时价格。 |
| `contractMultiplier` | string | `true` | 合约的乘数。 |
| `icebergAllowed` | string | `false` | 是否支持冰山订单。 |

在 contracts的filters信息组里:

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `filterType` | string | `PRICE_FILTER` | 过滤器类型。 |
| `minPrice` | float | `0.001` | 允许的最小价格。 |
| `maxPrice` | float | `100000.00000000` | 允许的最大价格。 |
| `tickSize` | float | `0.001` | 合约价格的精度。 |
| `minQty` | float | `0.01` | 允许的最小数量。 |
| `maxQty` | float | `100000.00000000` | 允许的最大数量。 |
| `stepSize` | float | `0.001` | 合约数量的精度。 |
| `minNotional` | float | `1` | 最小交易额限制(数量 * 价格) |

在riskLimits信息组里:

| name | type | example | description |
| :--- | :--- | :--- | :--- |
| `quantity` | float | `100` | 仓位小于此数且大于前一个档位的quantity的需要参照以下要求。 |
| `initialMargin` | float | `0.1` | 初始保证金率。 |
| `maintMargin` | float | `0.03` | 最小维持保证金率。 |

#### **Example:**

```javascript
{
  "timezone":"UTC",
  "serverTime":"1570701444309",
  "brokerFilters":[],
  "rateLimits":[
    {
      "rateLimitType":"ORDERS",
      "interval":"SECOND",
      "limit":20
    },
    {"rateLimitType":"ORDERS",
    "interval":"DAY",
    "limit":350000
  },{
    "rateLimitType":"REQUEST_WEIGHT",
    "interval":"MINUTE",
    "limit":1500
  }],
  "contracts":[
    {
      "filters":[
        {"minPrice":"0.01",
        "maxPrice":"100000.00000000",
        "tickSize":"0.01",
        "filterType":"PRICE_FILTER"
      },
      {
        "minQty":"1",
        "maxQty":"100000.00000000",
        "stepSize":"1",
        "filterType":"LOT_SIZE"
      },{
        "minNotional":"0.000001",
        "filterType":"MIN_NOTIONAL"
      }],
      "exchangeId":"301",
      "symbol":"BTC-PERP-REV",
      "symbolName":"BTC-PERP-REV",
      "status":"TRADING",
      "baseAsset":"BTC-PERP-REV",
      "baseAssetPrecision":"1",
      "quoteAsset":"USDT",
      "quoteAssetPrecision":"0.01",
      "icebergAllowed":false,
      "inverse":true,
      "index":"BTCUSDT",
      "marginToken":"TBTC",
      "marginPrecision":"0.00000001",
      "contractMultiplier":"1.0",
      "riskLimits":[
        {
          "riskLimitId":"200000001",
          "quantity":"1000000.0",
          "initialMargin":"0.01",
          "maintMargin":"0.005"
        },
        {
          "riskLimitId":"200000002",
          "quantity":"2000000.0",
          "initialMargin":"0.02",
          "maintMargin":"0.01"
        },
        {
          "riskLimitId":"200000003",
          "quantity":"3000000.0",
          "initialMargin":"0.03",
          "maintMargin":"0.015"
        },
        {
          "riskLimitId":"200000004",
          "quantity":"4000000.0",
          "initialMargin":"0.04",
          "maintMargin":"0.02"
        }
      ]
    }
  ]
}
```

### `insurance` **\(PENDING\)**

Get current insurance funding.

#### **Request Weight:**

0

#### **Request Url:**

```bash
GET /openapi/contract/v1/insurance
```

#### **Parameters：**

| name | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `NO` |  | Input specific symbol to return the corresponding records. If not entered, records for all symbols will be returned. |
| `fromId` | long | `NO` |  | pagination,    return record which id &lt; fromId |
| `toId` | long | `NO` |  | pagination, return record which id &gt; toId. If toId is given, toId cannot be 0. |
| `limit` | integer | `NO` | 20 | Number of entries returned. |

#### **Response:**

| name | type | example | description |
| :--- | :--- | :--- | :--- |
| `id` | long | `1552272184833` | The ID of the record. |
| `timestamp` | long | `155227218483` | current server timestamp. |
| `value` | float | `23.23` | Balance of the insurance fund. |
| `unit` | string | `BTC` | Unit of the balance.. |

```javascript
{
  "BTC":[
    {
      "id": 1552272184833,
      "timestamp": 155227218483,
      "value": 23.23,
      "unit": "BTC"
    },...
  ],...
}
```

### `index`

标的指数价格

#### **Request Weight:**

0

#### **Request Url:**

```bash
GET /openapi/quote/v1/contract/index
```

#### **Parameters：**

| 名称 | 类型 | 是否强制 | 默认 | 描述 |
| :--- | :--- | :--- | :--- | :--- |
| symbol | string | `NO` |  | 标的指数名称。如果这个没有发送，所有标的指数的价格都会被返回。 |

will be returned.

#### **Response:**

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `index` | float | `8342.73` | 标的指数的价格。 |
| `EDP` | float | `8432.32` | 标的指数的EDP(预估交割价，过去10分钟指数价格的平均值）。 |

#### **Example:**

```javascript
{
  "index":{
    "BTCUSDT":"8243.21666667",
    "OKBUSDT":"1.482",
    "BNBUSDT":"31.2658",
    "HTUSDT":"3.1209",...
    },
  "edp":{
    "BTCUSDT":"8258.98505556",
    "OKBUSDT":"1.48578333",
    "BNBUSDT":"31.48741917",
    "HTUSDT":"3.14308",...
  }
}
```

### `fundingRate`

获取当前资金费率 (历史资金费率正在建设)

#### **Request Weight:**

0

#### **Request Url:**

```bash
GET /openapi/contract/v1/fundingRate
```

#### **Parameters：**

| 名称 | 类型 | 是否强制 | 默认 | 描述 |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `NO` |  | 合约名称。如果没有发送该参数，所有合约的资金费率都会被返回。 |
| `state` | string | `NO` | `current` | 获取current（当前）或者past（历史）的资金费率。 |
| `from` | long | `NO` |  | 开始时间戳。 |
| `to` | long | `NO` |  | 结束时间戳。 |
| `limit` | integer | `NO` | `20` | 返回条数。 |

#### **Response:**

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `symbol` | string | `BTCUSD` | 合约名称 |
| `intervalStart` | long | `1554710400000` | 本次结算开始时间。 |
| `intervalEnd` | long | `1554710400000` | 本次结算结束时间。 |
| `rate` | float | `0.00321` | 该次结算资金费率。 |
| `index` | float | `10076.34` | 结算时的指数价格。 |

#### **Example:**

```javascript
[
  {
    'symbol': 'BTC-PERP-REV',
    'intervalStart': '1570708800000',
    'intervalEnd': '1570737600000',
    'rate': '-0.000048567445464006'
  },...
]
```

### `depth`

获取当前订单簿的数据。

#### **Request Weight:**

根据数量会不一样，请求数量越多，重量越大:

| 数量 | 请求重量 |
| :--- | :--- |
| 5, 10, 20, 50, 100 | 1 |
| 500 | 5 |
| 1000 | 10 |

#### **Request Url:**

```text
GET /openapi/quote/v1/contract/depth
```

#### **Parameters:**

| 名称 | 类型 | 是否强制 | 默认 | 描述 |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `YES` |  | 用来获取订单簿的合约名称。 |
| `limit` | integer | `NO` | `100` \(max = 100\) | 返回bids和asks的数量 |

#### **Response:**

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `time` | long | `1550829103981` | 当前时间（Unix Timestamp，毫秒ms） |
| `bids` | list | \(如下\) | 所有bid的价格和数量信息，最优bid价格由上到下排列。 |
| `asks` | list | \(如下\) | 所有ask的价格和数量信息，最优ask价格由上到下排列。 |

bids和asks所对应的信息组代表了订单簿的所有价格以及价格对应数量的信息，由最优价格从上到下排列。

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `''` | float | `123.10` | 价格 |
| `''` | float | `300` | 当前价格对应的数量 |

#### **Example:**

```javascript
{
  "time": 1555049455783,
  "bids": [
   ["78.82", "0.526"],//[Price, Quantity]
   ["77.24", "1.22"],
   ["76.65", "1.043"],
   ["76.58", "1.34"],
   ["75.67", "1.52"],
   ["75.12", "0.635"],
   ["75.02", "0.72"],
   ["75.01", "0.672"],
   ["73.73", "1.282"],
   ["73.58", "1.116"],
   ["73.45", "0.471"],
   ["73.44", "0.483"],
   ["72.32", "0.383"],
   ["72.26", "1.283"],
   ["72.11", "0.703"],
   ["70.61", "0.454"]],
   "asks": [
     ["122.96", "0.381"],//[Price, Quantity]
     ["144.46", "1"],
     ["155.55", "0.065"],
     ["160.16", "0.052"],
     ["200", "0.775"],
     ["249", "0.17"],
     ["250", "1"],
     ["300", "1"],
     ["400", "1"],
     ["499", "1"]]
   }
```

### `trades`

获取某个合约最近成交订单的信息。

#### **Request Weight:**

1

#### **Request URL:**

```text
GET /openapi/quote/v1/contract/trades
```

#### **Parameters：**

| 名称 | 类型 | 是否强制 | 默认 | 描述 |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `YES` |  | 合约名称 |
| `limit` | integer | `NO` \(clamped to max 1000\) | `100` | 返回成交订单的数量 |

#### **Response:**

| 名称 | 类型 | 例子 | 描述 |
| :--- | :--- | :--- | :--- |
| `price` | float | `0.055` | The price of the trade |
| `time` | long | `1537797044116` | Current timestamp \(ms\) |
| `qty` | float | `5` | The quantity traded |
| `isBuyerMaker` | string | `true` | Maker or taker of the trade. `true`= maker, `false` = taker |

#### **Example:**

```javascript
[
  {
    "price": "1.21",
    "time": 1555034474064,
    "qty": "0.725",
    "isBuyerMaker": False
  },...
]
```

### `klines`

Retrieves the kline information \(open, high, trade volume, etc.\) for a specific contract.

#### **Request Weight:**

1

#### **Request URL:**

```text
GET /openapi/quote/v1/contract/klines
```

#### **Parameters：**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `YES` |  | Name of the contract. |
| `interval` | string | `YES` |  | Interval of the kline. Possible values include: `1m`,`5m`,`15m`,`30m`,`1h`,`1d`,`1w`,`1M` |
| `limit` | integer | `NO` | `1000` | Number of entries returned. Max is capped at 1000. |
| `to` | integer | `NO` |  | timestamp of the last datapoint |

#### **Response:**

| name | type | example | description |
| :--- | :--- | :--- | :--- |
| `''` | long | `1538728740000` | Open Time |
| `''` | float | `36.00000` | Open |
| `''` | float | `36.00000` | High |
| `''` | float | `36.00000` | Low |
| `''` | float | `36.00000` | Close |
| `''` | float | `148976.11427815` | Trade volume amount |
| `''` | long | `1538728740000` | Close time |
| `''` | float | `2434.19055334` | Quote asset volume |
| `''` | integer | `308` | Number of trades |
| `''` | float | `1756.87402397` | Taker buy base asset volume |
| `''` | float | `28.46694368` | Taker buy quote asset volume |

#### **Example:**

```javascript
[
  [
    1538728740000, //'opentime'
    "36.000000000000000000", //'open'
    "36.000000000000000000", //'high'
    "36.000000000000000000", //'low':
    "36.000000000000000000", //'close'
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368"       // Taker buy quote asset volume
  ],...
]
```

`base asset` means the quantity is expressed as the amount of contracts that were received by the buyer.

`quote asset` means the amount of tokens paid to acquire the contracts.

## Private API

### `order`

Places order for a contract. This API endpoint requires your request to be signed.

#### **Request Weight:**

1

#### **Request URL:**

```bash
POST /openapi/contract/v1/order
```

#### **Parameters：**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `YES` |  | Name of the contract. |
| `side` | string | `YES` |  | Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`. |
| `orderType` | string | `YES` |  | The order type, possible types: `LIMIT`, `STOP` |
| `quantity` | float | `YES` |  | The number of contracts to buy. |
| `leverage` | float | `YES`\(**NOT REQUIRED** for \*\_CLOSE" orders\) |  | Leverage of the order. |
| `price` | float | `NO`. **REQUIRED** for \(`LIMIT` & `INPUT`\) orders |  | Price of the order |
| `priceType` | string | `NO` | `INPUT` | The price type, possible types include: `INPUT`, `OPPONENT`, `QUEUE`, `OVER`, and `MARKET`. |
| `triggerPrice` | float | `NO`. **REQUIRED** for `STOP` orders. |  | The price at which the trigger order will be executed. |
| `timeInForce` | string | `NO` | `GTC` | Time in force for `LIMIT` orders. Possible values include `GTC`,`FOK`,`IOC`,`LIMIT_MAKER`. |
| `clientOrderId` | string/long | `YES` |  | A unique ID of the order \(user defined\) |

**NOTE** For **Market Orders**, you need to set `orderType` as **`LIMIT`** **AND** `priceType` as **`MARKET`**.

You can get contracts' price, quantity precision configuration data in the `brokerInfo` endpoint.

Note: if your balance does not meet the margin requirement \(which is the minimum margin requirement + open position fee + close position fee\), "_insufficient balance_" error message will be returned.

For detailed information regarding various _price types_ and _order types_. Please refer to the explanation section in the end.

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `time` | long | `1570759718825` | Timestamp when the order is created. |
| `updateTime` | long | `1551062936784` | Last time this order was updated |
| `orderId` | integer | `469961015902208000` | ID of the order. |
| `clientOrderId` | string | `213443` | A unique ID of the order. |
| `symbol` | string | `BTC-PERP-REV` | Name of the contract. |
| `price` | float | `8200` | Price of the order. |
| `leverage` | float | `4` | Leverage of the order. |
| `origQty` | float | `1.01` | Quantity ordered |
| `executedQty` | float | `1.01` | Quantity of orders that has been executed |
| `avgPrice` | float | `4754.24` | Average price of filled orders. |
| `marginLocked` | float | `200` | Amount of margin locked for this order. This includes the actually margin needed plus fees to open and close the position. |
| `orderType` | string | `YES` | The order type, possible types: `LIMIT`, `STOP` |
| `priceType` | string | `INPUT` | The price type. Possible values include `INPUT`, `OPPONENT`, `QUEUE`, `OVER`, and `MARKET`. |
| `side` | string | `BUY` | Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`. |
| `status` | string | `NEW` | The state of the order.Possible values include `NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, and `REJECTED`. |
| `timeInForce` | string | `GTC` | Time in force. Possible values include `GTC`,`FOK`,`IOC`, and `LIMIT_MAKER` |
| `fees` |  |  | Fees incurred for this order. |

#### **Example:**

```javascript
{
  'time': '1570759718825',
  'updateTime': '0',
  'orderId': '469961015902208000',
  'clientOrderId': '6423344174',
  'symbol': 'BTC-PERP-REV',
  'price': '8200',
  'leverage': '12.08',
  'origQty': '5',
  'executedQty': '0',
  'avgPrice': '0',
  'marginLocked': '0.00005047',
  'orderType': 'LIMIT',
  'side': 'BUY_OPEN',
  'fees': [],
  'timeInForce': 'GTC',
  'status': 'NEW',
  'priceType': 'INPUT'
}
```

### `cancel`

Cancels an order, specified by `orderId` or `clientOrderId`. This API endpoint requires your request to be signed.

#### **Request Weight:**

1

#### **Request Url:**

```bash
DELETE /openapi/contract/v1/order/cancel
```

#### **Parameter:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `orderId` | integer | `NO` |  | The order ID of the order to be canceled |
| `clientOrderId` | string | `NO` |  | Unique client customized ID of the order. |
| `orderType` | string | `YES` |  | The order type, possible types: `LIMIT` and `STOP`. |

One **MUST** be provided for either of these two parameters.

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `time` | long | `1551062936784` | Timestamp when the order is created. |
| `updateTime` | long | `1551062936784` | Last time this order was updated |
| `orderId` | integer | `891` | ID of the order. |
| `clientOrderId` | string | `213443` | A unique ID of the order. |
| `symbol` | string | `BTC-PERP-REV` | Name of the contract. |
| `price` | float | `4765.29` | Price of the order. |
| `leverage` | float | `4` | Leverage of the order. |
| `origQty` | float | `1.01` | Quantity ordered |
| `executedQty` | float | `1.01` | Quantity of orders that has been executed |
| `avgPrice` | float | `4754.24` | Average price of filled orders. |
| `marginLocked` | float | `200` | Amount of margin locked for this order. This includes the actually margin needed plus fees to open and close the position. |
| `orderType` | string | `LIMIT` | The order type, possible types: `LIMIT` and `STOP`. |
| `priceType` | string | `INPUT` | The price type. Possible values include `INPUT`, `OPPONENT`, `QUEUE`, `OVER`, and `MARKET`. |
| `side` | string | `BUY_OPEN` | Direction of the order. Possible values include `BUY_OPEN`, `BUY_CLOSE`, `SELL_OPEN` and `SELL_CLOSE` |
| `status` | string | `NEW` | The state of the order.Possible values include `NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, and `REJECTED`. |
| `timeInForce` | string | `GTC` | Time in force. Possible values include `GTC`,`FOK`,`IOC`, and `LIMIT_MAKER` |
| `fees` |  |  | Fees incurred for this order. |

In the `fees` field:

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `feeToken` | string | `USDT` | Fee token kind. |
| `fee` | float | `0` | Actual transaction fees occurred. |

#### **Example:**

```javascript
{
  'time': '1570759718825',
  'updateTime': '0',
  'orderId': '469961015902208000',
  'clientOrderId': '6423344174',
  'symbol': 'BTC-PERP-REV',
  'price': '8200',
  'leverage': '12.08',
  'origQty': '5',
  'executedQty': '0',
  'avgPrice': '0',
  'marginLocked': '0',
  'orderType': 'LIMIT',
  'side': 'BUY_OPEN',
  'fees': [],
  'timeInForce': 'GTC',
  'status': 'CANCELED',
  'priceType': 'INPUT' //status will always be `CANCELED` for cancel request
}
```

#### `batchCancel`

Cancel orders en masse. \(**PENDING: batch cancel for STOP orders**\)

#### **Request Weight:**

1

#### **Request Url:**

```bash
DELETE /openapi/contract/v1/order/batchCancel
```

#### **Parameter:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string/list | `NO` |  | The symbol ids of the cancel orders |

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `message` | string | `success` | The message response of the cancel request. |
| `timestamp` | long | `1541161088303` | The timestamp when the response is returned. |

#### **Example:**

```javascript
{
  'message':'success',
  'timestamp':1541161088303
}
```

#### `openOrders`

Retrieves open orders. This API endpoint requires your request to be signed.

#### **Request Weight:**

1

#### **Request Url:**

```bash
GET /openapi/contract/v1/openOrders
```

#### **Parameters:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `NO` |  | Symbol to return open orders for. If not sent, orders of all contracts will be returned. |
| `orderId` | integer | `NO` |  | Order ID |
| `side` | string | `NO` |  | Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`. |
| `orderType` | string | `YES` |  | The order type, possible types: `LIMIT` and `STOP`. |
| `limit` | integer | `NO` | `20` | Number of entries to return. |

If `orderId` is set, it will get orders &lt; that `orderId`. Otherwise most recent orders are returned.

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `time` | long | `1551062936784` | Timestamp when the order is created. |
| `updateTime` | long | `1551062936784` | Last time this order was updated |
| `orderId` | integer | `891` | ID of the order. |
| `clientOrderId` | string | `213443` | A unique ID of the order. |
| `symbol` | string | `BTC-PERP-REV` | Name of the contracts. |
| `price` | float | `4765.29` | Price of the order. |
| `leverage` | float | `4` | Leverage of the order. |
| `origQty` | float | `1.01` | Quantity ordered |
| `executedQty` | float | `1.01` | Quantity of orders that has been executed |
| `avgPrice` | float | `4754.24` | Average price of filled orders. |
| `marginLocked` | float | `200` | Amount of margin locked for this order. This includes the actually margin needed plus fees to open and close the position. |
| `orderType` | string | `LIMIT` | The order type, possible types: `LIMIT` and `STOP`. |
| `priceType` | string | `INPUT` | The price type. Possible values include `INPUT`, `OPPONENT`, `QUEUE`, `OVER`, and `MARKET`. |
| `side` | string | `BUY_OPEN` | Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`. |
| `status` | string | `NEW` | The state of the order.Possible values include `NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, and `REJECTED`. |
| `timeInForce` | string | `GTC` | Time in force. Possible values include `GTC`,`FOK`,`IOC`, and `LIMIT_MAKER`. |
| `fees` |  |  | Fees incurred for this order. |

#### **Example:**

```javascript
[
  {
    'time': '1570760254539',
    'updateTime': '0',
    'orderId': '469965509788581888',
    'clientOrderId': '1570760253946',
    'symbol': 'BTC-PERP-REV',
    'price': '8502.34',
    'leverage': '20',
    'origQty': '222',
    'executedQty': '0',
    'avgPrice': '0',
    'marginLocked': '0.00130552',
    'orderType': 'LIMIT',
    'side': 'BUY_OPEN',
    'fees': [],
    'timeInForce': 'GTC',
    'status': 'NEW',
    'priceType': 'INPUT'
  },...
]
```

### `historyOrders`

Retrieves history of orders that have been partially or fully filled or canceled. This API endpoint requires your request to be signed.

#### **Request Weight:**

1

#### **Request Url:**

```bash
GET /openapi/contract/v1/historyOrders
```

#### **Parameters:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `NO` |  | Symbol to return open orders for. If not sent, orders of all contracts will be returned. |
| `orderId` | integer | `NO` |  | Order ID |
| `side` | string | `NO` |  | Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`. |
| `orderType` | string | `YES` |  | The order type, possible types: `LIMIT`, `STOP` |
| `limit` | integer | `NO` | `20` | Number of entries to return. |

If `orderId` is set, it will get orders &lt; that `orderId`. Otherwise most recent orders are returned.

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `time` | long | `1551062936784` | Timestamp when the order is created. |
| `updateTime` | long | `1551062936784` | Last time this order was updated |
| `orderId` | integer | `891` | ID of the order. |
| `clientOrderId` | string | `213443` | A unique ID of the order. |
| `symbol` | string | `BTC-PERP-REV` | Name of the contracts. |
| `price` | float | `4765.29` | Price of the order. |
| `leverage` | float | `4` | Leverage of the order. |
| `origQty` | float | `1.01` | Quantity ordered |
| `executedQty` | float | `1.01` | Quantity of orders that has been executed |
| `avgPrice` | float | `4754.24` | Average price of filled orders. |
| `marginLocked` | float | `200` | Amount of margin locked for this order. This includes the actually margin needed plus fees to open and close the position. |
| `orderType` | string | `LIMIT` | The order type, possible types: `LIMIT` and `STOP`. |
| `priceType` | string | `INPUT` | The price type. Possible values include `INPUT`, `OPPONENT`, `QUEUE`, `OVER`, and `MARKET`. |
| `side` | string | `BUY_OPEN` | Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`. |
| `status` | string | `NEW` | The state of the order.Possible values include `NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, and `REJECTED`. |
| `timeInForce` | string | `GTC` | Time in force. Possible values include `GTC`,`FOK`,`IOC`, and `LIMIT_MAKER`. |
| `fees` |  |  | Fees incurred for this order. |

#### **Example:**

```javascript
[
  {
    'time': '1570759718825',
    'updateTime': '0',
    'orderId': '469961015902208000',
    'clientOrderId': '6423344174',
    'symbol': 'BTC-PERP-REV',
    'price': '8200',
    'leverage': '12.08',
    'origQty': '5',
    'executedQty': '0',
    'avgPrice': '0',
    'marginLocked': '0',
    'orderType': 'LIMIT',
    'side': 'BUY_OPEN',
    'fees': [],
    'timeInForce': 'GTC',
    'status': 'CANCELED',
    'priceType': 'INPUT'
  },...
]
```

### `getOrder`

Get details on a specific order, regardless of order state.

#### **Request Weight:**

1

#### **Request Url:**

```bash
GET /openapi/contract/v1/getOrder
```

#### **Parameters:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `orderId` | integer | `NO` |  | Order ID. |
| `clientOrderId` | string | `NO` |  | Unique client customized ID of the order. |
| `orderType` | string | `YES` |  | The order type, possible types: `LIMIT`, `STOP` |

**Either `orderId` or `clientOrderId` must be sent**

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `time` | long | `1551062936784` | Timestamp when the order is created. |
| `updateTime` | long | `1551062936784` | Last time this order was updated |
| `orderId` | integer | `891` | ID of the order. |
| `clientOrderId` | string | `213443` | A unique ID of the order. |
| `symbol` | string | `BTC-PERP-REV` | Name of the contracts. |
| `price` | float | `4765.29` | Price of the order. |
| `leverage` | float | `4` | Leverage of the order. |
| `origQty` | float | `1.01` | Quantity ordered |
| `executedQty` | float | `1.01` | Quantity of orders that has been executed |
| `avgPrice` | float | `4754.24` | Average price of filled orders. |
| `marginLocked` | float | `200` | Amount of margin locked for this order. This includes the actually margin needed plus fees to open and close the position. |
| `orderType` | string | `LIMIT` | The order type, possible types: `LIMIT` and `STOP`. |
| `priceType` | string | `INPUT` | The price type. Possible values include `INPUT`, `OPPONENT`, `QUEUE`, `OVER`, and `MARKET`. |
| `side` | string | `BUY_OPEN` | Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`. |
| `status` | string | `NEW` | The state of the order.Possible values include `NEW`, `PARTIALLY_FILLED`, `FILLED`, `CANCELED`, and `REJECTED`. |
| `timeInForce` | string | `GTC` | Time in force. Possible values include `GTC`,`FOK`,`IOC`, and `LIMIT_MAKER`. |
| `fees` |  |  | Fees incurred for this order. |

#### **Example:**

```javascript
{
  'time': '1570760254539',
  'updateTime': '0',
  'orderId': '469965509788581888',
  'clientOrderId': '1570760253946',
  'symbol': 'BTC-PERP-REV',
  'price': '8502.34',
  'leverage': '20',
  'origQty': '222',
  'executedQty': '0',
  'avgPrice': '0',
  'marginLocked': '0.00130552',
  'orderType': 'LIMIT',
  'side': 'BUY_OPEN',
  'fees': [],
  'timeInForce': 'GTC',
  'status': 'NEW',
  'priceType': 'INPUT'
}
```

### `myTrades`

Retrieve the trade history of the account. This API endpoint requires your request to be signed.

#### **Request Weight:**

1

#### **Request Url:**

```bash
GET /openapi/contract/v1/myTrades
```

#### **Parameters:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `NO` |  | Name of the contract. If not sent, trades for all symbols will be returned. |
| `limit` | integer | `NO` | `20` | The number of trades returned \(clamped to max 1000\) |
| `side` | string | `NO` |  | Direction of the order. |
| `fromId` | integer | `NO` |  | TradeId to fetch from. |
| `toId` | integer | `NO` |  | TradeId to fetch to. |

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `time` | long | `1551062936784` | Timestamp when the order is created. |
| `tradeId` | long | `49366` | The ID for the trade |
| `orderId` | integer | `891` | ID of the order. |
| `matchOrderId` | long | `630491432` | ID of the match order. |
| `symbolId` | string | `BTC-PERP-REV` | Name of the contract. |
| `price` | float | `4765.29` | Price of the trade. |
| `quantity` | float | `1.01` | Quantity of the trade. |
| `feeTokenId` | string | `USDT` | Fee token name. |
| `fee` |  |  | Fee of the trade. |
| `side` | string | `BUY` | Direction of the order. Possible values include `BUY_OPEN`, `SELL_OPEN`, `BUY_CLOSE`, and `SELL_CLOSE`. |
| `orderType` | string | `LIMIT` | The order type, possible types: LIMIT, MARKET |

#### **Example:**

```javascript
[
  {
    'time': '1570760582848',
    'tradeId': '469968263995080704',
    'orderId': '469968263793737728',
    "matchOrderId": 436002617267469062,
    'accountId': '456552319339779840',
    'symbolId': 'BTC-PERP-REV',
    'price': '8531.17',
    'quantity': '100',
    'feeTokenId': 'TBTC',
    'fee': '0.00000586',
    'type': 'LIMIT',
    'side': 'BUY_OPEN'
  },...
]
```

### `positions`

Retrieves current positions. This API endpoint requires your request to be signed.

#### **Request Weight:**

1

#### **Request Url:**

```bash
GET /openapi/contract/v1/positions
```

#### **Parameters:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `NO` |  | Name of the contract. If not sent, positions for all contracts will be returned. |
| `side` | string | `NO` |  | `LONG` or `SHORT`. Direction of the position. If not sent, positions for both sides will be returned. |

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `symbol` | string | `BTC-PERP-REV` | Name of the contract. |
| `side` | string | `LONG` | Position side. |
| `avgPrice` | float | `100` | Average price for opening the position. |
| `position` | float | `20` | Amount of contracts opened |
| `available` | float | `15` | Amount of contracts available to close. |
| `leverage` | float | `5` | Leverage of the position |
| `lastPrice` | float | `100` | Last trade price of the symbol. |
| `positionValue` | float | `2000` | Current position value. |
| `flp` | float | `80` | Forced liquidation price. |
| `margin` | float | `20` | Margin for this position. |
| `marginRate` | float | `0.2` | Margin rate for current position. |
| `unrealizedPnL` | float | `0.0` | Unrealized profit and loss for current position held. |
| `profitRate` | float | `0.0000333` | Rate of return for the position. |
| `realizedPnL` | float | `6.8` | Cumulative realized profit and loss for this `symbol`. |

#### **Example:**

```javascript
[
  {
    'symbol': 'BTC-PERP-REV',
    'side': 'LONG',
    'avgPrice': '8183.11',
    'position': '1100',
    'available': '1100',
    'leverage': '6',
    'lastPrice': '8572.53',
    'positionValue': '0.12833346',
    'flp': '7523.65',
    'margin': '0.01251335',
    'marginRate': '0.14',
    'unrealizedPnL': '0.00608975',
    'profitRate': '0.0000333',
    'realizedPnL': '-0.00006721'
  },...
]
```

### `account`

This endpoint is used to retrieve contract account balance. This endpoint requires you to be signed.

#### **Request Weight:**

1

#### **Request Url:**

```bash
GET  /openapi/contract/v1/account
```

#### **Parameters:**

None

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `total` | float | `131.06671401` | Total balance. |
| `availableMargin` | float | `131.0545541` | Available margin for use. |
| `positionMargin` | float | `0.01215991` | Margin for positions. |
| `orderMargin` | float | `0` | Margin locked for open orders. |

#### **Example:**

```javascript
{
  "TBTC": {
    "total":"131.06671401",
    "availableMargin":"131.0545541",
    "positionMargin":"0.01215991",
    "orderMargin":"0"
  },...
}
```

### `modifyMargin`

Modify margin for a specific symbol. This endpoint requires your position to be signed.

#### **Request Weight:**

1

#### **Request Url:**

```bash
POST  /openapi/contract/v1/modifyMargin
```

#### **Parameters:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `symbol` | string | `YES` |  | The symbol's margin to be modified. |
| `side` | string | `YES` |  | LONG or SHORT. Direction of the position. |
| `amount` | float | `YES` |  | Amount of margin to be added \(Positive Value\) or removed \(Negative Value\). Please note that this amount refers to the underlying quote asset of the asset. |

#### **Response:**

| Name | type | example | description |
| :--- | :--- | :--- | :--- |
| `symbol` | string | `BTC-PERP-REV` | The name of the contract. |
| `margin` | float | `12.3` | Updated margin for the symbol. |
| `timestamp` | long | `1541161088303` | Updated timestamp |

#### **Example:**

```javascript
{
  'symbol':'BTC-PERP-REV',
  'margin': 15,
  'timestamp': 1541161088303
}
```

### `transfer` **\(PENDING\)**

This endpoint is used to transfer funds across different accounts. This endpoint requires you to be signed.

#### **Request Weight:**

1

#### **Request Url:**

```bash
POST  /openapi/v1/transfer
```

#### **Parameters:**

| Parameter | type | required | default | description |
| :--- | :--- | :--- | :--- | :--- |
| `from` | string | `YES` |  | Transfer from which account. |
| `to` | string | `YES` |  | Transfer to which account. |
| `currency` | string | `YES` |  | The intended currency to transfer. \(`USDT`, `BTC`, etc.\) |
| `amount` | float | `YES` |  | Amount of currency to transfer. |

Currently supports transferring assets across `wallet`, `option`, and `contract` accounts.

#### **Response:**

A confirmation message will be returned.

#### **Example:**

```javascript
{
  'message': 'success',
  'timestamp': 1541161088303
}
```
