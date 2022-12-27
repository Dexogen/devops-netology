# 05-virt-02-iaac

## Задача 1

- Опишите своими словами основные преимущества применения на практике IaaC паттернов.
    > - Подготока шаблонов ускорояет развертывание 
    > - Однородность конфигураций
    > - Ускорение и упрощение жизни разработчиков
- Какой из принципов IaaC является основополагающим?
    > Разработка инфраструктуры идентична разработке кода.

## Задача 2

- Чем Ansible выгодно отличается от других систем управление конфигурациями?
    > Низкий порог входа, требует знания достаточно популярного и простого языка Python. Не требует наличия агентов на хостах.
- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?
    > push надежнее, всегда можно быть уверенным, что конфигурация "доехала" (или нет) до конечного хоста.
    

## Задача 3

Установить на личный компьютер:

- VirtualBox
- Vagrant
- Ansible

*Приложить вывод команд установленных версий каждой из программ, оформленный в markdown.*

```bash
❯ vboxmanage --version
6.1.38_Ubuntur153438
❯ vagrant --version
Vagrant 2.3.2
❯ ansible --version
ansible [core 2.13.6]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.6 (main, Nov  2 2022, 18:53:38) [GCC 11.3.0]
  jinja version = 3.0.3
  libyaml = True
```


## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

- Создать виртуальную машину.
- Зайти внутрь ВМ, убедиться, что Docker установлен с помощью команды
```
docker ps
```
> Done