# Public Rest API for Piexgo 

# General API Information
* The  base prod-endpoint is: **https://api.piexgo.com**
* The base test-endpoint is: **https://api.piexgo.co**
* All endpoints return either a JSON object or array.


* Any endpoint can return an ERROR; the error payload is as follows:
```javascript
{
  "err_code": 400,
  "msg": "Invalid symbol."
}
```


# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it.
* API-keys are passed into the Rest API via the `KEY` and `signature`
  header.
* API-keys and secret-keys **are case sensitive**.
* By default, API-keys can access all secure routes.




* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

# SIGNED Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string`.
* Endpoints use `HMAC SHA512` signatures. The `HMAC SHA512 signature` is a keyed `HMAC SHA512` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **case sensitive**.


## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recv_window`, may be sent to specify the number of milliseconds after timestamp the request is valid for. If `recv_window` is not sent, it defaults to 5000.
* The logic is as follows:
```javascript
if (timestamp < (server_time + 1000) && (server_time - timestamp) <= recv_window) {
  // process request
} else {
  // reject request
}
```



## SIGNED Endpoint Examples for POST /api/v1/order
Here is a step-by-step example of how to send a vaild signed payload.

Key | Value
------------ | ------------
apiKey | 23db31c6-6fd7-4041-9388-81dea6a05086
secretKey | 7bea08b911022f8eece9dbd0b86facb86011c666


Parameter | Value
------------ | ------------
price | 6830
quantity | 10
side | BUY/SELL
symbol | BTC_USDT
type | LIMIT/MARKET/STOP_LIMIT
stop_price | 6830
recv_window | 5000
timestamp | 1552356480000




### Example 1 (LIMIT): As a query string
* **Sort the query string components by byte order.:** price=6830&quantity=10&recv_window=5000&side=BUY&symbol=BTC_USDT&timestamp=1552356480000&type=LIMIT
* **HMAC SHA512 signature:** Calculate the Signature:
The signature is calculated with the query string and secret key as inputs to a keyed hash function.

```
[linux]$ echo -n "price=6830&quantity=10&recv_window=5000&side=BUY&symbol=BTC_USDT&timestamp=1552356480000&type=LIMIT" | openssl dgst -sha512 -hmac "4ad49bd70fa3674800e10e37d3cfe4ce433a8b94"
(stdin)= fc3729658558e0ca42c12c46b94feb269cdf3d87c4a649896c457472b4e4ab4ec1640b0882d5de9a33ed282c33bc171c95f3315b95b578fd64f87bde4ce945cd
```

* Add the resulting value to the query header as a Signature parameter. 

### Example 2 (MARKET): As a query string
* **Sort the query string components by byte order.:** 
quantity=10&recv_window=5000&side=BUY&symbol=BTC_USDT&timestamp=1552356480000&type=MARKET
* **HMAC SHA512 signature:** Calculate the Signature:
The signature is calculated with the query string and secret key as inputs to a keyed hash function.

```
[linux]$ echo -n "quantity=10&recv_window=5000&side=BUY&symbol=BTC_USDT&timestamp=1552356480000&type=MARKET" | openssl dgst -sha512 -hmac "4ad49bd70fa3674800e10e37d3cfe4ce433a8b94"
(stdin)= 340b0fdc5adcaa3116d1bf2623bae1efb3baefb6d027cfc2d07df9e650f33fa2e3d4a2a1af2099556ab2487caad2efb184c459d97eda38e24659ffd9bdffe9de
```

* Add the resulting value to the query header as a Signature parameter. 

### Example 3 (STOP_LIMIT): As a query string
* **Sort the query string components by byte order.:** price=6830&quantity=10&recv_window=5000&side=BUY&stop_price=6830&symbol=BTC_USDT&timestamp=1552356480000&type=STOP_LIMIT
* **HMAC SHA512 signature:** Calculate the Signature:
The signature is calculated with the query string and secret key as inputs to a keyed hash function.

```
[linux]$ echo -n "price=6830&quantity=10&recv_window=5000&side=BUY&stop_price=6830&symbol=BTC_USDT&timestamp=1552356480000&type=STOP_LIMIT" | openssl dgst -sha512 -hmac "4ad49bd70fa3674800e10e37d3cfe4ce433a8b94"
(stdin)= d7fad2e98edf9dc49ec37537195c5a143a8f51615d74a3282391f6b7f861b152b83518d8e93fdd1ce63fc5936ab70ee336d9e89364c704e85204c040d20dbb41
```

