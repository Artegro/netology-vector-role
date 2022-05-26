# Домашнее задание к занятию "6.2. SQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.
Приведите получившуюся команду или docker-compose манифест.

*   Ответ
```bash
# Скопировали образ
$ docker pull postgres:12  # Скопировали образ

#  сделали свой файл
$ nano Dockerfile       
    # base image
    FROM postgres:12
    RUN mkdir /vol1
    VOLUME /vol1
    RUN mkdir /vol2
    VOLUME /vol2

#  Смонтировали свой образ по докерфайлу    
$ docker build -t my:v2 ./         

# Запустили контейнер
$ docker run -it -d my:v2 bash     # Запустили контейнер

#  Проверяем чо котейнер запустился
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS      NAMES
2e22fe70322f   my:v2           "docker-entrypoint.s…"   9 seconds ago   Up 8 seconds   5432/tcp   gallant_williamson


# Проверям что волумы подключились
$ docker exec -it 2e22fe70322 bash
# Заходим в контейнер и создаем файл в /vol2 
root@2e22fe70322f:/# ls
bin  boot  dev  docker-entrypoint-initdb.d  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  vol1  vol2
root@2e22fe70322f:/# echo  test > /vol2/test.txt
root@2e22fe70322f:/# exit

# проверяем что файл создался в волуме докера на хосте
$ sudo ls -l /var/lib/docker/volumes/b0e5a1349f54b78b6240322d9dec52070098b764aad0fdcf4724b3133e244ad9/_data
total 4
-rw-r--r-- 1 root root 5 May 26 14:23 test.txt
# Смотрим конфигурацию ,
 $ docker inspect -f "" 2e22fe70322f
 "Mounts": [
            {
                "Type": "volume",
                "Name": "297686860b2428404be3b040e25f23233443c21ec540ee450d7720a026fe66a7",
                "Source": "/var/lib/docker/volumes/297686860b2428404be3b040e25f23233443c21ec540ee450d7720a026fe66a7/_data",
                "Destination": "/var/lib/postgresql/data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "ca11875b5f3eb121c7509975b279c09eb1f345eb77f7b759b183b36745861af4",
                "Source": "/var/lib/docker/volumes/ca11875b5f3eb121c7509975b279c09eb1f345eb77f7b759b183b36745861af4/_data",
                "Destination": "/vol1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "b0e5a1349f54b78b6240322d9dec52070098b764aad0fdcf4724b3133e244ad9",
                "Source": "/var/lib/docker/volumes/b0e5a1349f54b78b6240322d9dec52070098b764aad0fdcf4724b3133e244ad9/_data",
                "Destination": "/vol2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""

```
Кажется я перестарался и получилось в 3-мя ...

## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
*   Ответ 
```bash
# su - postgres
$ createuser --interactive --pwprompt
Enter name of role to add: test-admin-user
Enter password for new role:
Enter it again:
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) n
Shall the new role be allowed to create more new roles? (y/n) n
$  createdb  test_db
```
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
- создайте пользователя test-simple-user  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)
* Ответ 
```bash
test_db=# CREATE TABLE orders (цена INT, id SERIAL PRIMARY KEY, наименование VARCHAR(50) UNIQUE NOT NULL);
CREATE TABLE
```
Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)
* Ответ 
```bash
test_db=# CREATE TYPE address AS (addr VARCHAR(200), index INT);
CREATE TYPE
test_db=# CREATE TABLE clients (id SERIAL PRIMARY KEY, фамилия VARCHAR(50) NOT NULL, заказ INT REFERENCES orders (id),  contry address);
CREATE TABLE
test_db=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner
--------+---------+-------+----------
 public | clients | table | postgres
 public | orders  | table | postgres

```
*   Далее раздаем права согласно заданию 

```bash
test_db=# GRANT ALL ON clients, orders TO "test-admin-user";
GRANT

test_db=# create user "test-simple-user" with encrypted password '123';
CREATE ROLE

test_db=# GRANT SELECT, INSERT, UPDATE, DELETE ON clients, orders TO "test-simple-user";
GRANT
```
Приведите:
- итоговый список БД после выполнения пунктов выше,
- Ответ  
```bash

postgres=# \list
                                     List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |       Access privileges
-----------+----------+----------+------------+------------+--------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres                  +
           |          |          |            |            | postgres=CTc/postgres         +
           |          |          |            |            | "test-admin-user"=CTc/postgres
(4 rows)
```

- описание таблиц (describe)
- Ответ  
```bash
test_db=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner
--------+---------+-------+----------
 public | clients | table | postgres
 public | orders  | table | postgres
(2 rows)
```

- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- Ответ  
```bash
test_db=# SELECT table_name FROM information_schema.tables
WHERE table_schema NOT IN ('information_schema','pg_catalog');
 table_name
------------
 clients
 orders
```

- список пользователей с правами над таблицами test_db
- Ответ  
```bash
test_db=# \dp
                                              Access privileges
 Schema |          Name          |   Type   |        Access privileges         | Column privileges | Policies
--------+------------------------+----------+----------------------------------+-------------------+----------
 public | clients                | table    | postgres=arwdDxt/postgres       +|                   |
        |                        |          | "test-simple-user"=arwd/postgres |                   |
 public | clients_id_seq         | sequence |                                  |                   |
 public | clients_заказ_seq | sequence |                                  |                   |
 public | orders                 | table    | postgres=arwdDxt/postgres       +|                   |
        |                        |          | "test-simple-user"=arwd/postgres |                   |
 public | orders_id_seq          | sequence |                                  |                   |
(5 rows)

```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

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

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.
*   Ответ 
```bash
INSERT INTO orders (id, наименование, цена) VALUES     (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
test_db=# SELECT * FROM orders;
 цена | id | наименование
----------+----+--------------------------
       10 |  1 | Шоколад
     3000 |  2 | Принтер
      500 |  3 | Книга
     7000 |  4 | Монитор
     4000 |  5 | Гитара
# проверил на одной записи 
test_db=# INSERT INTO clients (id, фамилия, contry) VALUES (1, 'Иванов Иван Иванович', ('USA', 1));
INSERT 0 1
# Удачно , заносим остальные
test_db=# INSERT INTO clients (id, фамилия, contry) VALUES (1, 'Иванов Иван Иванович', ('USA', 1)), (2, 'Петров Петр Петрович', ('Canada', 1)), (3, 'Иоганн Себастьян Бах', ('Japan', 83)), (4, 'Ронни Джеймс Дио', ('Russia', 7)), (5, 'Ritchie Blackmore', ('Russia', 7));
INSERT 0 5
test_db=# select * from clients;
 id |             фамилия             | заказ |   contry
----+----------------------------------------+------------+------------
  1 | Иванов Иван Иванович |            | (USA,1)
  2 | Петров Петр Петрович |            | (Canada,1)
  3 | Иоганн Себастьян Бах |            | (Japan,83)
  4 | Ронни Джеймс Дио         |            | (Russia,7)
  5 | Ritchie Blackmore                      |            | (Russia,7)
(5 rows)
```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
 
Подсказк - используйте директиву `UPDATE`.

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
