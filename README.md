# API Яндекс.Прилавок

## Цель проекта
  протестировать новую функциональность API v.3.1.1 в Яндекс.Прилавок:

- **Работа с наборами:** возможность добавлять продукты в набор — ручка  **POST /api/v1/kits/{id}/products**.
  
- **Работа с курьерами:** возможность проверить, есть ли доставка курьерской службой «Привезём быстро» и сколько она стоит. Ручка **POST /fast-delivery/v3.1.1/calculate-delivery.xml**.
  
- **Работа с корзиной:**
  - возможность получить список продуктов, которые добавили в корзину. Ручка **GET /api/v1/orders/id**;
  - возможность добавлять продукты в корзину. Ручка **PUT /api/v1/orders/:id**;
  - возможность удалять корзину. Ручка **DELETE/api/v1/orders/:id.**

## **Задачи проекта:**

1. Изучить документацию к API _(ниже)_. Требования к бэкенду <https://code.s3.yandex.net/qa/files/backend_requirements.pdf>.

2. Спроектировать тесты в виде чек-листа, чтобы покрыть новую функциональность. Авторизацию проверять не нужно.  

3. Протестировать API через Postman и завести баг-репорты.


## Выполнение проекта:

**Чек-листы, баг-репорты** 

<https://docs.google.com/spreadsheets/d/1fNgn-F_SD62ktNr1-qjSHnAYo5vT0IppQpypCoTxmmY/edit?usp=sharing>



# Документация к API 
## **Добавление продуктов в набор**

**POST**

```javascript
/api/v1/kits/id/products
```

- Пример заголовков

```javascript
{
    "Content-Type": "application/json"
}
```

### **Параметр**

| Название | Тип | Описание |
|----|----|----|
| id | number | id набора в таблице kit_model. Передается урл-параметром |
| productsList | string | productList набора. Массив, содержащий id продуктов и их количества. Передается в теле запроса. |

- Изменение набора /api/v1/kits/8

```javascript
{
    "productsList": [
        {
            "id": 1,
            "quantity": 2
        },
        {
            "id": 6,
            "quantity": 2
        }
    ]
}
```

- Ответ: Успешное добавление продуктов в набор

```javascript
     HTTP/1.1 200 OK
 {
    "id": 2,
    "name": "Мой набор для выходных",
    "productsList": [
        {
            "id": 1,
            "name": "Икра красная Белое море",
            "price": 45,
            "weight": 5,
            "units": "кг",
            "quantity": 2
        },
        {
            "id": 5,
            "name": "Багет французский",
            "price": 15,
            "weight": 1,
            "units": "кг",
            "quantity": 2
        }
    ],
    "productsCount": 4
}
```

- Ошибка: Наборов, подходящих под условие, не нашлось

```javascript
HTTP/1.1 404 Not found.
{
       "code": 404,
       "message": "Not found"
}
```

## **Доставка: "Привезём быстро"**

Взаимодействие осуществляется с помощью XML

 **POST**

```javascript
/fast-delivery/v3.1.1/calculate-delivery.xml
```

- Пример заголовков

```javascript
{
    "Content-Type": "application/xml"
}
```

### **Параметр**

| Название | Тип | Описание |
|----|----|----|
| productsCount | number | Количество продуктов в заказе |
| productsWeight | number | Вес продуктов |
| deliveryTime | number | Планируемое время доставки |

- Пример запроса

```javascript
<InputModel>
    <productsCount>2</productsCount>
    <productsWeight>5.1</productsWeight>
    <deliveryTime>20</deliveryTime>
</InputModel>
```

- Ответ: Пример ответа

```javascript
HTTP/1.1 200 OK
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="43" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

## **Получение продуктов в корзине**

**GET**

```javascript
/api/v1/orders/id
```

### **Параметр**

| Название | Тип | Описание |
|----|----|----|
| id | string | id корзины в таблице order_model. Передается url-параметром |

- Получение корзины

```javascript
/api/v1/orders/6
```

- Ответ: Успешное получение продуктов из корзины

```javascript
HTTP/1.1 200 OK
[
    {
           "id": 1,
           "name": "Сок Jumex апельсин без сахара",
           "price": 149,
           "weight": 473,
           "units": "мл",
           "quantity": 3
       },
    {
           "id": 4,
           "name": "Sprite классический",
           "price": 79,
           "weight": 900,
           "units": "мл",
           "quantity": 4
    }
]
```

- Ошибка: Корзина не найдена

```javascript
HTTP/1.1 404 Not found.
{
       "code": 404,
       "message": "Not found"
}
```

  

## **Добавление товаров в корзину**

**PUT**

```javascript
/api/v1/orders/id
```

- Добавление товаров в корзину

```javascript
{
    "productsList": [
        {
            "id": 1,
            "quantity": 4
        },
        {
            "id": 5,
            "quantity": 2
        },
        {
            "id": 3,
            "quantity": 1
        },
        {
            "id": 4,
            "quantity": 1
        }
    ]
}
```

- Ответ: Успешное добавление товаров в корзину

```javascript
 HTTP/1.1 200 OK
 {
"productsList": [
    {
        "id": 1,
        "quantity": 10
    },
    {
        "id": 5,
        "quantity": 10
    },
    {
        "id": 3,
        "quantity": 9
    },
    {
        "id": 4,
        "quantity": 5
    }
],
        "status": 0,
        "deliveryPriceOur": 30,
        "deliveryTime": "25~30",
        "courierService": "Привезём быстро",
        "deliveryPrice": 0,
        "wareHouse": "Шведский дом",
        "userId": 1,
        "id": 5,
        "productsCost": 75,
        "finalCCost": 174
 }
```

- Ошибка: Корзина не найдена

```javascript
HTTP/1.1 404 Not found.
{
       "code": 404,
       "message": "Not found"
}
```

- Ошибка: Нет склада, способного обработать Ваш заказ

```javascript
HTTP/1.1 409 Conflict.
{
       "code": 409,
       "message": "Нет склада, способного обработать Ваш заказ"
}
```

## **Удаление корзины**

**DELETE**

```javascript
/api/v1/orders/:id
```

### **Параметр**

| Название | Тип | Описание |
|----|----|----|
| id | number | id корзины в таблице order_model. Передается урл-параметром |

- Ответ: Успешное удаление корзины

```javascript
HTTP/1.1 200 OK
{
       "ok": true
}
```
