# Public Rest API for Piexgo 

# General API Information
* The  base prod-endpoint is: **https://api.piexgo.com**
* The base test-endpoint is: **https://api.piexgo.co**
* All endpoints return either a JSON object or array.


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

### Assets information
```
GET /api/v2/assets
```
Returns information about currencies


**Parameters:**
NONE

**Response:**

Field | Type |  Description
------|------ | ------------
market | STRING | Name of the currency
deposit_status | INT | Designates whether (0) or not (1) deposits are disabled
withdraw_status | INT | Designates whether (0) or not (1) withdrawals are disabled
status | INT | Designates whether (3) or not (1,2) this currency has been delisted from the exchange
withdraw_fee | DECIMAL | The network fee necessary to withdraw this currency
withdraw_min | DECIMAL | The minimum number of blocks necessary before a withdraw can be credited to an account
deposit_min | DECIMAL | The minimum number of blocks necessary before a deposit can be credited to an account

```javascript
Example:

curl https://api.piexgo.com/api/v2/assets

SUCCESS:
[{
    "market":"USDT",
    "deposit_status":0,
    "withdraw_status":0,
    "status":3,  // 3-enable, 1 or 2-disable
    "withdraw_fee":2,
    "withdraw_min":10,
    "deposit_min":1
},
......
{
    "market":"BTC",
    "deposit_status":0,
    "withdraw_status":0,
    "status":3,
    "withdraw_fee":0.0005,
    "withdraw_min":0.01,
    "deposit_min":0.0001
}]

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

Retrieves summary information for each currency pair listed on the exchange.
```
GET /api/v2/ticker
```

**Parameters:**
NONE

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
is_frozen | INT | Price change percentage.

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
        "symbol":"BTC_USDT",
        "is_frozen":0
    },
    ......
}

FAILED:
{}
```
