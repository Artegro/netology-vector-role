# Домашнее задание к занятию "2. Работа с Playbook"

## Подготовка к выполнению

1. (Необязательно) Изучите, что такое [clickhouse](https://www.youtube.com/watch?v=fjTNS2zkeBs) и [vector](https://www.youtube.com/watch?v=CgEhyffisLY)

    Выполнено

2. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.

    [Готово](https://github.com/Artegro/netology/tree/main/HomeWork_8.2)

3. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

    Выполнено
    
4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.
Запустим контейнер с
```
docker run --name clickhouse-01 -d -p 8123:8123 pycontribs/centos:7 sleep 36000000  && docker run --name vector-01 -d pycontribs/centos:7 sleep 36000000
```
Проверим что все стартовало
```
docker ps
CONTAINER ID   IMAGE                 COMMAND            CREATED          STATUS          PORTS     NAMES
ea52b2822393   pycontribs/centos:7   "sleep 36000000"   8 seconds ago    Up 6 seconds              vector-01
47749b99ec28   pycontribs/centos:7   "sleep 36000000"   10 minutes ago   Up 10 minutes             clickhouse-01
```

## Основная часть

1. Приготовьте свой собственный inventory файл `prod.yml`.
Подправим prod.yml согласно тому , что будем работат ьс контейнером
```
cat /home/artegro/GitHub/HomeWork_8.2/playbook/inventory/prod_new.yml
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_connection: docker

vector:
  hosts:
    vektor-01:
      ansible_connection: docker
```
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev).
Готово
Добавляем

3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, установить vector.
5. Запустите и исправьте ошибки, если они есть.

Для начала установим ansible-lint
```
apt install ansible-lint
```
Запускаем
```
root@test:/home/artegro/GitHub/HomeWork_8.2/playbook# ansible-lint site.yml
root@test:/home/artegro/GitHub/HomeWork_8.2/playbook#
```
Прохогдит теб выводов

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.

Запускаем и получаем следующее, ошибку так как действий не происходит и провертить установку не получиться.
```
 ansible-playbook -i inventory/prod_new.yml site.yml --check
...
...
clickhouse-01              : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=1    ignored=0   
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0 
```
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

Запускаем
```
ansible-playbook -i inventory/prod_new.yml site.yml --diff
...
...
clickhouse-01              : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
Видим что все хорошо.

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

Повторяем запуск
```
ansible-playbook -i inventory/prod_new.yml site.yml --diff
...
...
clickhouse-01              : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
vector-01                  : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
Результат получает иденпотентный

9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.


10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.