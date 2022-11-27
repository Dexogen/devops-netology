# Оглавление
* [06-db-01-basics](#06-db-01-basics)
* [06-db-02-sql](#06-db-02-sql)
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

> ```yaml
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