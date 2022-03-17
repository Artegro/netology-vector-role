# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | никакое , будет ошибка типов, мы к int добавляем str  |
| Как получить для переменной `c` значение 12?  | c = str(a) + b  |
| Как получить для переменной `c` значение 3?  |  c = a + int(b)  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os
import sys
c_patch = "cd "
c_patch += sys.argv[1]
bash_command = [c_patch, "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
  if result.find('изменено') != -1:
    prepare_result = result.replace('\tизменено:   ', '')
    print(os.path.realpath(prepare_result))


```

### Вывод скрипта при запуске при тестировании:
```
root@test:/home/art/devops-netology/devops-netology# ./test1/1.py ./
/home/art/devops-netology/devops-netology/   test1/1.md
/home/art/devops-netology/devops-netology/   test1/1.py


```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт: 
*   скрипт тут остался тот же, я зачем то сразу наисал ввод пути по параметру, проверка является ли папка репозиторием скрипт проходит без изменений корректно , покрайней мере завершается нормально с нормальныи ответом
```python
#!/usr/bin/env python3

import os
import sys
c_patch = "cd "
c_patch += sys.argv[1]
bash_command = [c_patch, "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
  if result.find('изменено') != -1:
    prepare_result = result.replace('\tизменено:   ', '')
    print(os.path.realpath(prepare_result))


```

### Вывод скрипта при запуске при тестировании:  
*   тут я задаю заведомо каталог не являющийся репозиторием
```
root@test:/home/art/devops-netology/devops-netology# ./test1/1.py /home/art/devops-netology
fatal: не найден git репозиторий (или один из родительских каталогов): .git


```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
*  если нужен постоянны опрос то все это можно завернуть в цикс с условием выхода например при изменения ip или сделать вызов в кроне, что мне кажется более логичным , данные на вход берется из json файла и туда же записывается результат
* содержимое файла name.json 
* {"drive.google.com": "64.233.164.194", "mail.google.com": "108.177.14.17", "google.com": "74.125.131.113"}
* 
```python
#!/usr/bin/env python3

import socket
import json

with open('/mnt/c/Users/Art/PycharmProjects/devops-netology/test/name.json', 'r') as j:
    name_listold = json.load(j)
name_list = name_listold.copy()
for names in name_list:
    ips =  socket.gethostbyname(names)
    name_list[names] = ips
    if name_list.get(names) != name_listold.get(names):
        print("[error]: ", names, " IP mismatch: ", name_listold.get(names), " New ip: ", name_list.get(names),)
    else:
        print(names, "new ip: ", name_list.get(names))
with open('/mnt/c/Users/Art/PycharmProjects/devops-netology/test/name.json', 'w') as j:
   j.write(json.dumps(name_list))
```

### Вывод скрипта при запуске при тестировании:
*   соответственно вывод скрипта
```
art@PONYO:/mnt/c/Users/Art/PycharmProjects/devops-netology/test$ python3 2.py
[error]:  drive.google.com  IP mismatch:  64.233.164.194  New ip:  64.233.161.194
[error]:  mail.google.com  IP mismatch:  108.177.14.17  New ip:  64.233.162.17
[error]:  google.com  IP mismatch:  74.125.131.113  New ip:  173.194.221.113
art@PONYO:/mnt/c/Users/Art/PycharmProjects/devops-netology/test$ python3 2.py
drive.google.com new ip:  64.233.161.194
mail.google.com new ip:  64.233.162.17
google.com new ip:  173.194.221.113


```


