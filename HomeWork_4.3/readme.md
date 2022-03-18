# Домашнее задание к занятию "4.3. Языки разметки JSON и YAML"


## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t", 
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }   
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43 
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис
```
    { "info" : "Sample JSON output from our service\t", # так же тут если нам надо экраировать \t то надо добавить \ , поулчаем \\t , но это возможно и не ошибка
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175   # тут можно было бы добавить кавычки . но п оскольку это число int оно будет воспринято как число а не строка , хотя логически тут должен быть ip адрес и записан в формате "7.1.7.5" тут логическая ошика а не ошибка синтаксиса , может нам и надо передать туда число а не адрес... воля создателя неизвесна .
            }   # тут забыли запятую, долно быть так },
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43  # тут забыли закрывающие кавычки ", должно быть тык "ip" : "71.78.22.43"
            }
        ]
    }
    исправленный выглядет так:
        { "info" : "Sample JSON output from our service\t", 
        "elements" :[
            { "name" : "first", 
            "type" : "server", 
            "ip" : 7175
            },
            { "name" : "second", 
            "type" : "proxy", 
            "ip" : "71.78.22.43"
            }
        ]
    }
```

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import socket
import json
import yaml

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
with open('/mnt/c/Users/Art/PycharmProjects/devops-netology/test/name.yaml', 'w') as yam:
    yam.write(yaml.dump(name_list))

```

### Вывод скрипта при запуске при тестировании:
```
# Первое тестирование
drive.google
art@PONYO:/mnt/c/Users/Art/PycharmProjects/devops-netology/test$ python3 2.py
[error]:  drive.google.com  IP mismatch:  64.233.161.194  New ip:  74.125.205.194
[error]:  mail.google.com  IP mismatch:  64.233.162.17  New ip:  173.194.221.83
[error]:  google.com  IP mismatch:  173.194.221.113  New ip:  74.125.131.113
============  
# второе тестирование
art@PONYO:/mnt/c/Users/Art/PycharmProjects/devops-netology/test$ python3 2.py
drive.google.com new ip:  74.125.205.194
[error]:  mail.google.com  IP mismatch:  173.194.221.83  New ip:  173.194.73.83
google.com new ip:  74.125.131.113

```

### json-файл(ы), который(е) записал ваш скрипт:
# Первое тестирование
```json
{"drive.google.com": "74.125.205.194", "mail.google.com": "173.194.221.83", "google.com": "74.125.131.113"}  
```
=====================
# второе тестирование
```json
{"drive.google.com": "74.125.205.194", "mail.google.com": "173.194.73.83", "google.com": "74.125.131.113"}
```

### yml-файл(ы), который(е) записал ваш скрипт:
# Первое тестирование
```yaml 
drive.google.com: 74.125.205.194
google.com: 74.125.131.113
mail.google.com: 173.194.221.83
```
==============
# второе тестирование
```yaml
drive.google.com: 74.125.205.194
google.com: 74.125.131.113
mail.google.com: 173.194.73.83
```