* Add the resulting value to the query header as a Signature parameter. 





### Check server time
```
GET /api/v1/time
```
Get the current server time


**Parameters:**
NONE

**Response:**

```javascript
{
    "server_time":1552356480000
}
```



### Pairs information
```
GET /api/v2/symbols
```
Current exchange  symbol information


**Parameters:**
NONE

**Response:**

Field | Type |  Description
------------ | ------------ | ------------ 
data | array | An array of symbols

```javascript
Example:

curl https://api.piexgo.com/api/v2/symbols

SUCCESS:
["BTC_USDT","ETH_USDT","ETH_BTC",......]

FAILED:
{}
```

## Market Data endpoints
### Order book
```
GET /api/v2/orderBook
```

**Parameters:**

Parameter | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | A pair like BTC_ETH or all
depth  | INT    | NO  | Default depth is 10. Max depth is 50.


**Response:**

The response data will be two arrays and timestamp

Field |	Type | Description
------------ | ------------ | ------------ 
bids | array | The bids currently on the book. These are offers to buy at a given price
asks | array | The asks currently on the book. These are offers to sell at a given price
timestamp | timestamp | Unix timestamp in milliseconds for when the last updated time occurred.

The bids and the asks are grouped by price, so each entry may represent multiple orders at that price. Each element of the array will be a JSON object

Field | Type |  Description
------------ | ------------ | ------------ 
price | decimal | The price
quantity | decimal | The total quantity at the price

```javascript
Example:

curl https://api.piexgo.com/api/v2/orderBook?symbol=all&depth=10

SUCCESS:
{
    "BTC_USDT":{
        "asks":[
            {"price":"500.563","quantity":"1000"}
            {"price":"410.65855","quantity":"110"}
            ......
            {"price":"451.7412","quantity":"210"}
        ],
        "bids":[
            {"price":"1523.956","quantity":"20"}
            {"price":"457.85456","quantity":"100"}
            ......
            {"price":"854.445855","quantity":"11000"}
        ],
        "timestamp":1571911416469
    },
    "ETH_USDT":{
        "asks":[
            {"price":"500.563","quantity":"1000"}
            {"price":"410.65855","quantity":"110"}
            ......
            {"price":"451.7412","quantity":"210"}
        ],
        "bids":[
            {"price":"1523.956","quantity":"20"}
            {"price":"457.85456","quantity":"100"}
            ......
            {"price":"854.445855","quantity":"11000"}
        ],
        "timestamp":1571911416469
    },
    ......
}

FAILED:
{}
```

### Recent trades
```
GET /api/v2/trades
```
Returns the past 200 trades for a given market.

**Parameters:**

Parameter | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | A pair like BTC_ETH

**Response:**

Field | Type |  Description
------------ | ------------ | ------------ 
time | LONG |  Order deal time
direct | INT | 0-BUY, 1-SELL 
price | STRING | Order average deal price
vol | STRING | Order deal quantity 


```javascript
Example:

curl https://api.piexgo.com/api/v2/trades?symbol=BTC_USDT

SUCCESS:
[
    {
        "time":1571901738933,
        "direct":0, //0-BUY, 1-SELL 
        "price":"10000.0014000000",
        "vol":"0.00100000"
    },
    {
        "time": 1553821666904,
        "direct": 0, //0-BUY, 1-SELL 
        "price": "4029.7800000000",
        "vol": "0.48000000"
    },
    ......
]

FAILED:
{}
```


### 24H ticker statistics

Returns the 24-hour volume for all markets as well as totals for primary currencies.
```
GET /api/v2/24hVolume
```

**Parameters:**
NONE

**Response:**

Field | Type |  Description
------------ | ------------ | ------------ 
symbol | STRING | BTC_USDT
--- | DECIMAL | BTC deal quantity
--- | DECIMAL | USDT deal quantity

```javascript
Example:

curl https://api.piexgo.com/api/v2/24hVolume

SUCCESS:
{
    "ALGO_USDT":{
        "ALGO":0,
        "USDT":0
    },
    "BCH_USDT":{
        "BCH":20005.25,
        "USDT":100.02
    },
    ......
}

FAILED:
{}
```


