# MySQL_One_To_Many_and_Many_To_Many

## Foreign Key
```
// Creating the customers and orders tables
mysql> CREATE TABLE customers(
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     first_name VARCHAR(100),
    ->     last_name VARCHAR(100),
    ->     email VARCHAR(100)
    -> );
Query OK, 0 rows affected (0.13 sec)

mysql> CREATE TABLE orders(
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     order_date DATE,
    ->     amount DECIMAL(8,2),
    ->     customer_id INT,
    ->     FOREIGN KEY(customer_id) REFERENCES customers(id)
    -> );
Query OK, 0 rows affected (0.15 sec)

// Inserting some customers and orders
mysql> INSERT INTO customers (first_name, last_name, email)
    -> VALUES ('Boy', 'George', 'george@gmail.com'),
    ->        ('George', 'Michael', 'gm@gmail.com'),
    ->        ('David', 'Bowie', 'david@gmail.com'),
    ->        ('Blue', 'Steele', 'blue@gmail.com'),
    ->        ('Bette', 'Davis', 'bette@aol.com');
Query OK, 5 rows affected (0.02 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql>
mysql> INSERT INTO orders (order_date, amount, customer_id)
    -> VALUES ('2016/02/10', 99.99, 1),
    ->        ('2017/11/11', 35.50, 1),
    ->        ('2014/12/12', 800.67, 2),
    ->        ('2015/01/03', 12.50, 2),
    ->        ('1999/04/11', 450.25, 5);
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

// This INSERT fails because of our fk constraint.  No user with id: 98
mysql> INSERT INTO orders (order_date, amount, customer_id)
    -> VALUES ('2016/06/06', 33.67, 98);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`ann_arbor`.`orders`,
CONSTRAINT `orders_ibfk_1` FOREIGN KEY (`customer_id`) REFERENCES `customers` (`id`))
```
## Cross Joins
```
// Finding Orders Placed By George: 2 Step Process
mysql> SELECT id FROM customers WHERE last_name='George';
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.01 sec)

mysql> SELECT * FROM orders WHERE customer_id = 1;
+----+------------+--------+-------------+
| id | order_date | amount | customer_id |
+----+------------+--------+-------------+
|  1 | 2016-02-10 |  99.99 |           1 |
|  2 | 2017-11-11 |  35.50 |           1 |
+----+------------+--------+-------------+
2 rows in set (0.01 sec)

// Finding Orders Placed By George: Using a subquery
mysql> SELECT * FROM orders WHERE customer_id =
    ->     (
    ->         SELECT id FROM customers
    ->         WHERE last_name='George'
    ->     );
+----+------------+--------+-------------+
| id | order_date | amount | customer_id |
+----+------------+--------+-------------+
|  1 | 2016-02-10 |  99.99 |           1 |
|  2 | 2017-11-11 |  35.50 |           1 |
+----+------------+--------+-------------+
2 rows in set (0.01 sec)

// Cross Join Craziness
mysql> SELECT * FROM customers, orders;
+----+------------+-----------+------------------+----+------------+--------+-------------+
| id | first_name | last_name | email            | id | order_date | amount | customer_id |
+----+------------+-----------+------------------+----+------------+--------+-------------+
|  1 | Boy        | George    | george@gmail.com |  1 | 2016-02-10 |  99.99 |           1 |
|  2 | George     | Michael   | gm@gmail.com     |  1 | 2016-02-10 |  99.99 |           1 |
|  3 | David      | Bowie     | david@gmail.com  |  1 | 2016-02-10 |  99.99 |           1 |
|  4 | Blue       | Steele    | blue@gmail.com   |  1 | 2016-02-10 |  99.99 |           1 |
|  5 | Bette      | Davis     | bette@aol.com    |  1 | 2016-02-10 |  99.99 |           1 |
|  1 | Boy        | George    | george@gmail.com |  2 | 2017-11-11 |  35.50 |           1 |
|  2 | George     | Michael   | gm@gmail.com     |  2 | 2017-11-11 |  35.50 |           1 |
|  3 | David      | Bowie     | david@gmail.com  |  2 | 2017-11-11 |  35.50 |           1 |
|  4 | Blue       | Steele    | blue@gmail.com   |  2 | 2017-11-11 |  35.50 |           1 |
|  5 | Bette      | Davis     | bette@aol.com    |  2 | 2017-11-11 |  35.50 |           1 |
|  1 | Boy        | George    | george@gmail.com |  3 | 2014-12-12 | 800.67 |           2 |
|  2 | George     | Michael   | gm@gmail.com     |  3 | 2014-12-12 | 800.67 |           2 |
|  3 | David      | Bowie     | david@gmail.com  |  3 | 2014-12-12 | 800.67 |           2 |
|  4 | Blue       | Steele    | blue@gmail.com   |  3 | 2014-12-12 | 800.67 |           2 |
|  5 | Bette      | Davis     | bette@aol.com    |  3 | 2014-12-12 | 800.67 |           2 |
|  1 | Boy        | George    | george@gmail.com |  4 | 2015-01-03 |  12.50 |           2 |
|  2 | George     | Michael   | gm@gmail.com     |  4 | 2015-01-03 |  12.50 |           2 |
|  3 | David      | Bowie     | david@gmail.com  |  4 | 2015-01-03 |  12.50 |           2 |
|  4 | Blue       | Steele    | blue@gmail.com   |  4 | 2015-01-03 |  12.50 |           2 |
|  5 | Bette      | Davis     | bette@aol.com    |  4 | 2015-01-03 |  12.50 |           2 |
|  1 | Boy        | George    | george@gmail.com |  5 | 1999-04-11 | 450.25 |           5 |
|  2 | George     | Michael   | gm@gmail.com     |  5 | 1999-04-11 | 450.25 |           5 |
|  3 | David      | Bowie     | david@gmail.com  |  5 | 1999-04-11 | 450.25 |           5 |
|  4 | Blue       | Steele    | blue@gmail.com   |  5 | 1999-04-11 | 450.25 |           5 |
|  5 | Bette      | Davis     | bette@aol.com    |  5 | 1999-04-11 | 450.25 |           5 |
+----+------------+-----------+------------------+----+------------+--------+-------------+
25 rows in set (0.00 sec)
```

