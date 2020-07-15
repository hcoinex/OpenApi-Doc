# 用户数据流推送

## 基本信息

* 一个用户数据流的`listenKey`在创建之后的有效期只有60分钟
* 如果对`listenKey`做`PUT`请求可以延长有效期60分钟 
* 如果对`listenKey`做`DELETE`请求会关闭推送
* 用户数据流推送在这个端点 **/openapi/ws/** 访问
* 单一API连接的有效期只有24小时，请做好在24小时后被断开连接的准备
* 用户信息流返回在订单繁忙期**不保证**顺序正常，**请使用E字段进行排序**

## 推送接口

### 创建listenKey

```text
POST /openapi/v1/userDataStream
```

创建一个新的用户信息流。如果没有发送keepalive，推送将会在60分钟后断开。

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Response:**

```javascript
{
  "listenKey": "1A9LWJjuMwKWYP4QQPw34GRm8gz3x5AephXSuqcDef1RnzoBVhEeGE963CoS1Sgj"
}
```

### 延长listenKey有效期

```text
PUT /openapi/v1/userDataStream
```

发送PUT请求会有效期延长至本次调用后60分钟，建议每30分钟发送一个ping。

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| listenKey | STRING | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Response:**

```javascript
{}
```

### 关闭listenKey

```text
DELETE /openapi/v1/userDataStream
```

关闭用户数据流。

**Weight:** 1

**Parameters:**

| 名称 | 类型 | 是否强制 | 描述 |
| :--- | :--- | :--- | :--- |
| listenKey | STRING | YES |  |
| recvWindow | LONG | NO |  |
| timestamp | LONG | YES |  |

**Response:**

```javascript
{}
```

## Web Socket Payloads

### 账户更新

使用`outboundAccountInfo` event进行账户更新。

**Payload:**

```javascript
{
  "e": "outboundAccountInfo",   // Event type
  "E": 1499405658849,           // Event time
  // "m": 0,                       // Maker commission rate (bips)
  // "t": 0,                       // Taker commission rate (bips)
  // "b": 0,                       // Buyer commission rate (bips)
  // "s": 0,                       // Seller commission rate (bips)
  "T": true,                    // Can trade?
  "W": true,                    // Can withdraw?
  "D": true,                    // Can deposit?
  // "u": 1499405658848,           // Time of last account update
  "B": [                        // Balances changed
    {
      "a": "LTC",               // Asset
      "f": "17366.18538083",    // Free amount
      "l": "0.00000000"         // Locked amount
    }
  ]
}
```

### 订单更新

订单通过`executionReport`事件进行更新。详细说明信息请查看 API文档。通过将`Z`除以`z`可以找到平均价格。

**Payload:**

```javascript
{
  "e": "executionReport",        // Event type
  "E": 1499405658658,            // Event time
  "s": "ETHBTC",                 // Symbol
  "c": 1000087761,               // Client order ID
  "S": "BUY",                    // Side
  "o": "LIMIT",                  // Order type
  "f": "GTC",                    // Time in force
  "q": "1.00000000",             // Order quantity
  "p": "0.10264410",             // Order price
  // "P": "0.00000000",             // Stop price
  // "F": "0.00000000",             // Iceberg quantity
  // "g": -1,                       // Ignore
  // "x": "NEW",                    // Current execution type
  "X": "NEW",                    // Current order status
  // "r": "NONE",                   // Order reject reason; will be an error code.
  "i": 4293153,                  // Order ID
  "l": "0.00000000",             // Last executed quantity
  "z": "0.00000000",             // Cumulative filled quantity
  "L": "0.00000000",             // Last executed price
  "n": "0",                      // Commission amount
  "N": null,                     // Commission asset
  "u": true,                     // Is the trade normal, ignore for now
  // "T": 1499405658657,            // Transaction time
  // "t": -1,                       // Trade ID
  // "I": 8641984,                  // Ignore
  "w": true,                     // Is the order working? Stops will have
  "m": false,                    // Is this trade the maker side?
  // "M": false,                    // Ignore
  "O": 1499405658657,            // Order creation time
  "Z": "0.00000000"              // Cumulative quote asset transacted quantity
}
```

**Execution types:**

* NEW（新订单）
* PARTIALLY_FILLED（部分成交）
* FILLED（全部成交）
* CANCELED（已撤销）
* REJECTED（已拒绝）

