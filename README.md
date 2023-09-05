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





---