## Inner Join
![inner_join](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/inner_join.png)
```
// IMPLICIT INNER JOIN
mysql> SELECT * FROM customers, orders
    -> WHERE customers.id = orders.customer_id;
+----+------------+-----------+------------------+----+------------+--------+-------------+
| id | first_name | last_name | email            | id | order_date | amount | customer_id |
+----+------------+-----------+------------------+----+------------+--------+-------------+
|  1 | Boy        | George    | george@gmail.com |  1 | 2016-02-10 |  99.99 |           1 |
|  1 | Boy        | George    | george@gmail.com |  2 | 2017-11-11 |  35.50 |           1 |
|  2 | George     | Michael   | gm@gmail.com     |  3 | 2014-12-12 | 800.67 |           2 |
|  2 | George     | Michael   | gm@gmail.com     |  4 | 2015-01-03 |  12.50 |           2 |
|  5 | Bette      | Davis     | bette@aol.com    |  5 | 1999-04-11 | 450.25 |           5 |
+----+------------+-----------+------------------+----+------------+--------+-------------+
5 rows in set (0.00 sec)

mysql> SELECT first_name, last_name, order_date, amount
    -> FROM customers, orders
    ->     WHERE customers.id = orders.customer_id;
+------------+-----------+------------+--------+
| first_name | last_name | order_date | amount |
+------------+-----------+------------+--------+
| Boy        | George    | 2016-02-10 |  99.99 |
| Boy        | George    | 2017-11-11 |  35.50 |
| George     | Michael   | 2014-12-12 | 800.67 |
| George     | Michael   | 2015-01-03 |  12.50 |
| Bette      | Davis     | 1999-04-11 | 450.25 |
+------------+-----------+------------+--------+
5 rows in set (0.00 sec)

// EXPLICIT INNER JOINS
mysql> SELECT * FROM customers
    -> JOIN orders
    ->     ON customers.id = orders.customer_id;
+----+------------+-----------+------------------+----+------------+--------+-------------+
| id | first_name | last_name | email            | id | order_date | amount | customer_id |
+----+------------+-----------+------------------+----+------------+--------+-------------+
|  1 | Boy        | George    | george@gmail.com |  1 | 2016-02-10 |  99.99 |           1 |
|  1 | Boy        | George    | george@gmail.com |  2 | 2017-11-11 |  35.50 |           1 |
|  2 | George     | Michael   | gm@gmail.com     |  3 | 2014-12-12 | 800.67 |           2 |
|  2 | George     | Michael   | gm@gmail.com     |  4 | 2015-01-03 |  12.50 |           2 |
|  5 | Bette      | Davis     | bette@aol.com    |  5 | 1999-04-11 | 450.25 |           5 |
+----+------------+-----------+------------------+----+------------+--------+-------------+
5 rows in set (0.00 sec)

mysql> SELECT first_name, last_name, order_date, amount
    -> FROM customers
    -> JOIN orders
    ->     ON customers.id = orders.customer_id;
+------------+-----------+------------+--------+
| first_name | last_name | order_date | amount |
+------------+-----------+------------+--------+
| Boy        | George    | 2016-02-10 |  99.99 |
| Boy        | George    | 2017-11-11 |  35.50 |
| George     | Michael   | 2014-12-12 | 800.67 |
| George     | Michael   | 2015-01-03 |  12.50 |
| Bette      | Davis     | 1999-04-11 | 450.25 |
+------------+-----------+------------+--------+
5 rows in set (0.00 sec)

mysql> SELECT
    ->     first_name,
    ->     last_name,
    ->     SUM(amount) AS total_spent
    -> FROM customers
    -> JOIN orders
    ->     ON customers.id = orders.customer_id
    -> GROUP BY orders.customer_id
    -> ORDER BY total_spent DESC;
+------------+-----------+-------------+
| first_name | last_name | total_spent |
+------------+-----------+-------------+
| George     | Michael   |      813.17 |
| Bette      | Davis     |      450.25 |
| Boy        | George    |      135.49 |
+------------+-----------+-------------+
3 rows in set (0.20 sec)
```
## Left Joins
![left_join](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/left_join.png)
```
mysql> SELECT * FROM customers
    -> LEFT JOIN orders
    ->     ON customers.id = orders.customer_id;
+----+------------+-----------+------------------+------+------------+--------+-------------+
| id | first_name | last_name | email            | id   | order_date | amount | customer_id |
+----+------------+-----------+------------------+------+------------+--------+-------------+
|  1 | Boy        | George    | george@gmail.com |    1 | 2016-02-10 |  99.99 |           1 |
|  1 | Boy        | George    | george@gmail.com |    2 | 2017-11-11 |  35.50 |           1 |
|  2 | George     | Michael   | gm@gmail.com     |    3 | 2014-12-12 | 800.67 |           2 |
|  2 | George     | Michael   | gm@gmail.com     |    4 | 2015-01-03 |  12.50 |           2 |
|  3 | David      | Bowie     | david@gmail.com  | NULL | NULL       |   NULL |        NULL |
|  4 | Blue       | Steele    | blue@gmail.com   | NULL | NULL       |   NULL |        NULL |
|  5 | Bette      | Davis     | bette@aol.com    |    5 | 1999-04-11 | 450.25 |           5 |
+----+------------+-----------+------------------+------+------------+--------+-------------+
7 rows in set (0.00 sec)

mysql> SELECT first_name, last_name, order_date, amount
    -> FROM customers
    -> LEFT JOIN orders
    ->     ON customers.id = orders.customer_id;
+------------+-----------+------------+--------+
| first_name | last_name | order_date | amount |
+------------+-----------+------------+--------+
| Boy        | George    | 2016-02-10 |  99.99 |
| Boy        | George    | 2017-11-11 |  35.50 |
| George     | Michael   | 2014-12-12 | 800.67 |
| George     | Michael   | 2015-01-03 |  12.50 |
| David      | Bowie     | NULL       |   NULL |
| Blue       | Steele    | NULL       |   NULL |
| Bette      | Davis     | 1999-04-11 | 450.25 |
+------------+-----------+------------+--------+
7 rows in set (0.00 sec)

mysql> SELECT
    ->     first_name,
    ->     last_name,
    ->     IFNULL(SUM(amount), 0) AS total_spent
    -> FROM customers
    -> LEFT JOIN orders
    ->     ON customers.id = orders.customer_id
    -> GROUP BY customers.id
    -> ORDER BY total_spent;
+------------+-----------+-------------+
| first_name | last_name | total_spent |
+------------+-----------+-------------+
| Blue       | Steele    |        0.00 |
| David      | Bowie     |        0.00 |
| Boy        | George    |      135.49 |
| Bette      | Davis     |      450.25 |
| George     | Michael   |      813.17 |
+------------+-----------+-------------+
5 rows in set (0.00 sec)
```
## Right Joins
![right_join](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/right_join.png)
```
mysql> SELECT * FROM customers
    -> LEFT JOIN orders
    ->     ON customers.id = orders.customer_id;
+----+------------+-----------+------------------+------+------------+--------+-------------+
| id | first_name | last_name | email            | id   | order_date | amount | customer_id |
+----+------------+-----------+------------------+------+------------+--------+-------------+
|  1 | Boy        | George    | george@gmail.com |    1 | 2016-02-10 |  99.99 |           1 |
|  1 | Boy        | George    | george@gmail.com |    2 | 2017-11-11 |  35.50 |           1 |
|  2 | George     | Michael   | gm@gmail.com     |    3 | 2014-12-12 | 800.67 |           2 |
|  2 | George     | Michael   | gm@gmail.com     |    4 | 2015-01-03 |  12.50 |           2 |
|  3 | David      | Bowie     | david@gmail.com  | NULL | NULL       |   NULL |        NULL |
|  4 | Blue       | Steele    | blue@gmail.com   | NULL | NULL       |   NULL |        NULL |
|  5 | Bette      | Davis     | bette@aol.com    |    5 | 1999-04-11 | 450.25 |           5 |
+----+------------+-----------+------------------+------+------------+--------+-------------+
7 rows in set (0.07 sec)
mysql> SELECT * FROM orders
    -> RIGHT JOIN customers
    ->     ON customers.id = orders.customer_id;
+------+------------+--------+-------------+----+------------+-----------+------------------+
| id   | order_date | amount | customer_id | id | first_name | last_name | email            |
+------+------------+--------+-------------+----+------------+-----------+------------------+
|    1 | 2016-02-10 |  99.99 |           1 |  1 | Boy        | George    | george@gmail.com |
|    2 | 2017-11-11 |  35.50 |           1 |  1 | Boy        | George    | george@gmail.com |
|    3 | 2014-12-12 | 800.67 |           2 |  2 | George     | Michael   | gm@gmail.com     |
|    4 | 2015-01-03 |  12.50 |           2 |  2 | George     | Michael   | gm@gmail.com     |
| NULL | NULL       |   NULL |        NULL |  3 | David      | Bowie     | david@gmail.com  |
| NULL | NULL       |   NULL |        NULL |  4 | Blue       | Steele    | blue@gmail.com   |
|    5 | 1999-04-11 | 450.25 |           5 |  5 | Bette      | Davis     | bette@aol.com    |
+------+------------+--------+-------------+----+------------+-----------+------------------+
7 rows in set (0.00 sec)
```
## Joins Challenge (1)
### Write The Schema and Inset data
![joins_challenge_1](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/joins_challenge_1.png)
```
mysql> CREATE TABLE students (
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     first_name VARCHAR(100)
    -> );
Query OK, 0 rows affected (0.22 sec)

mysql>
mysql>
mysql> CREATE TABLE papers (
    ->     title VARCHAR(100),
    ->     grade INT,
    ->     student_id INT,
    ->     FOREIGN KEY (student_id)
    ->         REFERENCES students(id)
    ->         ON DELETE CASCADE
    -> );
Query OK, 0 rows affected (0.06 sec)
mysql> INSERT INTO students (first_name) VALUES
    -> ('Caleb'),
    -> ('Samantha'),
    -> ('Raj'),
    -> ('Carlos'),
    -> ('Lisa');
Query OK, 5 rows affected (0.06 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql>
mysql> INSERT INTO papers (student_id, title, grade ) VALUES
    -> (1, 'My First Book Report', 60),
    -> (1, 'My Second Book Report', 75),
    -> (2, 'Russian Lit Through The Ages', 94),
    -> (2, 'De Montaigne and The Art of The Essay', 98),
    -> (4, 'Borges and Magical Realism', 89);
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0
```
## Joins Challenge (2)
![joins_challenge_2](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/joins_challenge_2.png)
```
mysql> SELECT first_name, title, grade
    -> FROM students
    -> INNER JOIN papers
    ->     ON students.id = papers.student_id
    -> ORDER BY grade DESC;
+------------+---------------------------------------+-------+
| first_name | title                                 | grade |
+------------+---------------------------------------+-------+
| Samantha   | De Montaigne and The Art of The Essay |    98 |
| Samantha   | Russian Lit Through The Ages          |    94 |
| Carlos     | Borges and Magical Realism            |    89 |
| Caleb      | My Second Book Report                 |    75 |
| Caleb      | My First Book Report                  |    60 |
+------------+---------------------------------------+-------+
5 rows in set (0.02 sec)
mysql> SELECT first_name, title, grade
    -> FROM students
    -> RIGHT JOIN papers
    ->     ON students.id = papers.student_id
    -> ORDER BY grade DESC;
+------------+---------------------------------------+-------+
| first_name | title                                 | grade |
+------------+---------------------------------------+-------+
| Samantha   | De Montaigne and The Art of The Essay |    98 |
| Samantha   | Russian Lit Through The Ages          |    94 |
| Carlos     | Borges and Magical Realism            |    89 |
| Caleb      | My Second Book Report                 |    75 |
| Caleb      | My First Book Report                  |    60 |
+------------+---------------------------------------+-------+
5 rows in set (0.00 sec)
```

