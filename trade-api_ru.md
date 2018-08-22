# <a name="begin"/> Trade API v1

*[English](trade-api.md) ∙ [Русский](trade-api_ru.md)*
___

* [Оглавление](#main)
* [Аутентификация](#auth)
* [Методы](#methods)
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
    

 
## <a name="main"/> Оглавление

Данный API предоставляет возможность торговать на бирже и получать информацию об аккаунте.

Для использования данного API необходимо создать API-ключ.
API-ключ можно создать в профиле, раздел API-ключи. После создания API-ключа вам выдается ключ и секрет.
Обратите внимание на то, что получить секрет ключа можно только в первый час после создания ключа.
Данные API-ключа используются для аутентификации.

Все запросы к TradeAPI идут по следующему URL: https://wex.nz/tapi

Имя метода отправляется посредством POST-параметра method.
Все параметры методов отправляется через POST-параметры.
Все ответы от сервера приходят в формате JSON.
Для каждого запроса необходима аутентификация. Как произвести аутентификацию можно прочитать в соответствующем разделе данной документации.

При успешном выполнении запроса приходит ответ типа:

```javascript
{
  "success": 1,
  "return": {
    # результат
  }
}
```

Ответ при ошибке:

```javascript
{
  "success":0,
  "error": "<ошибка>"
}
```

Так же при ошибке может придти ответ не в JSON формате. Как правило это случается при превышении лимитов к API либо при непредвиденных ошибках.

##### [В начало](#begin)
___
## <a name="auth"/> Аутентификация

Аутентификация происходит посредством отправки следующих HTTP-заголовков: \
**Key** — API-ключ. Пример API-ключа: *46G9R9D6-WJ77XOIP-XH9HH5VQ-A3XN3YOZ-8T1R8I8T*
###### API-ключ создается в профиле, раздел API-ключи.

**Sign** — Подпись. POST-параметры (?nonce=1&param0=val0), подписанные секретным ключом с помощью HMAC-SHA512

Так же для успешной аутентификации необходимо посылать POST-параметр **nonce** с инкрементым каждый запрос числовым значением.

Пример использования значения nonce:

1 запрос: nonce=1 \
2 запрос: nonce=2 \
3 запрос: nonce=10 \
4 запрос: nonce=10 — выдаст ошибку, потому что nonce равен прошлому запросу \
5 запрос: nonce=11 \
6 запрос: nonce=9 — выдаст ошибку, потому что nonce меньше, чем значение nonce на API-ключе

Минимальное значение nonce - 1, максимальное - 4294967294.
Для обнуления значения nonce необходимо создать новый ключ.

##### [В начало](#begin)
___
## <a name="methods"/> Methods
### <a name="getInfo"/> getInfo

Возвращает информацию о текущем балансе пользователя, привилегиях API-ключа, количество открытых ордеров и время сервера.
Для использования данного метода необходима привилегия ключа info.
Ответ кэшируется на 60 секунд.

**Параметры:** \
Нет.

**Пример запроса:**

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

**Пример ответа:**
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

**funds**: Баланс аккаунта доступный к использованию. Не включает деньги на ваших открытых ордерах. \
**rights**: Привилегии текущего API-ключа. Привилегия withdraw в данный момент нигде не используется. \
**transaction_count**: Устарело, всегда 0. \
**open_orders**: Количество ваших открытых ордеров. \
**server_time**: Время сервера (MSK).

##### [В начало](#begin)
___
### <a name="Trade"/> Trade

Основной метод используя который можно создавать ордера и торговать на бирже.
Для использования данного метода необходима привилегия ключа trade.

С помощью данного метода можно создавать только лимит ордера, но можно эмулировать маркет ордера с помощью параметра rate. Например, **используя rate=0.1
при продаже можно продать по самой выгодней цене по рынку**.
У каждой пары разные ограничения на минимальную/максимальную суммы, минимальное количество и количество знаков после запятой. Все ограничения можно
получить с помощью метода info в PublicAPI v3.

**Параметры:**
 
| Параметр       | Описание                                                                      | Принимает значение                             | 
| :------------  |:------------------------------------------------------------------------------|:-----------------------------------------------|
| pair           | пара                                                                          |  Торгуемая пара, например: «btc_usd»           |
| type           | сторона сделки (покупка или продажа)                                          |  «buy» или «sell»                              |
| rate           | курс по которому необходимо купить/продать                                    |  число, например «7566.351»                    |
| mode           | тип ордера                                                                    |  "market" or "limit" (**limit** по умолчанию ) |
| amount         | количество которое необходимо купить/продать (обязательный для limit ордеров) |  число, например «1.4»                         |

Список пар можно получить используя метод info в PublicAPI v3.

**Пример запроса:**

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

**Пример ответа:**

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

**received**: Сколько валюты куплено/продано. \
**remains**: Сколько валюты осталось купить/продать (и на сколько был создан ордер). \
**order_id**: Имеет значение 0 если запрос был полностью удовлетворен встречными ордерами, в противном случае возращает идентификатор созданного
ордера. \
**funds**: Баланс после запроса. 

##### [В начало](#begin)
___
### <a name="ActiveOrders"/> ActiveOrders

Возращает список ваших активных ордеров.
Для использования данного метода необходима привилегия ключа info.

Если ордер пропадает из списка, то он был исполнен или отменен.

**Необязательные параметры:**

| Параметр       | Описание  | Принимает значение                   | Стандартное значение                  |
|:---------------|:----------|:-------------------------------------|:--------------------------------------|
| pair           | пара      | Торгуемая пара, например: «btc_usd»  | все пары                              |

Список пар можно получить используя метод info в PublicAPI v3.

**Пример запроса:**

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

**Пример ответа:**
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

**Ключи массива**: Идентификатор ордера. \
**pair**: Пара на которой был создан ордер. \
**type**: Тип ордера buy/sell. \
**amount**: Сколько осталось купить/продать. \
**rate**: Цена покупки/продажи. \
**timestamp_created**: Время когда был создан ордер. \
**status**: Устарел, всегда 0.

##### [В начало](#begin)
___
### <a name="OrderInfo"/> OrderInfo

Возращает информацию о конкретном ордере.
Для использования данного метода необходима привилегия ключа info.

**Параметры:**

| Параметр       | Описание                                     | Принимает значение                   | 
| :------------  |:---------------------------------------------|:-------------------------------------|
| order_id       | идентификатор ордера                         |  число, например «343152»            |

**Пример запроса:**

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

**Пример ответа:**

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

**Ключ массива**: Идентификатор ордера. \
**pair**: Пара на которой был создан ордер. \
**type**: Тип ордера, buy/sell. \
**start_amount**: Начальная сумма которая была при создании ордера. \
**amount**: Сколько осталось купить/продать. \
**rate**: Цена покупки/продажи. \
**timestamp_created**: Время когда был создан ордер. \
**status**: 0 - активен, 1 - исполненный ордер, 2 - отмененный, 3 - отмененный, но был частично исполнен.

##### [В начало](#begin)
___
### <a name="CancelOrder"/> CancelOrder

Метод предназначен для отмены ордера.
Для использования данного метода необходима привилегия ключа trade.

**Параметры:**

| Параметр       | Описание                                     | Принимает значение                   | 
| :------------  |:---------------------------------------------|:-------------------------------------|
| order_id       | идентификатор ордера                         |  число, например «343154»            |

**Пример запроса:**

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

**Пример ответа:**
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

**order_id**: Идентификатор отмененного ордера. \
**funds**: Баланс после запроса.

##### [В начало](#begin)
___
### <a name="CancelOrders"/> CancelOrders

Метод предназначен для отмены ордера или множества ордеров.
Для отмены более одного ордера, перечислите их через запятую в параметре order_id. Например: «1300,1301»
Для использования данного метода необходима привилегия ключа trade.

**Параметры:**

| Параметр       | Описание                                       | Принимает значение                   | 
| :------------  |:-----------------------------------------------|:-------------------------------------|
| order_id       | идентификатор ордера или ордеров через запятую |  число, например «1300,1301»         |

**Пример запроса:**

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

**Пример ответа:**
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

**cancel_result**: Результат отмены ордера. \
**funds**: Баланс после запроса.

##### [В начало](#begin)
___
### <a name="TradeHistory"/> TradeHistory

Возвращает историю сделок.
Для использования данного метода необходима привилегия ключа info.

**Необязательные параметры:**

| Параметр | Описание                                 | Принимает значение                   | Стандартное значение  |
| :--------|:-----------------------------------------|:-------------------------------------|:----------------------|
| from     | номер сделки, с которой начинать вывод   |  число                               | 0                     |
| count    | количество сделок на вывод               |  число                               | 1000                  |
| from_id  | id сделки, с которой начинать вывод      |  число                               | 0                     |
| end_id   | id сделки, на которой заканчивать вывод  |  число                               | ∞                     |
| order    | сортировка                               |  ASC или DESC                        | DESC                  |
| since    | с какого времени начинать вывод          |  UNIX time                           | 0                     |
| end      | на каком времени заканчивать вывод       |  UNIX time                           | ∞                     |
| pair     | пара, по которой выводить сделки         |  Торгуемая пара, пример: «btc_usd»   | все пары              |

При использовании параметров since или end, параметр order автоматически принимает значение ASC.
При использовании параметра since максимальная дата по которой можно получить историю - неделю назад.

**Пример запроса:**

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

**Пример ответа:**
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

**Ключи массива**: Идентификатор сделки. \
**pair**: Пара на которой была сделка. \
**type**: Тип сделки, buy/sell. \
**amount**: Количество купленного/проданного. \
**rate**: Цена покупки/продажи. \
**order_id**: Идентификатор ордера. \
**is_your_order**: Значение 1 если order_id является вашим ордером, в противном случае 0. \
**timestamp**: Время сделки.

##### [В начало](#begin)
___
### <a name="TransHistory"/> TransHistory

Возвращает историю транзакций.
Для использования данного метода необходима привилегия ключа info.

**Необязательные параметры:**

| Параметр | Описание                                     | Принимает значение                   | Стандартное значение  |
| :--------|:---------------------------------------------|:-------------------------------------|:----------------------|
| from     | номер тразнакции, с которой начинать вывод   |  число                               | 0                     |
| count    | количество транзакций на вывод               |  число                               | 1000                  |
| from_id  | id тразнакции, с которой начинать вывод      |  число                               | 0                     |
| end_id   | id тразнакции, на которой заканчивать вывод  |  число                               | ∞                     |
| order    | сортировка                                   |  ASC или DESC                        | DESC                  |
| since    | с какого времени начинать вывод              |  UNIX time                           | 0                     |
| end      | на каком времени заканчивать вывод           |  UNIX time                           | ∞                     |

При использовании параметров since или end, параметр order автоматически принимает значение ASC.

**Пример запроса:**

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

**Пример ответа:**
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

**Ключи массива**: Идентификатор транзакции. \
**type**: Тип транзакции. 1/2 – ввод/вывод, 4/5 – приход/расход. \
**amount**: Сумма транзакции. \
**currency**: Валюта транзакции. \
**desc**: Описание транзакции. \
**status**: Статус транзакции: 0 – отклонено/неуспешно, 1 – ожидание подтверждения, 2 – успешно, 3 – не подтверждено. \
**timestamp**: Время транзакции.

##### [В начало](#begin)
___
#### <a name="CoinDepositAddress"/> CoinDepositAddress

Метод предназначен для получения депозитного адреса криптовалюты.
Для использования данного метода необходима привилегия ключа info.
В данный момент метод не генерирует новые адреса. Поэтому произойдет ошибка, если вы запросите адрес по валюте, депозитный адрес для которой вы ранее не генерировали.

**Обязательные параметры:**

| Параметр       | Описание        | Принимает значение              | 
| :------------  |:----------------|:--------------------------------|
| coinName       | название валюты | «криптовалюта» (пример: «BTC»)  |

**Пример запроса:**

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

**Пример ответа**:
```javascript
{
  "success": 1,
  "return": {
    "address": "1BARCQAEjxqp1sKneiFtBVTZzhTHBTYcN5"
  }
}
```

**address**: адрес для депозитов.

##### [В начало](#begin)
___
### <a name="WithdrawCoin"/> WithdrawCoin

Метод предназначен для вывода криптовалюты. 
Для использования данного метода необходима привилегия ключа «withdraw», которую пока можно включить только после обращения в Службу поддержки.

Для подключения привилегии необходимо:

1. Создать API-ключ с белым списком IP;
2. Обратиться с запросом в Службу поддержки, в котором сообщить:
      - название привилегии ключа;
      - первые 8 символов API-ключа (напр. HKG82W66);
      - причины подключения привилегии.

**Внимание!** 
После включения данной привилегии можно будет вывести средства с вашего аккаунта без дополнительного подтверждений; 
Как только вам будет подключен данный метод, двухфакторная аутентификация больше не будет самой сильной защитой ваших активов;
WEX не сможет оказать помощь в возвращении ваших активов, в случае их кражи.


**Параметры:**

| Параметр       | Описание         | Принимает значение                                            | 
| :------------  |:-----------------|:--------------------------------------------------------------|
| coinName       | название валюты  | «криптовалюта», например: «BTC»                               |
| amount         | сумма для вывода | число, например «0.31»                                        |
| address        | адрес для вывода | адрес кошелька, например «1BARCQAEjxqp1sK...eiFtBVTZzhTHBTYcN5» |

**Пример запроса:**

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

**Пример ответа**:
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

**tId**: Идентификатор транзакции. \
**amountSent**: Фактически отправленная сумма, учитывая комиссию. \
**funds**: Баланс после запроса.

##### [В начало](#begin)
___
### <a name="CreateCoupon"/> CreateCoupon

Метод предназначен для создания купонов.

**Обратите внимание**: Для использования данного метода необходима привилегия ключа coupon, которую на данный момент можно включить только через
саппорт. \
Необходимо заранее создать API-ключ с белым списком IP, который вы будете использовать для данного метода и сообщить в тикете первые 8 символов ключа
(напр. HKG82W66) на котором необходимо подключить привилегию coupon. \
Обратите внимание - белый список IP является обязательным условием для получения привилегии coupon. В противном случае вам будет отказано в получении
данной привилегии. \
Так как никакого дополнительного подтверждения при использовании данного метода не нужно, после включения данной привилегии на вашем API-ключе вы
несете полную ответственность за сохранность секрета к данному ключу.

**Параметры:**

| Параметр       | Описание                                      | Принимает значение                   | 
|:---------------|:----------------------------------------------|:-------------------------------------|
| coinName       | название валюты                               | «криптовалюта», пример: «BTC»        |
| amount         | сумма для вывода                              | число                                |
| receiver       | имя пользователя, которому предназначен купон | логин пользователя, например «admin» |

**Пример запроса:**

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

**Пример ответа**:
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

**coupon**: Сгенерированный купон. \
**transID**: Идентификатор транзакции. \
**funds**: Баланс после запроса.

##### [В начало](#begin)
___
### <a name="RedeemCoupon"/> RedeemCoupon

Метод предназначен для погашения купонов.

**Обратите внимание**: Для использования данного метода необходима привилегия ключа coupon, которую на данный момент можно включить только через
саппорт. \
Необходимо заранее создать API-ключ с белым списком IP, который вы будете использовать для данного метода и сообщить в тикете первые 8 символов ключа
(напр. HKG82W66) на котором необходимо подключить привилегию coupon. \
Обратите внимание - белый список IP является обязательным условием для получения привилегии coupon. В противном случае вам будет отказано в получении
данной привилегии. \
Так как никакого дополнительного подтверждения при использовании данного метода не нужно, после включения данной привилегии на вашем API-ключе вы
несете полную ответственность за сохранность секрета к данному ключу.

**Параметры:**

| Параметр  | Описание | Принимает значение | 
|:----------|:---------|:-------------------|
| coupon    | купон    | WEXUSD... (пример) |

**Пример запроса:**

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

**Пример ответа**:
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

**couponAmount**: Сумма которая была погашена. \
**couponCurrency**: Валюта купона который был погашен. \
**transID**: Идентификатор транзакции. \
**funds**: Баланс после запроса.

##### [В начало](#begin)
___
## <a name="api-libraries"/> WEX API Libraries

Большинство примеров были написаны нашими пользователями. Мы не несем ответственность за их работоспособность и не осуществляем поддержку по ним. Вы
используете их на свой страх и риск. 

Если вы хотите видеть свой пример здесь, пожалуйста создайте тикет с заголовком «API пример для документации».

*Python*: https://github.com/madmis/wexapi by madmis \
*C#:* https://github.com/Falweek/WexAPI by Falweek \
*C#:* https://github.com/multiprogramm/WexAPI by multiprogramm\
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
*С# .NET:* https://bitbucket.org/EldarG/btceapi \
*C++/CLI:* http://pastebin.com/YvxmCRL9 \
*C++11:* https://github.com/halcyonx/btc-e-API \
*VB.NET:* http://pastebin.com/JmJZSsd7 \
*Objective-C:* https://github.com/backmeupplz/BTCEBot \
*Ruby:* https://github.com/cgore/ruby-btce \
*Swift:* https://bitbucket.org/MaximSh/yawzabot-btce-bot \
*Go:* https://github.com/alexpantyukhin/btceapi \
*Node.js:* https://www.npmjs.com/package/btc-e3

##### [В начало](#begin)
