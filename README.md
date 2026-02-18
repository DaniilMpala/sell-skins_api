# Документация

- [Внешняя интеграция](./Внешняя%20интеграция.md)
- [Внутренняя Интеграция](./Внутренняя%20интеграция.md)


# API Документация

- *apikey*: ****
- *domain*: sellskin.ru 

в Header: {authorization: Bearer *apikey*} и  {content-type: application/json}

### 1. Получение цен скупки

**Эндпоинт:** `/api/skins-updater/prices`

**Метод:** GET

**Тип Query запроса:**

```ts
export class GetPricesDto {
    game: '730' | '570' | '440' | '252490';
    withCountSell: boolean = false; 
}
```

**Пример Body запроса:**

Стим айди или трейд ссылку необходимо передать, чтобы мы понимали, что человек депозитит именно с этого аккаунта а не с левого.  
```json
{
  "game": "730",
  "withCountSell": false
}
```

**Тип ответа:**
```ts
class PricesDtoReturn {
    hash_name: string;
    allowed: boolean;
    priceBuy: any;
}[]
```

**Ответ:**
```json
[
  {
      hash_name: "";
      allowed: true;
      priceBuy: 12.2;
  },
  {
      hash_name: "";
      allowed: false;
      priceBuy: null;
  }
]
```

### 2. WebHook транзакции

**После Отмены/Принятия (в том числе в холд)**
  
Мы будем отсылать коллбек (каждые 5 минут) до тех пор, пока вы не ответите ответом 200 на запрос или не пройдет 24 часа с его создания.
### Описание полей сделки

- **`meta`** — метаданные, которые вы передаёте при создании транзакции  
  *(та самая meta data, отправленная при инициализации трейда)*

- **`cost`** — сумма транзакции в USD

- **`currency`** — в данном поле приходит зафиксированный курс валют на момент начало транзакции, можеет умножать на него cost и получать сумму в валюте другой

- **`isAccept`** — трейд окончательно принят

- **`settlementDate`** — время прохождение протекта, в миллисекундах, оно опцимонально!

- **`isCancel`** — трейд был отменён в одном из следующих состояний:
  - `Declined`
  - `Expired`
  - `Canceled`
  - `CanceledBySecondFactor`
  - `InvalidItems`
  - `Countered`

- **`isProtect`** — трейд принят, но находится в холде:
  - либо в статусе **Protect** (ожидание 8 дней для CS),
  - либо у пользователя был отключён аутентификатор

