
# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"

## Задача 1

Сценарий выполения задачи:

- создайте свой репозиторий на https://hub.docker.com;
- выберете любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.
### Ответ1 
Ссылка не репозитарий контенера https://hub.docker.com/r/agrod80/nqinxnet  
За основу был взят образ из контейнера ubuntu/nginx 
создан файл с нужной нам начинкой index.nginx-debian.html и перекопирован в контейнер 
Dockerfile 
```bash
# base image
FROM ubuntu/nginx
# Copy file to start list nqinx
COPY index.nginx-debian.html /var/www/html/index.nginx-debian.html
RUN /etc/init.d/nginx restart
```

Запускаем контейнер
```bash
$ docker run -d -p 80:80 nqinxtest:1.1
e22402fbbbdbfd8d90beb7b0bbf4b8e2be1e53222be49e71de75db080a4f77da
```
Проверяем что запустился и работает , так же смотрим его id
```bash
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                               NAMES
e22402fbbbdb   nqinxtest:1.1   "/docker-entrypoint.…"   7 seconds ago   Up 7 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   wonderful_hypatia
```
Проверяем что у нас веб сервер заработал и поменялся индекс файл на нужный нам
```bash
lynx localhost:80
   Hey, Netology
  I’m DevOps Engineer!
```
Все хорошо, создаем коммит и пушим в репозитарий
```bash
$ docker commit e22402fbbbdb agrod80/nqinxnet:1.1
sha256:18ba70320626e0dc69f5bed084ce0be2ef76477b78db8e9b1a5634763f7537c2
$  docker push agrod80/nqinxnet:1.1
c47dba99e931: Pushed
a013b0421977: Pushed
59ad5c28c184: Pushed
56b370a4aba8: Mounted from ubuntu/nginx
8365af196586: Mounted from ubuntu/nginx
33000485f1ac: Mounted from ubuntu/nginx
06babc60ed94: Mounted from ubuntu/nginx
1.1: digest: sha256:dae0054baeb40b278c16377bb7abc8d956649002221fdcdb9fdfdcbbaf710c70 size: 1776

```

## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

- Высоконагруженное монолитное java веб-приложение; 
```
Docker контейнеры , скорее всего в виртуальной среде на гипервизоре или кластере. 
+ овязываемя от железа . Простота и скорость разворачивания, миграции и балансировка нагрузки.
```
- Nodejs веб-приложение;
```
Docker контейнеры , скорее всего в виртуальной среде на гипервизоре или кластере. 
+ овязываемя от железа.  Простота и скорость разворачивания, миграции и балансировка нагрузки.
+ есть официальный образ в репозитарий на https://hub.docker.com/_/node
```
- Мобильное приложение c версиями для Android и iOS;
```
В этом кейса скрее всего подходит виртуализация (сам гипервизор не принципиален , oracl, vmware, kvm, hyper-v), так как нам надо разные OS,
```
- Шина данных на базе Apache Kafka;
```
Скорее всего тут мы будет делать на Физически машинах  да бы снизить издержки на виртуализацию. 
```
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
```
Ноды  elasticsearch , разместить на виртуальных или физических машинах, думаю тут потери виртуализации ресурсов не так критичны
Ноды logstash и kibana  в docker контейнерах. так же на виртуализаци, 
```
- Мониторинг-стек на базе Prometheus и Grafana;
```
Используем Docker в виртуальной среде, для мониторинга не так важна скорость операций ввода вывода на носители , 
Так же для  Prometheus и Grafana имеем официальные образы для docker контейнеров . что упростить старт и разворачиване, 
```
- MongoDB, как основное хранилище данных для java-приложения;
```
Под MongoDB использовать. в порядке предпочтения: физический, виртуальный.
```
- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.
```
Виртыулизация . 
Задумался о docker контейнере для этого с подмонтированной шарой , но для него опять таки откуда то надо будет брать конфиг, а он и есть Registry .. виртуализация лучше.
```

## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.
### Ответ
-  Скачеваем образы
```bash
$ docker pull debian
$ docker pull centos
```
- создаем папку 
```bash
$ mkdir data
```
-запускаем контейнеры
```bash
$ docker run --name=cent -d -v /home/vagrant/dock2/data:/data -t centos:latest
3ac81621c7a668559f1bb267f9913701bb803205fad45a375f3dc8fa1c2f96f9
$ docker run --name=deb -d -v /home/vagrant/dock2/data:/data -t debian:latest
62b371ef24ff0106ffa13eb8fcfc4605350d8dcee44db09e5b5b254793ba82a6
a$ docker ps
CONTAINER ID   IMAGE           COMMAND       CREATED          STATUS          PORTS     NAMES
62b371ef24ff   debian:latest   "bash"        2 seconds ago    Up 1 second               deb
3ac81621c7a6   centos:latest   "/bin/bash"   26 seconds ago   Up 25 seconds             cent
```
- Далее по порядкуЖ
- Создаем файл с текстом в контейнере с центос  
- Создаем файл на хосте и проверяем на нем листинг
- Проверяем листинг и содержимое файла на контейнере дебиан 
```bash
$ docker exec -it cent "/bin/bash"
[root@3ac81621c7a6 /]# echo "Test centos" > /data/1.txt
[root@3ac81621c7a6 /]# ls /data
1.txt
[root@3ac81621c7a6 /]# exit
exit

vagrant@server1:~/dock2/data$ ls
1.txt
vagrant@server1:~/dock2/data$ touch 2.txt
vagrant@server1:~/dock2/data$ ls
1.txt  2.txt
vagrant@server1:~/dock2/data$ docker exec -it deb "bash"

root@62b371ef24ff:/# ls /data
1.txt  2.txt
root@62b371ef24ff:/# cat /data/1.txt
Test centos
root@62b371ef24ff:/# exit
exit
```



