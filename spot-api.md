# 币币交易

## 术语解释

* `base asset` 指的是symbol的quantity（即数量）。
* `quote asset` 指的是symbol的price（即价格）。

## ENUM 定义

**Symbol 状态:**

* TRADING - 交易中
* HALT - 终止
* BREAK - 断开

**Symbol 类型:**

* SPOT - 现货

**资产类型:**

* CASH - 现金
* MARGIN - 保证金

**订单状态:**

* NEW - 新订单，暂无成交
* PARTIALLY\_FILLED - 部分成交
* FILLED - 完全成交
* CANCELED - 已取消
* PENDING\_CANCEL - 等待取消
* REJECTED - 被拒绝

**订单类型:**

* LIMIT - 限价单
* MARKET - 市价单
* LIMIT\_MAKER - maker限价单
* STOP\_LOSS \(unavailable now\) - 暂无
* STOP\_LOSS\_LIMIT \(unavailable now\) - 暂无
* TAKE\_PROFIT \(unavailable now\) - 暂无
* TAKE\_PROFIT\_LIMIT \(unavailable now\) - 暂无
* MARKET\_OF\_PAYOUT \(unavailable now\) - 暂无

**订单方向:**

* BUY - 买单
* SELL - 卖单

**订单时效类型:**

* GTC
* IOC
* FOK

**k线/烛线图区间:**

m -&gt; 分钟; h -&gt; 小时; d -&gt; 天; w -&gt; 周; M -&gt; 月

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 1w
* 1M

**频率限制类型 (rateLimitType)**

* REQUESTS\_WEIGHT
* ORDERS

**频率限制区间**

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

## 通用接口

### 测试连接

```text
GET /openapi/v1/ping
```

测试REST API的连接。

**Weight:** 0

**Parameters:** NONE

**Response:**

```javascript
{}
```

### 服务器时间

```text
GET /openapi/v1/time
```

测试连接并获取当前服务器的时间。

**Weight:** 0

**Parameters:** NONE

**Response:**

```javascript
{
  "serverTime": 1538323200000
}
```

### Broker信息

```text
GET /openapi/v1/brokerInfo
```

当前broker交易规则和symbol信息

**Weight:** 0

**Parameters:** NONE

**Response:**

```javascript
{
  "timezone": "UTC",
  "serverTime": 1538323200000,
  "rateLimits": [{
      "rateLimitType": "REQUESTS_WEIGHT",
      "interval": "MINUTE",
      "limit": 1500
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "SECOND",
      "limit": 20
    },
    {
      "rateLimitType": "ORDERS",
      "interval": "DAY",
      "limit": 350000
    }
  ],
  "brokerFilters":[],
  "symbols": [{
    "symbol": "ETHBTC",
    "status": "TRADING",
    "baseAsset": "ETH",
    "baseAssetPrecision": "0.001",
    "quoteAsset": "BTC",
    "quotePrecision": "0.01",
    "icebergAllowed": false,
    "filters": [{
      "filterType": "PRICE_FILTER",
      "minPrice": "0.00000100",
      "maxPrice": "100000.00000000",
      "tickSize": "0.00000100"
    }, {
      "filterType": "LOT_SIZE",
      "minQty": "0.00100000",
      "maxQty": "100000.00000000",
      "stepSize": "0.00100000"
    }, {
      "filterType": "MIN_NOTIONAL",
      "minNotional": "0.00100000"
    }]
  }]
}
```

## 行情接口

### 订单簿

```text
GET /openapi/quote/v1/depth
```

**Weight:** 根据limit不同:

| Limit | Weight |
| :--- | :--- |
| 5, 10, 20, 50, 100 | 1 |
| 500 | 5 |
| 1000 | 10 |

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| symbol | STRING | YES |  |
| limit | INT | NO | 默认 100; 最大 100 |

**Caution:** 如果设置limit=0会返回很多数据。

**Response:**

[价格, 数量]

```javascript
{
  "bids": [
    [
      "3.90000000",   // 价格
      "431.00000000"  // 数量
    ],
    [
      "4.00000000",
      "431.00000000"
    ]
  ],
  "asks": [
    [
      "4.00000200",  // 价格
      "12.00000000"  // 数量
    ],
    [
      "5.10000000",
      "28.00000000"
    ]
  ]
}
```

### 最近成交

```text
GET /openapi/quote/v1/trades
```

获取当前最新成交（最多500）

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| symbol | STRING | YES |  |
| limit | INT | NO | Default 500; max 1000. |

