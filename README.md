# API Документация

- *apikey*: ****
- *domain*: http://167.235.201.24

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

О ручке, у нас есть кеш инвентаря юзера и получение(обновление) его, вот эта ручка /refresh обновляет и выводит полученные, юзайте только для первого запроса и если юзер сам решит обновить, не над юзать при каждой открытие вкладки инвентаря его, ибо есть рейтлимиты у стима на получение инвента, они все же большие с учетом, что это через ботов получаем, но все же, возможен случай, что в рейтлимит уйдут все боты и нельзя будет поулчить в этом случае ошибка 400 или 500
Для кэшированного инвентаря юзайте ручку первую без /refresh, ее пофиг скок юзать, она из нашей бд данные подтягивает, вывод одинаковый

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
  "items": [
    {
      "appid": "App ID",
      "contextid": "Context ID",
      "assetid": "Asset ID",
      "priceBuy": "Optional price"
    }
  ]
}
```
**Ответ:**
```json
{
  "typeResp": "success" | "error",
  "textResp": "Response text",
  "SteamIDBot": "Optional Steam Bot ID",
  "tradeId": "Optional Trade ID",
  "transactionId": "Optional Transaction ID",
  "status": "Optional Status",
  "createdAt": "Optional Creation Timestamp"
}
```

**Пример успешного ответа:**
```json
{
  "typeResp": "success",
  "textResp": "Транзакция создана",
  "SteamIDBot": "7656119954....",
  "tradeId": "73681....",
  "transactionId": 15,
  "status": "create",
  "createdAt": 172556.... (в миллисекундах)
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
