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
---