**Response:**

```javascript
[
  {
    "price": "4.00000100",
    "qty": "12.00000000",
    "time": 1499865549590,
    "isBuyerMaker": true
  }
]
```

### k线/烛线图数据

```text
GET /openapi/quote/v1/klines
```

symbol的k线/烛线图数据 K线会根据开盘时间而辨别。

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| symbol | STRING | YES |  |
| interval | ENUM | YES |  |
| startTime | LONG | NO |  |
| endTime | LONG | NO |  |
| limit | INT | NO | 默认500; 最大1000. |

* 如果startTime和endTime没有发送，只有最新的K线会被返回。

**Response:**

```javascript
[
  [
    1499040000000,      // Open time
    "0.01634790",       // Open
    "0.80000000",       // High
    "0.01575800",       // Low
    "0.01577100",       // Close
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368"       // Taker buy quote asset volume
  ]
]
```

### 24小时ticker价格变化数据

```text
GET /openapi/quote/v1/ticker/24hr
```

24小时价格变化数据。**注意** 如果没有发送symbol，会返回很多数据。

**Weight:** 如果只有一个symbol，1; 如果symbol没有被发送，**40**。

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| symbol | STRING | NO |  |

* 如果symbol没有被发送，所有symbol的数据都会被返回。

**Response:**

```javascript
{
  "time": 1538725500422,
  "symbol": "ETHBTC",
  "bestBidPrice": "4.00000200",
  "bestAskPrice": "4.00000200",
  "lastPrice": "4.00000200",
  "openPrice": "99.00000000",
  "highPrice": "100.00000000",
  "lowPrice": "0.10000000",
  "volume": "8913.30000000"
}
```

OR

```javascript
[
  {
    "time": 1538725500422,
    "symbol": "ETHBTC",
    "lastPrice": "4.00000200",
    "openPrice": "99.00000000",
    "highPrice": "100.00000000",
    "lowPrice": "0.10000000",
    "volume": "8913.30000000"
 }
]
```

### Symbol价格

```text
GET /openapi/quote/v1/ticker/price
```

单个或多个symbol的最新价

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| symbol | STRING | NO |  |

* 如果symbol没有发送，所有symbol的最新价都会被返回。

**Response:**

```javascript
{
  "price": "4.00000200"
}
```

OR

```javascript
[
  {
    "symbol": "LTCBTC",
    "price": "4.00000200"
  },
  {
    "symbol": "ETHBTC",
    "price": "0.07946600"
  }
]
```

### Symbol最佳订单簿价格

```text
GET /openapi/quote/v1/ticker/bookTicker
```

单个或者多个symbol的最佳买单卖单价格。

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| symbol | STRING | NO |  |

* 如果symbol没有被发送，所有symbol的最佳订单簿价格都会被返回。

**Response:**

```javascript
{
  "symbol": "LTCBTC",
  "bidPrice": "4.00000000",
  "bidQty": "431.00000000",
  "askPrice": "4.00000200",
  "askQty": "9.00000000"
}
```

OR

```javascript
[
  {
    "symbol": "LTCBTC",
    "bidPrice": "4.00000000",
    "bidQty": "431.00000000",
    "askPrice": "4.00000200",
    "askQty": "9.00000000"
  },
  {
    "symbol": "ETHBTC",
    "bidPrice": "0.07946700",
    "bidQty": "9.00000000",
    "askPrice": "100000.00000000",
    "askQty": "1000.00000000"
  }
]
```

## 账户接口

### 创建新订单   \(TRADE\)

```text
POST /openapi/v1/order  (HMAC SHA256)
```

发送一个新的订单

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| symbol | STRING | YES |  |
| assetType | STRING | NO |  |
| side | ENUM | YES |  |
| type | ENUM | YES |  |
| timeInForce | ENUM | NO |  |
| quantity | DECIMAL | YES |  |
| price | DECIMAL | NO |  |
| newClientOrderId | STRING | NO | 一个自己给订单定义的ID，如果没有发送会自动生成。 |
| stopPrice | DECIMAL | NO | 与 STOP_LOSS, STOP_LOSS_LIMIT, TAKE_PROFIT, 和TAKE_PROFIT_LIMIT 订单一起使用. 当前不可用 |
| icebergQty | DECIMAL | NO | 与 LIMIT, STOP_LOSS_LIMIT, 和 TAKE_PROFIT_LIMIT 来创建冰山订单. 当前不可用 |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

