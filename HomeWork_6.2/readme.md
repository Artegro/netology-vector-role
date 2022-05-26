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

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db
*   Ответ
```bash
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
