# <a name="begin"/> Public API v3

*[English](public-api.md) ∙ [Русский](public-api_ru.md)*
___

* [Contents](#main)
* [Methods](#methods)
    * [info](#info)
    * [ticker](#ticker)
    * [depth](#depth)
    * [trades](#trades)

## <a name="main"/> Contents

This api provides access to such information as tickers of currency pairs, active orders on different pairs, the latest trades for each pair etc.

All API requests are made from this address: \
https://wex.link/api/3/<method_name>/<pair_listing>

Currency pairs are hyphen-separated (-), e.g.: \
https://wex.link/api/3/ticker/btc_usd-btc_rur

You can use as many pairs in the listing as you wish. Duplicates are not allowed. It is also possible to use only one pair: \
https://wex.link/api/3/ticker/btc_usd \
A set of pairs works with all the methods presented in the public api except info.

All information is cached every 2 seconds, so there's no point in making more frequent requests. \
All API responses have the following format JSON.

**Important!** API will display an error if we disable one of the pairs listed in your request. If you are not going to synchronize the state of pairs using the method info, you need to send the GET-parameter ignore_invalid equal to 1, e.g.: \
https://wex.link/api/3/ticker/btc_usd-btc_btc?ignore_invalid=1 \
Without the parameter ignore_invalid this request would have caused an error because of a non-existent pair.

##### [to the top](#begin)
___
## <a name="methods"/> Methods
### <a name="info"/> info

This method provides all the information about currently active pairs, such as the maximum number of digits after the decimal point, the minimum price, the maximum price, the minimum transaction size, whether the pair is hidden, the commission for each pair.

**Request example:**

```bash
curl -X GET \
  https://wex.link/api/3/info
```

**Response example:**

```javascript
{
  "server_time": 1370814956,
  "pairs": {
    "btc_usd": {
      "decimal_places": 3,
      "min_price": 0.1,
      "max_price": 400,
      "min_amount": 0.01,
      "hidden": 0,
      "fee": 0.2
    }
    # etc
  }
}
```

**decimal_places**: number of decimals allowed during trading. \
**min_price**: minimum price allowed during trading. \
**max_price**: maximum price allowed during trading. \
**min_amount**: minimum sell / buy transaction size. \
**hidden**: whether the pair is hidden, 0 or 1. \
**fee**: commission for this pair.

A hidden pair (hidden=1) remains active but is not displayed in the list of pairs on the main page. \
The Commission is displayed for all users, it will not change even if it was reduced on your account in case of promotional pricing. \
If one of the pairs is disabled, it will simply disappear from the list.

##### [to the top](#begin)
___
### <a name="ticker"/> ticker

This method provides all the information about currently active pairs, such as: the maximum price, the minimum price, average price, trade volume, trade volume in currency, the last trade, Buy and Sell price. \
All information is provided over the past 24 hours.

**Request example:**

```bash
curl -X GET \
  https://wex.link/api/3/ticker/btc_usd
```

**Response example:**
```javascript
{
  "btc_usd": {
    "high": 109.88,
    "low": 91.14,
    "avg": 100.51,
    "vol": 1632898.2249,
    "vol_cur": 16541.51969,
    "last": 101.773,
    "buy": 101.9,
    "sell": 101.773,
    "updated": 1370816308
  }
  # etc
}
```

**high**: maximum price. \
**low**: minimum price. \
**avg**: average price. \
**vol**: trade volume. \
**vol_cur**: trade volume in currency. \
**last**: the price of the last trade. \
**buy**: buy price. \
**sell**: sell price. \
**updated**: last update of cache.

##### [to the top](#begin)
___
### <a name="depth"/> depth

This method provides the information about active orders on the pair.

Additionally it accepts an optional GET-parameter limit, which indicates how many orders should be displayed (150 by default). \
Is set to less than 5000.

**Request example:**

```bash
curl -X GET \
  https://wex.link/api/3/depth/btc_usd
```

**Response example:**

```javascript
{
  "btc_usd": {
    "asks": [
      [
        103.426,
        0.01
      ],
      [
        103.5,
        15
      ],
      [
        103.504,
        0.425
      ],
      [
        103.505,
        0.1
      ],
      # etc
    ],
    "bids": [
      [
        103.2,
        2.48502251
      ],
      [
        103.082,
        0.46540304
      ],
      [
        102.91,
        0.99007913
      ],
      [
        102.83,
        0.07832332
      ],
      # etc
    ]
  }
  # etc
}
```

**asks**: Sell orders. \
**bids**: Buy orders.

##### [to the top](#begin)
___
### <a name="trades"/> trades

This method provides the information about the last trades.

Additionally it accepts an optional GET-parameter limit, which indicates how many orders should be displayed (150 by default). \
The maximum allowable value is 5000.

**Request example:**

```bash
curl -X GET \
  https://wex.link/api/3/trades/btc_usd
```

**Response example:**

```javascript
{
  "btc_usd": [
    {
      "type": "ask",
      "price": 103.6,
      "amount": 0.101,
      "tid": 4861261,
      "timestamp": 1370818007
    },
    {
      "type": "bid",
      "price": 103.989,
      "amount": 1.51414,
      "tid": 4861254,
      "timestamp": 1370817960
    },
    # etc
  ]
  # etc
}
```

**type**: ask – Sell, bid – Buy. \
**price**: Buy price/Sell price. \
**amount**: the amount of asset bought/sold. \
**tid**: trade ID. \
**timestamp**: UNIX time of the trade.

##### [to the top](#begin)