在`type`上的额外强制参数:

| 类型 | 额外强制参数 |
| :--- | :--- |
| `LIMIT` | `timeInForce`, `quantity`, `price` |
| `MARKET` | `quantity` |
| `STOP_LOSS` | `quantity`, `stopPrice` 当前不可用|
| `STOP_LOSS_LIMIT` | `timeInForce`, `quantity`,  `price`, `stopPrice` 当前不可用|
| `TAKE_PROFIT` | `quantity`, `stopPrice` 当前不可用|
| `TAKE_PROFIT_LIMIT` | `timeInForce`, `quantity`, `price`, `stopPrice` 当前不可用|
| `LIMIT_MAKER` | `quantity`, `price` |

Other info:

* `LIMIT_MAKER` are `LIMIT` orders that will be rejected if they would immediately match and trade as a taker.
* `STOP_LOSS` and `TAKE_PROFIT` will execute a `MARKET` order when the `stopPrice` is reached.
* Any `LIMIT` or `LIMIT_MAKER` type order can be made an iceberg order by sending an `icebergQty`.
* Any order with an `icebergQty` MUST have `timeInForce` set to `GTC`.

Trigger order price rules against market price for both MARKET and LIMIT versions:

* Price above market price: `STOP_LOSS` `BUY`, `TAKE_PROFIT` `SELL`
* Price below market price: `STOP_LOSS` `SELL`, `TAKE_PROFIT` `BUY`

**Response:**

```javascript
{
  "orderId": 28,
  "clientOrderId": "6k9M212T12092"
}
```

### 测试新订单 \(TRADE\)

```text
POST /openapi/v1/order/test (HMAC SHA256)
```

用signature和recvWindow测试生成新订单。 创建和验证一个新订单但是不送入撮合引擎。

**Weight:** 1

**Parameters:**

和 `POST /openapi/v1/order` 一样。

**Response:**

```javascript
{}
```

### 查询订单 \(USER\_DATA\)

```text
GET /openapi/v1/order (HMAC SHA256)
```

查询订单状态。

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| orderId | LONG | NO |  |
| origClientOrderId | STRING | NO |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

Notes:

* 单一  `orderId` 或者 `origClientOrderId` 必须被发送。 
* 对于某些历史数据  `cummulativeQuoteQty`  可能会 < 0, 这说明数据当前不可用。

**Response:**

```javascript
{
  "symbol": "LTCBTC",
  "orderId": 1,
  "clientOrderId": "9t1M2K0Ya092",
  "price": "0.1",
  "origQty": "1.0",
  "executedQty": "0.0",
  "cummulativeQuoteQty": "0.0",
  "avgPrice": "0.0",
  "status": "NEW",
  "timeInForce": "GTC",
  "type": "LIMIT",
  "side": "BUY",
  "stopPrice": "0.0",
  "icebergQty": "0.0",
  "time": 1499827319559,
  "updateTime": 1499827319559,
  "isWorking": true
}
```

### 取消订单 \(TRADE\)

```text
DELETE /openapi/v1/order  (HMAC SHA256)
```

取消当前正在交易的订单。

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| orderId | LONG | NO |  |
| clientOrderId | STRING | NO |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

单一 `orderId` 或者 `clientOrderId`。

**Response:**

```javascript
{
  "symbol": "LTCBTC",
  "clientOrderId": "tU721112KM",
  "orderId": 1,
  "status": "CANCELED"
}
```

### 当前订单 \(USER\_DATA\)

```text
GET /openapi/v1/openOrders  (HMAC SHA256)
```

获取当前单个或者多个symbol的当前订单。 **注意** 如果没有发送symbol，会返回很多数据。

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| symbol | String | NO |  |
| orderId | LONG | NO |  |
| limit | INT | NO | Default 500; max 1000. |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Notes:**

* 如果 `orderId`设定好了，会筛选订单小于 `orderId`的。否则会返回最近的订单信息。

**Response:**

```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "clientOrderId": "t7921223K12",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "avgPrice": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true
  }
]
```

### History orders \(USER\_DATA\)

```text
GET /openapi/v1/historyOrders (HMAC SHA256)
```

GET all orders of the account; canceled, filled or rejected.

**Weight:** 5

**Parameters:**

| Name | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| symbol | String | NO |  |
| orderId | LONG | NO |  |
| startTime | LONG | NO |  |
| endTime | LONG | NO |  |
| limit | INT | NO | Default 500; max 1000. |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Notes:**