### Symbol ticker

This endpoint retrieves information about recent trading activity for the symbol.
```
GET /api/v2/ticker
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO | A pair like BTC_ETH or all,Default all

**Response:**
Retrieves summary information for each currency pair listed on the exchange. Fields include:

Field | Type |  Description
------------ | ------------ | ------------ 
symbol | STRING | A pair like BTC_USDT
last_price | DECIMAL | The price of the last executed trade
highest_bid | DECIMAL | The highest bid currently available
lowest_ask | DECIMAL | The lowest ask currently available
base_volume | DECIMAL | Base units traded in the last 24 hours.
quote_volume | DECIMAL | Quoted units traded in the last 24 hours.
percent_change | DECIMAL | Price change percentage.

```javascript
Example:

curl https://api.piexgo.com/api/v2/ticker?symbol=all

SUCCESS:
{
    "BTC_USDT":{
        "base_volume":264.5,
        "highest_bid":11002,
        "last_price":10,
        "lowest_ask":1,
        "percent_change":-99.8989898989899,
        "quote_volume":1411528.22,
        "symbol":"BTC_USDT"
    },
    ......
}

FAILED:
{}
```


## Trade or Account endpoints
### New order  (TRADE)
```
POST /api/v1/order  (HMAC signature)
```
Send in a new order.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
price | DECIMAL | NO | Order price
quantity | DECIMAL | YES | Order quantity
side | ENUM | YES | BUY/SELL
symbol | STRING | YES | Order market
type | ENUM | YES | LIMIT/MARKET/STOP_LIMIT
stop_price | DECIMAL | NO | STOP_LIMIT Order stop pirce
recv_window | LONG | YES | Expire times
timestamp | LONG | YES | Server time



Additional mandatory parameters based on `type`:

Type | Additional mandatory parameters
------------ | ------------
`LIMIT` |  `quantity`, `price`
`MARKET` | `quantity`
`STOP_LIMIT` | `quantity`, `price`, `stop_price`


**Response:**

```javascript
{
    "err_code": 0,
    "msg": "ok",
    "order_id": "1234567890"
}
```

### Cancel order  (TRADE)
```
POST /api/v1/cancelOrder  (HMAC signature)
```
Cancel an open order.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | Order market
order_id | STRING | YES | Order ID
recv_window | LONG | YES | Expire times
timestamp | LONG | YES | Server time

**Response:**

```javascript
{
    "err_code":0,
    "msg":"ok"
}
```

### Cancel orders  (TRADE)
```
POST /api/v1/cancelOrders  (HMAC signature)
```
Cancel multiple open orders.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | Order market
order_id | STRING | YES | Orders ID Separated by commas (Up to 50.)
recv_window | LONG | YES | Expire times
timestamp | LONG | YES | Server time

**Response:**

```javascript
{
    "err_code":0,
    "msg":"ok"
}
```

### Single order info (USER_DATA) 
```
POST /api/v1/orderInfo  (HMAC signature)
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | Order market
order_id | STRING | YES | Order ID

**Response:**

Name | Type | Description
------------ | ------------ | ------------
order_price | STRING | Order price |
price | STRING | Order average deal price |
volume | STRING | Order quantity |
filled_volume | STRING | Order filled quantity |
fee | STRING | Order fee |
type | STRING | buy/sell |
timestamp | LONG | Order created time |
update_time | LONG | Order updated time |
order_id | STRING | Order ID |
market | STRING | Market |
status | INT | Order status |
flag | INT | 0-LIMIT,1-MARKET,2-STOP_LIMIT |

**Definitions**

Order status (status)

* 0 : Order is pending
* 2 : Order is done
* 4 : Order is cancelled (Partial deal)
* 5 : Order is refused
* -1 : STOP_LIMIT order is pending

```javascript
{
    "err_code":0,
    "msg":"OK",
    "data":{
        "order_price": "3330.4800000000",
        "price": "3330.4800000000",
        "volume": "2.00000000",
        "filled_volume": "2.00000000",
        "fee":"0.00029486",
        "type": "buy",
        "timestamp": 1553831017,
        "update_time": 1553848365,
        "order_id": "1111070125244588064",
        "market": "BTC_USDT",
        "status": 0,
        "flag": 0,
    }
}
```