- **`statusTradeNumber`** — номер состояния оффера в Steam  
  Соответствует enum [`ETradeOfferState`](https://developer.valvesoftware.com/wiki/Steam_Web_API/IEconService#ETradeOfferState)

- **`statusTrade`** — текстовое представление `statusTradeNumber`
#### Возможные значения `statusTrade`

| Код | Статус                     | Описание |
|-----|----------------------------|----------|
| 1   | `Invalid`                  | Некорректный или несуществующий оффер |
| 2   | `Active`                   | Активный трейд, ожидает действий |
| 3   | `Accepted`                 | Трейд успешно принят |
| 4   | `Countered`                | Сделано встречное предложение |
| 5   | `Expired`                  | Истёк по времени |
| 6   | `Canceled`                 | Отменён пользователем |
| 7   | `Declined`                 | Отклонён второй стороной |
| 8   | `InvalidItems`             | Некорректные или недоступные предметы |
| 9   | `CreatedNeedsConfirmation` | Создан, требуется подтверждение |
| 10  | `CanceledBySecondFactor`   | Отменён из-за второго фактора (2FA) |
| 11  | `InEscrow`                 | В холде (Escrow / Protect) |

Один из вариантов, как можно обрабатывать, это отслеживать поля isAccept, isCancel, isProtect всегда будет заполнено, одно из них. 
- Если isAccept - зачисляете баланс
- Если isCancel - отмена трейда
- Если isProtect - трейд в холде, в течение 8-15дней придет новый коллбек с статусом одним из выше

```ts
export class CallbackDto {
    SteamID: string; //Продающего
    date: Date;
    cost: number;
    statusTrade: string;
    statusTradeNumber: number;
    settlementDate?: number;
    offerId: string;
    meta?: any;
    currency?: Record<string, number>; //пример данных {"EUR": 90.5625, "USD": 79.0246}
    isProtect: boolean;
    isCancel: boolean;
    isAccept: boolean;
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

**Пример:**
```json
{
  "cost": 4.27,
  "date": "2025-10-03T17:44:07.767Z",
  "meta": {
    "test": 123
  },
  "currency": {
    "EUR": 0.92,
    "RUB": 92.5
  },
  "items": [
    {
      "appid": "730",
      "color": "#eb4b4b",
      "float": null,
      "assetId": "46549537737",
      "classid": "1813198903",
      "rungame": "steam://rungame/730/76561202255233023/+csgo_econ_action_preview%20S%owner_steamid%A46549537737D4929802143596962724",
      "icon_url": "i0CoZ81Ui0m-9KwlBY1L_18myuGuq1wfhWSaZgMttyVfPaERSR0Wqmu7LAocGIGz3UqlXOLrxM-vMGmW8VNxu5Dx60noTyL2kpnj9h1Y-s2pZKtuK8-ED3SExOJ3vuVWXSyxkBEYvjiBk5r0b3zBZlNzCpImF7MK40TtmtDmZuvktFSP39hCny782i5J7Xw95uYHBKQ7uvqANlqPdCA",
      "stickers": [],
      "contextid": "2",
      "createdAt": "2025-10-03T17:44:07.769Z",
      "hash_name": "Glock-18 | Wasteland Rebel (Field-Tested)",
      "instanceid": "7513465702",
      "paint_seed": null,
      "paint_index": null,
      "priceOnTransaction": 4.27
    }
  ],
  "SteamID": "76561199552140003",
  "offerId": "8478425233",
  "isAccept": true,
  "isCancel": false,
  "isProtect": false,
  "statusTrade": "Accepted",
  "statusTradeNumber": 3
}
```

### Заголовки, которые отправляет наш сервер в webhook:
- **`Content-Type: application/json`** — всегда такой.
- **`X-Signature: <sha256>`** — сигнатура формируется на основе вашего `apikey` и `payloadJson`.  
  Например: у нас есть JSON-объект (payload), который мы отправляем вам в примере выше. Мы преобразуем его в строку с помощью `JSON.stringify()`, получая формат вида текста и потом шифруем его через sha256
  ```json
  {"cost":0.289,"date":"2025-10-03T16:17..."}
  ```
   ```js
  crypto.createHmac('sha256', apikey).update(payloadJson).digest('hex')
  ```

### 3. Статистика интеграции

**Эндпоинт:** `/api/integration/stats/mine`

**Метод:** GET

**Ответ:**
```json
{
  "payouts": {
    "totalPaid": 1500,
    "creditAmount": 500,
    "creditAmountWithProtect": 700
  },
  "periodStats": {
    "last24h": {
      "success": { "total": 10, "games": { "730": 8, "440": 2 } },
      "protect": { "total": 2, "games": { "730": 2, "440": 0, "570": 0, "252490": 0 } },
      "cancel": { "total": 1, "games": { "570": 1, "440": 0, "730": 0, "252490": 0 } }
    },
    "last7d": { ... },
    "allTime": { ... }
  }
}
```

**Тип:**
```ts
export type AppId = '440' | '730' | '570' | '252490';

export class PeriodStatsItem {
    total: number; // Общее количество транзакций в этом статусе
    games: Record<AppId, number>; // Количество транзакций с разбивкой по appid игры
}

export class PeriodStats {
    success: PeriodStatsItem; // Данные по успешным транзакциям
    protect: PeriodStatsItem; // Данные по защищенным транзакциям (в холде)
    cancel: PeriodStatsItem; // Данные по отмененным транзакциям
}

export class IntegrationPayoutStats {
    totalPaid: number; // Общая сумма уже произведенных выплат
    creditAmount: number; // Сумма к выплате (все успешные - выплаты)
    creditAmountWithProtect: number; // Сумма к выплате включая холд (успешные + защищенные - выплаты)
}

export class IntegrationStatsDto {
    payouts: IntegrationPayoutStats; // Сводка по финансам и выплатам
    periodStats: {
        last24h: PeriodStats; // Статистика за последние 24 часа
        last7d: PeriodStats; // Статистика за последние 7 дней
        allTime: PeriodStats; // Статистика за всё время
    };
}
```

### 4. Информация о транзакции (Callback Data)

**Эндпоинт:** `/api/integration/transaction/:id/info/callback`

**Метод:** `GET`

Позволяет получить актуальные данные о транзакции в том же формате, который отправляется через Webhook (Callback). Полезно для проверки статуса, если Webhook не был получен.

**Параметры пути:**
* `id` (number): ID транзакции в нашей системе.

**Ответ:** `CallbackDto` (Это тот же объект, что и в 2. WebHook транзакции).

---