* If `orderId` is set, it will get orders &lt; that `orderId`. Otherwise most recent orders are returned.

**Response:**

```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "clientOrderId": "987yjj2Ym",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "cummulativeQuoteQty": "0.0",
    "avgPrice": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "updateTime": 1499827319559,
    "isWorking": true
  }
]
```

### Account information \(USER\_DATA\)

```text
GET /openapi/v1/account (HMAC SHA256)
```

GET current account information.

**Weight:** 5

**Parameters:**

| Name | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Response:**

```javascript
{
  "canTrade": true,
  "canWithdraw": true,
  "canDeposit": true,
  "updateTime": 123456789,
  "balances": [
    {
      "asset": "BTC",
      "free": "4723846.89208129",
      "locked": "0.00000000"
    },
    {
      "asset": "LTC",
      "free": "4763368.68006011",
      "locked": "0.00000000"
    }
  ]
}
```

### Account trade list \(USER\_DATA\)

```text
GET /openapi/v1/myTrades  (HMAC SHA256)
```

GET trades for a specific account.

**Weight:** 5

**Parameters:**

| Name | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| startTime | LONG | NO |  |
| endTime | LONG | NO |  |
| fromId | LONG | NO | TradeId to fetch from. |
| toId | LONG | NO | TradeId to fetch to. |
| limit | INT | NO | Default 500; max 1000. |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Notes:**

* If only `fromId` is set，it will get orders &lt; that `fromId` in descending order.
* If only `toId` is set, it will get orders &gt; that `toId` in ascending order.
* If `fromId` is set and `toId` is set, it will get orders &lt; that `fromId` and &gt; that `toId` in descending order.
* If `fromId` is not set and `toId` it not set, most recent order are returned in descending order.

**Response:**

```javascript
[
  {
    "symbol": "ETHBTC",
    "id": 28457,
    "orderId": 100234,
    "matchOrderId": 109834,
    "price": "4.00000100",
    "qty": "12.00000000",
    "commission": "10.10000000",
    "commissionAsset": "ETH",
    "time": 1499865549590,
    "isBuyer": true,
    "isMaker": false
  }
]
```

### Account deposit list \(USER\_DATA\)

```text
GET /openapi/v1/depositOrders  (HMAC SHA256)
```

GET deposit orders for a specific account.

**Weight:** 5

**Parameters:**

| Name | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| startTime | LONG | NO |  |
| endTime | LONG | NO |  |
| fromId | LONG | NO | Deposit OrderId to fetch from. Default gets most recent deposit orders. |
| limit | INT | NO | Default 500; max 1000. |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Notes:**

* If `fromId` is set, it will get orders &gt; that `fromId`. Otherwise most recent orders are returned.

**Response:**

```javascript
[
  {
    "orderId": 100234,
    "token": "EOS",
    "address": "deposit2jb",
    "addressTag": "19012584",
    "fromAddress": "clarkkent",
    "fromAddressTag": "19029901",
    "time": 1499865549590,
    "quantity": "1.01"
  }
]
```

### Sub-account list\(SUB\_ACCOUNT\_LIST\)

```text
POST /openapi/v1/subAccount/query
```

Query sub-account lists

**Parameters:**

None

**Weight:** 5

**Response:**

```javascript
[
    {
        "accountId": "122216245228131",
        "accountName": "",
        "accountType": 1,
        "accountIndex": 0 // main-account: 0, sub-account: 1
    },
    {
        "accountId": "482694560475091200",
        "accountName": "createSubAccountByCurl", // sub-account name
        "accountType": 1, // sub-account type 1. token trading 3. contract trading
        "accountIndex": 1
    },
    {
        "accountId": "422446415267060992",
        "accountName": "",
        "accountType": 3,
        "accountIndex": 0
    },
    {
        "accountId": "482711469199298816",
        "accountName": "createSubAccountByCurl",
        "accountType": 3,
        "accountIndex": 1
    },
]
```

### Internal Account Transfer \(ACCOUNT\_TRANSFER\)

```text
POST /openapi/v1/transfer
```

Internal transfer

**Weight:** 1

**Parameters:**

| Name | Typee | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| fromAccountType | int | YES | source account type: 1. token trading account 2.Options account 3. Contracts account |
| fromAccountIndex | int | YES | sub-account index\(valid when using main-account api, get sub-account indices from `SUB_ACCOUNT_LIST` endpoint\) |
| toAccountType | int | YES | Target account type: 1. token trading account 2.Options account 3. Contracts account |
| toAccountIndex | int | YES | sub-account index\(valid when using main-account api, get sub-account indices from `SUB_ACCOUNT_LIST` endpoint\) |
| tokenId | STRING | YES | tokenID |
| amount | STRING | YES | Transfer amount |

