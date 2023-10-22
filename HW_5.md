# HOMEWORK #5. Indexes.

### 1. Создать индекс к какой-либо из таблиц вашей БД.
```sql
-- Создать индекс на таблицу "Покупатели" в схеме "Потребители" на поле "Фамилия":
create index idx_customer_ln 
on consumer.customer(last_name);
```

### 2. Прислать текстом результат команды explain, в которой используется данный индекс.

> __План запроса буду получать:__
```sql
explain (costs, verbose, format json, analyze)
-- + запрос;
``` 
-- ----------
```sql
explain (costs, verbose, format json, analyze)
select * from consumer.customer c
where c.last_name = 'Vavilov';
```
```json
-- С использованием индекса:
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Parallel Aware": false,
      "Relation Name": "customer",
      "Schema": "consumer",
      "Alias": "c",
      "Startup Cost": 0.00,
      "Total Cost": 1.20,
      "Plan Rows": 1,
      "Plan Width": 1708,
      "Actual Startup Time": 0.012,
      "Actual Total Time": 0.013,
      "Actual Rows": 1,
      "Actual Loops": 1,
      "Output": ["id", "first_name", "last_name", "second_name", "alias", "phone", "email", "city", "is_company"],
      "Filter": "((c.last_name)::text = 'Vavilov'::text)",
      "Rows Removed by Filter": 15
    },
    "Planning Time": 0.077,
    "Triggers": [
    ],
    "Execution Time": 0.024
  }
]
-- Без использования индекса:
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Parallel Aware": false,
      "Relation Name": "customer",
      "Schema": "consumer",
      "Alias": "c",
      "Startup Cost": 0.00,
      "Total Cost": 1.20,
      "Plan Rows": 1,
      "Plan Width": 1708,
      "Actual Startup Time": 0.015,
      "Actual Total Time": 0.016,
      "Actual Rows": 1,
      "Actual Loops": 1,
      "Output": ["id", "first_name", "last_name", "second_name", "alias", "phone", "email", "city", "is_company"],
      "Filter": "((c.last_name)::text = 'Vavilov'::text)",
      "Rows Removed by Filter": 15
    },
    "Planning Time": 0.145,
    "Triggers": [
    ],
    "Execution Time": 0.031
  }
]
```

### 3.Реализовать индекс для полнотекстового поиска.
```sql
-- Добавлю в таблицу новое поле:
alter table consumer.customer add column display tsvector;

-- заполню его данными и воспользуюсь функцией, которая разберет данные на "токены":
update consumer.customer
set display = to_tsvector(first_name);	

-- создадим обратный индекс (к полнотекстовому поиску):
create index idx_search_title 
on consumer.customer using gin(display);

-- запрос + план запроса:
explain (costs, verbose, format json, analyze)
select * from consumer.customer c 
where c.display @@ to_tsquery('victor');
```
```json
-- С использованием gin-индекса:
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Parallel Aware": false,
      "Relation Name": "customer",
      "Schema": "consumer",
      "Alias": "c",
      "Startup Cost": 0.00,
      "Total Cost": 5.20,
      "Plan Rows": 1,
      "Plan Width": 1740,
      "Actual Startup Time": 0.018,
      "Actual Total Time": 0.085,
      "Actual Rows": 1,
      "Actual Loops": 1,
      "Output": ["id", "first_name", "last_name", "second_name", "alias", "phone", "email", "city", "is_company", "display"],
      "Filter": "(c.display @@ to_tsquery('victor'::text))",
      "Rows Removed by Filter": 15
    },
    "Planning Time": 5.652,
    "Triggers": [
    ],
    "Execution Time": 0.098
  }
]
-- Без использования gin-индекса:
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Parallel Aware": false,
      "Relation Name": "customer",
      "Schema": "consumer",
      "Alias": "c",
      "Startup Cost": 0.00,
      "Total Cost": 5.20,
      "Plan Rows": 1,
      "Plan Width": 1740,
      "Actual Startup Time": 0.020,
      "Actual Total Time": 0.092,
      "Actual Rows": 1,
      "Actual Loops": 1,
      "Output": ["id", "first_name", "last_name", "second_name", "alias", "phone", "email", "city", "is_company", "display"],
      "Filter": "(c.display @@ to_tsquery('victor'::text))",
      "Rows Removed by Filter": 15
    },
    "Planning Time": 1.236,
    "Triggers": [
    ],
    "Execution Time": 0.105
  }
]
```
### 4. Реализовать индекс на часть таблицы или индекс на поле с функцией.
```sql
-- Создать индекс по городу в схеме "Потребители" в таблице "Покупатели" по полю "Город":
CREATE INDEX idx_customer_city 
ON consumer.customer USING btree (lower((city)::text));

-- запрос + план запроса:
explain (costs, verbose, format json, analyze)
select * from consumer.customer c 
where lower(c.city) = 'sochi'
```

