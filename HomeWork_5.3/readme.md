
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
- Nodejs веб-приложение;
- Мобильное приложение c версиями для Android и iOS;
- Шина данных на базе Apache Kafka;
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
- Мониторинг-стек на базе Prometheus и Grafana;
- MongoDB, как основное хранилище данных для java-приложения;
- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.

## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.


---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
