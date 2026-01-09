**Задание 1**

На лекции рассматривались режимы репликации master-slave, master-
master, опишите их различия.

**Решение**

- `master-slave`

Slave - инстанс Базы-данных используется только для чтения ( read-only ), синхронизации данных с Master, запись должно производится только на Master.

- `master-master`

Инстансы Баз-данных равноценны, чтение и запись осуществляется на всех инстансах.

---

**Задание 2**

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

**Решение**

![Task2_1.bmp](https://github.com/user-attachments/files/24534400/2_1.bmp)

![Task2_2.bmp](https://github.com/user-attachments/files/24534402/2_2.bmp)

![Task2_3.bmp](https://github.com/user-attachments/files/24534407/2_3.bmp)

```bash
dev1@lhwdgzcbsh:~/.temp$ ls -lAht
total 16K
-rw-rw-r-- 1 dev1 dev1 1.1K Jan  9 18:29 compose.yml
-rw-rw-r-- 1 dev1 dev1   13 Jan  9 18:28 .s1.txt
-rw-rw-r-- 1 dev1 dev1   95 Jan  9 18:28 slave.cnf
-rw-rw-r-- 1 dev1 dev1   80 Jan  9 18:27 master.cnf
dev1@lhwdgzcbsh:~/.temp$
```

```bash
dev1@lhwdgzcbsh:~/.temp$ cat compose.yml
services:
  mysql-master:
    image: mysql:8.0
    container_name: mysql-master
    volumes:
      - mysql-master-volume:/var/lib/mysql
      - ./master.cnf:/etc/mysql/conf.d/master.cnf
    networks:
      - mysql-replication_network
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: "replicadb"
      MYSQL_ROOT_PASSWORD_FILE: /var/run/secrets/mysql_root_password
    secrets:
      - mysql_root_password
  mysql-slave:
    image: mysql:8.0
    container_name: mysql-slave
    volumes:
      - mysql-slave-volume:/var/lib/mysql
      - ./slave.cnf:/etc/mysql/conf.d/slave.cnf
    networks:
      - mysql-replication_network
    ports:
      - "3307:3306"
    environment:
      MYSQL_DATABASE: "replicadb"
      MYSQL_ROOT_PASSWORD_FILE: /var/run/secrets/mysql_root_password
    secrets:
      - mysql_root_password
    depends_on:
      - mysql-master

networks:
  mysql-replication_network:
    driver: bridge

volumes:
  mysql-master-volume:
  mysql-slave-volume:

secrets:
  mysql_root_password:
    file: ./.s1.txt
dev1@lhwdgzcbsh:~/.temp$

dev1@lhwdgzcbsh:~/.temp$ cat slave.cnf
[mysqld]
server-id=2
relay_log=mysql-relay-bin
log_bin=mysql-bin
binlog_format=ROW
read_only=1
dev1@lhwdgzcbsh:~/.temp$

dev1@lhwdgzcbsh:~/.temp$ cat master.cnf
[mysqld]
server-id=1
log_bin=mysql-bin
binlog_format=ROW
binlog_do_db=replicadb
dev1@lhwdgzcbsh:~/.temp$
```

---

```sql
# MASTER HOST
dev1@lhwdgzcbsh:~/.temp$ docker exec -it mysql-master mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.44 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE USER 'repl'@'%' IDENTIFIED WITH `mysql_native_password` BY '**REDACTED_FOR_PRIVACY**'; GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%'; FLUSH PRIVILEGES; SHOW MASTER STATUS;
Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      827 | replicadb    |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql>
```

```sql
# SLAVE HOST
dev1@lhwdgzcbsh:~/.temp$ docker exec -it mysql-slave mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.44 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CHANGE MASTER TO
    ->   MASTER_HOST='mysql-master',
    ->   MASTER_PORT=3306,
    ->   MASTER_USER='repl',
    ->   MASTER_PASSWORD='**REDACTED_FOR_PRIVACY**',
    ->   MASTER_LOG_FILE='mysql-bin.000003',
    ->   MASTER_LOG_POS=827; START SLAVE; SHOW SLAVE STATUS\G
Query OK, 0 rows affected, 9 warnings (0.00 sec)

Query OK, 0 rows affected, 1 warning (0.02 sec)

*************************** 1. row ***************************
               Slave_IO_State: Checking source version
                  Master_Host: mysql-master
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 827
               Relay_Log_File: mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 827
              Relay_Log_Space: 157
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 0
                  Master_UUID:
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Reading event from the relay log
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.00 sec)

mysql>
mysql>
```

```sql
# MASTER HOST
mysql> USE `replicadb`;
Database changed
mysql>
mysql> CREATE TABLE `user` ( `id` INT NOT NULL AUTO_INCREMENT PRIMARY KEY, `name` VARCHAR(10) NOT NULL ); INSERT INTO `user` (`name`) VALUES ('Tom'), ('Sam');
Query OK, 0 rows affected (0.00 sec)

Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql>
```

```sql
# SLAVE HOST
mysql> USE `replicadb`; SELECT * FROM `user`;
Database changed
+----+------+
| id | name |
+----+------+
|  1 | Tom  |
|  2 | Sam  |
+----+------+
2 rows in set (0.00 sec)

mysql>
```

---

**Задание 3***

Выполните конфигурацию master-master репликации. Произведите проверку.

Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.

**Решение**

![Task3_1.bmp](https://github.com/user-attachments/files/24534430/3_1.bmp)

![Task3_2.bmp](https://github.com/user-attachments/files/24534437/3_2.bmp)

![Task3_3.bmp](https://github.com/user-attachments/files/24534472/3_3.bmp)

```bash
dev1@lhwdgzcbsh:~/.temp$ ls -lAht
total 16K
-rw-rw-r-- 1 dev1 dev1 1022 Jan  9 19:06 compose.yml
-rw-rw-r-- 1 dev1 dev1   80 Jan  9 19:05 master2.cnf
-rw-rw-r-- 1 dev1 dev1   13 Jan  9 18:28 .s1.txt
-rw-rw-r-- 1 dev1 dev1   80 Jan  9 18:27 master1.cnf
dev1@lhwdgzcbsh:~/.temp$
```

```bash
dev1@lhwdgzcbsh:~/.temp$ cat compose.yml
services:
  mysql-master1:
    image: mysql:8.0
    container_name: mysql-master1
    volumes:
      - mysql-master1-volume:/var/lib/mysql
      - ./master1.cnf:/etc/mysql/conf.d/master1.cnf
    networks:
      - mysql-replication_network
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: "replicadb"
      MYSQL_ROOT_PASSWORD_FILE: /var/run/secrets/mysql_root_password
    secrets:
      - mysql_root_password
  mysql-master2:
    image: mysql:8.0
    container_name: mysql-master2
    volumes:
      - mysql-master2-volume:/var/lib/mysql
      - ./master2.cnf:/etc/mysql/conf.d/master2.cnf
    networks:
      - mysql-replication_network
    ports:
      - "3307:3306"
    environment:
      MYSQL_DATABASE: "replicadb"
      MYSQL_ROOT_PASSWORD_FILE: /var/run/secrets/mysql_root_password
    secrets:
      - mysql_root_password

networks:
  mysql-replication_network:
    driver: bridge

volumes:
  mysql-master1-volume:
  mysql-master2-volume:

secrets:
  mysql_root_password:
    file: ./.s1.txt
dev1@lhwdgzcbsh:~/.temp$

dev1@lhwdgzcbsh:~/.temp$ cat master1.cnf
[mysqld]
server-id=1
log_bin=mysql-bin
binlog_format=ROW
binlog_do_db=replicadb
dev1@lhwdgzcbsh:~/.temp$

dev1@lhwdgzcbsh:~/.temp$ cat master2.cnf
[mysqld]
server-id=2
log_bin=mysql-bin
binlog_format=ROW
binlog_do_db=replicadb
dev1@lhwdgzcbsh:~/.temp$
```

```sql
# MASTER1 HOST
dev1@lhwdgzcbsh:~/.temp$ docker exec -it mysql-master1 mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.44 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE USER 'repl'@'%' IDENTIFIED WITH `mysql_native_password` BY '**REDACTED_FOR_PRIVACY**'; GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%'; FLUSH PRIVILEGES; SHOW MASTER STATUS;

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      827 | replicadb    |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql>
mysql> CHANGE MASTER TO
    ->   MASTER_HOST='mysql-master2',
    ->   MASTER_PORT=3306,
    ->   MASTER_USER='repl',
    ->   MASTER_PASSWORD='**REDACTED_FOR_PRIVACY**',
    ->   MASTER_LOG_FILE='mysql-bin.000003',
    ->   MASTER_LOG_POS=827; START SLAVE; SHOW SLAVE STATUS\G
Query OK, 0 rows affected, 9 warnings (0.01 sec)

Query OK, 0 rows affected, 1 warning (0.03 sec)

*************************** 1. row ***************************
               Slave_IO_State: Checking source version
                  Master_Host: mysql-master2
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 827
               Relay_Log_File: 7e37b9c2f900-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 827
              Relay_Log_Space: 157
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 2
                  Master_UUID: de24d785-ed8e-11f0-bf9c-9265b539c18c
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.00 sec)

mysql>
mysql> USE `replicadb`;
Database changed
mysql>
mysql> CREATE TABLE `user` ( `id` INT NOT NULL AUTO_INCREMENT PRIMARY KEY, `name` VARCHAR(10) NOT NULL ); INSERT INTO `user` (`name`) VALUES ('Tom'), ('Sam');
Query OK, 0 rows affected (0.01 sec)

Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql>
mysql> SELECT * FROM `user`;
+----+------+
| id | name |
+----+------+
|  1 | Tom  |
|  2 | Sam  |
+----+------+
2 rows in set (0.00 sec)

mysql>
```

```sql
# MASTER2 HOST
dev1@lhwdgzcbsh:~/.temp$ docker exec -it mysql-master2 mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.44 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE USER 'repl'@'%' IDENTIFIED WITH `mysql_native_password` BY '**REDACTED_FOR_PRIVACY**'; GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%'; FLUSH PRIVILEGES; SHOW MASTER STATUS;

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      827 | replicadb    |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql>

mysql> CHANGE MASTER TO
    ->   MASTER_HOST='mysql-master1',
    ->   MASTER_PORT=3306,
    ->   MASTER_USER='repl',
    ->   MASTER_PASSWORD='**REDACTED_FOR_PRIVACY**',
    ->   MASTER_LOG_FILE='mysql-bin.000003',
    ->   MASTER_LOG_POS=827; START SLAVE; SHOW SLAVE STATUS\G
Query OK, 0 rows affected, 9 warnings (0.00 sec)

Query OK, 0 rows affected, 1 warning (0.03 sec)

*************************** 1. row ***************************
               Slave_IO_State: Checking source version
                  Master_Host: mysql-master1
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 827
               Relay_Log_File: d675ddc9c8b1-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 827
              Relay_Log_Space: 157
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 0
                  Master_UUID:
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: waiting for handler commit
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.00 sec)

mysql>
mysql> USE `replicadb`;
Database changed
mysql>
mysql> SELECT * FROM `user`;
+----+------+
| id | name |
+----+------+
|  1 | Tom  |
|  2 | Sam  |
+----+------+
2 rows in set (0.00 sec)

mysql>
```

