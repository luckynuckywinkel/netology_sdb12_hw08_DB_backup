# Домашнее задание к занятию «Резервное копирование баз данных», Лебедев А.И., FOPS-10


---

### Задание 1. Резервное копирование

### Кейс
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования. 

Необходимо описать, какие варианты резервного копирования подходят в случаях: 

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

*Приведите ответ в свободной форме.*  

### Ответ:  



---

### Задание 2. PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

*Приведите ответ в свободной форме.*  

### Ответ:  



---

### Задание 3. MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL. 

3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?

*Приведите ответ в свободной форме.*  

### Решение на примере установленного в доккере MySQL:   

- Создадим таблицу и наполним ее несколькими таблицами:

```
mysql> create database backup_test;
Query OK, 1 row affected (0.00 sec)
```

---  

```
mysql> CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(30));
Query OK, 0 rows affected (0.04 sec)

mysql> show tables;
+-----------------------+
| Tables_in_backup_test |
+-----------------------+
| users                 |
+-----------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE cities (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(30));
Query OK, 0 rows affected (0.03 sec)

mysql> show tables;
+-----------------------+
| Tables_in_backup_test |
+-----------------------+
| cities                |
| users                 |
+-----------------------+
2 rows in set (0.01 sec)
```

---  

```
mysql> INSERT INTO users(name) VALUES ("Ivan Petrov"),("Sasha Ivanov"),("Max Fry"),("Johnny Depp");
Query OK, 4 rows affected (0.01 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> INSERT INTO cities(name) VALUES ("Berlin"),("Paris"),("Valencia"),("Tokyo");
Query OK, 4 rows affected (0.01 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> select * from users;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | Ivan Petrov  |
|  2 | Sasha Ivanov |
|  3 | Max Fry      |
|  4 | Johnny Depp  |
+----+--------------+
4 rows in set (0.00 sec)

mysql> select * from cities;
+----+----------+
| id | name     |
+----+----------+
|  1 | Berlin   |
|  2 | Paris    |
|  3 | Valencia |
|  4 | Tokyo    |
+----+----------+
4 rows in set (0.00 sec)
```


- Сделаем полный бэкап нашей маленькой базы:

```
root@sql:/home/vagrant# docker exec -it fa25621f8469 bash
bash-4.4# mysqldump -v -u admin -p --databases backup_test --single-transaction --flush-logs --source-data=2 > /tmp/full.sql
-- Connecting to localhost...
Enter password:
-- main : logs flushed successfully!
-- Starting transaction...
-- Setting savepoint...
-- Retrieving table structure for table cities...
-- Sending SELECT query...
-- Retrieving rows...
-- Rolling back to savepoint sp...
-- Retrieving table structure for table users...
-- Sending SELECT query...
-- Retrieving rows...
-- Rolling back to savepoint sp...
-- Releasing savepoint...
-- Disconnecting from localhost...
bash-4.4# cd /tmp/
bash-4.4# ls -lai
total 16
3021685 drwxrwxrwt 1 root root 4096 Sep  5 10:39 .
3037214 drwxr-xr-x 1 root root 4096 Jul 31 12:01 ..
3037308 -rw-r--r-- 1 root root 3044 Sep  5 10:39 full.sql
3037241 drwxr-xr-x 3 root root 4096 Jul 31 12:01 mysql
```

- А вот с инкрементным бэкапированием в контейнере у меня возникли трудности. Т.к. мой контейнер не предполагал наличия в нем пакетного менеджера, а уж, тем более, уьилиты для инкрементного копирования, дальнейшее задание предполагало создание доккерфайла и кастомизацию моего файла доккер-компоуз. И я решил пойти по чуть более легкому пути, развернув терраформом и ансиблом машину с MySQL не в контейнере на YC и создав там такую же базу.

- Для инкрементного юэкапирования я буду использовать утилиту **Percona XtraBackup**. Почитав форумы, я решил выбрать именно ее. Устанавливаем по инструкции с официального сайта и пишем для нее конфиг на bash по инструкции:

```
#!/bin/bash

# Путь к Percona XtraBackup
XTRABACKUP=/usr/bin/innobackupex

# Путь, где будут храниться инкрементные бэкапы
BACKUP_DIR=/home/winkel/backup

# Путь к директории с полным бэкапом
FULL_BACKUP_DIR=/home/winkel/backup

# Создает инкрементный бэкап
$XTRABACKUP --incremental $BACKUP_DIR --incremental-basedir=$FULL_BACKUP_DIR
```

- Настраиваем планировщик crontab, чтобы он делал бэкап каждые две минуты (что б быстрее):

```
  GNU nano 3.2                                                                                             /tmp/crontab.ivFUmm/crontab

# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command


*/2 * * * * /home/winkel/inc_backup.sh
```

- Напомню, что мы заполнили таблицу тестовыми данными и уже сделали полный бэкап. Добавим еще несколько городов:

```
mysql> INSERT INTO cities(name) VALUES ("Moscow"),("Samara"),("Kiev"),("Minsk");
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> select * from cities;
+----+----------+
| id | name     |
+----+----------+
|  1 | Berlin   |
|  2 | Paris    |
|  3 | Valencia |
|  4 | Tokyo    |
|  5 | Moscow   |
|  6 | Samara   |
|  7 | Kiev     |
|  8 | Minsk    |
+----+----------+
8 rows in set (0.00 sec)
```

- Посмотрим, сыпятся ли нам инкрементные бэкапы в директрию, которую мы указали:

```
root@fhm3d67gee8do13fdsv6:/home/winkel/backup# ls -lai
total 36
48246 drwxr-xr-x 8 root   root   4096 Sep  5 11:50 .
 3279 drwxr-xr-x 6 winkel winkel 4096 Sep  5 11:49 ..
48294 drwxr-x--- 2 root   root   4096 Sep  5 11:40 2023-09-05_11-40-01
48296 drwxr-x--- 2 root   root   4096 Sep  5 11:42 2023-09-05_11-42-01
48298 drwxr-x--- 2 root   root   4096 Sep  5 11:44 2023-09-05_11-44-01
48301 drwxr-x--- 2 root   root   4096 Sep  5 11:46 2023-09-05_11-46-01
48303 drwxr-x--- 2 root   root   4096 Sep  5 11:48 2023-09-05_11-48-01
48306 drwxr-x--- 2 root   root   4096 Sep  5 11:50 2023-09-05_11-50-01
48247 -rw-r--r-- 1 root   root   3043 Sep  5 11:23 full.sql
root@fhm3d67gee8do13fdsv6:/home/winkel/backup#
```

- Что ж, посмотрим, что у нас выйдет. Дропаем базу данных, создаем снова, заливаем полный бэкап и смотрим:

```
mysql> source /home/winkel/backup/full.sql
...
mysql> select * from cities;
+----+----------+
| id | name     |
+----+----------+
|  1 | Berlin   |
|  2 | Paris    |
|  3 | Valencia |
|  4 | Tokyo    |
+----+----------+
4 rows in set (0.00 sec)

mysql> show tables;
+-----------------------+
| Tables_in_backup_test |
+-----------------------+
| cities                |
| users                 |
+-----------------------+
2 rows in set (0.01 sec)
```

- А дальше у меня начались веселые приключения :) Оказывается, нельзя просто так взять и подсунуть Percona XtraBackup фулловый бэкап, сделанный mysqldump'ом. Что ж делаем бэкап этим софтом. Я сам на это подписался:

```
root@fhm3d67gee8do13fdsv6:/home/winkel# xtrabackup --backup --target-dir=/home/winkel/backup --user=root --password=strongpassword --databases backup_test


2023-09-05T13:24:17.024925-00:00 0 [Note] [MY-011825] [Xtrabackup] recognized server arguments: --server-id=1 --log_bin=/var/log/mysql/mybin.log --datadir=/var/lib/mysql
2023-09-05T13:24:17.025183-00:00 0 [Note] [MY-011825] [Xtrabackup] recognized client arguments: --user=root --backup=1 --target-dir=/home/winkel/backup --user=root --password=* --databases=backup_test
xtrabackup version 8.0.34-29 based on MySQL server 8.0.34 Linux (x86_64) (revision id: 5ba706ee)
230905 13:24:17  version_check Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_group=xtrabackup' as 'root'  (using password: YES).
230905 13:24:17  version_check Connected to MySQL server
230905 13:24:17  version_check Executing a version check against the server...
230905 13:24:17  version_check Done.
2023-09-05T13:24:17.218141-00:00 0 [Note] [MY-011825] [Xtrabackup] Connecting to MySQL server host: localhost, user: root, password: set, port: not set, socket: not set
2023-09-05T13:24:17.226047-00:00 0 [Note] [MY-011825] [Xtrabackup] Using server version 8.0.33
2023-09-05T13:24:17.228989-00:00 0 [Note] [MY-011825] [Xtrabackup] Executing LOCK INSTANCE FOR BACKUP ...
2023-09-05T13:24:17.229904-00:00 0 [Note] [MY-011825] [Xtrabackup] uses posix_fadvise().
2023-09-05T13:24:17.229960-00:00 0 [Note] [MY-011825] [Xtrabackup] cd to /var/lib/mysql
2023-09-05T13:24:17.229994-00:00 0 [Note] [MY-011825] [Xtrabackup] open files limit requested 0, set to 1024
2023-09-05T13:24:17.247131-00:00 0 [Note] [MY-011825] [Xtrabackup] using the following InnoDB configuration:
2023-09-05T13:24:17.247179-00:00 0 [Note] [MY-011825] [Xtrabackup] innodb_data_home_dir = .
2023-09-05T13:24:17.247198-00:00 0 [Note] [MY-011825] [Xtrabackup] innodb_data_file_path = ibdata1:12M:autoextend
2023-09-05T13:24:17.247238-00:00 0 [Note] [MY-011825] [Xtrabackup] innodb_log_group_home_dir = ./
2023-09-05T13:24:17.247257-00:00 0 [Note] [MY-011825] [Xtrabackup] innodb_log_files_in_group = 2
2023-09-05T13:24:17.247283-00:00 0 [Note] [MY-011825] [Xtrabackup] innodb_log_file_size = 50331648
2023-09-05T13:24:17.249853-00:00 0 [Note] [MY-011825] [Xtrabackup] inititialize_service_handles suceeded
2023-09-05T13:24:17.385676-00:00 0 [Note] [MY-011825] [Xtrabackup] Connecting to MySQL server host: localhost, user: root, password: set, port: not set, socket: not set
2023-09-05T13:24:17.390944-00:00 0 [Note] [MY-011825] [Xtrabackup] Redo Log Archiving is not set up.
2023-09-05T13:24:17.500517-00:00 0 [Note] [MY-012953] [InnoDB] Disabling background ibuf IO read threads.
2023-09-05T13:24:17.500686-00:00 1 [Note] [MY-011825] [Xtrabackup] >> log scanned up to (19646156)
2023-09-05T13:24:17.702926-00:00 0 [Note] [MY-011825] [Xtrabackup] Generating a list of tablespaces
2023-09-05T13:24:17.703020-00:00 0 [Note] [MY-012204] [InnoDB] Scanning './'
2023-09-05T13:24:17.704178-00:00 0 [Note] [MY-012208] [InnoDB] Completed space ID check of 2 files.
2023-09-05T13:24:17.704525-00:00 0 [Warning] [MY-012091] [InnoDB] Allocated tablespace ID 4 for backup_test/cities, old maximum was 0
****
2023-09-05T13:24:19.943716-00:00 0 [Note] [MY-011825] [Xtrabackup] Transaction log of lsn (19646156) to (19646156) was copied.
2023-09-05T13:24:20.152642-00:00 0 [Note] [MY-011825] [Xtrabackup] completed OK!
```

