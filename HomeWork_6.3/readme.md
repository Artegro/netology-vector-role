# Домашнее задание к занятию "6.3. MySQL"

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с данным контейнером.
* Ответ 
```bash
# Создаем свой образ на основе официального 8-й версии
$ docker pull mysql:8-debian
$ nano Dockerfile
    # base image
    FROM  mysql:8-debian
    RUN mkdir /vol1
$ docker build -t mysql:v1 ./

$ docker volume create mysql-conf
mysql-conf

# Запускаем контейнер и монтируем в него волиум
$ docker run  --name mysql123 -e MYSQL_ROOT_PASSWORD=mysql -v ca11875b5f3eb121c7509975b279c09eb1f345eb77f7b759b183b36745861af4:/vol1 -v mysql-conf:/etc/mysql/ -d mysql:v1
f5ae22aaabdc81a336b185aaae4291c93dd5cc5724150adaa35655b154b452bc
vagrant@server1:~/dockmysql$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
f5ae22aaabdc   mysql:v1        "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql2

# Копируем дамп базы
cp ./test_dump.sql /var/lib/docker/volumes/ca11875b5f3eb121c7509975b279c09eb1f345eb77f7b759b183b36745861af4/_data/test_dump.sql

# заходи в контуйнер и смотрим дамп 
$ docker exec -it mysql123 bash
nano /vol1/test_dump.sql
#  Смотрим бамп , в нем есть таблица orders и наподнгение к ней.

# Команда \h и её вывод
:/# mysql -uroot -p
mysql> \h

For information about MySQL products and services, visit:
   http://www.mysql.com/
For developer information, including the MySQL Reference Manual, visit:
   http://dev.mysql.com/
To buy MySQL Enterprise support, training, or other products, visit:
   https://shop.mysql.com/

List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for `help'.
clear     (\c) Clear the current input statement.
connect   (\r) Reconnect to the server. Optional arguments are db and host.
delimiter (\d) Set statement delimiter.
edit      (\e) Edit command with $EDITOR.
ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.
pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
print     (\p) Print current command.
prompt    (\R) Change your mysql prompt.
quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.
source    (\.) Execute an SQL script file. Takes a file name as an argument.
status    (\s) Get status information from the server.
system    (\!) Execute a system shell command.
tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
use       (\u) Use another database. Takes database name as argument.
charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
warnings  (\W) Show warnings after every statement.
nowarning (\w) Don't show warnings after every statement.
resetconnection(\x) Clean session context.
query_attributes Sets string parameters (name1 value1 name2 value2 ...) for the next query to pick up.
ssl_session_data_print Serializes the current SSL session data to stdout or file
For server side help, type 'help contents'

# Восстанавливаем базу
mysql> create database test_db;
Query OK, 1 row affected (0.08 sec)
/# mysql -uroot -p mysql test_db < /vol1/test_dump.sql

# Смотрм что  внутри
mysql>  use test_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)

mysql> select count(*) from orders where price > 300;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)

```

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней 
- количество попыток авторизации - 3 
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.
* Ответ
```bash

mysql> CREATE USER 'test'@'localhost' IDENTIFIED BY 'test-pass';
Query OK, 0 rows affected (0.09 sec)

