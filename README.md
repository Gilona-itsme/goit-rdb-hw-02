# Нормалізація бази даних: замовлення, клієнти, товари
# Database Normalization: Orders, Customers, Products

Домашнє завдання з проєктування реляційних баз даних: нормалізація вихідної таблиці до 3НФ, побудова ER-діаграми та створення таблиць у MySQL.

*Relational database design homework: normalizing a source table to 3NF, building an ER diagram, and creating the tables in MySQL.*

## Зміст / Contents

- [Файли / Files](#файли--files)
- [Початкова таблиця / Initial table](#початкова-таблиця--initial-table)
- [1НФ / 1NT](#1нф--1nt)
- [2НФ / 2NT](#2нф--2nt)
- [3НФ / 3NT](#3нф--3nt)
- [ER-діаграма / ER diagram](#er-діаграма--er-diagram)
- [Таблиці БД / Database tables](#таблиці-бд--database-tables)
- [Запуск / Setup](#запуск--setup)

## Файли / Files

| Файл / File | Опис / Description |
|---|---|
| [`p0_initial_table.png`](./p0_initial_table.png) | Початкова ненормалізована таблиця / Original unnormalized table |
| [`p1_1NT.png`](./p1_1NT.png) | Таблиця в 1НФ / Table in 1NT |
| [`p2_2NT.png`](./p2_2NT.png) | Таблиці в 2НФ / Tables in 2NT |
| [`p3_3NT.png`](./p3_3NT.png) | Таблиці в 3НФ / Tables in 3NT |
| [`p4_ER_diagram.png`](./p4_ER_diagram.png) | ER-діаграма / ER diagram |
| [`p5_DB_tables.sql`](./p5_DB_tables.sql) | SQL-скрипт створення таблиць (без даних) / SQL script creating the tables (no data) |

## Початкова таблиця / Initial table

Складене поле «Назва товару і кількість» містить кілька товарів в одному рядку; адреса та ім'я клієнта дублюються для кожного замовлення.

*The composite "product name and quantity" field holds several products in one row; customer name and address are duplicated for every order.*

| Номер_замовлення | Назва_товару і кількість | Адреса_клієнта | Дата_замовлення | Клієнт |
|---|---|---|---|---|
| 101 | Лептоп: 3, Мишка: 2 | Хрещатик 1 | 2023-03-15 | Мельник |
| 102 | Принтер: 1 | Басейна 2 | 2023-03-16 | Шевченко |
| 103 | Мишка: 4 | Комп'ютерна 3 | 2023-03-17 | Коваленко |

## 1НФ / 1NT

Складене поле розбито на атомарні значення — по одному товару в рядку.

*The composite field is split into atomic values — one product per row.*

| Order_number | Product_name | Quantity | Customer_address | Order_date | Customer |
|---|---|---|---|---|---|
| 101 | Laptop | 3 | Khreshchatyk 1 | 2023-03-15 | Melnyk |
| 101 | Mouse | 2 | Khreshchatyk 1 | 2023-03-15 | Melnyk |
| 102 | Printer | 1 | Baseina 2 | 2023-03-16 | Shevchenko |
| 103 | Mouse | 4 | Kompiuterna 3 | 2023-03-17 | Kovalenko |

## 2НФ / 2NT

Усунено часткові залежності: дата, клієнт і адреса залежать лише від `Order_number`, а не від складеного ключа `(Order_number, Product_name)`.

*Partial dependencies removed: date, customer and address depend only on `Order_number`, not on the composite key `(Order_number, Product_name)`.*

**Orders**(Order_number, Order_date, Customer, Customer_address)
**Order_details**(Order_number, Product_name, Quantity)

## 3НФ / 3NT

Усунено транзитивну залежність `Order_number → Customer → Customer_address`; клієнтів і товари винесено в окремі таблиці.

*Transitive dependency `Order_number → Customer → Customer_address` removed; customers and products moved into separate tables.*

- **Orders**(Order_number, Order_date, Customer FK)
- **Customers**(Customer PK, Customer_name, Customer_address)
- **Order_details**(Order_number, Product_name FK → id_product, Quantity)
- **Product**(id_product PK, Product_name)

## ER-діаграма / ER diagram

- `customers` 1 — ∞ `orders`: клієнт може мати багато замовлень, замовлення належить одному клієнту. / *a customer can place many orders; each order belongs to one customer.*
- `orders` 1 — ∞ `order_details`: замовлення містить багато позицій. / *an order contains many line items.*
- `products` 1 — ∞ `order_details`: товар може входити в багато позицій. / *a product can appear in many line items.*
- `orders` M — N `products` реалізовано через проміжну таблицю `order_details`. / *the M:N relationship between orders and products is implemented via the junction table `order_details`.*

Діаграма: [`p4_ER_diagram.png`](./p4_ER_diagram.png)

## Таблиці БД / Database tables

Схема без даних, лише колонки та зв'язки — файл [`p5_DB_tables.sql`](./p5_DB_tables.sql).

*Schema only, no data — columns and relationships in [`p5_DB_tables.sql`](./p5_DB_tables.sql).*

- **customers**(id PK, name, address)
- **products**(id PK, title_product UNIQUE)
- **orders**(id PK, date, id_customer FK → customers.id)
- **order_details**(id_product FK → products.id, quantity, id_order FK → orders.id)

```
customers ──< orders ──< order_details >── products
```

> Примітка: у назві таблиці `сustomers` у SQL-файлі перша літера кирилична (`с`), а не латинська — варто виправити перед фінальним запуском на іншому середовищі, щоб уникнути плутанини з кодуванням назв.
>
> *Note: the table name `сustomers` in the SQL file starts with a Cyrillic `с`, not a Latin one — worth fixing before running on another environment to avoid identifier-encoding confusion.*

## Запуск / Setup

```bash
mysql -u root -p < p5_DB_tables.sql
```

Або імпортувати скрипт у MySQL Workbench і згенерувати ER-діаграму через **Database → Reverse Engineer**.

*Or import the script into MySQL Workbench and generate the ER diagram via **Database → Reverse Engineer**.*