```json
-- С использованием индекса:
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Parallel Aware": false,
      "Relation Name": "customer",
      "Schema": "consumer",
      "Alias": "c",
      "Startup Cost": 0.00,
      "Total Cost": 1.24,
      "Plan Rows": 1,
      "Plan Width": 1740,
      "Actual Startup Time": 0.014,
      "Actual Total Time": 0.028,
      "Actual Rows": 3,
      "Actual Loops": 1,
      "Output": ["id", "first_name", "last_name", "second_name", "alias", "phone", "email", "city", "is_company", "display"],
      "Filter": "(lower((c.city)::text) = 'sochi'::text)",
      "Rows Removed by Filter": 13
    },
    "Planning Time": 0.221,
    "Triggers": [
    ],
    "Execution Time": 0.042
  }
]
-- Без использования индекса:
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Parallel Aware": false,
      "Relation Name": "customer",
      "Schema": "consumer",
      "Alias": "c",
      "Startup Cost": 0.00,
      "Total Cost": 1.24,
      "Plan Rows": 1,
      "Plan Width": 1740,
      "Actual Startup Time": 0.017,
      "Actual Total Time": 0.033,
      "Actual Rows": 3,
      "Actual Loops": 1,
      "Output": ["id", "first_name", "last_name", "second_name", "alias", "phone", "email", "city", "is_company", "display"],
      "Filter": "((c.city)::text = 'Sochi'::text)",
      "Rows Removed by Filter": 13
    },
    "Planning Time": 0.254,
    "Triggers": [
    ],
    "Execution Time": 0.048
  }
]
```

### 5. Создать индекс на несколько полей.
```sql
-- Создать индекс в схеме "Потребители" по таблице Покупатели на поля: "Имя" и "Фамилия".
create index idx_customer_fn_ln 
on consumer.customer(first_name, last_name);

-- запрос + план запроса:
explain (costs, verbose, format json, analyze)
select * from consumer.customer c 
where c.first_name = 'Victor'
and   c.last_name = 'Arephev'
```
```json
-- С использованием индекса:
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Parallel Aware": false,
      "Relation Name": "customer",
      "Schema": "consumer",
      "Alias": "c",
      "Startup Cost": 0.00,
      "Total Cost": 1.24,
      "Plan Rows": 1,
      "Plan Width": 1740,
      "Actual Startup Time": 0.010,
      "Actual Total Time": 0.014,
      "Actual Rows": 1,
      "Actual Loops": 1,
      "Output": ["id", "first_name", "last_name", "second_name", "alias", "phone", "email", "city", "is_company", "display"],
      "Filter": "(((c.first_name)::text = 'Victor'::text) AND ((c.last_name)::text = 'Arephev'::text))",
      "Rows Removed by Filter": 15
    },
    "Planning Time": 0.163,
    "Triggers": [
    ],
    "Execution Time": 0.028
  }
]

-- Без использования индекса:
[
  {
    "Plan": {
      "Node Type": "Seq Scan",
      "Parallel Aware": false,
      "Relation Name": "customer",
      "Schema": "consumer",
      "Alias": "c",
      "Startup Cost": 0.00,
      "Total Cost": 1.24,
      "Plan Rows": 1,
      "Plan Width": 1740,
      "Actual Startup Time": 0.010,
      "Actual Total Time": 0.015,
      "Actual Rows": 1,
      "Actual Loops": 1,
      "Output": ["id", "first_name", "last_name", "second_name", "alias", "phone", "email", "city", "is_company", "display"],
      "Filter": "(((c.first_name)::text = 'Victor'::text) AND ((c.last_name)::text = 'Arephev'::text))",
      "Rows Removed by Filter": 15
    },
    "Planning Time": 0.252,
    "Triggers": [
    ],
    "Execution Time": 0.031
  }
]
```
### 6. Написать комментарии к каждому из индексов.
> Комментарии даны по каждому заданию.

### 7. Описать что и как делали и с какими проблемами столкнулись.
> Комментарии даны по каждому заданию, проблем никаких не было.

