# Домашнее задание к занятию 2. «SQL - Потапов Василий»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

## Ответ 1

```docker
docker run --name postgre1 -d -e POSTGRES_HOST_AUTH_METHOD=trust -v C:/Temp/sql_base:/var/lib/postgresql/data -v C:/Temp/sql_backup:/tmp/backup postgres:12
```
`- e POSTGRES_HOST_AUTH_METHOD=trust` - устанавливает переменную окружения `POSTGRES_HOST_AUTH_METHOD` в значение `trust`. Это означает, что аутентификация хоста будет доверенной, что позволяет подключаться к серверу PostgreSQL без аутентификации.

`-v C:/Temp/sql_base:/var/lib/postgresql/data` - монтирует каталог `C:/Temp/sql_base` с хост-системы в каталог `/var/lib/postgresql/data` внутри контейнера. 

`-v C:/Temp/sql_backup:/tmp/backup` - монтирует каталог `C:/Temp/sql_backup` с хост-системы в каталог `/tmp/backup внутри контейнера`. 

Это позволяет хранить данные PostgreSQL на хост-системе.

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;

## Ответ 2

Подключаемся к Docker-контейнеру:

```docker
docker exec -it postgre1 psql -U postgres
```

Создаем пользователя test_admin:
```sql
CREATE USER test_admin WITH PASSWORD '12345678';
```
Создаем БД:
```sql
CREATE DATABASE test_db;
```
Переключаем БД:
```sql
\c test_db
You are now connected to database "test_db" as user "postgres".
```

Cоздаем таблицы orders, clients:

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    наименование VARCHAR,
    цена INTEGER
);

CREATE TABLE clients (
    id SERIAL PRIMARY KEY,
    фамилия VARCHAR,
    страна_проживания VARCHAR,
    заказ_id INT REFERENCES orders(id)
);

CREATE INDEX country_index ON clients (страна_проживания);
```

Предоставляем привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
```sql
GRANT ALL ON orders, clients TO "test_admin";
```
Создайте пользователя test-simple-user;
```sql
CREATE USER test_user WITH PASSWORD '123456';
```
Предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.
```sql
GRANT SELECT,INSERT,UPDATE,DELETE ON orders, clients to "test_user";
```
Приведите:

- итоговый список БД после выполнения пунктов выше;
```sql
SELECT datname FROM pg_database;

  datname
-----------
 postgres
 test_db
 template1
 template0
(4 rows)
```
- описание таблиц (describe);
```sql
\d+ clients

                                                                  Table "public.clients"
      Column       |       Type        | Collation | Nullable |               Default               | Storage  | Compression | Stats target | Description
-------------------+-------------------+-----------+----------+-------------------------------------+----------+-------------+--------------+-------------
 id                | integer           |           | not null | nextval('clients_id_seq'::regclass) | plain    |             |              |
 фамилия           | character varying |           |          |                                     | extended |             |              |
 страна_проживания | character varying |           |          |                                     | extended |             |              |
 заказ_id          | integer           |           |          |                                     | plain    |             |              |
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "country_index" btree ("страна_проживания")
Foreign-key constraints:
    "clients_заказ_id_fkey" FOREIGN KEY ("заказ_id") REFERENCES orders(id)
Access method: heap
```

```sql
\d+ orders

                                                               Table "public.orders"
    Column    |       Type        | Collation | Nullable |              Default               | Storage  | Compression | Stats target | Description
--------------+-------------------+-----------+----------+------------------------------------+----------+-------------+--------------+-------------
 id           | integer           |           | not null | nextval('orders_id_seq'::regclass) | plain    |             |              |
 наименование | character varying |           |          |                                    | extended |             |              |
 цена         | integer           |           |          |                                    | plain    |             |              |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_id_fkey" FOREIGN KEY ("заказ_id") REFERENCES orders(id)
Access method: heap
```

- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db:

```sql
SELECT table_name,grantee,privilege_type 
FROM information_schema.table_privileges
WHERE table_schema NOT IN ('information_schema','pg_catalog');

 table_name |  grantee   | privilege_type
------------+------------+----------------
 orders     | postgres   | INSERT
 orders     | postgres   | SELECT
 orders     | postgres   | UPDATE
 orders     | postgres   | DELETE
 orders     | postgres   | TRUNCATE
 orders     | postgres   | REFERENCES
 orders     | postgres   | TRIGGER
 clients    | postgres   | INSERT
 clients    | postgres   | SELECT
 clients    | postgres   | UPDATE
 clients    | postgres   | DELETE
 clients    | postgres   | TRUNCATE
 clients    | postgres   | REFERENCES
 clients    | postgres   | TRIGGER
 orders     | test_admin | INSERT
 orders     | test_admin | SELECT
 orders     | test_admin | UPDATE
 orders     | test_admin | DELETE
 orders     | test_admin | TRUNCATE
 orders     | test_admin | REFERENCES
 orders     | test_admin | TRIGGER
 clients    | test_admin | INSERT
 clients    | test_admin | SELECT
 clients    | test_admin | UPDATE
 clients    | test_admin | DELETE
 clients    | test_admin | TRUNCATE
 clients    | test_admin | REFERENCES
 clients    | test_admin | TRIGGER
 orders     | test_user  | INSERT
 orders     | test_user  | SELECT
 orders     | test_user  | UPDATE
 orders     | test_user  | DELETE
 clients    | test_user  | INSERT
 clients    | test_user  | SELECT
 clients    | test_user  | UPDATE
 clients    | test_user  | DELETE
