Задание 1
Опишите не менее семи таблиц, из которых состоит база данных. Определите:

какие данные хранятся в этих таблицах,
какой тип данных у столбцов в этих таблицах, если данные хранятся в PostgreSQL.
Начертите схему полученной модели данных. Можете использовать онлайн-редактор: https://app.diagrams.net/

Этапы реализации:

Внимательно изучите предоставленный вам файл с данными и подумайте, как можно сгруппировать данные по смыслу.
Разбейте исходный файл на несколько таблиц и определите список столбцов в каждой из них.
Для каждого столбца подберите подходящий тип данных из PostgreSQL.
Для каждой таблицы определите первичный ключ (PRIMARY KEY).
Определите типы связей между таблицами.
Начертите схему модели данных. На схеме должны быть чётко отображены:
все таблицы с их названиями,
все столбцы с указанием типов данных,
первичные ключи (они должны быть явно выделены),
линии, показывающие связи между таблицами.
Результатом выполнения задания должен стать скриншот получившейся схемы базы данных.

Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению. Вы можете их выполнить, если хотите глубже и шире разобраться в материале.

Задание 2*
Разверните СУБД Postgres на своей хостовой машине, на виртуальной машине или в контейнере docker.
Опишите схему, полученную в предыдущем задании, с помощью скрипта SQL.
Создайте в вашей полученной СУБД новую базу данных и выполните полученный ранее скрипт для создания вашей модели данных.
В качестве решения приложите SQL скрипт и скриншот диаграммы.

---
<img width="641" height="371" alt="1" src="https://github.com/user-attachments/assets/9b05a463-92b8-4a10-be26-d4c7e536a3b9" />

workers:
```sql
CREATE TABLE workers (
worker_id       SERIAL          PRIMARY KEY,
full_name       VARCHAR(100)    NOT NULL,
salary          DECIMAL(12, 2)  NOT NULL,
position_id     INTEGER         NOT NULL,
department_id   INTEGER         NOT NULL,
hire_date       DATE            NOT NULL,
project_id      INTEGER,
branch_id       INTEGER         NOT NULL,
FOREIGN KEY (position_id) REFERENCES positions(position_id),
FOREIGN KEY (department_id) REFERENCES departments(department_id),
FOREIGN KEY (project_id) REFERENCES projects(project_id),
FOREIGN KEY (branch_id) REFERENCES branches(branch_id)
);
```

positions:
```sql
CREATE TABLE positions (
position_id     SERIAL      PRIMARY KEY,
position_name   VARCHAR(50) NOT NULL UNIQUE,
department_type VARCHAR(20) NOT NULL
);
```

departments
```
CREATE TABLE departments (
department_id SERIAL PRIMARY KEY,
name VARCHAR(100) NOT NULL,
);
```

projects
```sql
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

branches
```sql
CREATE TABLE branches (
branch_id   SERIAL          PRIMARY KEY,
address     VARCHAR(100)    NOT NULL UNIQUE
);
```

```bash
task1=# \d
                      List of relations
 Schema |             Name              |   Type   |  Owner
--------+-------------------------------+----------+----------
 public | branches                      | table    | postgres
 public | branches_branch_id_seq        | sequence | postgres
 public | departments                   | table    | postgres
 public | departments_department_id_seq | sequence | postgres
 public | positions                     | table    | postgres
 public | positions_position_id_seq     | sequence | postgres
 public | projects                      | table    | postgres
 public | projects_project_id_seq       | sequence | postgres
 public | workers                       | table    | postgres
 public | workers_worker_id_seq         | sequence | postgres
(10 rows)

task1=#
```
