# <a name="begin"/> Push API

*[English](push-api.md) ∙ [Русский](push-api_ru.md)*
___

* [Contents](#main)
* [Channels](#channels)
    * [depth](#depth)
    * [trades](#trades)

## <a name="main"/> Contents

This API provides an opportunity to access trading information in real time. \
With it you can get information about changes in the order book and about transactions in real time.

This API is Provided using the service Pusher. You can read Pusher documentation [here](https://pusher.com/docs). \
App key - ee987526a24ba107824c \
Cluster - eu

___
## <a name="channels"/> Channels
### <a name="depth"/> depth

Information about changes in order books is sent on these channels.

Channels are formatted as "<pair_name>.depth", for example btc_usd.depth \
The event always has the name "depth" and is sent when new trades are made on the pair.

**Event example:**

```javascript
{
  "ask": [
    [
      "2508.179",
      "0"
    ],
    [
      "2466.842",
      "0.15"
    ],
	...
  ],
  "bid": [
	...
  ]
}
```

**ask**: changes to the sales order book. \
**bid**: changes to the buy order book.

All changes to the order book are sent as arrays, where the first value is the price, and the second change at this price. \
If the change is set to 0, it means that there are no offers in the order book at this price (the order has been canceled or fully executed).

A change other than zero reflects the current offer in the order book.

##### [to the top](#begin)
___
### <a name="trades"/> trades

These channels send information about the transactions.

Channels are formatted as "<pair_name>.trades", for example btc_usd.trades \
The event always has the name "trades" and is sent when new trades are made on the pair.

**Event example:**

```javascript
[
  [
    "buy",
    "2467.762",
    "0.02862282"
  ],
  [
    "sell",
    "2463.105",
    "0.00428728"
  ],
  # etc
]
```

**buy**: purchase transaction. \
**sell**: sale transaction.

##### [to the top](#begin)