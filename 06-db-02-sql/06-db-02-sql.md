# Домашнее задание к занятию 2. «SQL»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

## Ответ 1



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

Подключаемся к Docker контейнеру:

`docker exec -it 3ca74064fd97 psql -U postgres`

Создаем пользователя:

`postgres=# CREATE USER test_admin WITH PASSWORD '12345678';`

Создаем БД:

`postgres=# CREATE DATABASE test_db;`

Выбираем БД:

`postgres=# psql test_db`

Cоздаем табилцу orders:

`CREATE TABLE orders (`
`    id SERIAL PRIMARY KEY,`
`    наименование VARCHAR(255),`
`    цена INTEGER`
`);`

`CREATE TABLE clients (`
`    id SERIAL PRIMARY KEY,`
`    фамилия VARCHAR(255),`
`    страна_проживания VARCHAR(255),`
`    заказ_id INT REFERENCES orders(id)`
`);`

Предоставляем привилегии на все операции пользователю test-admin-user на таблицы БД test_db;

`postgres=# GRANT ALL ON orders, clients TO "test_admin";`

Создайте пользователя test-simple-user;

`CREATE USER test_user WITH PASSWORD '123456';`

Предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

`GRANT SELECT,INSERT,UPDATE,DELETE ON orders, clients to "test_user";`

Приведите:

- итоговый список БД после выполнения пунктов выше;

`SELECT datname FROM pg_database;`

`  datname`
`-----------`
` postgres`
` test_db`
` template1`
` template0`
`(4 rows)`

- описание таблиц (describe);

`SELECT table_name, table_type`
`FROM information_schema.tables`
`WHERE table_schema = 'public';`

 `table_name | table_type`
`------------+------------`
` orders     | BASE TABLE`
` clients    | BASE TABLE`
`(2 rows)`

- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;

`SELECT table_name,grantee,privilege_type` 
`FROM information_schema.table_privileges`
`WHERE table_schema NOT IN ('information_schema','pg_catalog');`

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

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