### Current open orders (USER_DATA) 
```
POST /api/v1/openOrders  (HMAC signature)
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
page | STRING | YES | Starting from zero

* 50 records per page, page parameters starting from 0

**Response:**

Name | Type | Description
------------ | ------------ | ------------
stop | STRING | Order stop price |
remain | STRING | Order remain quantity
market_price | STRING | MARKET Order price
stop | STRING | STOP_LIMIT Order stop price

```javascript
{
    "err_code":0,
    "msg":"OK",
    "total":10,
    "next_page":false,
    "list":[
        {
            "order_price": "3330.4800000000",
            "stop": "0.0000000000",
            "price": "3330.4800000000",
            "market_price": "0.0000000000",
            "volume": "2.00000000",
            "remain": "2.00000000",
            "amount": "0.00000000",
            "type": "buy",
            "timestamp": 1553831017,
            "order_id": "1111070125244588064",
            "market": "BTC_USDT",
            "status": 0,
            "flag": 0
        }
    ]
}
```

### Transaction History orders (USER_DATA)
```
POST /api/v1/tradeOrderHistory  (HMAC signature)
```
Get transaction order hisroty.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
page | STRING | YES | Starting from zero



**Response:**

```javascript
{
    "err_code":0,
    "msg":"OK",
    "total":10,
    "next_page":false,
    "list":[
        {
            "order_price": "3330.4800000000",
            "stop": "0.0000000000",
            "price": "3330.4800000000",
            "market_price": "0.0000000000",
            "volume": "2.00000000",
            "remain": "2.00000000",
            "amount": "0.00000000",
            "type": "buy",
            "timestamp": 1553831017,
            "order_id": "1111070125244588064",
            "market": "BTC_USDT",
            "status": 4,
            "flag": 0
        }
    ]
}
```

### Canceled History orders (USER_DATA)
```
POST /api/v1/canceledOrderHistory  (HMAC signature)
```
Get canceled order hisroty.


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
page | STRING | YES | Starting from zero



**Response:**

```javascript
{
    "err_code":0,
    "msg":"OK",
    "total":10,
    "next_page":false,
    "list":[
        {
            "order_price": "3330.4800000000",
            "stop": "0.0000000000",
            "price": "3330.4800000000",
            "market_price": "0.0000000000",
            "volume": "2.00000000",
            "remain": "2.00000000",
            "amount": "0.00000000",
            "type": "buy",
            "timestamp": 1553831017,
            "order_id": "1111070125244588064",
            "market": "BTC_USDT",
            "status": 4,
            "flag": 0
        }
    ]
}
```

### History orders (USER_DATA)
```
POST /api/v1/orderHistory  (HMAC signature)(To be removed) 
```
Get all order hisroty (filled or canceled).


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | Order market
page | STRING | YES | Starting from zero



**Response:**

```javascript
{
    "err_code":0,
    "msg":"OK",
    "total":10,
    "next_page":false,
    "list":[
        {
            "order_price": "3330.4800000000",
            "stop": "0.0000000000",
            "price": "3330.4800000000",
            "market_price": "0.0000000000",
            "volume": "2.00000000",
            "remain": "2.00000000",
            "amount": "0.00000000",
            "type": "buy",
            "timestamp": 1553831017,
            "order_id": "1111070125244588064",
            "market": "BTC_USDT",
            "status": 4,
            "flag": 0
        }
    ]
}
```

### Account information (USER_DATA)
```
POST /api/v1/account  (HMAC signature)
```
Get current account information.


**Parameters:**
NONE

**Response:**

Name | Type | Description
------------ | ------------ | ------------
market | STRING | Symbol |
balance | STRING | Symbol balance |
locked | STRING | Symbol locked |
otc_balance | STRING | Symbol otc balance |
otc_locked | STRING | Symbol otc locked |

```javascript
{
    "err_code":0,
    "msg":"ok",
    "data":[
        {
            "market": "USDT",   
            "balance": "10048.28242756",
            "locked": "0.00000000",
            "otc_balance": "0.00000000",
            "otc_locked": "0.00000000",
        }
    ]
}
```