**Response:**

```javascript
{
    "success":"true" // success
}
```

**Explanation**

1. Either transferring or receiving account must be the main account \(Token trading account\)
2. Main account api can support transferring to other account\(including sub-accounts\) and receiving from other accounts
3. **Sub-account API only supports transferring from current account to the main-account. Therefore `fromAccountType\fromAccountIndex\toAccountType\toAccountIndex` should be left empty.**

### Check Balance Flow \(BALANCE\_FLOW\)

```text
POST /openapi/v1/balance_flow
```

Check blance flow

**Weight:** 5

**Parameters:**

| Name | Type | Mandatory | Description |  |  |
| :--- | :--- | :--- | :--- | :--- | :--- |
| accountType | int | no | Account account\_type | 默认1 |  |
| accountIndex | int | no | Account account\_index | 默认0 |  |
| tokenId | string | no | token\_id | eg: BTC |  |
| fromFlowId | long | no |  | Query data that id &lt; fromFlowId |  |
| endFlowId | long | no |  | Query data that id &gt; | endFlowId |
| startTime | long | no | Start Time | Timestamp \(millisecond\) |  |
| endTime | long | no | End Time | Timestamp \(millisecond\) |  |
| limit | integer | no | Number of entries | Default is 50，max is 100 |  |

**Response:**

```javascript
[
    {
        "id": "539870570957903104",
        "accountId": "122216245228131",
        "tokenId": "BTC",
        "tokenName": "BTC",
        "flowTypeValue": 51, // balance flow type
        "flowType": "USER_ACCOUNT_TRANSFER", // balance flow type name
        "flowName": "Transfer", // balance flow type Explanation
        "change": "-12.5", // change
        "total": "379.624059937852365", // total asset after change
        "created": "1579093587214"
    },
    {
        "id": "536072393645448960",
        "accountId": "122216245228131",
        "tokenId": "USDT",
        "tokenName": "USDT",
        "flowTypeValue": 7,
        "flowType": "AIRDROP",
        "flowName": "Airdrop",
        "change": "-2000",
        "total": "918662.0917630848",
        "created": "1578640809195"
    }
]
```

**Explanation**

1. Main-account API can query balance flow for token account and other accounts\(including sub-accounts, or designated `accountType` and `accountIndex` accounts\)
2. Sub-account API can only query current sub-account, therefore `accountType` and `accountIndex` is not required.

**Please see the following for balance flow types**

| Category | Parameter Type Name | Parameter Type Id | Explanation |
| :--- | :--- | :--- | :--- |
| General Balance Flow | TRADE | 1 | trades |
| General Balance Flow | FEE | 2 | trading fees |
| General Balance Flow | TRANSFER | 3 | transfer |
| General Balance Flow | DEPOSIT | 4 | deposit |
| Derivatives | MAKER\_REWARD | 27 | maker reward |
| Derivatives | PNL | 28 | PnL from contracts |
| Derivatives | SETTLEMENT | 30 | Settlement |
| Derivatives | LIQUIDATION | 31 | Liquidation |
| Derivatives | FUNDING\_SETTLEMENT | 32 | 期货等的资金费率结算 |
| Internal Transfer | USER\_ACCOUNT\_TRANSFER | 51 | userAccountTransfer 专用，流水没有subjectExtId |
| OTC | OTC\_BUY\_COIN | 65 | OTC buy coin |
| OTC | OTC\_SELL\_COIN | 66 | OTC sell coin |
| OTC | OTC\_FEE | 73 | OTC fees |
| OTC | OTC\_TRADE | 200 | Old OTC balance flow |
| Campaign | ACTIVITY\_AWARD | 67 | Campaign reward |
| Campaign | INVITATION\_REFERRAL\_BONUS | 68 | 邀请返佣 |
| Campaign | REGISTER\_BONUS | 69 | Registration reward |
| Campaign | AIRDROP | 70 | Airdrop |
| Campaign | MINE\_REWARD | 71 | Mining reward |

## User data stream API

Specifics on how user data streams work is in another document.

### Start user data stream \(USER\_STREAM\)

```text
POST /openapi/v1/userDataStream
```

Start a new user data stream. The stream will close after 60 minutes unless a keepalive is sent.

