# Документаций по Kuna API v3

API организовано по принципам REST. \
Все методы делятся на публичные (public) и приватные (private).

Для того что бы осуществлять приватные запросы, вы должны быть зерегистрированным пользователем на сайте [kuna.io](https://kuna.io), и иметь специальный **API Token**, который состоит из публичного (`publicKey`) и приватного (`privateKey`) ключей.

Для публичных методов ключ и подпись запроса не нужны.

API Token можно получить у себя в кабинете, по ссылке https://kuna.io/api_tokens.

## Библиотеки для работы с API v3
Наши проекты находятся в стадии разработки. Мы их выпустим сразу как только будем уверены что они безопасны и стабильны.

**Проекты от комьюнити** \
На свой страх и риск, вы можете использовать реализации API v3 от комъюнити.

* JavaScript библиотека: [CoinWizard/kuna-sdk](https://github.com/CoinWizard/kuna-sdk)


## Схема и сериализация данных в API

### 1) Запросы
Доступ к данным осуществляется через стандартные HTTPS запросы в кодировке UTF-8. Данные отправляется и принимаются в формате JSON.


### 2) Домен
Для всех методов, публичных или приватных, используеся один домен
```bash
https://api.kuna.io
```


К примеру, URL для получения актуального стакана заявок
```bash
https://api.kuna.io/v3/book/{market}
```
где `{market}` это ключ пары, `btcuah`, `kunbtc` или другой


### 3) Заголовки (Headers)
Заголовки, которые нужны для успешного запроса.

| Заголовок     | Описание                       |
|---------------|--------------------------------|
| Accept        | Должен быть `application/json` |
| Content-Type  | Только для запросов где есть тело. Должен быть `application/json` |
| Kun-Nonce     | Метка времени запроса. Указывается в формате Unix Time Stamp в милисекундах (ms). Только для приватных методов. |
| Kun-ApiKey    | Публичный ключ вашего API Token. Только для приватных методов. |
| Kun-Signature | Подпись запроса. Только для приватных методов. |



### 4) Подпись для приватных запросов
Подпись нужно указать в заголовке запроса под ключем `Kun-Signature`.

Она формируется по формуле
```
HEX(
    HMAC-SHA384(
      {apiPath} + {nonce} + JSON({body}),
      {privateKey}
    )
)
```

Где,

| Значение        | Описание                                     |
| --------------- | -------------------------------------------- |
| `{apiPath}`     | Метод, к примеру `/v3/auth/kuna_codes/count` |
| `{nonce}`       | Метка времени запроса. Указывается в формате Unix Time Stamp в милисекундах (ms). Это же значние должно быть и в заголовке под ключем `Kun-Nonce` |
| `{body}`        | Данные, которые передаются в теле запроса. Должны быть в формате JSON. В случае `GET` запросов, когда тело не передается, используется пустой JSON объект - `{}`. |
| `{privateKey}`  | Приватный ключ вашего API Token              |


#### Пример подписи с использованием JavaScript

```javascript
const crypto = require('crypto');

const publicKey = '';
const privateKey = '';

const apiPath = '/v3/auth/kuna_codes/issued-by-me';
const nonce = new Date().getTime();
const body = {};

const signatureString = `${apiPath}${nonce}${JSON.stringify(body)}`;

const signature = crypto
    .createHmac('sha384', apiSecret)
    .update(signatureString)
    .digest('hex');

console.log(signature); // вывидет подпись запроса в HEX формате
```


### 5) Пример приватного запроса с cURL

```bash
curl -X GET \
    https://api.kuna.io/v3/auth/kuna_codes/issued-by-me \
    -H 'Accept: application/json' \
    -H 'Kun-Nonce: 1560007410000' \
    -H 'Kun-ApiKey: vPNvF9ArqV4HqMzpAIyaLvToJJ1x1rfRZP5jNrQf' \
    -H 'Kun-Signature: 0d34c19a5125d68fe2e7fb3a3b58e162cc53e166d1e7790deb5d79f6cb04aad1d5e01daeb3ecf8871c3b767a8ea289ea'
```
###### Не пытайтесь использовать этот API Key. Он для примера :)



## Публичные методы

### 1) Время на сервере
Этот метод возвращает текущее время на сервере Kuna. Полезно, если нужно проверить доступность API.

```
GET /v3/timestamp
```

**Пример ответа**
```bash
{
  "timestamp": 1560005994,
  "timestamp_miliseconds": 1560005994692
}
```


### 2) Список доступных валют
Вернет список валют, доступных на Kuna.

```
GET /v3/currencies
```

**Пример ответа**
```bash
[
  {
    "id": 2,
    "code": "btc",
    "name": "Bitcoin",
    "has_memo": false,
    "icons": {
      "std": "https://kuna.io/icons/currency/std/btc.svg",
      "xl": "https://kuna.io/icons/currency/xl/btc.svg",
      "png_2x": "https://kuna.io/icons/currency/png/BTC@2x.png",
      "png_3x": "https://kuna.io/icons/currency/png/BTC@3x.png"
    },
    "coin": true,
    "explorer_link": "https://www.blockchain.com/btc/tx/#{txid}",
    "sort_order": 5,
    "precision": {
      "real": 8,
      "trade": 6
    }
  }
]
```

### 3) Курс валют на бирже
Этот метод позволяет вам получить курсы валют в USD, UAH, BTC, EUR и в рублике, для каждой из существующих валют на Kuna.

```
GET /v3/exchange-rates/{currency}
```

**Примеры запроса**
```bash
# вернет объект с курсами для USD
https://api.kuna.io/v3/exchange-rates/usd

# если не указать currency, то метод вернет массив всех доступных валют с их курсами
https://api.kuna.io/v3/exchange-rates
```

**Пример ответа для USD**
```bash
{
  "currency": "usd",  # ключ валюты
  "usd": 1,           # курс к доллару соединенных штатов
  "uah": 26.595721,   # курс к гривне Украины
  "btc": 0.0001276,   # курс к Bitcoin
  "eur": 1.13,        # курс к Евро
  "rub": 0.0153752    # курс к рублику
}
```


### 4) Рынки
Этот метод возвращает список валютных пар (рынков), которые доступны для торговли.

```
GET /v3/markets
```

**Пример ответа**
```bash
[
  {
    "id": "btcusdt",
    "base_unit": "btc",
    "quote_unit": "usdt",
    "base_precision": 6,
    "quote_precision": 2,
    "display_precision": 1,
    "price_change": -1.89
  }
]
```


### 5) Последние данные по рынку (тикеры)
Этот метод возвращает тикеры по всем рынкам, либо по конкретному.

Тикер представляет собой общий обзор состояния рынка. Он показывает текущую лучшую цену спроса и предложения, а также цену последней сделки. Он также включает в себя информацию, такую как дневной объем и сколько цена изменилась за последний день.

```
GET /v3/tickers?symbols={symbols}
```

**Примеры запроса**
```bash
# Вернет информацию по btcuah, kunbtc и ethuah
curl https://api.kuna.io/v3/tickers?symbols=btcuah,kunbtc,ethuah

# Вернет информацию только по рынку btcuah
curl https://api.kuna.io/v3/tickers?symbols=btcuah

# Вернет информацию по всем активным рынкам
curl https://api.kuna.io/v3/tickers?symbols=ALL
```

**Пример ответа**
```bash
[
  [
    "btcuah",   # символ рынка
    208001,     # цена BID
    11200693,   # размер стакана BID
    208499,     # цена ASK
    29.255569,  # размер стакана ASK
    5999,       # изменение цены за 24 часа в котируемой валюте 
    -2.8,       # изменение цены за 24 часа в процентах
    208001,     # последняя цена
    11.3878,    # объем торгов за 24 часа в базовой валюте
    215301,     # максимальная цена за 24 часа
    208001      # минимальная цена за 24 часа
  ]
]
```

### 6) Биржевой стакан
Метод для биржевого стакана позволяет вам отслеживать состояние книги заказов.

```
GET /v3/book/{symbol}
```

**Пример ответа**
```bash
[
  [
    208008,    # цена
    0.048076,  # обьем
    1          # количество заявок
  ],
  [
    212220,    # цена
    -0.091058, # обьем
    1          # количество заявок
  ]
]
```

Обратите внимание весь стакан для BID и ASK возвращается одним массивом.

Если обьем > 0, то имеем данные для BID стакана. \
Если обьем < 0, то данные для ASK стакана.


### 6) История сделок

😱 _Coming soon_ \
Этот метод еще в разработке и появится так скоро как мы сможем его реализовать.


## Приватные методы
Каждый приватный метод требует подписи, механизм которой описан в \
**Схема и сериализация данных в API > Подпись для приватных методов**.


### 1) Данные аккаунта
Данный метод возвращает данные об аккаунте, такие как Email, Kuna-ID, активна ли двухфакторная аутентификация.

```bash
POST /v3/auth/me
```

**Пример ответа**
```bash
{
  "email": "my-email@gmail.com",
  "kunaid": "kunaid-XXXXXXXXXXXX",
  "two_factor": true,
  "withdraw_confirmation": false,
  "public_keys": {
    "deposit_sdk_uah_public_key": "pk_live_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "deposit_sdk_usd_public_key": "pk_live_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "deposit_sdk_rub_public_key": "pk_live_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  },
  "announcements": false
}
```


### 2) Баланс аккаунта
Метод баланса аккаунта возвращает массив всех балансов всех валют вашего акаунта на Kuna вместе с информацией о достпных средствах.

```bash
POST /v3/auth/r/wallets
```

**Пример ответа**
```bash
[
  [
    'exchange', 
    'USD',      # символ валюты
    1208370,    # полный баланс аккаунта
    null,       
    1208370     # доступные средства
  ],
  [
    'exchange',
    'KUN',
    800000,
    null,
    760000
  ],
]
```


### 3) Создать ордер

😱 _Coming soon_ \
Покачто, этот метод не доступен. Но вы можете создать заявку, используя API v2.


### 4) Список активных ордеров
Этот метод вернет список всех ваших активных ордеров

```bash
POST /v3/auth/r/orders/{?markets}
```

**Примеры запроса**
```bash
# Вернет список всех активных ордеров
curl -X POST \
  "https://api.kuna.io/v3/auth/r/orders" \
  -d "{}"

# Вернет список активных ордеров только на паре BTC/UAH
curl -X POST \
  "https://api.kuna.io/v3/auth/r/orders/btcuah" \
  -d "{}"
```

**Пример ответа**
```bash
[
  [
    100279610,      # ID ордера
    null,           # не используется
    null,           # не используется
    'xrpuah',       # код рынка (валютной пары)
    1560089091000,  # время создания (timestamp в милисекундах)
    1560089091000,  # время обновления (timestamp в милисекундах)
    '45.0',         # объем ордера
    '-45.0',        # изначальный обьем ордера
                    # если значение < 0 то ордер на продажу
                    # если > 0, то ордер на покупку
    'LIMIT',        # тип ордера (LIMIT or MARKET)
    null,           # не используется
    null,           # не используется
    null,           # не используется
    null,           # не используется
    'ACTIVE',       # статус ордера
    null,           # не используется
    null,           # не используется
    '14.0',         # цена ордера
    '0.0'           # средняя сделок по ордеру
  ]
]
```


### 5) Список исполненных ордеров
Этот метод вернет список исполненных ордеров

```bash
POST /v3/auth/r/orders/{?markets}/hist
```

**Примеры запросов**
```bash
# Вернет список исполненных ордеров ордеров для любой пары
curl -X POST \
  "https://api.kuna.io/v3/auth/r/orders/hist" \
  -d "{}"

# Вернет список исполненных ордеров только для пары BTC/UAH
curl -X POST \
  "https://api.kuna.io/v3/auth/r/orders/btcuah/hist" \
  -d "{}"
```

**Параметры тела запроса**

| Параметр | Обязательный | Описание           |
|----------|--------------|--------------------|
| start    | нет          | Показать ордера с этой даты. Временя в миллисекундах. По умолчанию 2 недели назад. |
| end      | нет          | Показать ордера до этой даты. Временя в миллисекундах. По умолчанию текущее время.  |
| limit    | нет          | Количество записей. По умолчанию 25 |
| sort     | нет          | 1 или -1. Порядок сортировки ордеров. По умолчанию ордера возвращаются в порядке убывания. |


**Пример ответа**
```bash
[
  [
    100279610,      # ID ордера
    null,           # не используется
    null,           # не используется
    'xrpuah',       # код рынка (валютной пары)
    1560089091000,  # время создания (timestamp в милисекундах)
    1560089091000,  # время обновления (timestamp в милисекундах)
    '45.0',         # объем ордера
    '-45.0',        # изначальный обьем ордера
                    # если значение < 0 то ордер на продажу
                    # если > 0, то ордер на покупку
    'LIMIT',        # тип ордера (LIMIT or MARKET)
    null,           # не используется
    null,           # не используется
    null,           # не используется
    null,           # не используется
    'EXECUTED',     # статус ордера
    null,           # не используется
    null,           # не используется
    '14.0',         # цена ордера
    '13.0'          # средняя сделок по ордеру
  ]
]
```


### 6) Список сделок по ордеру

```bash
POST /v3/auth/r/order/{market}:{order_id}/trades
```


**Пример запроса**
```bash
# Вернет список сделок для ордера ID 10000000 в паре BTC/UAH
curl -X POST \
  "https://api.kuna.io/v3/auth/r/order/btcuah:10000000/trades" \
  -d "{}"
```


**Параметры запроса**

| Параметр         | Описание                                  |
|------------------|-------------------------------------------|
| market           | Обязательный параметр. Код валютной пары. |
| order_id         | Обязательный параметр. ID ордера.         |


**Пример ответа**
```bash
[
  [
    2538815,            # ID сделки
    'ethuah',           # валютная пара
    1559667470000,      # время сделки
    98140806,           # ID ордера
    '0.032925',         # сумма сделки
    '6798.0',           # курс сделки
    null,               # не используется
    null,               # не используется
    -1,                 # 1 - maker, -1 taker
    '0.0000823125',     # размер комиссии
    'eth'               # валюта комиссии
  ]
]
```


### 7) Отменить ордер
Этот метод позволяет отменить один или несколько активных ордеров.

```bash
POST /v3/order/cancel
```

**Пример запроса**
```bash
# Отменит ордер с ID 1000000 
curl -X POST \ 
  "https://api.kuna.io/v3/order/cancel"
  -d '{"order_id": 1000000}'

# А так можно отменить сразу несколько ордеров, ID 1000000 и ID 1000001
curl -X 'POST' \
  "https://api.kuna.io/v3/order/cancel/multy" \
  -d '{"order_ids": [1000000, 1000001]}'
```


**Пример ответа**
```bash
{
  "id": 100279610,                  # ID ордера
  "side": "sell",                   # Операция ордера - продажа или покупка
  "type": "limit",                  # Тип ордера, limit или market
  "price": "14.0",                  # Цена ордера
  "avg_execution_price": "0.0",     # Средняя цена сделок по ордеру
  "symbol": "xrpuah",               # Валютная пара ордера
  "timestamp": 1560089091000,       # Время закрытия ордера
  "original_amount": "45.0",        # Изначальный объем ордера
  "remaining_amount": "45.0",       # Объем ордера на момент закрытия
  "executed_amount": "0.0",         # Исполненный объем ордера 
  "is_cancelled": null,
  "is_hidden": null,
  "is_live": null,
  "was_forced": null,
  "exchange": null
}
```


## Kuna Codes
Kuna имеет особый функционал для трансфера средств между аккаунтами, без комиссий и СМС. Он называется **Kuna Code**.

Kuna Code имеет структуру: \
**857ny**-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-KUN-KCode

Где первый сегмент, это публичный ключ кода. Потом идет 8 сегментов по 5 символов, это приватное тело Kuna Code, и 2 завершающих сегмента, это валюта и метка что это код от куны (KCode).

**Статусы кодов**

| Название статуса | Описание                          |
|------------------|-----------------------------------|
| created          | Код был создан                    |
| processing       | Код в процессе выпуска            |
| unconfirmed      | Код ожидает подтверждения         |
| active           | Код можно активировать            |
| redeeming        | В процессе активации              |
| redeemed         | Активирован                       |
| onhold           | Код под подозрением               |
| canceled         | Код отменен                       |

### 1) Проверить Kuna Code
Единственный публичный метод для Kuna Code. \
Этот метод позволяет проверить доступность Куна кода по его первому сегменту, а так-же узнать валюту, сумму, использован он или еще нет.

```bash
GET /v3/kuna_codes/{code}/check
```

**Параметры пути**
```
[code] - 5 первых символов Kuna Code.
```

**Пример ответа**
```bash
{
  "id": 519,                        # внутренний ID
  "sn": 'p9MajCjlLo72',             # ID для указания к супорту
  "code": "11111",                  # первый сегмент кода
  "recipient": "all",               # Kuna-ID того кто может активировать код.
                                    # Если 'all' то активировать может кто угодно
  "amount": 20000,                  # сумма кода
  "currency": "xrp",                # currency кода
  "status": "active",               # статус (active, redeemed, expired)
  "non_refundable_before": null,    # до этого времени владелец кода не может его активировать.              
  "created_at": "2019-03-20T13:00:00+02:00",  # время создания кода                            
  "redeemed_at": null               # время активации кода
}
```

### 2) Создать Kuna Code
Можно создать Kuna Code с помощью этого метода. Через API можно указать Kuna ID получателя, дату, до которой код не будет доступен создателю на активацию, публичную и приватную заметку.

⚠️ Если у вас активирована двухфакторная аутентификация, то в запросе нужно указать **otp** пароль.

```bash
POST /v3/auth/kuna_codes
```

**Параметры запроса**

| Параметр              | Обязательный | Описание                            |
|-----------------------|--------------|-------------------------------------|
| otp                   | нет          | Код двухфакторной аутентификации    |
| recipient             | нет          | Получатель кода. По умолчанию код может актировать кто угодно, но если указать Kuna ID, то код сможет ввести только этот аккаунт. |
| **amount**            | да           | Сумма кода                          |
| **currency**          | да           | Валюта кода                         |
| non_refundable_before | нет          | Формат ISO8601. Дата до которой владелец кода не сможет его активировать. |
| comment               | нет          | Публичная заметка к коду.           |
| private_comment       | нет          | Приватная заметка к коду, которая видна только владельцу кода. |


**Пример ответа**
```bash
{
  "id": 519,
  "sn": "p9MajCjlLo72",
  "code": "857ny-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-XXXXX-KUN-KCode",
  "recipient": "all",
  "amount": "2938",
  "currency": "xrp",
  "status": "active",
  "non_refundable_before": "2019-08-20T13:00:00+02:00",
  "created_at": "2019-03-20T13:00:00+02:00",
  "redeemed_at": null,
  "comment": "Try to activate inside your Plark Wallet",
  "private_comment": "Ripple to the MOON!"
}
```


