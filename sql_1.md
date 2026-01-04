**Задание 1**

Получите уникальные названия районов из таблицы с адресами, которые начинаются на “K” и заканчиваются на “a” и не содержат пробелов.

**Решение**

![Task1.bmp](https://github.com/user-attachments/files/24423424/1_1.bmp)

```sql
mysql> SELECT DISTINCT(`district`) FROM `address` WHERE `district` LIKE 'K%' AND `district` LIKE '%a' AND `district` NOT LIKE '% %' AND `district` NOT REGEXP ' ';
+-----------+
| district  |
+-----------+
| Kanagawa  |
| Kalmykia  |
| Kaduna    |
| Karnataka |
| Kütahya  |
| Kerala    |
| Kitaa     |
+-----------+
7 rows in set (0.01 sec)

mysql>
```

---

**Задание 2**

Получите из таблицы платежей за прокат фильмов информацию по платежам, которые выполнялись в промежуток с 15 июня 2005 года по 18 июня 2005 года включительно и стоимость которых превышает 10.00.

![Task2.bmp](https://github.com/user-attachments/files/24423438/1_2.bmp)

```sql
mysql> SELECT * FROM `payment` WHERE `payment_date` BETWEEN '2005-06-15 00:00:00' AND '2005-06-18 23:59:59' AND `amount` > 10.00;
+------------+-------------+----------+-----------+--------+---------------------+---------------------+
| payment_id | customer_id | staff_id | rental_id | amount | payment_date        | last_update         |
+------------+-------------+----------+-----------+--------+---------------------+---------------------+
|        908 |          33 |        1 |      1301 |  10.99 | 2005-06-15 09:46:33 | 2006-02-15 22:12:36 |
|       7017 |         260 |        1 |      2091 |  10.99 | 2005-06-17 18:09:04 | 2006-02-15 22:14:58 |
|       8272 |         305 |        1 |      2166 |  11.99 | 2005-06-17 23:51:21 | 2006-02-15 22:15:47 |
|      12888 |         477 |        1 |      2306 |  10.99 | 2005-06-18 08:33:23 | 2006-02-15 22:19:46 |
|      13892 |         516 |        1 |      1718 |  10.99 | 2005-06-16 14:52:02 | 2006-02-15 22:20:47 |
|      14620 |         544 |        2 |      1434 |  10.99 | 2005-06-15 18:30:46 | 2006-02-15 22:21:35 |
|      15313 |         572 |        2 |      1889 |  10.99 | 2005-06-17 04:05:12 | 2006-02-15 22:22:22 |
+------------+-------------+----------+-----------+--------+---------------------+---------------------+
7 rows in set (0.01 sec)

mysql>
```

---

**Задание 3**

Получите последние пять аренд фильмов.

![Task3.bmp](https://github.com/user-attachments/files/24423441/1_3.bmp)

```sql
mysql> SELECT * FROM `rental` ORDER BY `rental_id` DESC LIMIT 5;
+-----------+---------------------+--------------+-------------+---------------------+----------+---------------------+
| rental_id | rental_date         | inventory_id | customer_id | return_date         | staff_id | last_update         |
+-----------+---------------------+--------------+-------------+---------------------+----------+---------------------+
|     16049 | 2005-08-23 22:50:12 |         2666 |         393 | 2005-08-30 01:01:12 |        2 | 2006-02-15 21:30:53 |
|     16048 | 2005-08-23 22:43:07 |         2019 |         103 | 2005-08-31 21:33:07 |        1 | 2006-02-15 21:30:53 |
|     16047 | 2005-08-23 22:42:48 |         2088 |         114 | 2005-08-25 02:48:48 |        2 | 2006-02-15 21:30:53 |
|     16046 | 2005-08-23 22:26:47 |         4364 |          74 | 2005-08-27 18:02:47 |        2 | 2006-02-15 21:30:53 |
|     16045 | 2005-08-23 22:25:26 |          772 |          14 | 2005-08-25 23:54:26 |        1 | 2006-02-15 21:30:53 |
+-----------+---------------------+--------------+-------------+---------------------+----------+---------------------+
5 rows in set (0.00 sec)

mysql>
```

---

**Задание 4**

Одним запросом получите активных покупателей, имена которых Kelly или Willie.

Сформируйте вывод в результат таким образом:

все буквы в фамилии и имени из верхнего регистра переведите в нижний регистр,
замените буквы 'll' в именах на 'pp'.

![Task4.bmp](https://github.com/user-attachments/files/24423524/1_4.bmp)

```sql
mysql> SELECT
    -> REPLACE(LOWER(`first_name`), 'll', 'pp') AS `first_name_custom`,
    -> LOWER(last_name) AS `last_name_lower`
    -> FROM
    -> `customer`
    -> WHERE
    -> `first_name` IN ('Kelly', 'Willie')
    -> AND `active` = 1;
+-------------------+-----------------+
| first_name_custom | last_name_lower |
+-------------------+-----------------+
| keppy             | torres          |
| wippie            | howell          |
| wippie            | markham         |
| keppy             | knott           |
+-------------------+-----------------+
4 rows in set (0.01 sec)

mysql>
```

---

Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

**Задание 5***

Выведите Email каждого покупателя, разделив значение Email на две отдельных колонки: в первой колонке должно быть значение, указанное до @, во второй — значение, указанное после @.

![Task5.bmp](https://github.com/user-attachments/files/24423478/1_5.bmp)

![Task5_1.bmp](https://github.com/user-attachments/files/24423479/1_5_1.bmp)


```sql
mysql> SELECT SUBSTRING_INDEX(`email`, '@', 1) AS `before_at_sign`, SUBSTRING_INDEX(`email`, '@', -1) AS `after_at_sign` FROM `customer`;
+-----------------------+--------------------+
| before_at_sign        | after_at_sign      |
+-----------------------+--------------------+
| MARY.SMITH            | sakilacustomer.org |
| PATRICIA.JOHNSON      | sakilacustomer.org |
| LINDA.WILLIAMS        | sakilacustomer.org |
| BARBARA.JONES         | sakilacustomer.org |
| ELIZABETH.BROWN       | sakilacustomer.org |
| JENNIFER.DAVIS        | sakilacustomer.org |
| MARIA.MILLER          | sakilacustomer.org |
| SUSAN.WILSON          | sakilacustomer.org |
| MARGARET.MOORE        | sakilacustomer.org |

...

| TRACY.HERRMANN        | sakilacustomer.org |
| SETH.HANNON           | sakilacustomer.org |
| KENT.ARSENAULT        | sakilacustomer.org |
| TERRANCE.ROUSH        | sakilacustomer.org |
| RENE.MCALISTER        | sakilacustomer.org |
| EDUARDO.HIATT         | sakilacustomer.org |
| TERRENCE.GUNDERSON    | sakilacustomer.org |
| ENRIQUE.FORSYTHE      | sakilacustomer.org |
| FREDDIE.DUGGAN        | sakilacustomer.org |
| WADE.DELVALLE         | sakilacustomer.org |
| AUSTIN.CINTRON        | sakilacustomer.org |
+-----------------------+--------------------+
599 rows in set (0.00 sec)

mysql>
```

---

**Задание 6***

Доработайте запрос из предыдущего задания, скорректируйте значения в новых колонках: первая буква должна быть заглавной, остальные — строчными.

![Task6.bmp](https://github.com/user-attachments/files/24423496/1_6.bmp)

![Task6_1.bmp](https://github.com/user-attachments/files/24423495/1_6_1.bmp)


```sql
mysql> SELECT CONCAT( UPPER(LEFT(SUBSTRING_INDEX(`email`, '@', 1), 1)), LOWER(SUBSTRING(SUBSTRING_INDEX(`email`, '@', 1), 2)) ) AS `before_at_sign`, CONCAT( UPPER(LEFT(SUBST
RING_INDEX(`email`, '@', -1), 1)), LOWER(SUBSTRING(SUBSTRING_INDEX(`email`, '@', -1), 2)) ) AS `after_at_sign` FROM `customer`;
+-----------------------+--------------------+
| before_at_sign        | after_at_sign      |
+-----------------------+--------------------+
| Mary.smith            | Sakilacustomer.org |
| Patricia.johnson      | Sakilacustomer.org |
| Linda.williams        | Sakilacustomer.org |
| Barbara.jones         | Sakilacustomer.org |
| Elizabeth.brown       | Sakilacustomer.org |
| Jennifer.davis        | Sakilacustomer.org |

...

| Sergio.stanfield      | Sakilacustomer.org |
| Marion.ocampo         | Sakilacustomer.org |
| Tracy.herrmann        | Sakilacustomer.org |
| Seth.hannon           | Sakilacustomer.org |
| Kent.arsenault        | Sakilacustomer.org |
| Terrance.roush        | Sakilacustomer.org |
| Rene.mcalister        | Sakilacustomer.org |
| Eduardo.hiatt         | Sakilacustomer.org |
| Terrence.gunderson    | Sakilacustomer.org |
| Enrique.forsythe      | Sakilacustomer.org |
| Freddie.duggan        | Sakilacustomer.org |
| Wade.delvalle         | Sakilacustomer.org |
| Austin.cintron        | Sakilacustomer.org |
+-----------------------+--------------------+
599 rows in set (0.00 sec)

mysql>
```
