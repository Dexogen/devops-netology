# Оглавление
* [06-db-01-basics](#06-db-01-basics)
* [06-db-02-sql](#06-db-02-sql)
* [06-db-03-mysql](#06-db-03-mysql)
* [06-db-04-postgresql](#06-db-04-postgresql)
* [06-db-05-elasticsearch](#06-db-05-elasticsearch)
---

# 06-db-01-basics

## Задача 1

Архитектор ПО решил проконсультироваться у вас, какой тип БД 
лучше выбрать для хранения определенных данных.

Он вам предоставил следующие типы сущностей, которые нужно будет хранить в БД:

- Электронные чеки в json виде
> Документно-ориентированные, классика выбора бд для json
- Склады и автомобильные дороги для логистической компании
> Реляционные. Большой объем связанных между собой данных, требующих создание связей между таблицами. Если речь идет о исключительно логистической задаче построения маршрутов, то тут подойдет графовая бд.
- Генеалогические деревья
> Иерархическая структура, соответствующие БД.
- Кэш идентификаторов клиентов с ограниченным временем жизни для движка аутенфикации
> Ключ-значение. Хранение в оперативке, возможность ограничить время жизни.
- Отношения клиент-покупка для интернет-магазина
> Вполне подойдут реляционные, т.к. явно требуются несколько связанных таблиц.

Выберите подходящие типы СУБД для каждой сущности и объясните свой выбор.

## Задача 2

Вы создали распределенное высоконагруженное приложение и хотите классифицировать его согласно 
CAP-теореме. Какой классификации по CAP-теореме соответствует ваша система, если 
(каждый пункт - это отдельная реализация вашей системы и для каждого пункта надо привести классификацию). А согласно PACELC-теореме, как бы вы классифицировали данные реализации?

- Данные записываются на все узлы с задержкой до часа (асинхронная запись)
    > ### CAP
    > Асинхронность говорит об отсутствии согласованности. Значит предполагается высокая доступность и устойчивость к разделению (т.к. узлы могут длительное время не общаться). Получается это `AP` система.
    > ### PACELC
    > При разделении баз доступны и неконсистентны, значит `PA`. При отсутствии разделения дальнейший приоритет неизвестен, но можно предположить, что задержка в достижении консистентности обусловлена необходимостью быстрого отклика. Будем считать, что это `PA/EL`.
- При сетевых сбоях, система может разделиться на 2 раздельных кластера
    > ### CAP
    > Система устойчива к разделению `P` и судя по формулировке должна быть к этому готова и должна продолжать свою работу. Значит это `AP`.
    > ### PACELC
    > При разделении система сохраняет доступность, но не консистентность, значит PA. Как и в прошлой системе дальнейший приоритет неизвестен, так что это равновероятно может быть `PA/EL` или `PA/EC`.
- Система может не прислать корректный ответ или сбросить соединение
    > ### CAP
    > Ответ не всегда есть, значит это может быть `CP`.
    > ### PACELC
    > При сетевом разделении базы недоступны, но вероятно консистентны. В консистентном состоянии будет стремиться ее сохранить любой ценой. Веротяно, это `PC/EC`

## Задача 3

Могут ли в одной системе сочетаться принципы BASE и ACID? Почему?
> Не могут, т.к. во главе каждого принципа стоят разные цели. BASE - это про производительность, а ACID про доступность и надежность.
## Задача 4

Вам дали задачу написать системное решение, основой которого бы послужили:

- фиксация некоторых значений с временем жизни
- реакция на истечение таймаута

Вы слышали о key-value хранилище, которое имеет механизм [Pub/Sub](https://habr.com/ru/post/278237/). 
Что это за система? Какие минусы выбора данной системы?
> Система Pub/Sub, это способ построения передачи сообщений внутри проекта. Между паблишерами и сабскрайберами (последним даже не нужно знать о существовании первых).
> Под описанные требования нужна in-memory БД, т.к. у нас есть требования к скорости и работе с таймаутами. Это может быть Redis и Memcached.
> Из минусов - необходимость быть готовым к потере данных в оперативной памяти при сбое сервера или заморачиваться со сбросом данных на диск (persistence режим у редиса, например).

# 06-db-02-sql

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

> ```yaml[]
> version: '3.5'
> 
> services:
>     postgres:
>         container_name: postgres_container
>         image: postgres:12
>         environment:
>             POSTGRES_USER: postgres
>             POSTGRES_PASSWORD: postgres
>             PGDATA: /data/postgres
>         volumes:
>             - /ssd-raid-1/docker/postgres/data:/data/postgres
>             - /ethernity/docker/postgres/backup:/backup/postgres
>         ports:
>             - 15432:5432
>         networks:
>             - postgres
>         restart: unless-stopped
> 
> networks:
>     postgres:
>         driver: bridge
> ```

## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
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

> ```sql
> CREATE DATABASE test_db;
> CREATE TABLE "public"."orders" (
>  "id" SERIAL PRIMARY KEY,
>  "name" VARCHAR(255) NOT NULL,
>  "price" INT NOT NULL
> );
> 
> CREATE TABLE "public"."clients" (
>  "id" SERIAL PRIMARY KEY,
>  "lastname" VARCHAR(255) NOT NULL,
>  "country" VARCHAR(255) NOT NULL,
>  "order" INTEGER REFERENCES "public"."orders" (id)
> );
>
> CREATE INDEX "country" ON "public"."clients" (country);
>
> CREATE ROLE test_admin_user WITH LOGIN PASSWORD 'test_admin_password';
> GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO  test_admin_user;
>
> CREATE ROLE test_simple_user WITH LOGIN PASSWORD 'test_simple_password';
> GRANT SELECT,INSERT,UPDATE,DELETE ON ALL TABLES IN SCHEMA public TO test_simple_user;
> ```

> ```sql
> postgres=# \l
>                                  List of databases
>    Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
> -----------+----------+----------+------------+------------+-----------------------
>  postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
>  template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
>            |          |          |            |            | postgres=CTc/postgres
>  template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
>            |          |          |            |            | postgres=CTc/postgres
>  test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres         +
>            |          |          |            |            | postgres=CTc/postgres
> (4 rows)
> postgres=# \c test_db
> test_db=# \d+
>                             List of relations
>  Schema |      Name      |   Type   |  Owner   |    Size    | Description 
> --------+----------------+----------+----------+------------+-------------
>  public | clients        | table    | postgres | 8192 bytes | 
>  public | clients_id_seq | sequence | postgres | 8192 bytes | 
>  public | orders         | table    | postgres | 0 bytes    | 
>  public | orders_id_seq  | sequence | postgres | 8192 bytes | 
> (4 rows)
> test_db=# \d+ orders
>                                                         Table "public.orders"
>  Column |          Type          | Collation | Nullable |              Default               | Storage 
> --------+------------------------+-----------+----------+------------------------------------+----------
>  id     | integer                |           | not null | nextval('orders_id_seq'::regclass) | plain
>  name   | character varying(255) |           | not null |                                    | extended
>  price  | integer                |           | not null |                                    | plain
> Indexes:
>     "orders_pkey" PRIMARY KEY, btree (id)
> Referenced by:
>     TABLE "clients" CONSTRAINT "clients_order_fkey" FOREIGN KEY ("order") REFERENCES orders(id)
> Access method: heap
> 
> test_db=# \d+ clients
>                                                          Table "public.clients"
>   Column  |          Type          | Collation | Nullable |               Default               | Storage
> ----------+------------------------+-----------+----------+-------------------------------------+----------
>  id       | integer                |           | not null | nextval('clients_id_seq'::regclass) | plain
>  lastname | character varying(255) |           | not null |                                     | extended
>  country  | character varying(255) |           | not null |                                     | extended
>  order    | integer                |           |          |                                     | plain
> Indexes:
>     "clients_pkey" PRIMARY KEY, btree (id)
>     "country" btree (country)
> Foreign-key constraints:
>     "clients_order_fkey" FOREIGN KEY ("order") REFERENCES orders(id)
> Access method: heap
> ```

> ```sql
> -- Список доступов к таблицам в базе
> SELECT grantee, string_agg(privilege_type, ', ') AS privileges
> FROM information_schema.role_table_grants 
> WHERE table_catalog='test_db' AND table_schema='public'
> GROUP BY grantee;
>      grantee      |                                                          privileges                                                          
> ------------------+------------------------------------------------------------------------------------------------------------------------------
> postgres         | INSERT, SELECT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER, INSERT, SELECT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER
> test_admin_user  | TRIGGER, UPDATE, DELETE, INSERT, SELECT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRUNCATE, REFERENCES, TRIGGER, INSERT, SELECT
> test_simple_user | SELECT, UPDATE, DELETE, UPDATE, DELETE, INSERT, INSERT, SELECT
> (3 rows)
> -- Не очень понял в чем разница с предыдущим вопросом, но список самих юзеров еще можно получить так:
> test_db=# \du+
> ```
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

> ```sql
> INSERT INTO "public"."orders" (name, price)
> VALUES 
>   ('Шоколад', 10), 
>   ('Принтер', 3000), 
>   ('Книга', 500), 
>   ('Монитор', 7000), 
>   ('Гитара', 4000 );
> ```

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

> ```sql
> INSERT INTO "public"."clients" (lastname, country)
> VALUES 
>   ('Иванов Иван Иванович', 'USA'),
>   ('Петров Петр Петрович', 'Canada'),
>   ('Иоганн Себастьян Бах', 'Japan'),
>   ('Ронни Джеймс Дио', 'Russia'),
>   ('Ritchie Blackmore', 'Russia');
> ```

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.

> ```sql
> SELECT COUNT(*) FROM "public"."orders";
> 5
> SELECT COUNT(*) FROM "public"."clients";
> 5
> ```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

> ```sql
> UPDATE "public"."clients"
> SET "order" = (
> 	SELECT id 
> 	FROM "public"."orders" 
> 	WHERE "name"='Книга') 
> WHERE "lastname"='Иванов Иван Иванович';
> 
> UPDATE "public"."clients"
> SET "order" = (
> 	SELECT id 
> 	FROM "public"."orders" 
> 	WHERE "name"='Монитор') 
> WHERE "lastname"='Петров Петр Петрович';
> 
> UPDATE "public"."clients"
> SET "order" = (
> 	SELECT id 
> 	FROM "public"."orders" 
> 	WHERE "name"='Гитара') 
> WHERE "lastname"='Иоганн Себастьян Бах';
> ```

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
> ```sql
> SELECT cli.* 
> FROM "public"."clients" cli 
>   JOIN "public"."orders" ord 
>   ON cli.order = ord.id;
>  id |       lastname       | country | order
> ----+----------------------+---------+-------
>   1 | Иванов Иван Иванович | USA     |     3
>   2 | Петров Петр Петрович | Canada  |     4
>   3 | Иоганн Себастьян Бах | Japan   |     5
> ```
Подсказка - используйте директиву `UPDATE`.

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).
> ```sql
> EXPLAIN SELECT cli.* 
> FROM "public"."clients" cli 
>   JOIN "public"."orders" ord 
>   ON cli.order = ord.id;
>
>                                  QUERY PLAN
> ----------------------------------------------------------------------------
>  Hash Join  (cost=11.57..24.20 rows=70 width=1040)
>    Hash Cond: (ord.id = cli."order")
>    ->  Seq Scan on orders ord  (cost=0.00..11.40 rows=140 width=4)
>    ->  Hash  (cost=10.70..10.70 rows=70 width=1040)
>          ->  Seq Scan on clients cli  (cost=0.00..10.70 rows=70 width=1040)
> ```
Приведите получившийся результат и объясните что значат полученные значения.
> `EXPLAIN` выводит логику работы ядра при выполнении запросов.
> `Hash Join` - операция с несколькими субоперациями, используется для объединения двух наборов записей. 
> Cначала `Hash Join` вызывает `Hash`, который в свою очередь вызывает `Seq Scan` по `orders`. Потом `Hash` создает в памяти/диске хэш/массив/словарь со строками из источника, хэшированными с помощью того, что используется для объединения данных (в нашем случае это столбец `id` в `orders`).
> Потом `Hash Join` запускает вторую субоперацию `Seq Scan` по `clients` и для каждой строки из неё, делает следующее:
> 1. Проверяет, есть ли ключ `join` (`clients.order` в данном случае) в хэше, возвращенном операцией `Hash`.
> 2. Если нет, данная строка из субоперации игнорируется (не будет возвращена).
> 3. Если ключ существует, `Hash Join` берет строки из хэша и, основываясь на этой строке, с одной стороны, и всех строках хэша, с другой стороны, генерирует вывод строк.
>
> Теперь относительно данных из скобок (на примере первого `Seq Scan`):
> - `cost` - это два значения, описывающее затратность операции. Первое число `0.00` - затраты на получение первой строки, второе `11.40` - всех строк;
> - `rows` - приблизительное кол-во возвращаемых планировщиком строк (`140`) при выполнении операции `Seq Scan`;
> - `width` - средний размер одной строки в байтах (`4`).

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления.

> ```bash
> # Первый контейнер
> pg_dump -U postgres test_db > /backup/postgres/test_db.sql
> # Второй контейнер
> psql -U postgres -c 'create database test_db;'
> psql -U postgres test_db < /backup/postgres/test_db.sql
> ```
> Задачи перенести роли и юзеров не было, но это тоже не проблема:
> ```bash
> pg_dumpall -U postgres --roles-only > roles.sql
> ```

# 06-db-03-mysql

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

> ```yaml
> version: "3"
> 
> services:
>   db:
>     image: mysql:8.0
>     volumes:
>       - /ssd-raid-1/docker/mysql/data:/var/lib/mysql
>       - /ethernity/docker/mysql/backup:/backup/mysql
>     restart: always
>     ports:
>       - "3307:3306"
>     environment:
>       MYSQL_ROOT_PASSWORD: root
>       MYSQL_DATABASE: test_db
>       MYSQL_ROOT_HOST: '%'
> ```

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

> ```bash
> wget -O - -o /dev/null https://raw.githubusercontent.com/netology-code/virt-homeworks/virt-11/06-db-03-mysql/test_data/test_dump.sql | docker exec -i mysql8 mysql -uroot -proot test_db
> ```

Перейдите в управляющую консоль `mysql` внутри контейнера.

> ```bash
> docker exec -it mysql8 mysql -Uroot -proot
>```

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

> ```sql
> \status
> --------------
> mysql  Ver 8.0.31 for Linux on x86_64 (MySQL Community Server - GPL)
> ...
>```

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

> ```sql
> SHOW tables;
> +-------------------+
> | Tables_in_test_db |
> +-------------------+
> | orders            |
> +-------------------+
> 1 row in set (0.00 sec)
>```

**Приведите в ответе** количество записей с `price` > 300.

> ```sql
> SELECT COUNT(*)
> FROM orders
> WHERE price > 300
> +----------+
> | COUNT(*) |
> +----------+
> |        1 |
> +----------+
> 1 row in set (0.00 sec)
>```

В следующих заданиях мы будем продолжать работу с данным контейнером.

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней 
- количество попыток авторизации - 3 
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

> ```sql
> CREATE USER 'test'@'localhost' 
> 	IDENTIFIED WITH mysql_native_password BY 'test-pass'
> 	WITH MAX_CONNECTIONS_PER_HOUR 100
> 	PASSWORD EXPIRE INTERVAL 180 DAY
> 	FAILED_LOGIN_ATTEMPTS 3
> 	ATTRIBUTE '{"first_name":"James", "last_name":"Pretty"}';
> ```

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
> ```sql
> GRANT SELECT ON test_db.* TO test;
>```

Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

> ```sql
> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE user = 'test';
> +------+------+-----------+
> | USER | HOST | ATTRIBUTE |
> +------+------+-----------+
> | test | %    | NULL      |
> +------+------+-----------+
> 1 row in set (0.00 sec)
> ```
## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

> ```sql
> SELECT table_schema,table_name,engine FROM information_schema.tables WHERE table_schema = DATABASE();
> +--------------+------------+--------+
> | TABLE_SCHEMA | TABLE_NAME | ENGINE |
> +--------------+------------+--------+
> | test_db      | orders     | InnoDB |
> +--------------+------------+--------+
> 1 row in set (0.00 sec)
> ```

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`

> ```sql
> ALTER TABLE orders ENGINE = 'MyISAM';
> ALTER TABLE orders ENGINE = 'InnoDB';
> ```

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

> #### /etc/my.cnf
> ```bash
> innodb_flush_log_at_trx_commit = 2 # Скорость IO важнее сохранности данных
> innodb_file_per_table = ON # Нужна компрессия таблиц для экономии места на диске
> innodb_log_buffer_size = 1M # Размер буффера с незакомиченными транзакциями 1 Мб
> innodb_buffer_pool_size = 19G # Буффер кеширования 30% от ОЗУ (64GB)
> innodb_log_file_size = 100M # Размер файла логов операций 100 Мб
> ```

# 06-db-04-postgresql

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

# 06-db-05-elasticsearch

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения

Далее мы будем работать с данным экземпляром elasticsearch.

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`