(36 rows)
```

## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:

- запросы,
- результаты их выполнения.

```sql
INSERT INTO orders (наименование,цена) VALUES
('Шоколад',10),
('Принтер',3000),
('Книга',500),
('Монитор',7000),
('Гитара',4000);
```

```sql
ALTER TABLE clients ALTER COLUMN заказ_id DROP NOT NULL;
```

```sql
INSERT INTO clients (фамилия,страна_проживания,заказ_id) VALUES
('Иванов Иван Иванович','USA',NULL),
('Петров Петр Петрович','Canada',NULL),
('Иоганн Себастьян Бах','Japan',NULL),
('Ронни Джеймс Дио','Russia',NULL),
('Ritchie Blackmore','Russia',NULL);
```

Количество записей для таблиц:
```sql
select count(*) from clients;
 count
-------
     5
```

```sql
select count(*) from orders;
 count
-------
     5
```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
 
Подсказка: используйте директиву `UPDATE`.

## Ответ 4

Запросы на изменение:

```sql
UPDATE clients
SET заказ_id = (SELECT id FROM orders WHERE наименование = 'Книга')
WHERE фамилия = 'Иванов Иван Иванович';

UPDATE clients
SET заказ_id = (SELECT id FROM orders WHERE наименование = 'Монитор')
WHERE фамилия = 'Петров Петр Петрович';

UPDATE clients
SET заказ_id = (SELECT id FROM orders WHERE наименование = 'Гитара')
WHERE фамилия = 'Иоганн Себастьян Бах';
```
```sql
select * from clients;
 id |       фамилия        | страна  | заказ_id
----+----------------------+---------+----------
  9 | Ронни Джеймс Дио     | Russia  |
 10 | Ritchie Blackmore    | Russia  |
  6 | Иванов Иван Иванович | USA     |        3
  7 | Петров Петр Петрович | Canada  |        4
  8 | Иоганн Себастьян Бах | Japan   |        5
  
select * from orders;
 id |    наименование | цена
----+---------+--------------
  1 |    Шоколад      |    10
  2 |    Принтер      |  3000
  3 |    Книга        |   500
  4 |    Монитор      |  7000
  5 |    Гитара       |  4000
```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

## Ответ 5

```sql
EXPLAIN select * from clients;
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=810 width=72)
(1 row)
```

**Seq Scan** — означает, что используется последовательное, блок за блоком, чтение данных таблицы clients

**Cost** - некая виртуальная величина призванная оценить затратность операции. Первое значение 0.00 — затраты на получение первой строки. Второе — 18.10 — затраты на получение всех строк.

Единица измерения **cost** – «извлечение одной страницы в последовательном (sequential) порядке». То есть оценивается и время, и использование ресурсов.

**rows** — приблизительное количество возвращаемых строк при выполнении операции Seq Scan. Это значение возвращает планировщик.

**width** - это оценка PostgreSQL того, сколько, в среднем, байт содержится в одной строке, возвращенной в рамках данной операции

Как я понял, вывод этой информации - ожидания планировщика.

А если дать команду analyze и повторить запрос, то количество строк будет более реалистичным, и cost поменяется - потому что БД проведёт анализ.


## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

## Ответ 6

Создаем бэкап ролей из контейнера postgre1:

```docker
docker exec -it postgre1 pg_dumpall -U postgres --roles-only -f /tmp/backup/roles.sql
```

Создаем бэкап БД из контейнера postgre1:

```docker
docker exec -it postgre1 pg_dump -h localhost -U postgres -F t -f /tmp/backup/backup_1.tar test_db
```

Запускаем новый докер-контейнер postgre2:

```docker
docker run --name postgre2 -d -e POSTGRES_HOST_AUTH_METHOD=trust -v C:/Temp/sql2_base:/var/lib/postgresql/data -v C:/Temp/sql2_backup:/tmp/backup postgres:12
```

Создаем БД в докер-контейнере postgre2:

```docker
docker exec -it postgre2 psql -U postgres -c "CREATE DATABASE test_db WITH ENCODING='UTF-8';"
```
Восстаналвиваем роли из бэкапа:

```docker
docker exec -it postgre2 psql -U postgres -f /tmp/backup/roles.sql                   
```
Восстанавлием БД из бэкапа:

```sql
docker exec -it postgre2 pg_restore -U postgres -Ft -v -d test_db /tmp/backup/backup_1.tar

pg_restore: connecting to database for restore
pg_restore: creating TABLE "public.clients"
pg_restore: creating SEQUENCE "public.clients_id_seq"
pg_restore: creating SEQUENCE OWNED BY "public.clients_id_seq"
pg_restore: creating TABLE "public.orders"
pg_restore: creating SEQUENCE "public.orders_id_seq"
pg_restore: creating SEQUENCE OWNED BY "public.orders_id_seq"
pg_restore: creating DEFAULT "public.clients id"
pg_restore: creating DEFAULT "public.orders id"
pg_restore: processing data for table "public.clients"
pg_restore: processing data for table "public.orders"
pg_restore: executing SEQUENCE SET clients_id_seq
pg_restore: executing SEQUENCE SET orders_id_seq
pg_restore: creating CONSTRAINT "public.clients clients_pkey"
pg_restore: creating CONSTRAINT "public.orders orders_pkey"
pg_restore: creating INDEX "public.country_index"
pg_restore: creating FK CONSTRAINT "public.clients clients_заказ_id_fkey"
pg_restore: creating ACL "public.TABLE clients"
pg_restore: creating ACL "public.TABLE orders"
```
