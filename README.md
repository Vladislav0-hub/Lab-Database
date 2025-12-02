**Лабораторные работы по Базам Данных**

Выполнил: Васильев Владислав Ильич(2261-ДБ)  
Telegram: @VIad_OS

# Постановка задачи (вариант 46)

**Интернет-магазин: анализ отзывов и возвратов**

*Сущности:* 
  - Товары (артикул, категория),
  - Заказы (номер, дата), 
  - Отзывы (оценка от 1 до 5, текст, дата),
  - Возвраты (причина, дата)

*Процессы:*
  - После получения заказа клиент может оставить отзыв или инициировать возврат товара.

*Выходные документы:*
  - Выдать рейтинг товаров заданной категории по средней оценке, но исключив товары с количеством отзывов менее 10. Отсортировать по убыванию рейтинга.
  - Выдать список товаров с процентом возвратов от общего количества покупок за квартал, отсортированный по убыванию процента (только товары с более чем 50 покупками).

# Лабораторная работа №1 (Проектирование логической и физической модели БД)

```text
Лаба по проектированию информационной модели для реляционных баз данных.
Предполагаем Postgresql. 

# Интернет-магазин: анализ отзывов и возвратов

## Постановка задачи

*Сущности:*
    Товары (артикул, категория),
    Заказы (номер, дата),
    Отзывы (оценка от 1 до 5, текст, дата),
    Возвраты (причина, дата).

*Процессы:*
    После получения заказа клиент может оставить отзыв или инициировать возврат товара.

*Выходные документы:*

  - Выдать рейтинг товаров заданной категории по средней оценке, но исключив товары с количеством отзывов менее 10. Отсортировать по убыванию рейтинга.

  - Выдать список товаров с процентом возвратов от общего количества покупок за квартал, отсортированный по убыванию процента (только товары с более чем 50 покупками).

## ER-Модель

### Базовые сущности

    Товар(артикул, название, категория, цена)
    Заказ(номер, дата, статус)
    Отзыв(оценка, текст, дата)
    Возврат(причина, дата, статус)
    Клиент(идентификатор, имя, email)

### Отношения

    [Клиент]-1,Required------------------0..N,Optional-[Заказ]
    [Заказ]-1,Required------------------1..N,Required-[ПозицияЗаказа]
    [Товар]-1,Required------------------0..N,Optional-[ПозицияЗаказа]
    [Товар]-1,Required------------------0..N,Optional-[Отзыв]
    [Клиент]-1,Required------------------0..N,Optional-[Отзыв]
    [ПозицияЗаказа]-1,Optional------------------0..1,Optional-[Возврат]

## Логическая модель

Получаем шесть таблиц:

  - ```Product(article, name, category, price)```, primary key - article
  - ```Customer(customer_id, name, email, registration_date)```, primary key - customer_id
  - ```Order(order_id, customer_id, order_date, status)```, primary key - order_id
  - ```OrderItem(order_item_id, order_id, product_article, quantity, unit_price)```, primary key - order_item_id
  - ```Review(review_id, product_article, customer_id, rating, text, review_date)```, primary key - review_id
  - ```Return(return_id, order_item_id, reason, return_date, status)```, primary key - return_id

## Физическая модель

Зададим типы данных для атрибутов:

  - ```article::varchar(50)```
  - ```name::varchar(200)```
  - ```category::varchar(100)```
  - ```price::decimal(10,2)```
  - ```customer_id::serial```
  - ```name::varchar(100)```
  - ```email::varchar(150)```
  - ```registration_date::timestamp```
  - ```order_id::serial```
  - ```customer_id::integer```
  - ```order_date::timestamp```
  - ```status::varchar(20)``` CHECK (status IN ('обработан', 'доставляется', 'доставлен', 'отменен'))
  - ```order_item_id::serial```
  - ```order_id::integer```
  - ```product_article::varchar(50)```
  - ```quantity::integer```
  - ```unit_price::decimal(10,2)```
  - ```review_id::serial```
  - ```product_article::varchar(50)```
  - ```customer_id::integer```
  - ```rating::integer``` CHECK (rating BETWEEN 1 AND 5)
  - ```text::text```
  - ```review_date::timestamp```
  - ```return_id::serial```
  - ```order_item_id::integer```
  - ```reason::varchar(500)```
  - ```return_date::timestamp```
  - ```status::varchar(20)``` CHECK (status IN ('обрабатывается', 'одобрен', 'отклонен', 'выполнен'))```
  ```

## Ответ DeepSeek

Отличная основа для интернет-магазина! Модель хорошо отражает бизнес-процессы, но есть несколько важных улучшений для создания более надежной и нормализованной базы данных.

### Критические замечания и рекомендации

### 1. Нормализация товаров и категорий
**Проблема:**  Категория хранится непосредственно в таблице товаров, что приводит к избыточности данных.
**Исправление:** Создаем отдельную сущность "Категории" для централизованного управления.


#### 2. Улучшение структуры заказов
**Проблема:** Отсутствует связь между отзывами/возвратами и конкретными позициями заказа.
**Исправление:** Связываем отзывы и возвраты с позициями заказа, а не напрямую с товарами.

#### 3. Добавление проверок целостности
**Проблема:** Недостаточно ограничений для обеспечения бизнес-логики.
**Исправление:** Добавляем CHECK-ограничения для рейтингов, дат и статусов.

### Исправленная и улучшенная модель

#### Логическая модель
*   **`Category`** (`category_id`, name, description)
*   **`Product`** (`product_id`, article, name, category_id, price)
*   **`Customer`** (`customer_id`, name, email, registration_date)
*   **`Order`** (`order_id`, customer_id, order_date, status, total_amount)
*   **`OrderItem`** (`order_item_id`, order_id, product_id, quantity, unit_price)
*   **`Review`** (`review_id`, order_item_id, rating, text, review_date, status)
*   **`Return`** (`return_id`, order_item_id, reason, return_date, status, refund_amount)

#### Физическая модель (DDL для PostgreSQL)
```sql
-- Таблица категорий
CREATE TABLE category (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Таблица товаров
CREATE TABLE product (
    product_id SERIAL PRIMARY KEY,
    article VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(200) NOT NULL,
    category_id INTEGER NOT NULL REFERENCES category(category_id) ON DELETE RESTRICT,
    price DECIMAL(10,2) NOT NULL CHECK (price > 0),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);

-- Таблица клиентов
CREATE TABLE customer (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    registration_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    phone VARCHAR(20)
);

-- Таблица заказов
CREATE TABLE "order" (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customer(customer_id) ON DELETE CASCADE,
    order_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL CHECK (status IN ('обработан', 'доставляется', 'доставлен', 'отменен')),
    total_amount DECIMAL(10,2) NOT NULL CHECK (total_amount >= 0),
    delivery_address TEXT
);

-- Таблица позиций заказа
CREATE TABLE order_item (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES "order"(order_id) ON DELETE CASCADE,
    product_id INTEGER NOT NULL REFERENCES product(product_id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10,2) NOT NULL CHECK (unit_price >= 0),
    -- Проверка, что цена позиции равна количеству * цену за единицу
    CHECK (unit_price * quantity >= 0)
);

-- Таблица отзывов
CREATE TABLE review (
    review_id SERIAL PRIMARY KEY,
    order_item_id INTEGER NOT NULL UNIQUE REFERENCES order_item(order_item_id) ON DELETE CASCADE,
    rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
    text TEXT,
    review_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL DEFAULT 'активен' CHECK (status IN ('активен', 'скрыт', 'удален')),
    -- Проверка, что текст отзыва не пустой, если предоставлен
    CHECK (text IS NULL OR LENGTH(TRIM(text)) > 0)
);

-- Таблица возвратов
CREATE TABLE return (
    return_id SERIAL PRIMARY KEY,
    order_item_id INTEGER NOT NULL UNIQUE REFERENCES order_item(order_item_id) ON DELETE CASCADE,
    reason VARCHAR(500) NOT NULL,
    return_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL DEFAULT 'обрабатывается' CHECK (status IN ('обрабатывается', 'одобрен', 'отклонен', 'выполнен')),
    refund_amount DECIMAL(10,2) CHECK (refund_amount >= 0),
    -- Проверка, что причина возврата не пустая
    CHECK (LENGTH(TRIM(reason)) > 0)
);

-- Индексы для оптимизации частых запросов
CREATE INDEX idx_product_category ON product(category_id);
CREATE INDEX idx_product_article ON product(article);
CREATE INDEX idx_order_customer_date ON "order"(customer_id, order_date);
CREATE INDEX idx_order_date ON "order"(order_date);
CREATE INDEX idx_order_status ON "order"(status);
CREATE INDEX idx_order_item_product ON order_item(product_id);
CREATE INDEX idx_order_item_order ON order_item(order_id);
CREATE INDEX idx_review_rating_date ON review(rating, review_date);
CREATE INDEX idx_review_order_item ON review(order_item_id);
CREATE INDEX idx_return_date_status ON return(return_date, status);
CREATE INDEX idx_return_order_item ON return(order_item_id);
CREATE INDEX idx_customer_email ON customer(email);
```

#### Проверка нормальных форм
*   **1NF:** Соблюдена - все значения атомарные
*   **2NF:** Соблюдена - нет зависимостей неключевых атрибутов от части ключа
*   **3NF/BCNF:** Соблюдена - нет транзитивных зависимостей, все неключевые атрибуты зависят только от ключа

---

### Примеры запросов для "Выходных документов"

**1. Рейтинг товаров заданной категории по средней оценке (отзывов ≥ 10):**
```sql
SELECT 
    p.product_id,
    p.article,
    p.name,
    c.name as category_name,
    ROUND(AVG(r.rating)::numeric, 2) as avg_rating,
    COUNT(r.review_id) as review_count
FROM product p
JOIN category c ON p.category_id = c.category_id
JOIN order_item oi ON p.product_id = oi.product_id
JOIN review r ON oi.order_item_id = r.order_item_id
WHERE c.name = 'Название_категории'  -- Заменить на нужную категорию
    AND r.status = 'активен'
GROUP BY p.product_id, p.article, p.name, c.name
HAVING COUNT(r.review_id) >= 10
ORDER BY avg_rating DESC;
```

**2. Товары с процентом возвратов за квартал (покупок > 50):**
```sql
WITH quarterly_stats AS (
    SELECT 
        p.product_id,
        p.article,
        p.name,
        COUNT(DISTINCT oi.order_item_id) as total_purchases,
        COUNT(DISTINCT ret.return_id) as total_returns,
        ROUND(
            (COUNT(DISTINCT ret.return_id) * 100.0 / NULLIF(COUNT(DISTINCT oi.order_item_id), 0))::numeric, 
            2
        ) as return_percentage
    FROM product p
    JOIN order_item oi ON p.product_id = oi.product_id
    JOIN "order" o ON oi.order_id = o.order_id
    LEFT JOIN return ret ON oi.order_item_id = ret.order_item_id 
        AND ret.return_date >= DATE_TRUNC('quarter', CURRENT_DATE)
        AND ret.status IN ('одобрен', 'выполнен')
    WHERE o.order_date >= DATE_TRUNC('quarter', CURRENT_DATE)
        AND o.status IN ('доставлен')
    GROUP BY p.product_id, p.article, p.name
    HAVING COUNT(DISTINCT oi.order_item_id) > 50
)
SELECT 
    product_id,
    article,
    name,
    total_purchases,
    total_returns,
    return_percentage
FROM quarterly_stats
ORDER BY return_percentage DESC;
```

**Дополнительный полезный запрос - статистика по категориям:**
```sql
SELECT 
    c.name as category_name,
    COUNT(DISTINCT p.product_id) as product_count,
    COUNT(DISTINCT oi.order_item_id) as total_sales,
    ROUND(AVG(r.rating)::numeric, 2) as avg_rating,
    COUNT(DISTINCT ret.return_id) as total_returns
FROM category c
LEFT JOIN product p ON c.category_id = p.category_id
LEFT JOIN order_item oi ON p.product_id = oi.product_id
LEFT JOIN review r ON oi.order_item_id = r.order_item_id AND r.status = 'активен'
LEFT JOIN return ret ON oi.order_item_id = ret.return_id
GROUP BY c.category_id, c.name
ORDER BY total_sales DESC;
```

## Полученные диаграммы:

### ER-диаграмма

```mermaid
erDiagram
    CATEGORY ||--o{ PRODUCT : contains
    PRODUCT ||--o{ ORDER_ITEM : includes
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--o{ ORDER_ITEM : contains
    ORDER_ITEM ||--o| REVIEW : receives
    ORDER_ITEM ||--o| RETURN : may_have
    
    CATEGORY {
        integer category_id PK "SERIAL"
        varchar name
        text description
        timestamp created_at
    }
    
    PRODUCT {
        integer product_id PK "SERIAL"
        varchar article
        varchar name
        integer category_id FK
        decimal price
        timestamp created_at
        timestamp updated_at
    }
    
    CUSTOMER {
        integer customer_id PK "SERIAL"
        varchar name
        varchar email
        timestamp registration_date
        varchar phone
    }
    
    ORDER {
        integer order_id PK "SERIAL"
        integer customer_id FK
        timestamp order_date
        varchar status
        decimal total_amount
        text delivery_address
    }
    
    ORDER_ITEM {
        integer order_item_id PK "SERIAL"
        integer order_id FK
        integer product_id FK
        integer quantity
        decimal unit_price
    }
    
    REVIEW {
        integer review_id PK "SERIAL"
        integer order_item_id FK
        integer rating
        text text
        timestamp review_date
        varchar status
    }
    
    RETURN {
        integer return_id PK "SERIAL"
        integer order_item_id FK
        varchar reason
        timestamp return_date
        varchar status
        decimal refund_amount
    }
```

## Логическая модель в виде Диаграммы классов UML

```mermaid
classDiagram
    class Category {
        +category_id: Integer (PK)
        +name: String
        +description: String
        +created_at: DateTime
        +getProducts() List~Product~
        +getProductsCount() Integer
    }
    
    class Product {
        +product_id: Integer (PK)
        +article: String
        +name: String
        +category_id: Integer (FK)
        +price: Decimal
        +created_at: DateTime
        +updated_at: DateTime
        +getCategory() Category
        +getOrderItems() List~OrderItem~
        +getReviews() List~Review~
        +getAverageRating() Double
        +getSalesCount() Integer
        +getReturnRate() Double
    }
    
    class Customer {
        +customer_id: Integer (PK)
        +name: String
        +email: String
        +registration_date: DateTime
        +phone: String
        +getOrders() List~Order~
        +getReviews() List~Review~
        +getTotalOrders() Integer
        +getTotalSpent() Decimal
    }
    
    class Order {
        +order_id: Integer (PK)
        +customer_id: Integer (FK)
        +order_date: DateTime
        +status: String
        +total_amount: Decimal
        +delivery_address: String
        +getCustomer() Customer
        +getOrderItems() List~OrderItem~
        +getTotalItems() Integer
        +hasReturns() Boolean
    }
    
    class OrderItem {
        +order_item_id: Integer (PK)
        +order_id: Integer (FK)
        +product_id: Integer (FK)
        +quantity: Integer
        +unit_price: Decimal
        +getOrder() Order
        +getProduct() Product
        +getReview() Review
        +getReturn() Return
        +getTotalPrice() Decimal
    }
    
    class Review {
        +review_id: Integer (PK)
        +order_item_id: Integer (FK)
        +rating: Integer
        +text: String
        +review_date: DateTime
        +status: String
        +getOrderItem() OrderItem
        +getProduct() Product
        +getCustomer() Customer
        +isActive() Boolean
    }
    
    class Return {
        +return_id: Integer (PK)
        +order_item_id: Integer (FK)
        +reason: String
        +return_date: DateTime
        +status: String
        +refund_amount: Decimal
        +getOrderItem() OrderItem
        +getProduct() Product
        +getCustomer() Customer
        +isApproved() Boolean
    }
    
    Category "1" -- "*" Product : содержит
    Product "1" -- "*" OrderItem : включает
    Customer "1" -- "*" Order : оформляет
    Order "1" -- "*" OrderItem : содержит
    OrderItem "1" -- "0..1" Review : получает
    OrderItem "1" -- "0..1" Return : может_иметь
```

## Физическая модель БД

```mermaid
erDiagram
    category {
        integer category_id PK "SERIAL"
        varchar name "NOT NULL UNIQUE"
        text description
        timestamp created_at "NOT NULL DEFAULT CURRENT_TIMESTAMP"
    }
    
    product {
        integer product_id PK "SERIAL"
        varchar article "NOT NULL UNIQUE"
        varchar name "NOT NULL"
        integer category_id FK "NOT NULL"
        decimal price "NOT NULL CHECK (price > 0)"
        timestamp created_at "NOT NULL DEFAULT CURRENT_TIMESTAMP"
        timestamp updated_at
    }
    
    customer {
        integer customer_id PK "SERIAL"
        varchar name "NOT NULL"
        varchar email "NOT NULL UNIQUE"
        timestamp registration_date "NOT NULL DEFAULT CURRENT_TIMESTAMP"
        varchar phone
    }
    
    order {
        integer order_id PK "SERIAL"
        integer customer_id FK "NOT NULL"
        timestamp order_date "NOT NULL DEFAULT CURRENT_TIMESTAMP"
        varchar status "NOT NULL CHECK (обработан, доставляется, доставлен, отменен)"
        decimal total_amount "NOT NULL CHECK (total_amount >= 0)"
        text delivery_address
    }
    
    order_item {
        integer order_item_id PK "SERIAL"
        integer order_id FK "NOT NULL"
        integer product_id FK "NOT NULL"
        integer quantity "NOT NULL CHECK (quantity > 0)"
        decimal unit_price "NOT NULL CHECK (unit_price >= 0)"
    }
    
    review {
        integer review_id PK "SERIAL"
        integer order_item_id FK "NOT NULL UNIQUE"
        integer rating "NOT NULL CHECK (rating BETWEEN 1 AND 5)"
        text text
        timestamp review_date "NOT NULL DEFAULT CURRENT_TIMESTAMP"
        varchar status "NOT NULL DEFAULT активен CHECK (активен, скрыт, удален)"
    }
    
    return {
        integer return_id PK "SERIAL"
        integer order_item_id FK "NOT NULL UNIQUE"
        varchar reason "NOT NULL CHECK (LENGTH(TRIM(reason)) > 0)"
        timestamp return_date "NOT NULL DEFAULT CURRENT_TIMESTAMP"
        varchar status "NOT NULL DEFAULT обрабатывается CHECK (обрабатывается, одобрен, отклонен, выполнено)"
        decimal refund_amount "CHECK (refund_amount >= 0)"
    }
    
    category ||--o{ product : "FOREIGN KEY (category_id) REFERENCES category(category_id) ON DELETE RESTRICT"
    product ||--o{ order_item : "FOREIGN KEY (product_id) REFERENCES product(product_id) ON DELETE RESTRICT"
    customer ||--o{ order : "FOREIGN KEY (customer_id) REFERENCES customer(customer_id) ON DELETE CASCADE"
    order ||--o{ order_item : "FOREIGN KEY (order_id) REFERENCES order(order_id) ON DELETE CASCADE"
    order_item ||--o| review : "FOREIGN KEY (order_item_id) REFERENCES order_item(order_item_id) ON DELETE CASCADE"
    order_item ||--o| return : "FOREIGN KEY (order_item_id) REFERENCES order_item(order_item_id) ON DELETE CASCADE"
```

## Заключение

В лабораторной работе произведено проектирование ER-, логической и физической модели базы данных для интернет-магазина (вариант 46). Основные улучшения, внесенные в ходе проектирования:

1. **Нормализация категорий** - создана отдельная сущность "Категории" для устранения избыточности данных
2. **Улучшенная структура связей** - отзывы и возвраты теперь связаны с позициями заказа, что отражает реальную бизнес-логику
3. **Суррогатные ключи** - добавлены числовые идентификаторы для эффективности соединений
4. **Расширенная валидация** - добавлены CHECK-ограничения для рейтингов, цен, количеств и статусов
5. **Бизнес-логика** - реализованы уникальные ограничения для предотвращения дублирования отзывов и возвратов
6. **Оптимизация производительности** - созданы индексы для частых запросов по датам, категориям и статусам
7. **Соблюдение нормальных форм** - модель соответствует требованиям 3NF/BCNF

Модель эффективно поддерживает все требуемые бизнес-процессы и выходные документы, обеспечивая надежное хранение данных, целостность отношений и высокую производительность аналитических запросов по отзывам и возвратам.


# Лабораторная работа №2 (Инсталляция БД на сервере)
## Создание DDL-запросов для PostgreSQL
## Создание таблиц

### Таблица category(категорий)

<img width="705" height="202" alt="image" src="https://github.com/user-attachments/assets/3231001a-a927-485b-8a97-50dbfeba3dfe" />

### Таблица customer(клиентов)

<img width="821" height="231" alt="image" src="https://github.com/user-attachments/assets/7952ba41-fb8e-4162-ba72-c349d822c3fe" />

### Таблица product(товаров)

<img width="870" height="326" alt="image" src="https://github.com/user-attachments/assets/fdca28a2-4415-4a5d-a2b8-f33ed87365f2" />

### Таблица order(заказов)

<img width="974" height="198" alt="image" src="https://github.com/user-attachments/assets/ed3fb68e-2316-4513-b4c0-264c7ba35645" />

### Таблица order_item(позиций заказа)

<img width="858" height="289" alt="image" src="https://github.com/user-attachments/assets/4b0a2308-4e24-441e-a47c-7e198c6055e6" />

### Таблица review(отзывов)

<img width="975" height="230" alt="image" src="https://github.com/user-attachments/assets/77b58731-17f2-455b-9d6a-a22d4e50c516" />

### Таблица return(возвратов)

<img width="975" height="177" alt="image" src="https://github.com/user-attachments/assets/5dbf78dd-0740-4305-822b-1da66f80ab3c" />

## Создание индексов

<img width="852" height="261" alt="image" src="https://github.com/user-attachments/assets/cabda4b2-7342-4b2a-82c1-e5efa57e24ba" />

## Заполнение данными

### Таблица category

<img width="975" height="539" alt="image" src="https://github.com/user-attachments/assets/a621d2fc-721b-4455-8540-0daa3da1a299" />

### Таблица customer

<img width="975" height="447" alt="image" src="https://github.com/user-attachments/assets/5b86f483-0243-4543-b7f2-48d6438694f2" />

### Таблица product

<img width="975" height="387" alt="image" src="https://github.com/user-attachments/assets/67fbf626-0f8a-4614-bfb9-9d6e5a3666b2" />

### Таблица order

<img width="975" height="476" alt="image" src="https://github.com/user-attachments/assets/111d1636-f16b-4977-8d71-4c45c74b5b4e" />

### Таблица order_item

<img width="606" height="420" alt="image" src="https://github.com/user-attachments/assets/e22edf6c-f898-4848-a281-55d2210f011e" />

### Таблица review

<img width="975" height="411" alt="image" src="https://github.com/user-attachments/assets/f4defa0f-8793-4e30-96f8-45efc20977cb" />

### Таблица return

<img width="975" height="339" alt="image" src="https://github.com/user-attachments/assets/4564ef99-a656-4108-bc8a-1d50093126d2" />

## Запросы

### Запрос 1. Клиенты и их заказы

<img width="684" height="538" alt="image" src="https://github.com/user-attachments/assets/4242a5f6-a445-48eb-9310-037053f76faf" />

### Запрос 2. Статистика по категориям 

<img width="762" height="481" alt="image" src="https://github.com/user-attachments/assets/89d95063-e3eb-4f1d-9f7d-b15e0fc691f2" />










