**Задание 1**

Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию:

фамилия и имя сотрудника из этого магазина;
город нахождения магазина;
количество пользователей, закреплённых в этом магазине.

![Task2_1.bmp](https://github.com/user-attachments/files/24423638/2_1.bmp)



```sql
mysql> SELECT
    ->     `s`.`last_name` AS 'staff_second_name',
    ->     `s`.`first_name` AS 'staff_first_name',
    ->     `c`.`city` AS 'store_city',
    ->     COUNT(`cust`.`customer_id`) AS 'customer_count'
    -> FROM
    ->     `store` `st`
    ->     JOIN `staff` `s` ON `st`.`manager_staff_id` = `s`.`staff_id`
    ->     JOIN `address` `a` ON `st`.`address_id` = `a`.`address_id`
    ->     JOIN `city` `c` ON `a`.`city_id` = `c`.`city_id`
    ->     JOIN `customer` `cust` ON `st`.`store_id` = `cust`.`store_id`
    -> GROUP BY
    ->     `st`.`store_id`, `s`.`last_name`, `s`.`first_name`, `c`.`city`
    -> HAVING
    ->     COUNT(`cust`.`customer_id`) > 300
    -> ORDER BY
    ->     COUNT(`cust`.`customer_id`) DESC;
+-------------------+------------------+------------+----------------+
| staff_second_name | staff_first_name | store_city | customer_count |
+-------------------+------------------+------------+----------------+
| Hillyer           | Mike             | Lethbridge |            326 |
+-------------------+------------------+------------+----------------+
1 row in set (0.00 sec)

mysql>
```

---

**Задание 2**

Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

![Task2_2.bmp](https://github.com/user-attachments/files/24423681/2_2.bmp)

```sql
mysql> SELECT COUNT(*) AS `movies_longer_than_average`
    -> FROM `film` `f1`
    -> WHERE `f1`.`length` > (
    ->     SELECT AVG(`length`)
    ->     FROM `film`
    -> );
+----------------------------+
| movies_longer_than_average |
+----------------------------+
|                        489 |
+----------------------------+
1 row in set (0.00 sec)

mysql>
```

---

**Задание 3**

Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

![Task2_3.bmp](https://github.com/user-attachments/files/24423738/2_3.bmp)

```sql
mysql> SELECT
    ->     MONTHNAME(`payment_date`) AS `month_name`,
    ->     YEAR(`payment_date`) AS `year_num`,
    ->     SUM(`amount`) AS `total_payment`,
    ->     COUNT(DISTINCT `r`.`rental_id`) AS `rental_count`
    -> FROM `payment` `p`
    -> LEFT JOIN `rental` `r` ON `p`.`rental_id` = `r`.`rental_id`
    -> GROUP BY YEAR(`payment_date`), MONTH(`payment_date`), MONTHNAME(`payment_date`)
    -> HAVING `total_payment` = (
    ->     SELECT SUM(`amount`)
    ->     FROM `payment` `p2`
    ->     GROUP BY YEAR(`p2`.`payment_date`), MONTH(`p2`.`payment_date`)
    ->     ORDER BY SUM(`amount`) DESC
    ->     LIMIT 1
    -> )
    -> ORDER BY `year_num`, MONTH(`payment_date`);
+------------+----------+---------------+--------------+
| month_name | year_num | total_payment | rental_count |
+------------+----------+---------------+--------------+
| July       |     2005 |      28368.91 |         6709 |
+------------+----------+---------------+--------------+
1 row in set (0.03 sec)

mysql>
```