- Что ж, так как мы уже добавили новые города, давайте добавим еще людей, чтобы потом проверить инкремент:

```
mysql> INSERT INTO users(name) VALUES ("Sean Penn"),("Bob Marley"),("Elice Cooper"),("Brad Pitt");
Query OK, 4 rows affected (0.02 sec)
Records: 4  Duplicates: 0  Warnings: 0
```

- Теперь попробуем сделать инкремент. Я буду использовать просто команду в строку для проверки. Потом это все можно будет также завернуть в скрипт и в кронтаб:

```
root@fhm3d67gee8do13fdsv6:/home/winkel/backup# xtrabackup --backup --target-dir=/home/winkel/incremental_backups --incremental-basedir=/home/winkel/backup --user=root --password=strongpassword --databases backup_test

2023-09-05T13:34:25.204004-00:00 0 [Note] [MY-011825] [Xtrabackup] recognized server arguments: --server-id=1 --log_bin=/var/log/mysql/mybin.log --datadir=/var/lib/mysql
2023-09-05T13:34:25.204313-00:00 0 [Note] [MY-011825] [Xtrabackup] recognized client arguments: --user=root --backup=1 --target-dir=/home/winkel/incremental_backups --incremental-basedir=/home/winkel/backup --user=root --password=* --databases=backup_test
xtrabackup version 8.0.34-29 based on MySQL server 8.0.34 Linux (x86_64) (revision id: 5ba706ee)
230905 13:34:25  version_check Connecting to MySQL server with DSN 'dbi:mysql:;mysql_read_default_group=xtrabackup' as 'root'  (using password: YES).
230905 13:34:25  version_check Connected to MySQL server
230905 13:34:25  version_check Executing a version check against the server...
230905 13:34:25  version_check Done.
2023-09-05T13:34:25.282077-00:00 0 [Note] [MY-011825] [Xtrabackup] Connecting to MySQL server host: localhost, user: root, password: set, port: not set, socket: not set
2023-09-05T13:34:25.288256-00:00 0 [Note] [MY-011825] [Xtrabackup] Using server version 8.0.33
2023-09-05T13:34:25.291050-00:00 0 [Note] [MY-011825] [Xtrabackup] Executing LOCK INSTANCE FOR BACKUP ...
2023-09-05T13:34:25.291706-00:00 0 [Note] [MY-011825] [Xtrabackup] incremental backup from 19646156 is enabled.
2023-09-05T13:34:25.292045-00:00 0 [Note] [MY-011825] [Xtrabackup] uses posix_fadvise().
2023-09-05T13:34:25.292095-00:00 0 [Note] [MY-011825] [Xtrabackup] cd to /var/lib/mysql
2023-09-05T13:34:25.292122-00:00 0 [Note] [MY-011825] [Xtrabackup] open files limit requested 0, set to 1024
2023-09-05T13:34:25.292553-00:00 0 [Note] [MY-011825] [Xtrabackup] using the following InnoDB configuration:
***
2023-09-05T13:34:29.083740-00:00 0 [Note] [MY-011825] [Xtrabackup] Transaction log of lsn (19652754) to (19652754) was copied.
2023-09-05T13:34:29.192720-00:00 0 [Note] [MY-011825] [Xtrabackup] completed OK!
```

- Алилуййййя! Что ж. Грохаем базу еще разок. И восстанавливаем фулл:

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| backup_test        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> drop database backup_test;
Query OK, 2 rows affected (0.13 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

---  









---
