## Оглавление

* [03-sysadmin-01-terminal](#03-sysadmin-01-terminal)
* [03-sysadmin-02-terminal](#03-sysadmin-02-terminal)
* [03-sysadmin-03-os](#03-sysadmin-03-os)
* [03-sysadmin-04-os](#03-sysadmin-04-os)
* [03-sysadmin-05-fs](#03-sysadmin-05-fs)
* [03-sysadmin-06-net](#03-sysadmin-06-net)
* [03-sysadmin-07-net](#03-sysadmin-07-net)
* [03-sysadmin-08-net](#03-sysadmin-08-net)
* [03-sysadmin-09-security](#03-sysadmin-09-security)
---

## 03-sysadmin-01-terminal
1-7. У меня дома сервер с Proxmox на борту, так что ДЗ выполняю в виртуалке на нём.

8. Ознакомиться с разделами `man bash`, почитать о настройках самого bash:
* какой переменной можно задать длину журнала `history`, и на какой строчке manual это описывается?

  ```bash
  HISTSIZE (строка 862 в мануле)
  ```
* что делает директива `ignoreboth` в bash?

  ```bash
  Не сохранять команды начинающиеся с пробела и одинаковые команды, выполненные подряд (ignorespace + ignoredup)
  ```
  
9. В каких сценариях использования применимы скобки `{}` и на какой строчке `man bash` это описано?

  ```bash
  Это Brace Expansion (строка 1091 в мануле). Может использоваться для получения списка аргументов.
  echo {a..z} # алфавит
  echo {1..9} # последовательность
  echo {1..20..2} # инкремент в последовательности
  echo {a,b,c,d} # просто список
  ```

10. С учётом ответа на предыдущий вопрос, как создать однократным вызовом `touch` 100000 файлов? Получится ли аналогичным образом создать 300000? Если нет, то почему?

  ```bash
  touch {1..100000}.tmp
  ```
  
11.1 В man bash поищите по `/\[\[`. Что делает конструкция `[[ -d /tmp ]]`
  ```bash
  Двойные фигурные скобки - это расширение CONDITIONAL EXPRESSIONS. Данная конструкция проверяет существование папки /tmp.
  ```

11.2 Что же вернется в нашем случае, 0 или 1? Для проверки можно использовать команду echo $?
  ```bash
  В нашем случае вернется код 0, свидетельствующий о том, что папка существует.
  [[ -d /tmp ]]; echo $?
  0
  [[ -d /some_not_existing_dir ]]; echo $?
  1
  ```

12. Основываясь на знаниях о просмотре текущих (например, PATH) и установке новых переменных; командах, которые мы рассматривали, добейтесь в выводе type -a bash в виртуальной машине наличия первым пунктом в списке:
  ```bash
  bash is /tmp/new_path_directory/bash
  bash is /usr/local/bin/bash
  bash is /bin/bash
  ```
(прочие строки могут отличаться содержимым и порядком)

В качестве ответа приведите команды, которые позволили вам добиться указанного вывода или соответствующие скриншоты.

  ```bash
  mkdir -p /tmp/new_path_directory && touch /tmp/new_path_directory/bash
  PATH="/tmp/new_path_directory:$PATH"
  type -a bash
  ```

13. Чем отличается планирование команд с помощью `batch` и `at`?

  ```bash
  Обе являются частью пакета at.
  at - запуск команд в определённое время
  batch - выполнение команд когда загрузка системы ниже указанного значения (по дефолту 1.5)
  ```

14. Завершите работу виртуальной машины чтобы не расходовать ресурсы компьютера и/или батарею ноутбука.

## 03-sysadmin-02-terminal

1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.
  ```
  Это встроенная команда оболочки. Не будучи встроенной ей бы пришлось менять аттрибут родительского процесса.
  ```
2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.
  ```
  grep <some_string> <some_file> -c
  ```
3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
  ```
  # Прародитель всех юзерспейсов
  /sbin/init
  ```
4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?
  ```
  # посмотреть номер терминала, в который хотим получить вывод
  tty
  /dev/pts/1
  # выполнить из другой сессии
  ls > /dev/pts/1
  ```
5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
  ```
  cat package_list.txt
  apt
  apt-utils
  apt-transport-https
  
  apt show $(<package_list.txt) > out.txt
  grep "utils" < out.txt > one_more_time.txt
  ```
6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?
  ```
  Да, никаких сложностей не должно возникнуть. Выполняю через SSH и вижу вывод через iLO консоль сервера
  echo "Some text" > /dev/tty0
  ```
7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?
  ```
  Потому что мы создали файловый дескриптор, который перенаправили в STDOUT. Теперь то, что мы будем подавать в него будет вести в STDOUT.
  ```
8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.
  ```
  cat package_list.txt 3>&2 2>&1 1>&3 | grep utils -c
  # 3>&2 - новый дескриптор в stderr 
  # 2>&1 - stderr в stdout 
  # 1>&3 - stdout в новый дескриптор
  ```
9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?
  ```
  Это переменные окружения курильщика. Получить переменные окружения здорового человека можно через env или printenv.
  ```
10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.
  ```
  /proc/<PID>/cmdline - команда со всеми ее аргументами в момент запуска
  /proc/<PID>/exe - симлинк на файл процесса
  ```
11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.
  ```
  sse4_2
  ```
12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

    ```bash
	vagrant@netology1:~$ ssh localhost 'tty'
	not a tty
    ```

	Почитайте, почему так происходит, и как изменить поведение.
  ```
  При выполнении команды через ssh не выделяется псевдотерминал. В мануле ssh есть специальный ключ
       -t      Force pseudo-terminal allocation.  This can be used to execute arbitrary screen-based programs on a re‐
             mote machine, which can be very useful, e.g. when implementing menu services.  Multiple -t options force
             tty allocation, even if ssh has no local tty.
  ```
13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.
  ```
  ❯ ping 1>&2 ya.ru
  PING ya.ru(ya.ru (2a02:6b8::2:242)) 56 data bytes
  64 bytes from ya.ru (2a02:6b8::2:242): icmp_seq=1 ttl=56 time=3.34 ms
  64 bytes from ya.ru (2a02:6b8::2:242): icmp_seq=2 ttl=56 time=3.45 ms
  ^Z
  [1]  + 893191 suspended  ping ya.ru >&2
  ❯ disown ping
  disown: warning: job is suspended, use `kill -CONT -893191' to resume
  > screen -d -m reptyr 893191
  ```
14. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.
  ```
  Из манула: tee - read from standard input and write to standard output and files.
  Соответственно при такой конструкции tee выполняется под sudo, а значит сможет писать в файл.
  ```

## 03-sysadmin-03-os
1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`. Обратите внимание, что `strace` выдаёт результат своей работы в поток stderr, а не в stdout.
	```bash
	chdir("/tmp")         = 0
	```
2. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.
	```bash
	[pid 697590] openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
	```
3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
	```bash
	❯ ping 1.1.1.1 > /tmp/some_file &
	❯ rm /tmp/some_file
	❯ pgrep ping
	1212906
	❯ lsof -p 1212906 | grep some_file
	ping    1212906 root    1w   REG    8,1     5807     1191 /tmp/some_file (deleted)
	❯ : > /proc/1212906/fd/1
	```
4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
	```
	Нет, это уже завершенный процесс.
	```
5. В iovisor BCC есть утилита `opensnoop`:
    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
	```bash
	/var/run/utmp
	/usr/local/share/dbus-1/system-services
	/usr/share/dbus-1/system-services
	/lib/dbus-1/system-services
	```
6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
	```bash
	[pid 755144] uname({sysname="Linux", nodename="pve", ...}) = 0
	❯ man 2 uname
	...
	Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
	...
	```
7. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?
    ```
    && - работает только при успешном завершении первой команды.
    ; - выполняет команды последовательно в любом случае.
    set -e - в данном случае выполнение уже и так прекратится при ненулевом коде завершения первой команды, так что это не имеет смысла.
    ```
8. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
	```
	e - прерывание выполнения при ошибке любой команды кроме последней
	u - считать за ошибку пустые переменные
	x - печать команд и их аргументов
	o pipefail - вернуть в результате выполнения статус последней команды для завершения с ненулевым статусом или ноль, если ни одна команда не завершилась с ненулевым статусом
	```
9. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
	```
	❯ ps -eo stat | sort | uniq -c
     	29 I
     	68 I<
      	1 R+
      	1 Rs
	223 S
      	3 S+
     	70 S<
     	81 Sl
      	2 Sl+
      	1 S<Ls
      	1 S<Lsl
     	56 SN
      	1 SNl
      	1 S<s
     	74 Ss
      	1 Ss+
     	23 Ssl
      	2 Ssl+
	# Больше всего процессов в состоянии прерываемого сна.
	```

## 03-sysadmin-04-os

1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.
  
  	
	```bash
	# Скачал последний релиз-кандидат 1.4.0-rc.0, паспаковал и переименовал бинарник в название релиза, сделал симлинк (для удобства смены версий)
	❯ ln -s /opt/node-exporter/node_exporter-1.4.0-rc.0.linux-amd64 /usr/sbin/node_exporter
	# Создаем unit
	❯ cat /lib/systemd/system/node_exporter.service
	[Unit]
	Description=Prometheus Node Exporter

	[Service]
	User=node_exporter
	EnvironmentFile=/etc/sysconfig/node_exporter
	ExecStart=/usr/sbin/node_exporter $OPTIONS

	[Install]
	WantedBy=multi-user.target
	❯ cat /etc/sysconfig/node_exporter
	OPTIONS = --no-collector.wifi --no-collector.hwmon
	❯ systemctl enable node_exporter.service && systemctl start node_exporter.service
	❯ curl -s localhost:9102/metrics 
	# Получаем огромную портянку метрик

2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.
    ```bash
    # Для процессора (среднее время выполнения процессов в режие ядра), можно еще смотреть за iowait, чтобы обнаружить проблемы с сетью/диском и за user для пользовательских процессов
    node_cpu_seconds_total{mode="system"}
    # Для памяти (доступный объем)
    node_memory_MemAvailable_bytes
    # Для диска (доступное место)
    node_filesystem_avail_bytes
    # Для сети (пакетов получено), можно следить и за отправленными если это имеет смысл для конкретной системы
    node_network_receive_bytes_total
    ```
3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.
    ```bash
    # Неплохая тулза, спасибо.
    ```
4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
    ```bash
    ❯ sudo dmesg | grep "Hypervisor detected"
    QEMU
    # Это не всегда показательно, т.к. можно сконфигурировать виртуальную машину указав реальные smbios, выставив host-passthrough для процессора, указав "hidden state" и т.д.
    ```
5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
	```bash
	# Максимальное число дескрипторов для процесса, диапазон 8192–20000500
	❯ sysctl fs.nr_open
	fs.nr_open = 1048576
	# Перед этим значением есть еще два порога - ulimit (лимиты мягкие/жесткие для конкретного шелла) и перманентное ограничение в /etc/security/limits.conf
	```
6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
	```bash
	❯ unshare -f -p --mount-proc sleep 1h
	^Z
	[1]  + 3269918 suspended  unshare -f -p --mount-proc sleep 1h
	❯ ps | grep sleep
	3269919 pts/0    00:00:00 sleep
	❯ nsenter --target 3269919 -m -p 
	❯ pgrep sleep
	1
	```
7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?
	```bash
	# В читабельном виде это плодилка форков, уходящая в ограничение пространства имен.
	some(){
	  some | some &
	}
	some
	# После достижения лимита срабатывает Process Number Controller. Текущий лимит
	❯ ulimit -H -u
	3842
	# Ограничение можно поднять через ulimit, но опять же для конкретного шелла. Преманентные значения всё так же в /etc/security/limits.conf

## 03-sysadmin-05-fs
1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.
	```bash
	Done
	```
2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
	```bash
	Нет, не могут, т.к. указывают на один и тот же inode
3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
	```bash
	# Готово, только я использую для заданий домашний сервер с гипервизором.
	❯ lsblk
	NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
	...
	vdb     252:16   0  2.6G  0 disk
	vdc     252:32   0  2.6G  0 disk
	```
4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
	```bash
	❯ fdisk /dev/vdb
	...
	Command (m for help): g
	Created a new GPT disklabel (GUID: 547A4A63-BC13-024B-91E5-F014AD3E7381).
	
	Command (m for help): n
	Partition number (1-128, default 1):
	First sector (2048-5369822, default 2048):
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5369822, default 5369822): +2G

	Created a new partition 1 of type 'Linux filesystem' and of size 2 GiB.

	Command (m for help): n
	Partition number (2-128, default 2):
	First sector (4196352-5369822, default 4196352):
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5369822, default 5369822):

	Created a new partition 2 of type 'Linux filesystem' and of size 573 MiB.

	Command (m for help): w
	The partition table has been altered.
	Calling ioctl() to re-read partition table.
	Syncing disks.
	```
5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
	```bash
	❯ sfdisk -d /dev/vdb | sfdisk /dev/vdc
	```
6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.
	```bash
	❯ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/vd[bc]1
	```
7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
	```bash
	❯ sudo mdadm --create /dev/md1 --level=0 --raid-devices=2 /dev/vd[bc]2
	```
8. Создайте 2 независимых PV на получившихся md-устройствах.
	```bash
	❯ pvcreate /dev/md[01]
	❯ pvdisplay /dev/md[01]
	  "/dev/md0" is a new physical volume of "<2.00 GiB"
	  --- NEW Physical volume ---
	  PV Name               /dev/md0
	  VG Name
	  PV Size               <2.00 GiB
	  Allocatable           NO
	  PE Size               0
	  Total PE              0
	  Free PE               0
	  Allocated PE          0
	  PV UUID               XOvzwx-0PxV-096Y-tlBd-fGoT-2deS-jS4e26

	  "/dev/md1" is a new physical volume of "1.11 GiB"
	  --- NEW Physical volume ---
	  PV Name               /dev/md1
	  VG Name
	  PV Size               1.11 GiB
	  Allocatable           NO
	  PE Size               0
	  Total PE              0
	  Free PE               0
	  Allocated PE          0
	  PV UUID               nQi5mR-Zrh9-v4eK-EdkF-VTdz-yYsv-HB6otG
	```
9. Создайте общую volume-group на этих двух PV.
	```bash
	❯ vgcreate VG0 /dev/md[01]
	❯ vgdisplay VG0
	  --- Volume group ---
	  VG Name               VG0
	  System ID
	  Format                lvm2
	  Metadata Areas        2
	  Metadata Sequence No  1
	  VG Access             read/write
	  VG Status             resizable
	  MAX LV                0
	  Cur LV                0
	  Open LV               0
	  Max PV                0
	  Cur PV                2
	  Act PV                2
	  VG Size               <3.11 GiB
	  PE Size               4.00 MiB
	  Total PE              796
	  Alloc PE / Size       0 / 0
	  Free  PE / Size       796 / <3.11 GiB
	  VG UUID               dpaQpw-HSDN-Z3fT-aoPU-r1kI-OEpq-pq9wcs
	```
10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
	```bash
	❯ lvcreate --name LV0 --type linear --size 100M VG0 /dev/md1
	```
11. Создайте `mkfs.ext4` ФС на получившемся LV.
	```bash
	❯ mkfs.ext4 /dev/VG0/LV0
	```
12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
	```bash
	❯ mkdir /tmp/new && mount /dev/VG0/LV0 /tmp/new
	```
13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
	```bash
	❯ wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
	```
14. Прикрепите вывод `lsblk`.
	```bash
	❯ lsblk
	NAME          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
	loop0           7:0    0   48M  1 loop  /snap/snapd/16778
	loop1           7:1    0  103M  1 loop  /snap/lxd/23541
	loop2           7:2    0 63.2M  1 loop  /snap/core20/1623
	sr0            11:0    1    4M  0 rom
	vda           252:0    0    4G  0 disk
	├─vda1        252:1    0  3.9G  0 part  /
	├─vda14       252:14   0    4M  0 part
	└─vda15       252:15   0  106M  0 part  /boot/efi
	vdb           252:16   0  2.6G  0 disk
	├─vdb1        252:17   0    2G  0 part
	│ └─md0         9:0    0    2G  0 raid1
	└─vdb2        252:18   0  573M  0 part
	  └─md1         9:1    0  1.1G  0 raid0
		└─VG0-LV0 253:0    0  100M  0 lvm   /tmp/new
	vdc           252:32   0  2.6G  0 disk
	├─vdc1        252:33   0    2G  0 part
	│ └─md0         9:0    0    2G  0 raid1
	└─vdc2        252:34   0  573M  0 part
	  └─md1         9:1    0  1.1G  0 raid0
		└─VG0-LV0 253:0    0  100M  0 lvm   /tmp/new
	```
15. Протестируйте целостность файла:

    ```bash
    ❯ gzip -t /tmp/new/test.gz
    ❯ echo $?
    0
    ```
16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
	```bash
	❯ pvmove /dev/md1 /dev/md0
	  /dev/md1: Moved: 44.00%
  	  /dev/md1: Moved: 100.00%
	```
17. Сделайте `--fail` на устройство в вашем RAID1 md.
	```bash
	❯ mdadm /dev/md0 --fail /dev/vdb1
	mdadm: set /dev/vdb1 faulty in /dev/md0
	```
18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
	```bash
	❯ dmesg -e | tail
	[Sep24 21:08] md/raid1:md0: not clean -- starting background reconstruction
	[  +0.000003] md/raid1:md0: active with 2 out of 2 mirrors
	[  +0.000010] md0: detected capacity change from 0 to 4188160
	[  +0.002409] md: resync of RAID array md0
	[ +10.228247] md: md0: resync done.
	[ +10.423303] md1: detected capacity change from 0 to 2336768
	[Sep24 21:13] EXT4-fs (dm-0): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
	[Sep24 21:15] dm-1: detected capacity change from 204800 to 8192
	[Sep24 21:16] md/raid1:md0: Disk failure on vdb1, disabling device.
				  md/raid1:md0: Operation continuing on 1 devices.
	```
19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    ❯ gzip -t /tmp/new/test.gz
   	❯ echo $?
    0
    ```
20. Погасите тестовый хост, `vagrant destroy`.
	```bash
	Done
	```
--
## 03-sysadmin-06-net
1. Работа c HTTP через телнет.
- Подключитесь утилитой телнет к сайту stackoverflow.com
`telnet stackoverflow.com 80`
- отправьте HTTP запрос
```bash
GET /questions HTTP/1.0
HOST: stackoverflow.com
[press enter]
[press enter]
```
- В ответе укажите полученный HTTP код, что он означает?
2. Повторите задание 1 в браузере, используя консоль разработчика F12.
- откройте вкладку `Network`
- отправьте запрос http://stackoverflow.com
- найдите первый ответ HTTP сервера, откройте вкладку `Headers`
- укажите в ответе полученный HTTP код.
- проверьте время загрузки страницы, какой запрос обрабатывался дольше всего?
- приложите скриншот консоли браузера в ответ.
3. Какой IP адрес у вас в интернете?
4. Какому провайдеру принадлежит ваш IP адрес? Какой автономной системе AS? Воспользуйтесь утилитой `whois`
5. Через какие сети проходит пакет, отправленный с вашего компьютера на адрес 8.8.8.8? Через какие AS? Воспользуйтесь утилитой `traceroute`
6. Повторите задание 5 в утилите `mtr`. На каком участке наибольшая задержка - delay?
7. Какие DNS сервера отвечают за доменное имя dns.google? Какие A записи? воспользуйтесь утилитой `dig`
8. Проверьте PTR записи для IP адресов из задания 7. Какое доменное имя привязано к IP? воспользуйтесь утилитой `dig`

В качестве ответов на вопросы можно приложите лог выполнения команд в консоли или скриншот полученных результатов.
---
## 03-sysadmin-07-net
1. Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?
2. Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?
3. Какая технология используется для разделения L2 коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.
4. Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.
5. Сколько IP адресов в сети с маской /29 ? Сколько /29 подсетей можно получить из сети с маской /24. Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.
6. Задача: вас попросили организовать стык между 2-мя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP адреса? Маску выберите из расчета максимум 40-50 хостов внутри подсети.
7. Как проверить ARP таблицу в Linux, Windows? Как очистить ARP кеш полностью? Как из ARP таблицы удалить только один нужный IP?
8. [*] Установите эмулятор EVE-ng.
Инструкция по установке - https://github.com/svmyasnikov/eve-ng
Выполните задания на lldp, vlan, bonding в эмуляторе EVE-ng. 
---
## 03-sysadmin-08-net
1. Подключитесь к публичному маршрутизатору в интернет. Найдите маршрут к вашему публичному IP
```
telnet route-views.routeviews.org
Username: rviews
show ip route x.x.x.x/32
show bgp x.x.x.x/32
```
2. Создайте dummy0 интерфейс в Ubuntu. Добавьте несколько статических маршрутов. Проверьте таблицу маршрутизации.
3. Проверьте открытые TCP порты в Ubuntu, какие протоколы и приложения используют эти порты? Приведите несколько примеров.
4. Проверьте используемые UDP сокеты в Ubuntu, какие протоколы и приложения используют эти порты?
5. Используя diagrams.net, создайте L3 диаграмму вашей домашней сети или любой другой сети, с которой вы работали. 
6. [*] Установите Nginx, настройте в режиме балансировщика TCP или UDP.
7. [*] Установите bird2, настройте динамический протокол маршрутизации RIP.
8. [*] Установите Netbox, создайте несколько IP префиксов, используя curl проверьте работу API.

## 03-sysadmin-09-security