mysql> GRANT SELECT ON test_db.* TO 'test'@'localhost';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> ALTER USER 'test'@'localhost' IDENTIFIED BY 'test-pass' WITH MAX_QUERIES_PER_HOUR 100 PASSWORD EXPIRE INTERVAL 180 DAY FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME UNBOUNDED ATTRIBUTE '{"fam": "Pretty", "name": "James"}';
Query OK, 0 rows affected (0.13 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';
+------+-----------+------------------------------------+
| USER | HOST      | ATTRIBUTE                          |
+------+-----------+------------------------------------+
| test | localhost | {"fam": "Pretty", "name": "James"} |
+------+-----------+------------------------------------+
1 row in set (0.00 sec)
```
## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`
* Ответ
```bash
mysql> SET profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> SHOW PROFILES;
+----------+------------+-------------------+
| Query_ID | Duration   | Query             |
+----------+------------+-------------------+
|        1 | 0.00046300 | SET profiling = 1 |
+----------+------------+-------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SELECT TABLE_NAME,ENGINE,ROW_FORMAT,TABLE_ROWS,DATA_LENGTH,INDEX_LENGTH FROM information_schema.TABLES WHERE table_name = 'orders' and TABLE_SCHEMA = 'test_db' ORDER BY ENGINE asc;
+------------+--------+------------+------------+-------------+--------------+
| TABLE_NAME | ENGINE | ROW_FORMAT | TABLE_ROWS | DATA_LENGTH | INDEX_LENGTH |
+------------+--------+------------+------------+-------------+--------------+
| orders     | InnoDB | Dynamic    |          5 |       16384 |            0 |
+------------+--------+------------+------------+-------------+--------------+
1 row in set (0.01 sec)

mysql> alter table orders engine = MyISAM;
Query OK, 5 rows affected (0.07 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT TABLE_NAME,ENGINE,ROW_FORMAT,TABLE_ROWS,DATA_LENGTH,INDEX_LENGTH FROM information_schema.TABLES WHERE table_name = 'orders' and TABLE_SCHEMA = 'test_db' ORDER BY ENGINE asc;
+------------+--------+------------+------------+-------------+--------------+
| TABLE_NAME | ENGINE | ROW_FORMAT | TABLE_ROWS | DATA_LENGTH | INDEX_LENGTH |
+------------+--------+------------+------------+-------------+--------------+
| orders     | MyISAM | Dynamic    |          5 |       16384 |            0 |
+------------+--------+------------+------------+-------------+--------------+
1 row in set (0.00 sec)

mysql> alter table orders engine = InnoDB;
Query OK, 5 rows affected (0.17 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT TABLE_NAME,ENGINE,ROW_FORMAT,TABLE_ROWS,DATA_LENGTH,INDEX_LENGTH FROM information_schema.TABLES WHERE table_name = 'orders' and TABLE_SCHEMA = 'test_db' ORDER BY ENGINE asc;
+------------+--------+------------+------------+-------------+--------------+
| TABLE_NAME | ENGINE | ROW_FORMAT | TABLE_ROWS | DATA_LENGTH | INDEX_LENGTH |
+------------+--------+------------+------------+-------------+--------------+
| orders     | InnoDB | Dynamic    |          5 |       16384 |            0 |
+------------+--------+------------+------------+-------------+--------------+
1 row in set (0.00 sec)

mysql> SHOW PROFILES;
+----------+------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                                                                                                               |
+----------+------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|        1 | 0.00046300 | SET profiling = 1                                                                                                                                                                   |
|        2 | 0.01270350 | SELECT TABLE_NAME,ENGINE,ROW_FORMAT,TABLE_ROWS,DATA_LENGTH,INDEX_LENGTH FROM information_schema.TABLES WHERE table_name = 'orders' and TABLE_SCHEMA = 'test_db' ORDER BY ENGINE asc |
|        3 | 0.00078325 | alter table orders engen = MySQL                                                                                                                                                    |
|        4 | 0.00036550 | alter table orders engene = MySQL                                                                                                                                                   |
|        5 | 0.00014000 | alter table orders engine = MySQL                                                                                                                                                   |
|        6 | 0.07163850 | alter table orders engine = MyISAM                                                                                                                                                  |
|        7 | 0.00662325 | SELECT TABLE_NAME,ENGINE,ROW_FORMAT,TABLE_ROWS,DATA_LENGTH,INDEX_LENGTH FROM information_schema.TABLES WHERE table_name = 'orders' and TABLE_SCHEMA = 'test_db' ORDER BY ENGINE asc |
|        8 | 0.17093550 | alter table orders engine = InnoDB                                                                                                                                                  |
|        9 | 0.00647925 | SELECT TABLE_NAME,ENGINE,ROW_FORMAT,TABLE_ROWS,DATA_LENGTH,INDEX_LENGTH FROM information_schema.TABLES WHERE table_name = 'orders' and TABLE_SCHEMA = 'test_db' ORDER BY ENGINE asc |
+----------+------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
9 rows in set, 1 warning (0.00 sec)
```
## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

* Ответ :
  

innodb_log_buffer_size = 1M  
Размер буффера, в который помещаются транзакции в незакомиченном  
состоянии.      

innodb_log_file_size = 100M
Размер файла-лога операций.

innodb_flush_log_at_trx_commit = 0
Значение “0” даст наибольшую производительность. В этом случае буфер будет сбрасываться в лог файл независимо от транзакций. В этом случае риск потери данных возрастает.

innodb_buffer_pool_size = 1000M  треть от 3Гб ОЗУ
Размер буффера кеширования данных и индексов.  
```bash
root@2ce0ac0b466e:/# cat /proc/meminfo
MemTotal:        3087100 kB
```

innodb_file_per_table  = ON
При включении данной опции - таблицы хранятся по разным файлам.
Включение данного параметра требуется в случаях необходимости:
● освобождения места на диске при удалении таблиц (общий файл
может только увеличиваться)
● компрессии таблиц для экономии места на диске

```bash


/# cat /etc/mysql/my.cnf
...
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /vol1/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/

#custom
innodb_flush_log_at_trx_commit = 0
innodb_log_buffer_size = 1M
innodb_log_file_size = 100M
innodb_buffer_pool_size = 1000M
innodb_file_per_table  = ON
root@2ce0ac0b466e:/#

```
* проверяем что конфиг работает , для этого перезапустим контейнер
```bash
vagrant@server1:~$ docker restart mysql123
mysql123
vagrant@server1:~$ docker logs mysql123
...
2022-05-27T22:10:16.507409Z 0 [System] [MY-013172] [Server] Received SHUTDOWN from user <via user signal>. Shutting down mysqld (Version: 8.0.29).
2022-05-27T22:10:17.416168Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.29)  MySQL Community Server - GPL.
2022-05-27 22:10:17+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.29-1debian10 started.
2022-05-27 22:10:18+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-05-27 22:10:18+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.29-1debian10 started.
2022-05-27T22:10:18.814849Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.29) starting as process 1
2022-05-27T22:10:18.821775Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
vagrant@server1:~$ docker exec -it mysql123 bash
root@2ce0ac0b466e:/# mysql -p mysql
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.29 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> \q
```

---
