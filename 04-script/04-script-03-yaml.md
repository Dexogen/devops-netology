## 04-script-03-yaml
### Задание 1

### Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:

```json
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

#### Ваш скрипт:
```json
{
    "info": "Sample JSON output from our service",
    "elements": [{
            "name": "first",
            "type": "server",
            "ip": "7.1.7.5"
        }, {
            "name": "second",
            "type": "proxy",
            "ip": "71.78.22.43"
        }
    ]
}
```

### Задание 2

В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

#### Ваш скрипт:
```python
#!/usr/bin/python3

import socket
import time
import json
import yaml

services = {
                "drive.google.com": "0.0.0.0",
                "mail.google.com": "0.0.0.0",
                "google.com": "0.0.0.0"
            }

while True:
    for service in services.keys():
        ip_last = services[service]
        check_ip = socket.gethostbyname(service)
        if check_ip != ip_last:
            print(f"""[WARN] IP was changed for {service}: {ip_last} => {check_ip}""")
            services[service] = check_ip
        else:
            print(f"""[INFO] {ip_last}\t{service}""")
    with open('result.yml', 'w') as yaml_file:
        yaml.dump(services, yaml_file)
    with open('result.json', 'w') as json_file:
        json.dump(services, json_file)
    time.sleep(3)
```

#### Вывод скрипта при запуске при тестировании:
```
[WARN] IP was changed for drive.google.com: 0.0.0.0 => 74.125.205.194
[WARN] IP was changed for mail.google.com: 0.0.0.0 => 108.177.14.18
[WARN] IP was changed for google.com: 0.0.0.0 => 108.177.14.100
[INFO] 74.125.205.194   drive.google.com
[INFO] 108.177.14.18    mail.google.com
[INFO] 108.177.14.100   google.com
```

#### json-файл(ы), который(е) записал ваш скрипт:
```json
{
  "drive.google.com": "142.250.150.194",
  "mail.google.com": "74.125.205.17",
  "google.com": "64.233.162.138"
}
```

#### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
drive.google.com: 142.250.150.194
google.com: 64.233.162.138
mail.google.com: 74.125.205.17
```

### Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так как команды в нашей компании никак не могут прийти к единому мнению о том, какой формат разметки данных использовать: JSON или YAML, нам нужно реализовать парсер из одного формата в другой. Он должен уметь:
   * Принимать на вход имя файла
   * Проверять формат исходного файла. Если файл не json или yml - скрипт должен остановить свою работу
   * Распознавать какой формат данных в файле. Считается, что файлы *.json и *.yml могут быть перепутаны
   * Перекодировать данные из исходного формата во второй доступный (из JSON в YAML, из YAML в JSON)
   * При обнаружении ошибки в исходном файле - указать в стандартном выводе строку с ошибкой синтаксиса и её номер
   * Полученный файл должен иметь имя исходного файла, разница в наименовании обеспечивается разницей расширения файлов

#### Ваш скрипт:
```python
???
```

#### Пример работы скрипта:
???