## Joins Challenge (3)
![joins_challenge_3](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/joins_challenge_3.png)
```
mysql> SELECT first_name, title, grade
    -> FROM students
    -> LEFT JOIN papers
    ->     ON students.id = papers.student_id;
+------------+---------------------------------------+-------+
| first_name | title                                 | grade |
+------------+---------------------------------------+-------+
| Caleb      | My First Book Report                  |    60 |
| Caleb      | My Second Book Report                 |    75 |
| Samantha   | Russian Lit Through The Ages          |    94 |
| Samantha   | De Montaigne and The Art of The Essay |    98 |
| Raj        | NULL                                  |  NULL |
| Carlos     | Borges and Magical Realism            |    89 |
| Lisa       | NULL                                  |  NULL |
+------------+---------------------------------------+-------+
7 rows in set (0.00 sec)
```

## Joins Challenge (4)
![joins_challenge_4](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/joins_challenge_4.png)
```
mysql> SELECT
    ->     first_name,
    ->     IFNULL(title, 'MISSING'),
    ->     IFNULL(grade, 0)
    -> FROM students
    -> LEFT JOIN papers
    ->     ON students.id = papers.student_id;
+------------+---------------------------------------+------------------+
| first_name | IFNULL(title, 'MISSING')              | IFNULL(grade, 0) |
+------------+---------------------------------------+------------------+
| Caleb      | My First Book Report                  |               60 |
| Caleb      | My Second Book Report                 |               75 |
| Samantha   | Russian Lit Through The Ages          |               94 |
| Samantha   | De Montaigne and The Art of The Essay |               98 |
| Raj        | MISSING                               |                0 |
| Carlos     | Borges and Magical Realism            |               89 |
| Lisa       | MISSING                               |                0 |
+------------+---------------------------------------+------------------+
7 rows in set (0.00 sec)

```

