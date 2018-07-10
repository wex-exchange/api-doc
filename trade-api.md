# <a name="begin"/> Trade API v1

*[English](trade-api.md) ∙ [Русский](trade-api_ru.md)*
___

* [Contents](#main)
* [Authentication](#auth)
* [Methods](#methods)
    * [getInfo](#getInfo)
    * [Trade](#Trade)
    * [ActiveOrders](#ActiveOrders)
    * [OrderInfo](#OrderInfo)
    * [CancelOrder](#CancelOrder)
    * [CancelOrders](#CancelOrders)
    * [TradeHistory](#TradeHistory)
    * [TransHistory](#TransHistory)
    * [CoinDepositAddress](#CoinDepositAddress)
    * [WithdrawCoin](#WithdrawCoin)
    * [CreateCoupon](#CreateCoupon)
    * [RedeemCoupon](#RedeemCoupon)
* [WEX API Libraries](#api-libraries)

 
## <a name="main"/> Contents

This API allows to trade on the exchange and receive information about the account.

To use this API, you need to create an API key.
An API key can be created in your Profile in the API Keys section. After creating an API key you’ll receive a key and a secret.
Note that the Secret can be received only during the first hour after the creation of the Key.
API key information is used for authentication.

All requests to Trade API come from the following URL: https://wex.nz/tapi

The method name is sent via the POST-parameter method.
All method parameters are sent via the POST-parameters.
All server responses are received in the JSON format.
Each request needs an authentication. You can find out more on authentication in the relevant section of this documentation.

In the case of successful request, the response will be of the following type:

```javascript
{
  "success": 1,
  "return": {
    # result
  }
}
```

Response in the case of error:
```javascript
{
  "success": 0,
  "error": "<error>"
}
````

In the case of error you may also receive a response not in the JSON format. It usually happens if API limits are exceeded or in the case of unknown errors.

##### [to the top](#begin)
___
## <a name="auth"/> Authentication

Authentication is made by sending the following HTTP headers: \
**Key** — API key. API key examples: *46G9R9D6-WJ77XOIP-XH9HH5VQ-A3XN3YOZ-8T1R8I8T*
###### API keys are created in the Profile in the API keys section.

**Sign** — Signature. POST-parameters (?nonce=1&param0=val0), signed with a Secret key using HMAC-SHA512

For successful authentication you need to send a POST-parameter **nonce** with incremental numeric value for each request.

Example of using nonce values:

1 запрос: nonce=1 \
2 запрос: nonce=2 \
3 запрос: nonce=10 \
4 запрос: nonce=10 — an error will be displayed, because nonce is equal to the previous request \
5 запрос: nonce=11 \
6 запрос: nonce=9 — an error will be displayed, because nonce is smaller than the nonce value in the API key

Minimum nonce value - 1, maximum - 4294967294.
To reset the nonce value you need to create a new key.

##### [to the top](#begin)
___
## <a name="methods"/> Methods
### <a name="getInfo"/> getInfo

Returns information about the user’s current balance, API-key privileges, the number of open orders and Server Time.
To use this method you need a privilege of the key info. \
Response is cached for 60 seconds.

**Parameters:** \
None.

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=getInfo&nonce=$NONCE"
```

**Response example:**
```javascript
{
  "success": 1,
  "return": {
    "funds": {
      "usd": 325,
      "btc": 23.998,
      "ltc": 0
       # etc
    },
    "rights": {
      "info": 1,
      "trade": 0,
      "withdraw": 0
    },
    "transaction_count": 0,
    "open_orders": 1,
    "server_time": 1342123547
  }
}
```

**funds**: Your account balance available for trading. Doesn’t include funds on your open orders. \
**rights**: The privileges of the current API key. At this time the privilege to withdraw is not used anywhere. \
**transaction_count**: Deprecated, is equal to 0. \
**open_orders**: The number of your open orders. \
**server_time**: Server time (MSK).

##### [to the top](#begin)
___
### <a name="Trade"/> Trade

The basic method that can be used for creating orders and trading on the exchange.
To use this method you need an API key privilege to trade.

You can only create limit orders using this method, but you can emulate market orders using rate parameters. 
E.g. **using rate=0.1 you can sell at the best market price**.
Each pair has a different limit on the minimum/maximum amounts, the minimum amount and the number of digits after the decimal point. All limitations can be obtained using the info method in PublicAPI v3.

**Parameters:**
 
| Parameter      | Description                                  | Assumes value                                | 
| :------------  |:---------------------------------------------|:---------------------------------------------|
| pair           | pair                                         |  "currency_currency", for example: "btc_usd" |
| type           | order type                                   |  "buy" or "sell"                             |
| rate           | the rate at which you need to buy/sell       |  numerical, for example "1.4"                |
| amount         | the amount you need to buy / sell            |  numerical, for example "7566.351"           |

You can get the list of pairs using the info method in PublicAPI v3.

```bash
NONCE="123456" # Next API-key nonce
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=Trade&nonce=$NONCE&pair=btc_usd&type=sell&rate=7566.351&amount=1.4"
```

**Response example:**

```javascript
{
  "success": 1,
  "return": {
    "received": 0.5,
    "remains": 0,
    "order_id": 0,
    "funds": {
      "usd": 325,
      "btc": 2.498,
      "ltc": 0,
      # etc
    }
  }
}
```

**received**: The amount of currency bought/sold. \
**remains**: The remaining amount of currency to be bought/sold (and the initial order amount). \
**order_id**: Is equal to 0 if the request was fully “matched” by the opposite orders, otherwise the ID of the executed order will be returned. \
**funds**: Balance after the request. 

##### [to the top](#begin)
___
### <a name="ActiveOrders"/> ActiveOrders

Returns the list of your active orders.
To use this method you need a privilege of the info key.

If the order disappears from the list, it was either executed or canceled.

**Optional Parameters:**

| Parameter      | Description  | Assumes value                                | Standard value                 |
| :--------------|:-------------|:---------------------------------------------|:-------------------------------|
| pair           | pair         |  "currency_currency", for example: "btc_usd" | all pairs                      |

You can get the list of pairs using the info method in PublicAPI v3.

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=ActiveOrders&nonce=$NONCE&pair=btc_usd"
```

**Response example:**
```javascript
{
  "success": 1,
  "return": {
    "343152": {
      "pair": "btc_usd",
      "type": "sell",
      "amount": 12.345,
      "rate": 485,
      "timestamp_created": 1342448420,
      "status": 0
    }
    # etc
  }
}
```

**Array key**: Order ID. \
**pair**: The pair on which the order was created. \
**type**: Order type, buy/sell. \
**amount**: The amount of currency to be bought/sold. \
**rate**: Sell/Buy price. \
**timestamp_created**: The time when the order was created. \
**status**: Deprecated, is always equal to 0.

##### [to the top](#begin)
___
### <a name="OrderInfo"/> OrderInfo

Returns the information on particular order.
To use this method you need a privilege of the info key.

**Parameters:**

| Parameter      | Description   | Assumes value                    | 
| :--------------|:--------------|:---------------------------------|
| order_id       | order ID      |  numerical, for example "343152" |

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=OrderInfo&nonce=$NONCE&order_id=343152"
```

**Response example:**

```javascript
{
  "success": 1,
  "return": {
    "343152": {
      "pair": "btc_usd",
      "type": "sell",
      "start_amount": 13.345,
      "amount": 12.345,
      "rate": 485,
      "timestamp_created": 1342448420,
      "status": 0
    }
  }
}
```

**Array key**: Order ID. \
**pair**: The pair on which the order was created. \
**type**: Order type, buy/sell. \
**start_amount**: The initial amount at the time of order creation. \
**amount**: The remaining amount of currency to be bought/sold. \
**rate**: Sell/Buy price. \
**timestamp_created**: The time when the order was created. \
**status**: 0 - active, 1 – executed order, 2 - canceled, 3 – canceled, but was partially executed.

##### [to the top](#begin)
___
### <a name="CancelOrder"/> CancelOrder

This method is used for order cancellation.
To use this method you need a privilege of the trade key.

**Parameters:**

| Parameter      | Description     | Assumes value                    | 
| :------------  |:----------------|:---------------------------------|
| order_id       | order ID        |  numerical, for example "343154" |

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=CancelOrder&nonce=$NONCE&order_id=343154"
```

**Response example:**
```javascript
{
  "success": 1,
  "return": {
    "order_id": 343154,
    "funds": {
      "usd": 325,
      "btc": 24.998,
      "ltc": 0,
      # etc
    }
  }
}
```

**order_id**: The ID of canceled order. \
**funds**: Balance upon request.

##### [to the top](#begin)
___
### <a name="CancelOrders"/> CancelOrders

This method is used for order/orders cancellation.
To cancel more than one order, list them with a comma in the order_id parameter. For example: "1300,1301"
To use this method you need a privilege of the trade key.

**Parameters:**

| Parameter      | Description                               | Assumes value                       | 
| :------------  |:------------------------------------------|:------------------------------------|
| order_id       | order ID or order IDs separated by commas |  numerical, for example "1300,1400" |

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=CancelOrders&nonce=$NONCE&order_id=1300,1301"
```

**Response example:**
```javascript
{
  "success": 1,
  "return": {
    "cancel_result": {
      1300: {
        "success": 1
      },
      1301: {
        "sucess": 0,
        "error": "invalid status"
      }
    },
    "funds": {
      "btc": 24.998,
      "usd": 325,
      "ltc": 0,
      # etc
    }
  }
}
```

**cancel_result**: Order cancel result. \
**funds**: Balance upon request.

##### [to the top](#begin)
___
### <a name="TradeHistory"/> TradeHistory

Returns trade history.
To use this method you need a privilege of the info key.

**Optional Parameters:**

| Parameter | Description                              | Assumes value                               | Standard value |
| :---------|:-----------------------------------------|:--------------------------------------------|:---------------|
| from      | trade ID, from which the display starts  |  numerical                                  | 0              |
| count     | the number of trades for display         |  numerical                                  | 1000           |
| from_id   | trade ID, from which the display starts  |  numerical                                  | 0              |
| end_id    | trade ID on which the display ends       |  numerical                                  | ∞              |
| order     | sorting                                  |  ASC or DESC                                | DESC           |
| since     | the time to start the display            |  UNIX time                                  | 0              |
| end       | the time to end the display              |  UNIX time                                  | ∞              |
| pair      | pair to be displayed                     |  "currency_currency" for example: "btc_usd" | all pairs      |

When using parameters since or end, the order parameter automatically assumes the value ASC.
When using the since parameter the maximum time that can displayed is 1 week.

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=TradeHistory&nonce=$NONCE&pair=btc_usd"
```

**Response example:**
```javascript
{
  "success": 1,
  "return": {
    "166830": {
      "pair": "btc_usd",
      "type": "sell",
      "amount": 1,
      "rate": 450,
      "order_id": 343148,
      "is_your_order": 1,
      "timestamp": 1342445793
    }
  }
}
```

**Array keys**: Trade ID. \
**pair**: The pair on which the trade was executed. \
**type**: Trade type, buy/sell. \
**amount**: The amount of currency was bought/sold. \
**rate**: Sell/Buy price. \
**order_id**: Order ID. \
**is_your_order**: Is equal to 1 if order_id is your order, otherwise is equal to 0. \
**timestamp**: Trade execution time.

##### [to the top](#begin)
___
### <a name="TransHistory"/> TransHistory

Returns the history of transactions.
To use this method you need a privilege of the info key.

**Optional Parameters:**

| Parameter | Description                                   | Assumes value    | Standard value |
| :---------|:--------------------------------------------- |:-----------------|:---------------|
| from      | transaction ID, from which the display starts |  numerical       | 0              |
| count     | number of transaction to be displayed         |  numerical       | 1000           |
| from_id   | transaction ID, from which the display starts |  numerical       | 0              |
| end_id    | transaction ID on which the display ends      |  numerical       | ∞              |
| order     | sorting                                       |  ASC или DESC    | DESC           |
| since     | the time to start the display                 |  UNIX time       | 0              |
| end       | the time to end the display                   |  UNIX time       | ∞              |

When using the parameters since or end, the order parameter automatically assumes the value ASC.

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=TransHistory&nonce=$NONCE"
```

**Response example:**
```javascript
{
  "success": 1,
  "return": {
    "1081672": {
      "type": 1,
      "amount": 1.00000000,
      "currency": "BTC",
      "desc": "BTC Payment",
      "status": 2,
      "timestamp": 1342448420
    }
  }
}
```

**Array keys**: Transaction ID. \
**type**: Transaction type. 1/2 - deposit/withdrawal, 4/5 - credit/debit. \
**amount**: Transaction amount. \
**currency**: Transaction currency. \
**desc**: Transaction description. \
**status**: Transaction status. 0 - canceled/failed, 1 - waiting for acceptance, 2 - successful, 3 – not confirmed. \
**timestamp**: Transaction time.

##### [to the top](#begin)
___
#### <a name="CoinDepositAddress"/> CoinDepositAddress

This method can be used to retrieve the address for depositing crypto-currency. \
To use this method, you need the info key privilege. \
At present, this method does not generate new addresses. If you have never deposited in a particular crypto-currency 
and try to retrieve a deposit address, your request will return an error, because this address has not been generated yet.

**Obligatory Parameters:**

| Parameter      | Description  | Assumes value                  | 
| :--------------|:-------------|:-------------------------------|
| coinName       | coin name    | "currency", for example: "BTC" |

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=CoinDepositAddress&nonce=$NONCE&coinName=BTC"
```

**Response example**:
```javascript
{
  "success": 1,
  "return": {
    "address": "1BARCQAEjxqp1sKneiFtBVTZzhTHBTYcN5"
  }
}
```

**address**: address for deposits.

##### [to the top](#begin)
___
### <a name="WithdrawCoin"/> WithdrawCoin

The method is designed for cryptocurrency withdrawals.

**Please note**: You need to have the privilege of the Withdraw key to be able to use this method. You can make a request for enabling this privilege by submitting a ticket to Support. \
You need to create the API key that you are going to use for this method in advance. Please provide the first 8 characters of the key (e.g. HKG82W66) in your ticket to support. We'll enable the Withdraw privilege for this key. \
When using this method, there will be no additional confirmations of withdrawal. Please note that you are fully responsible for keeping the secret of the API key safe after we have enabled the Withdraw privilege for it.

**Parameters:**

| Parameter      | Description        | Assumes value                  | 
| :------------  |:-------------------|:-------------------------------|
| coinName       | coin name          | "currency", for example: "BTC" |
| amount         | withdrawal amount  | numerical                      |
| address        | withdrawal address | "address"                      |

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=WithdrawCoin&nonce=$NONCE&coinName=BTC&amount=0.31&address=1BARCQAEjxqp1sK...eiFtBVTZzhTHBTYcN5"
```

**Response example**:
```javascript
{
  "success": 1,
  "return": {
    "tId": 37832629,
    "amountSent": 0.009,
    "funds": {
      "usd": 325,
      "btc": 24.998,
      "ltc": 0,
      # etc
    }
  }
}
```

**tId**: Transaction ID. \
**amountSent**: The amount sent including commission. \
**funds**: Balance after the request.

##### [to the top](#begin)
___
### <a name="CreateCoupon"/> CreateCoupon

This method allows you to create Coupons.

**Please note**: In order to use this method, you need the Coupon key privilege. You can make a request to enable it by submitting a ticket to Support. \
You need to create the API key that you are going to use for this method in advance. Please provide the first 8 characters of the key (e.g. HKG82W66) in your ticket to support. We'll enable the Coupon privilege for this key. \
You must also provide us the IP-addresses from which you will be accessing the API. \
When using this method, there will be no additional confirmations of transactions. Please note that you are fully responsible for keeping the secret of the API key safe after we have enabled the Withdraw privilege for it.

**Parameters:**

| Parameter       | Description                                   | Assumes value                  | 
|:---------------|:-----------------------------------------------|:-------------------------------|
| coinName       | coin name                                      | "currency", for example: "BTC" |
| amount         | withdrawal amount                              | numerical                      |
| receiver       | name of user who is allowed to redeem the code | username                       |

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=CreateCoupon&nonce=$NONCE&coinName=BTC&amount=0.11&receiver=admin"
```

**Response example**:
```javascript
{
  "success": 1,
  "return": {
    "coupon": "WEXUSD69AA4BBX1UAZ32BPLKBR0QZTX5AENVNFWNZHNQDZ",
    "transID": 37832629,
    "funds": {
      "usd": 325,
      "btc": 24.998,
      "ltc": 0,
      # etc
    }
  }
}
```

**coupon**: Generated coupon. \
**transID**: Transaction ID. \
**funds**: Balance after the request.

##### [to the top](#begin)
___
### <a name="RedeemCoupon"/> RedeemCoupon

This method is used to redeem coupons.

**Please note**: In order to use this method, you need the Coupon key privilege. You can make a request to enable it by submitting a ticket to Support.\
You need to create the API key that you are going to use for this method in advance. 
Please provide the first 8 characters of the key (e.g. HKG82W66) in your ticket to support. We'll enable the Coupon privilege for this key. \
You must also provide us the IP-addresses from which you will be accessing the API. \
When using this method, there will be no additional confirmations of transactions. 
Please note that you are fully responsible for keeping the secret of the API key safe after we have enabled the Withdraw privilege for it.

**Parameters:**

| Parameter | Description | Assumes value       | 
|:----------|:------------|:--------------------|
| coupon    | coupon      | WEXUSD... (example) |

**Request example:**

```bash
SIGN="1382324e3d24e8579787135...f2107ffe61c9b8bb7" # Your API-key sign
KEY="J2UF00BF-OZ8X02EW-F7W37LYM-VCLWIBIU-XQG2HO1W" # Your API-key
NONCE="123456" # Next API-key nonce

curl -X POST \
  "https://wex.nz/tapi"\
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Key: $KEY" \
  -H "Sign: $SIGN" \
  -d "method=RedeemCoupon&nonce=$NONCE&coupon=WEXUSD...NZHNQDZ"
```

**Response example**:
```javascript
{
  "success": 1,
  "return": {
    "couponAmount": "1",
    "couponCurrency": "USD",
    "transID": 37832629,
    "funds": {
      "usd": 325,
      "btc": 24.998,
      "ltc": 0,
      # etc
    }
  }
}
```

**couponAmount**: The amount that has been redeemed. \
**couponCurrency**: The currency of the coupon that has been redeemed. \
**transID**: Transaction ID. \
**funds**: Balance after the request.

##### [to the top](#begin)
___
## <a name="api-libraries"/> WEX API Libraries

Most of the examples have been created by our users. We are not responsible for their performance and do not provide support for them. You can use them at your own risk.

If you want to see your example here, please create a ticket titled "API example for documentation".

*Python*: https://github.com/madmis/wexapi by madmis \
*C#:* https://github.com/Falweek/WexAPI by Falweek \
*Excel/VBA:* https://github.com/krijnsent/crypto_vba by Koen Rijnsent

**Deprecated**: 

*PHP:* http://pastebin.com/8fbMCguM \
*PHP:* https://github.com/marinu666/PHP-btce-api \
*Python:* http://pastebin.com/ec11hxcP \
*Python:* https://github.com/alanmcintyre/btce-api \
*Python:* https://github.com/t0pep0/btc-e.api.python \
*Python:* https://github.com/acidvegas/btc-e \
*Java:* http://pastebin.com/jyd9tACF \
*Java:* https://github.com/alexandersjn/btc_e_assist_api \
*Java:* https://github.com/Yocairo/BTCE_API_Lib \
*C#:* https://github.com/DmT021/BtceApi \
*C#:* https://github.com/multiprogramm/WexAPI \
*С# .NET:* https://bitbucket.org/EldarG/btceapi \
*C++/CLI:* http://pastebin.com/YvxmCRL9 \
*C++11:* https://github.com/halcyonx/btc-e-API \
*VB.NET:* http://pastebin.com/JmJZSsd7 \
*Objective-C:* https://github.com/backmeupplz/BTCEBot \
*Ruby:* https://github.com/cgore/ruby-btce \
*Swift:* https://bitbucket.org/MaximSh/yawzabot-btce-bot \
*Go:* https://github.com/alexpantyukhin/btceapi \
*Node.js:* https://www.npmjs.com/package/btc-e3

##### [to the top](#begin)