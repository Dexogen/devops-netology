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