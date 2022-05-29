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

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
