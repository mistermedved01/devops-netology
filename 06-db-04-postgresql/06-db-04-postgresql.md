# Домашнее задание к занятию 4. «PostgreSQL» - Потапов Василий

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

- вывода списка БД,
- подключения к БД,
- вывода списка таблиц,
- вывода описания содержимого таблиц,
- выхода из psql.

## Ответ 1

Поднимаем в Docker инстанс PostgreSQL 13, данные БД в volume:

```docker
docker run --name postgres13 -d -e POSTGRES_HOST_AUTH_METHOD=trust -v C:/Temp/postgres13:/var/lib/postgresql/data postgres:13
```

Подключаемся к Docker-контейнеру:

```docker
docker exec -it postgres13 psql -U postgres
```

Вывод списка БД:

```sql
\l
```

Подключение к БД:

```sql
\c <название_базы_данных>
```

Вывод списка таблиц:

```sql
\dt
```

Вывод описания содержимого таблиц:

```sql
\d+ <название_таблицы>
```

Выход из psql:

```sql
\q
```

## Задача 2

Используя `psql`, создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления, и полученный результат.

## Ответ 3

Создаем БД:

```sql
CREATE DATABASE test_database;
```

Выбираем БД:

```sql
\c test_database
```

Восстаналвием БД из бэкапа:

```sql
\i /var/lib/postgresql/data/test_dump111.sql
```
*единственный момент, пришлось поиграться с правами на файл для того чтобы контейнер увидел дамп базы

Ищем столбец таблицы `orders`  с наибольшим средним значением размера элементов в байтах.

```sql
ANALYZE VERBOSE public.orders;

INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE

SELECT tablename, attname, avg_width FROM pg_stats WHERE tablename='orders' ORDER BY avg_width DESC LIMIT 1;

tablename | attname | avg_width
-----------+---------+-----------
 orders    | title   |        16
(1 row)
```


## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

## Ответ 3

Провести разбиение:

```sql
CREATE TABLE orders_1 (CHECK (price < 499)) INHERITS (orders);
CREATE TABLE orders_2 (CHECK (price >= 499)) INHERITS (orders);
```

При необходимости перенести данные из основной таблицы в созданные (если этого не сделать - новые данные будут записываться в новые, а старые останутся на месте)

```sql
INSERT INTO orders_1 SELECT * FROM orders WHERE price < 499;
DELETE FROM only orders WHERE price < 499;
INSERT INTO orders_2 SELECT * FROM orders WHERE price >= 499;
DELETE FROM only orders WHERE price >= 499;
```

Изначально можно было создать такую таблицу:

```sql
CREATE TABLE public.orders_new (
id integer NOT NULL,
title character varying(80) NOT NULL,
price integer DEFAULT 0
)
PARTITION BY RANGE (price);
```

И дополнительно таблицы:

```sql
CREATE TABLE orders_new1 PARTITION OF orders_new FOR VALUES FROM ('0') TO ('499');
CREATE TABLE orders_new2 PARTITION OF orders_new FOR VALUES FROM ('499') TO ('999');
```

Правда в этом случае не получится добавить товар с ценой больше 999, и как создать таблицу для всего "больше 999" я найти не смог :)

Ещё нашёл мощную штуку - https://github.com/pgpartman/pg_partman

Похоже при огромных объёмах она оченеь помочь может :)


## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

## Ответ 4

Добавил бы слово UNIQUE.

```sql
CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) UNIQUE NOT NULL,
    price integer DEFAULT 0
);
```
