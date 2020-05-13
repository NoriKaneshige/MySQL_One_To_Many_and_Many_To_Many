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
![joins_challenge_6](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/joins_challenge_6_edit.png)
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
# Many To Many
![many_to_many_ex](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/many_to_many_ex.png)
## TV Joins Challenge
![tables_overview](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/tables_overview.png)
## Creating Tables
```
// CREATING THE REVIEWERS TABLE
Database changed
mysql> CREATE TABLE reviewers (
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     first_name VARCHAR(100),
    ->     last_name VARCHAR(100)
    -> );
Query OK, 0 rows affected (0.05 sec)

//　CREATING THE SERIES TABLE
mysql> CREATE TABLE series(
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     title VARCHAR(100),
    ->     released_year YEAR(4),
    ->     genre VARCHAR(100)
    -> );
Query OK, 0 rows affected (0.22 sec)

// CREATING THE REVIEWS TABLE
mysql> CREATE TABLE reviews (
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     rating DECIMAL(2,1),
    ->     series_id INT,
    ->     reviewer_id INT,
    ->     FOREIGN KEY(series_id) REFERENCES series(id),
    ->     FOREIGN KEY(reviewer_id) REFERENCES reviewers(id)
    -> );
Query OK, 0 rows affected (0.10 sec)

// INSERTING A BUNCH OF DATA
mysql> INSERT INTO series (title, released_year, genre) VALUES
    ->     ('Archer', 2009, 'Animation'),
    ->     ('Arrested Development', 2003, 'Comedy'),
    ->     ("Bob's Burgers", 2011, 'Animation'),
    ->     ('Bojack Horseman', 2014, 'Animation'),
    ->     ("Breaking Bad", 2008, 'Drama'),
    ->     ('Curb Your Enthusiasm', 2000, 'Comedy'),
    ->     ("Fargo", 2014, 'Drama'),
    ->     ('Freaks and Geeks', 1999, 'Comedy'),
    ->     ('General Hospital', 1963, 'Drama'),
    ->     ('Halt and Catch Fire', 2014, 'Drama'),
    ->     ('Malcolm In The Middle', 2000, 'Comedy'),
    ->     ('Pushing Daisies', 2007, 'Comedy'),
    ->     ('Seinfeld', 1989, 'Comedy'),
    ->     ('Stranger Things', 2016, 'Drama');
Query OK, 14 rows affected (0.02 sec)
Records: 14  Duplicates: 0  Warnings: 0

mysql> INSERT INTO reviewers (first_name, last_name) VALUES
    ->     ('Thomas', 'Stoneman'),
    ->     ('Wyatt', 'Skaggs'),
    ->     ('Kimbra', 'Masters'),
    ->     ('Domingo', 'Cortes'),
    ->     ('Colt', 'Steele'),
    ->     ('Pinkie', 'Petit'),
    ->     ('Marlon', 'Crafford');
Query OK, 7 rows affected (0.07 sec)
Records: 7  Duplicates: 0  Warnings: 0

mysql>     INSERT INTO reviews(series_id, reviewer_id, rating) VALUES
    ->     (1,1,8.0),(1,2,7.5),(1,3,8.5),(1,4,7.7),(1,5,8.9),
    ->     (2,1,8.1),(2,4,6.0),(2,3,8.0),(2,6,8.4),(2,5,9.9),
    ->     (3,1,7.0),(3,6,7.5),(3,4,8.0),(3,3,7.1),(3,5,8.0),
    ->     (4,1,7.5),(4,3,7.8),(4,4,8.3),(4,2,7.6),(4,5,8.5),
    ->     (5,1,9.5),(5,3,9.0),(5,4,9.1),(5,2,9.3),(5,5,9.9),
    ->     (6,2,6.5),(6,3,7.8),(6,4,8.8),(6,2,8.4),(6,5,9.1),
    ->     (7,2,9.1),(7,5,9.7),
    ->     (8,4,8.5),(8,2,7.8),(8,6,8.8),(8,5,9.3),
    ->     (9,2,5.5),(9,3,6.8),(9,4,5.8),(9,6,4.3),(9,5,4.5),
    ->     (10,5,9.9),
    ->     (13,3,8.0),(13,4,7.2),
    ->     (14,2,8.5),(14,3,8.9),(14,4,8.9);
Query OK, 47 rows affected (0.09 sec)
Records: 47  Duplicates: 0  Warnings: 0
```
## TV Joins Challenge (1)
![tv_joins_challenge_1](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/tv_joins_challenge_1.png)
```
mysql> SELECT
    ->     title,
    ->     rating
    -> FROM series
    -> JOIN reviews
    ->     ON series.id = reviews.series_id;
+----------------------+--------+
| title                | rating |
+----------------------+--------+
| Archer               |    8.0 |
| Archer               |    7.5 |
| Archer               |    8.5 |
| Archer               |    7.7 |
| Archer               |    8.9 |
| Arrested Development |    8.1 |
| Arrested Development |    6.0 |
| Arrested Development |    8.0 |
| Arrested Development |    8.4 |
| Arrested Development |    9.9 |
| Bob's Burgers        |    7.0 |
| Bob's Burgers        |    7.5 |
| Bob's Burgers        |    8.0 |
| Bob's Burgers        |    7.1 |
| Bob's Burgers        |    8.0 |
| Bojack Horseman      |    7.5 |
| Bojack Horseman      |    7.8 |
| Bojack Horseman      |    8.3 |
| Bojack Horseman      |    7.6 |
| Bojack Horseman      |    8.5 |
| Breaking Bad         |    9.5 |
| Breaking Bad         |    9.0 |
| Breaking Bad         |    9.1 |
| Breaking Bad         |    9.3 |
| Breaking Bad         |    9.9 |
| Curb Your Enthusiasm |    6.5 |
| Curb Your Enthusiasm |    7.8 |
| Curb Your Enthusiasm |    8.8 |
| Curb Your Enthusiasm |    8.4 |
| Curb Your Enthusiasm |    9.1 |
| Fargo                |    9.1 |
| Fargo                |    9.7 |
| Freaks and Geeks     |    8.5 |
| Freaks and Geeks     |    7.8 |
| Freaks and Geeks     |    8.8 |
| Freaks and Geeks     |    9.3 |
| General Hospital     |    5.5 |
| General Hospital     |    6.8 |
| General Hospital     |    5.8 |
| General Hospital     |    4.3 |
| General Hospital     |    4.5 |
| Halt and Catch Fire  |    9.9 |
| Seinfeld             |    8.0 |
| Seinfeld             |    7.2 |
| Stranger Things      |    8.5 |
| Stranger Things      |    8.9 |
| Stranger Things      |    8.9 |
+----------------------+--------+
47 rows in set (0.03 sec)
```
## TV Joins Challenge (2), Avg Rating
![tv_joins_challenge_2](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/tv_joins_challenge_2.png)
```
mysql> SELECT
    ->     title,
    ->     AVG(rating) as avg_rating
    -> FROM series
    -> JOIN reviews
    ->     ON series.id = reviews.series_id
    -> GROUP BY series.id
    -> ORDER BY avg_rating;
+----------------------+------------+
| title                | avg_rating |
+----------------------+------------+
| General Hospital     |    5.38000 |
| Bob's Burgers        |    7.52000 |
| Seinfeld             |    7.60000 |
| Bojack Horseman      |    7.94000 |
| Arrested Development |    8.08000 |
| Curb Your Enthusiasm |    8.12000 |
| Archer               |    8.12000 |
| Freaks and Geeks     |    8.60000 |
| Stranger Things      |    8.76667 |
| Breaking Bad         |    9.36000 |
| Fargo                |    9.40000 |
| Halt and Catch Fire  |    9.90000 |
+----------------------+------------+
12 rows in set (0.00 sec)
```
## TV Joins Challenge (3)
![tv_joins_challenge_3](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/tv_joins_challenge_3.png)
```
mysql> SELECT
    ->     first_name,
    ->     last_name,
    ->     rating
    -> FROM reviewers
    -> INNER JOIN reviews
    ->     ON reviewers.id = reviews.reviewer_id;
+------------+-----------+--------+
| first_name | last_name | rating |
+------------+-----------+--------+
| Thomas     | Stoneman  |    8.0 |
| Thomas     | Stoneman  |    8.1 |
| Thomas     | Stoneman  |    7.0 |
| Thomas     | Stoneman  |    7.5 |
| Thomas     | Stoneman  |    9.5 |
| Wyatt      | Skaggs    |    7.5 |
| Wyatt      | Skaggs    |    7.6 |
| Wyatt      | Skaggs    |    9.3 |
| Wyatt      | Skaggs    |    6.5 |
| Wyatt      | Skaggs    |    8.4 |
| Wyatt      | Skaggs    |    9.1 |
| Wyatt      | Skaggs    |    7.8 |
| Wyatt      | Skaggs    |    5.5 |
| Wyatt      | Skaggs    |    8.5 |
| Kimbra     | Masters   |    8.5 |
| Kimbra     | Masters   |    8.0 |
| Kimbra     | Masters   |    7.1 |
| Kimbra     | Masters   |    7.8 |
| Kimbra     | Masters   |    9.0 |
| Kimbra     | Masters   |    7.8 |
| Kimbra     | Masters   |    6.8 |
| Kimbra     | Masters   |    8.0 |
| Kimbra     | Masters   |    8.9 |
| Domingo    | Cortes    |    7.7 |
| Domingo    | Cortes    |    6.0 |
| Domingo    | Cortes    |    8.0 |
| Domingo    | Cortes    |    8.3 |
| Domingo    | Cortes    |    9.1 |
| Domingo    | Cortes    |    8.8 |
| Domingo    | Cortes    |    8.5 |
| Domingo    | Cortes    |    5.8 |
| Domingo    | Cortes    |    7.2 |
| Domingo    | Cortes    |    8.9 |
| Colt       | Steele    |    8.9 |
| Colt       | Steele    |    9.9 |
| Colt       | Steele    |    8.0 |
| Colt       | Steele    |    8.5 |
| Colt       | Steele    |    9.9 |
| Colt       | Steele    |    9.1 |
| Colt       | Steele    |    9.7 |
| Colt       | Steele    |    9.3 |
| Colt       | Steele    |    4.5 |
| Colt       | Steele    |    9.9 |
| Pinkie     | Petit     |    8.4 |
| Pinkie     | Petit     |    7.5 |
| Pinkie     | Petit     |    8.8 |
| Pinkie     | Petit     |    4.3 |
+------------+-----------+--------+
47 rows in set (0.01 sec)
```
## TV Joins Challenge (4), UNREVIEWED SERIES
![tv_joins_challenge_4](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/tv_joins_challenge_4.png)
```
mysql> SELECT title AS unreviewed_series
    -> FROM series
    -> LEFT JOIN reviews
    ->     ON series.id = reviews.series_id
    -> WHERE rating IS NULL;
+-----------------------+
| unreviewed_series     |
+-----------------------+
| Malcolm In The Middle |
| Pushing Daisies       |
+-----------------------+
2 rows in set (0.03 sec)
```
## TV Joins Challenge (5), GENRE AVG RATINGS
![tv_joins_challenge_5](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/tv_joins_challenge_5.png)
```
mysql> SELECT genre,
    ->        Round(Avg(rating), 2) AS avg_rating
    -> FROM   series
    ->        INNER JOIN reviews
    ->                ON series.id = reviews.series_id
    -> GROUP  BY genre;
+-----------+------------+
| genre     | avg_rating |
+-----------+------------+
| Animation |       7.86 |
| Comedy    |       8.16 |
| Drama     |       8.04 |
+-----------+------------+
3 rows in set (0.04 sec)
```
## TV Joins Challenge (6), Reviewer Stats 
![tv_joins_challenge_6](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/tv_joins_challenge_6.png)
```
mysql> SELECT first_name,
    ->        last_name,
    ->        Count(rating)                               AS COUNT,
    ->        Ifnull(Min(rating), 0)                      AS MIN,
    ->        Ifnull(Max(rating), 0)                      AS MAX,
    ->        Round(Ifnull(Avg(rating), 0), 2)            AS AVG,
    ->        IF(Count(rating) > 0, 'ACTIVE', 'INACTIVE') AS STATUS
    -> FROM   reviewers
    ->        LEFT JOIN reviews
    ->               ON reviewers.id = reviews.reviewer_id
    -> GROUP  BY reviewers.id;
+------------+-----------+-------+-----+-----+------+----------+
| first_name | last_name | COUNT | MIN | MAX | AVG  | STATUS   |
+------------+-----------+-------+-----+-----+------+----------+
| Thomas     | Stoneman  |     5 | 7.0 | 9.5 | 8.02 | ACTIVE   |
| Wyatt      | Skaggs    |     9 | 5.5 | 9.3 | 7.80 | ACTIVE   |
| Kimbra     | Masters   |     9 | 6.8 | 9.0 | 7.99 | ACTIVE   |
| Domingo    | Cortes    |    10 | 5.8 | 9.1 | 7.83 | ACTIVE   |
| Colt       | Steele    |    10 | 4.5 | 9.9 | 8.77 | ACTIVE   |
| Pinkie     | Petit     |     4 | 4.3 | 8.8 | 7.25 | ACTIVE   |
| Marlon     | Crafford  |     0 | 0.0 | 0.0 | 0.00 | INACTIVE |
+------------+-----------+-------+-----+-----+------+----------+
7 rows in set (0.06 sec)

mysql> SELECT
    ->   first_name,
    ->   last_name,
    ->   COUNT(rating) AS COUNT,
    ->   Ifnull(MIN(rating), 0) AS MIN,
    ->   Ifnull(MAX(rating), 0) AS MAX,
    ->   ROUND(Ifnull(AVG(rating), 0), 2) AS AVG,
    ->   CASE
    ->     WHEN COUNT(rating) >= 10 THEN 'POWER USER'
    ->     WHEN COUNT(rating) > 0 THEN 'ACTIVE'
    ->     ELSE 'INACTIVE'
    ->   END AS STATUS
    -> FROM reviewers
    -> LEFT JOIN reviews
    ->   ON reviewers.id = reviews.reviewer_id
    -> GROUP BY reviewers.id;
+------------+-----------+-------+-----+-----+------+------------+
| first_name | last_name | COUNT | MIN | MAX | AVG  | STATUS     |
+------------+-----------+-------+-----+-----+------+------------+
| Thomas     | Stoneman  |     5 | 7.0 | 9.5 | 8.02 | ACTIVE     |
| Wyatt      | Skaggs    |     9 | 5.5 | 9.3 | 7.80 | ACTIVE     |
| Kimbra     | Masters   |     9 | 6.8 | 9.0 | 7.99 | ACTIVE     |
| Domingo    | Cortes    |    10 | 5.8 | 9.1 | 7.83 | POWER USER |
| Colt       | Steele    |    10 | 4.5 | 9.9 | 8.77 | POWER USER |
| Pinkie     | Petit     |     4 | 4.3 | 8.8 | 7.25 | ACTIVE     |
| Marlon     | Crafford  |     0 | 0.0 | 0.0 | 0.00 | INACTIVE   |
+------------+-----------+-------+-----+-----+------+------------+
7 rows in set (0.01 sec)
```
## TV Joins Challenge (7), JOINING 3 TABLES!
![tv_joins_challenge_7](https://github.com/NoriKaneshige/MySQL_One_To_Many_and_Many_To_Many/blob/master/tv_joins_challenge_7.png)
```
mysql> SELECT
    ->     title,
    ->     rating,
    ->     CONCAT(first_name,' ', last_name) AS reviewer
    -> FROM reviewers
    -> INNER JOIN reviews
    ->     ON reviewers.id = reviews.reviewer_id
    -> INNER JOIN series
    ->     ON series.id = reviews.series_id
    -> ORDER BY title;
+----------------------+--------+-----------------+
| title                | rating | reviewer        |
+----------------------+--------+-----------------+
| Archer               |    8.0 | Thomas Stoneman |
| Archer               |    7.7 | Domingo Cortes  |
| Archer               |    8.5 | Kimbra Masters  |
| Archer               |    7.5 | Wyatt Skaggs    |
| Archer               |    8.9 | Colt Steele     |
| Arrested Development |    8.1 | Thomas Stoneman |
| Arrested Development |    6.0 | Domingo Cortes  |
| Arrested Development |    8.0 | Kimbra Masters  |
| Arrested Development |    8.4 | Pinkie Petit    |
| Arrested Development |    9.9 | Colt Steele     |
| Bob's Burgers        |    7.0 | Thomas Stoneman |
| Bob's Burgers        |    8.0 | Domingo Cortes  |
| Bob's Burgers        |    7.1 | Kimbra Masters  |
| Bob's Burgers        |    7.5 | Pinkie Petit    |
| Bob's Burgers        |    8.0 | Colt Steele     |
| Bojack Horseman      |    7.6 | Wyatt Skaggs    |
| Bojack Horseman      |    7.5 | Thomas Stoneman |
| Bojack Horseman      |    8.3 | Domingo Cortes  |
| Bojack Horseman      |    7.8 | Kimbra Masters  |
| Bojack Horseman      |    8.5 | Colt Steele     |
| Breaking Bad         |    9.9 | Colt Steele     |
| Breaking Bad         |    9.3 | Wyatt Skaggs    |
| Breaking Bad         |    9.5 | Thomas Stoneman |
| Breaking Bad         |    9.1 | Domingo Cortes  |
| Breaking Bad         |    9.0 | Kimbra Masters  |
| Curb Your Enthusiasm |    8.8 | Domingo Cortes  |
| Curb Your Enthusiasm |    7.8 | Kimbra Masters  |
| Curb Your Enthusiasm |    9.1 | Colt Steele     |
| Curb Your Enthusiasm |    6.5 | Wyatt Skaggs    |
| Curb Your Enthusiasm |    8.4 | Wyatt Skaggs    |
| Fargo                |    9.1 | Wyatt Skaggs    |
| Fargo                |    9.7 | Colt Steele     |
| Freaks and Geeks     |    8.5 | Domingo Cortes  |
| Freaks and Geeks     |    7.8 | Wyatt Skaggs    |
| Freaks and Geeks     |    9.3 | Colt Steele     |
| Freaks and Geeks     |    8.8 | Pinkie Petit    |
| General Hospital     |    4.3 | Pinkie Petit    |
| General Hospital     |    6.8 | Kimbra Masters  |
| General Hospital     |    5.8 | Domingo Cortes  |
| General Hospital     |    5.5 | Wyatt Skaggs    |
| General Hospital     |    4.5 | Colt Steele     |
| Halt and Catch Fire  |    9.9 | Colt Steele     |
| Seinfeld             |    8.0 | Kimbra Masters  |
| Seinfeld             |    7.2 | Domingo Cortes  |
| Stranger Things      |    8.9 | Domingo Cortes  |
| Stranger Things      |    8.9 | Kimbra Masters  |
| Stranger Things      |    8.5 | Wyatt Skaggs    |
+----------------------+--------+-----------------+
47 rows in set (0.02 sec)
```
