# API Документация

- *apikey*: ****
- *domain*: https://sell-skins.ru/

в Header: {authorization: Bearer *apikey*} и  {content-type: application/json}

## Основные эндпоинты

### 1. Получение цен

**Эндпоинт:** `/api/skins-updater/prices`

**Метод:** GET

**Параметры запроса:**
- `game` (необязательный): ID игры (730 - CS:GO, 570 - Dota 2, 440 - TF2, 252490 - Rust)

**Ответ:**
```json
[
  {
    "hash_name": "Festivized Dream Piped Brass Beast (Field-Tested)",
    "priceBuy": null,
    "allowed": false
  }
]
```

- `priceBuy`: `null` | `number` (float)
- `allowed`: `boolean`

### 2. Инвентарь Steam

Ручка главная получение! Обновление если принудительно юзер хочет обновить! Вещи кешеируеются на 12часов (обновляются принудительно ручкой рефрпешь!

#### 2.1 Получение инвентаря

**Эндпоинт:** `/api/steam/inventory`

**Метод:** GET

**Параметры запроса:**
- `SteamID` (обязательный): ID Steam пользователя
- `appid` (обязательный): ID приложения (730 - CS:GO, 570 - Dota 2 и т.д.)
- `contextid` (обязательный): ID контекста

**Ответ:**
```json
[
  {
    "stickers": [
      {
        "name": "Sticker Name",
        "src": "URL to sticker image"
      }
    ],
    "assetId": "Asset ID",
    "classid": "Class ID",
    "instanceid": "Instance ID",
    "hash_name": "Item Hash Name",
    "icon_url": "URL to item icon",
    "rungame": "Game Name or null",
    "color": "Item Color",
    "contextid": "Context ID",
    "appid": "App ID",
    "SteamID": "Steam ID",
    "priceBuy": "Optional price",
    "allowed": "Optional boolean"
  }
]
```
**Примечание:** Используйте `/api/steam/inventory/refresh` для обновления инвентаря только при первом запросе или по запросу пользователя. Не используйте при каждом открытии вкладки инвентаря из-за ограничений рейтлимита Steam.

#### 2.2 Обновление инвентаря

**Эндпоинт:** `/api/steam/inventory/refresh`

**Метод:** GET

**Параметры запроса:**
- `SteamID` (обязательный): ID Steam пользователя
- `appid` (обязательный): ID приложения
- `contextid` (обязательный): ID контекста

**Ответ:** Тот же формат, что и для получения инвентаря.

### 3. Создание трейда

Если  ошибка, то либо 500 либо 200 (но typeResp = 'error') если все ок то typeResp = 'success'

(В этой ручке еще будем проверять, чтобы цены которые вы нам передадите, ну то есть за который юзер берет, не отличались от наших новых цен в большую строну (если меньше мы примем), позже распишу какую, это на всякий если у вас сломается что то и не обновите цены от нас в течение 10мин и тд, наша подстраховачка будет)

**Эндпоинт:** `/api/transaction/`

**Метод:** POST

**Параметры запроса:**
```json
{
  "typeTransaction": "Integration CobaltLab Sell Skin", (именно так)
  "fullName": "Full Name", (опционально)
  "walletNumber": "Wallet Number", (опционально)
  "email": "Email Address", (опционально)
  "tradeurl": "Trade URL",
  "telegramContact": "Telegram Contact", (опционально)
  "paymentId": -1, (именно так)
  "externalId": "string", (опционально)
  "externalMessage": "string", (опционально)
  "items": [
    {
      "appid": "App ID",
      "contextid": "Context ID",
      "assetid": "Asset ID",
      "priceBuy": "price"
    }
  ]
}
```
**Ответ:**
```json
{
  "typeResp": "success" | "error",
  "textResp": 'Ошибка создания трейда'
                | 'No bot found'
                | 'No cookies found for the selected bot' "TradeBan"
                | "NewDevice"
                | "TargetCannotTrade"
                | "OfferLimitExceeded"
                | "ItemServerUnavailable"
                | <Будет ошибка на английском> ,
  "nicknameBot": "string";
  lvlSteamBot: number;
  avatarBot: string;
  "dateRegBot": "2024-09-11T07:11:41.749Z";
  "SteamIDBot": "Optional Steam Bot ID",
  "tradeId": "Optional Trade ID",
  "transactionId": "Optional Transaction ID",
  "status": "Optional Status",
  "createdAt": "Optional Creation Timestamp",
  items: Skin[];
}
```

class Skin {
  id: number;
  assetId: string;
  classid: string;
  instanceid: string;
  hash_name: string;
  float: number | null;
  paint_index: number | null;
  paint_seed: number | null;
  icon_url: string;
  rungame: string;
  color: string;
  stickers: Array<any>;
  createdAt: string;
  SteamID: string;
  appid: string;
  contextid: string;
  priceOnTransaction: number;
  tradeId: number;
  userId: number;
  botid: string;
};

**Пример успешного ответа:**
```json
{
  "typeResp": "success",
  "textResp": "Транзакция создана",
  lvlSteamBot: number;
  avatarBot: string;
  "nicknameBot": "string";
  "dateRegBot": "2024-09-11T07:11:41.749Z";
  "SteamIDBot": "7656119954....",
  "tradeId": "73681....",
  "transactionId": 15,
  "status": "create",
  "createdAt": 172556.... (в миллисекундах)
   items: Skin[];
}
```

**Примечание:** В этой ручке будет проверка цен. Если переданные цены выше чем цены в базе, вернет ошибку.

### 4. Коллбэки и статусы трейдов

**PayLoad:**
```json
 {
    SteamID: string;
    date: Date;
    cost: number;
    statusTrade: string;
    statusTradeNumber: number;
    offerId: string;
    externalId?: string;
    externalMessage?: string;
    items: {
        assetId: string;
        classid?: string | null;
        instanceid?: string | null;
        hash_name?: string | null;
        float?: number | null;
        paint_index?: number | null;
        paint_seed?: number | null;
        icon_url?: string | null;
        rungame?: string | null;
        color?: string | null;
        stickers?: any | null;
        createdAt: Date;
        appid?: string | null;
        contextid?: string | null;
        priceOnTransaction: number;
    }[];
}
```

### 5. Проверка доступности трейда

**Эндпоинт:** `/api/steam/trade/check

**Метод:** GET

**Параметры запроса Query:**
- `url ` (обязательный): трейд ссылка полная!

**Ответ:** 
```json
{
  "allow": boolean,
  "error":  'Not valid tradeUrl' 
	  | 'The service is temporarily unavailable for checking' 
	  | `Hold ${days} day`
	  | <"You cannot trade with cc235 because they have a trade ban.">,
"code": number,
}
```
**Code:**
- `0` : Все ок
- `1` : У нас проблемы, не можем проверить, пиши КОДЕРУ!
- `2` : Холд у юзера
- `3` : Почему то не может принять трейд в error написан ответ стима
- `4` : Не валидна отправленная ссылка

### 6. Проверка доступности сайта

**Эндпоинт:** `/api/transaction/status/check

**Метод:** GET

**Ответ:** 
```json
{}
```

### 7.Получение статистики транзакций

**Эндпоинт:** `/api/transaction/integration/stats

**Метод:** GET

**Параметры запроса:**
- `dateStart` (не обязательный): миллисекунды
- `dateEnd` (не обязательный): миллисекунды

**Ответ:** 
```json
[
    {
        "type": "Integration CobaltLab Sell Skin",
        "stats": [
            {
                "status": "cancel",
                "totalcost": 4.46
            },
            {
                "status": "create <новые созданнеые не принятые трейды>",
                "totalcost": 0.24
            },
            {
                "status": "paymenting <это то что ожидают выплаты>",
                "totalcost": 1.5
            }
        ]
    }
]
```
