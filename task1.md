
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
