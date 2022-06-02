# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql
# Ответ
```bash

$ docker run  --name pqsql1 -e POSTGRES_PASSWORD=psql -v ca11875b5f3eb121c7509975b279c09eb1f345eb77f7b759b183b36745861af4:/vol1 -d postgres:13
59bd264e3310017a16e19f038047dc34df10fba0c56ff8273c3a654bf3933a59

$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
59bd264e3310   postgres:13   "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   5432/tcp                                               pqsql1

:/# mkdir /vol1/psqldb
/# pg_createcluster 13 psql -d /vol1/psqldb
Creating new PostgreSQL cluster 13/psql ...
...
Ver Cluster Port Status Owner    Data directory Log file
13  psql    5433 down   postgres /vol1/psqldb   /var/log/postgresql/postgresql-13-psql.log

root@59bd264e3310:/# service postgresql -start
Usage: /etc/init.d/postgresql {start|stop|restart|reload|force-reload|status} [version ..]
root@59bd264e3310:/# service postgresql start
Starting PostgreSQL 13 database server: psql.
root@af12673a09ee:/# su - postgres

postgres=# \?
# -вывода списка БД
\l[+]   [PATTERN]      list databases
postgres=# \l+
                                                                   List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   |  Size   | Tablespace |                Description
-----------+----------+----------+------------+------------+-----------------------+---------+------------+--------------------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |                       | 7901 kB | pg_default | default administrative connection database
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +| 7753 kB | pg_default | unmodifiable empty database
           |          |          |            |            | postgres=CTc/postgres |         |            |
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +| 7901 kB | pg_default | default template for new databases
           |          |          |            |            | postgres=CTc/postgres |         |            |
(3 rows)

# - подключения к БД
\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
postgres=# \c
You are now connected to database "postgres" as user "postgres".

# - вывода списка таблиц
\dt[S+] [PATTERN]      list tables
postgres=# \dtS
                    List of relations
   Schema   |          Name           | Type  |  Owner
------------+-------------------------+-------+----------
 pg_catalog | pg_aggregate            | table | postgres
 pg_catalog | pg_am                   | table | postgres
...

# - вывода описания содержимого таблиц
 \d[S+]                 list tables, views, and sequences
 \d[S+]  NAME           describe table, view, sequence, or index

postgres=# \dS+
                                            List of relations
   Schema   |              Name               | Type  |  Owner   | Persistence |    Size    | Description
------------+---------------------------------+-------+----------+-------------+------------+-------------
 pg_catalog | pg_aggregate                    | table | postgres | permanent   | 56 kB      |
 pg_catalog | pg_am                           | table | postgres | permanent   | 40 kB      |
 pg_catalog | pg_amop                         | table | postgres | permanent   | 80 kB      |


# - выхода из psql 
\q                     quit psql  
```

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.
```bash
# su - postgres
postgres@af12673a09ee:~$ psql
psql (13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.
postgres=# CREATE DATABASE test_database;
CREATE DATABASE
postgres=# exit

postgres@af12673a09ee:~$ ls -lha /vol1/test
total 12K
-rw-rw-r-- 1 1000 1000 2.1K May 27 18:37 test_dump.sql

postgres@af12673a09ee:~$ psql test_database < /vol1/test/test_dump.sql
SET
...
(1 row)

ALTER TABLE
postgres@af12673a09ee:~$ psql

postgres=# \c test_database
You are now connected to database "test_database" as user "postgres".

test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 8 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE

test_database=#  SELECT avg_width FROM pg_stats WHERE tablename='orders';
 avg_width
-----------
         4
        16
         4
(3 rows)

```
* Ответ команда: SELECT avg_width FROM pg_stats WHERE tablename='orders';

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

* Ответ:
Можно было создать "секционированные" таблицы  указав в них парамерт "PARTITION BY RANGE (price);"  по параметру price , подробно описано [тут] (https://postgrespro.ru/docs/postgresql/10/ddl-partitioning#:~:text=PostgreSQL%20%D0%BF%D1%80%D0%B5%D0%B4%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D1%8F%D0%B5%D1%82%20%D0%B2%D0%BE%D0%B7%D0%BC%D0%BE%D0%B6%D0%BD%D0%BE%D1%81%D1%82%D1%8C%20%D1%83%D0%BA%D0%B0%D0%B7%D0%B0%D1%82%D1%8C%2C%20%D0%BA%D0%B0%D0%BA,%D0%BA%D0%BE%D1%82%D0%BE%D1%80%D1%8B%D0%B5%20%D0%B1%D1%83%D0%B4%D1%83%D1%82%20%D1%81%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D1%8F%D1%82%D1%8C%20%D0%BA%D0%BB%D1%8E%D1%87%20%D1%80%D0%B0%D0%B7%D0%B1%D0%B8%D0%B5%D0%BD%D0%B8%D1%8F)

```bash
test_database=# CREATE TABLE orders_1 (CHECK (price > 499)) INHERITS (orders)
test_database-# ;
CREATE TABLE
test_database=# CREATE TABLE orders_2 (CHECK (price <= 499)) INHERITS (orders);
CREATE TABLE
test_database=# INSERT INTO orders_over_499_price SELECT * FROM orders WHERE price > 499;
INSERT 0 3
test_database=# INSERT INTO orders_below_499_price SELECT * FROM orders WHERE price <= 499;
INSERT 0 5

test_database=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner
--------+----------+-------+----------
 public | orders   | table | postgres
 public | orders_1 | table | postgres
 public | orders_2 | table | postgres
(3 rows)
```

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
```bash
postgres@af12673a09ee:~$ pg_dump -U postgres -d test_database > /vol1/test/test_database.sql
# Сморим что в бекапе
postgres@af12673a09ee:~$ cat /vol1/test/test_database.sql
-- PostgreSQL database dump
CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL,
    price integer DEFAULT 0
);


# добавить критерий UNIQUE
CREATE TABLE public.orders (
    id integer,
    title character varying(80) UNIQUE NOT NULL,
    price integer
);
```
---