## Joins Challenge (5)
![joins_challenge_5](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/joins_challenge_5.png)
```
mysql> SELECT
    ->     first_name,
    ->     IFNULL(AVG(grade), 0) AS average
    -> FROM students
    -> LEFT JOIN papers
    ->     ON students.id = papers.student_id
    -> GROUP BY students.id
    -> ORDER BY average DESC;
+------------+---------+
| first_name | average |
+------------+---------+
| Samantha   | 96.0000 |
| Carlos     | 89.0000 |
| Caleb      | 67.5000 |
| Lisa       |  0.0000 |
| Raj        |  0.0000 |
+------------+---------+
5 rows in set (0.00 sec)

```
## Joins Challenge (6)
![joins_challenge_6](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/joins_challenge_6.png)
```
mysql> SELECT
    ->   first_name,
    ->   Ifnull(AVG(grade), 0) AS average,
    ->   CASE
    ->     WHEN AVG(grade) IS NULL THEN 'FAILING'
    ->     WHEN AVG(grade) >= 75 THEN 'PASSING'
    ->     ELSE 'FAILING'
    ->   END AS passing_status
    -> FROM students
    -> LEFT JOIN papers
    ->   ON students.id = papers.student_id
    -> GROUP BY students.id
    -> ORDER BY average DESC;
+------------+---------+----------------+
| first_name | average | passing_status |
+------------+---------+----------------+
| Samantha   | 96.0000 | PASSING        |
| Carlos     | 89.0000 | PASSING        |
| Caleb      | 67.5000 | FAILING        |
| Raj        |  0.0000 | FAILING        |
| Lisa       |  0.0000 | FAILING        |
+------------+---------+----------------+
5 rows in set (0.00 sec)
```

