# Документация

- [Внешняя интеграция](./Внешняя%20интеграция.md)
- [Внутренняя Интеграция](./Внутренняя%20интеграция.md)


API Документация
apikey: ****
domain: cobaltskins.tech
в Header: {authorization: Bearer apikey} и {content-type: application/json}

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
