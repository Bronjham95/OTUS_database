# HOMEWORK #9. Типы данных в MySQL.

### 1. Проанализировать типы данных в своем проекте, изменить при необходимости. В README указать что на что поменялось и почему.
> Анализ проведен, ничего не менял.

### 2.Добавить тип JSON в структуру. Проанализировать какие данные могли бы там хранится. привести примеры SQL для добавления записей и выборки.

> Добавление в таблицу products поля с типом json product_info:

```sql
ALTER TABLE Product
ADD product_info json NOT NULL;
```
-- ----------------------------------------
> Добавим данные в таблицу:
```sql
INSERT INTO store.product
(id, product_name, category_id, list_price, part_id product_info)
VALUES(1, 'milk', 1, 70.55, 1111, '{"id": 1, "good": {"Country": "Russia", "full_name": "House in the village", "Manufacturer": "Super Milk"}, "name": "Milk", "storage": {"Packed": "21.12.2023", "Weight": 1000, "Package": "Cardboard", "Produced": "20.12.2023", "Temperature": 3}}');

INSERT INTO store.product
(id, product_name, category_id, list_price, part_id, product_info)
VALUES(2, 'bread', 2, 32.1, 1111, '{"id": 2, "good": {"Country": "Russia", "full_name": "Bradley Cooper", "Manufacturer": "WoW, bread"}, "name": "Bread", "storage": {"Packed": "21.12.2023", "Weight": 200, "Package": "Polyethylene", "Produced": "20.12.2023", "Temperature": 12}}');

INSERT INTO store.product
(id, product_name, category_id, list_price, part_id, product_info)
VALUES(3, 'banana', 3, 40.1, 1111, '{"id": 3, "good": {"Country": "Russia", "full_name": "Chynga-Changa", "Manufacturer": "Bana-na"}, "name": "Banana", "storage": {"Packed": "19.12.2023", "Weight": 10000, "Package": "Box", "Produced": "10.12.2023", "Temperature": 18}}');

INSERT INTO store.product
(id, product_name, category_id, list_price, part_id, product_info)
VALUES(4, 'cheese', 1, 80.5, 1111, '{"id": 4, "good": {"Country": "Russia", "full_name": "Orbit", "Manufacturer": "Lugansk cheese factory"}, "name": "Cheese", "storage": {"Packed": "20.12.2023", "Weight": 500, "Package": "Polyethylene", "Produced": "19.12.2023", "Temperature": 5}}');
```
> Запросы с использованием json:
```sql
-- запрос №1
SELECT p.id, p.product_name, JSON_EXTRACT(p.product_info , "$.storage.Weight") FROM product p;
-- запрос №2
SELECT p.id, p.product_name, JSON_EXTRACT(p.product_info , "$.storage") FROM product p;
-- запрос №3
SELECT p.id, p.product_name, JSON_EXTRACT(p.product_info , "$.storage") FROM product p
where JSON_EXTRACT(p.product_info , "$.storage.Weight") < 500;
```