**Weight:** 1

**Parameters:**

| Name | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Response:**

```javascript
{
  "listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

### Keepalive user data stream \(USER\_STREAM\)

```text
PUT /openapi/v1/userDataStream
```

Keepalive a user data stream to prevent a time out. User data streams will close after 60 minutes. It's recommended to send a ping about every 30 minutes.

**Weight:** 1

**Parameters:**

| Name | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| listenKey | STRING | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Response:**

```javascript
{}
```

### Close user data stream \(USER\_STREAM\)

```text
DELETE /openapi/v1/userDataStream
```

Close out a user data stream.

**Weight:** 1

**Parameters:**

| Name | Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| listenKey | STRING | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Response:**

```javascript
{}
```

## Filters

Filters define trading rules on a symbol or an broker. Filters come in two forms: `symbol filters` and `broker filters`.

### Symbol filters

#### PRICE\_FILTER

The `PRICE_FILTER` defines the `price` rules for a symbol. There are 3 parts:

* `minPrice` defines the minimum `price`/`stopPrice` allowed.
* `maxPrice` defines the maximum `price`/`stopPrice` allowed.
* `tickSize` defines the intervals that a `price`/`stopPrice` can be increased/decreased by.

In order to pass the `price filter`, the following must be true for `price`/`stopPrice`:

* `price` &gt;= `minPrice`
* `price` &lt;= `maxPrice`
* \(`price`-`minPrice`\) % `tickSize` == 0

**/brokerInfo format:**

```javascript
  {
    "filterType": "PRICE_FILTER",
    "minPrice": "0.00000100",
    "maxPrice": "100000.00000000",
    "tickSize": "0.00000100"
  }
```

#### LOT\_SIZE

The `LOT_SIZE` filter defines the `quantity` \(aka "lots" in auction terms\) rules for a symbol. There are 3 parts:

* `minQty` defines the minimum `quantity`/`icebergQty` allowed.
* `maxQty` defines the maximum `quantity`/`icebergQty` allowed.
* `stepSize` defines the intervals that a `quantity`/`icebergQty` can be increased/decreased by.

In order to pass the `lot size`, the following must be true for `quantity`/`icebergQty`:

* `quantity` &gt;= `minQty`
* `quantity` &lt;= `maxQty`
* \(`quantity`-`minQty`\) % `stepSize` == 0

**/brokerInfo format:**

```javascript
  {
    "filterType": "LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
  }
```

#### MIN\_NOTIONAL

The `MIN_NOTIONAL` filter defines the minimum notional value allowed for an order on a symbol. An order's notional value is the `price` \* `quantity`.

**/brokerInfo format:**

```javascript
  {
    "filterType": "MIN_NOTIONAL",
    "minNotional": "0.00100000"
  }
```

#### MAX\_NUM\_ORDERS

The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on a symbol. Note that both "algo" orders and normal orders are counted for this filter.

**/brokerInfo format:**

```javascript
  {
    "filterType": "MAX_NUM_ORDERS",
    "limit": 25
  }
```

#### MAX\_NUM\_ALGO\_ORDERS

The `MAX_ALGO_ORDERS` filter defines the maximum number of "algo" orders an account is allowed to have open on a symbol. "Algo" orders are `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.

**/brokerInfo format:**

```javascript
  {
    "filterType": "MAX_NUM_ALGO_ORDERS",
    "maxNumAlgoOrders": 5
  }
```

#### ICEBERG\_PARTS

The `ICEBERG_PARTS` filter defines the maximum parts an iceberg order can have. The number of `ICEBERG_PARTS` is defined as `CEIL(qty / icebergQty)`.

**/brokerInfo format:**

```javascript
  {
    "filterType": "ICEBERG_PARTS",
    "limit": 10
  }
```

### Broker Filters

#### BROKER\_MAX\_NUM\_ORDERS

The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on the broker. Note that both "algo" orders and normal orders are counted for this filter.

**/brokerInfo format:**

```javascript
  {
    "filterType": "BROKER_MAX_NUM_ORDERS",
    "limit": 1000
  }
```

#### BROKER\_MAX\_NUM\_ALGO\_ORDERS

The `MAX_ALGO_ORDERS` filter defines the maximum number of "algo" orders an account is allowed to have open on the broker. "Algo" orders are `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.

**/brokerInfo format:**

```javascript
  {
    "filterType": "BROKER_MAX_NUM_ALGO_ORDERS",
    "limit": 200
  }
```

