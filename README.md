# DataLemur Medium Difficulty Problem Solving

## 1. User's Third Transaction
Assume you are given the table below on Uber transactions made by users. Write a query to obtain the third transaction of every user. Output the user id, spend, and transaction date.

`transactions` table:
|  Column Name       |  Type      |
|--------------------|------------|
|  user_id           |  integer    |
|  spend	           |  decimal    |
|  transaction_date  |  timestamp  |

`transactions` example output:
|user_id |spend   |transaction_date         |
|--------|--------|-------------------------|
|111     |100.50  |01/08/2022 12:00:00      |
|111     |55.00   |01/10/2022 12:00:00      |
|121     |36.00   |01/18/2022 12:00:00      |
|145     |24.99   |01/26/2022 12:00:00      |
|111     |89.60   |02/05/2022 12:00:00      |

SQL query:
 
    SELECT
      user_id,
      spend,
      transaction_date
    FROM
      (SELECT
        ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY transaction_date) AS no,
        user_id,
        spend,
        transaction_date
      FROM transactions) a
    WHERE a.no = 3;

Output:
|  user_id  |  spend  |  transaction_date  |
|-----------|---------|--------------------|
|111        |89.60    |02/05/2022 12:00:00  |
|121        |67.90    |04/03/2022 12:00:00  |
|263        |100.00    |07/12/2022 12:00:00  |

## 2. Second Highest Salary
Imagine you're an HR analyst at a tech company tasked with analyzing employee salaries. Your manager is keen on understanding the pay distribution and asks you to determine the second highest salary among all employees.

It's possible that multiple employees may share the same second highest salary. In case of duplicate, display the salary only once.

`employee` table:
|column_name   |type       |description                       |
|--------------|-----------|----------------------------------|
|employee_id   |integer    |The unique ID of the employee.    |
|name          |string     |The name of the employee.         |
|salary        |integer    |The salary of the employee.       |
|department_id |integer    |The department ID of the employee.|
|manager_id    |integer    |The manager ID of the employee.   |

`employee` example output:
|employee_id |name             |salary      |department_id |manager_id |
|------------|-----------------|------------|--------------|-----------|
|1           |Emma Thompson    |3800        |1             |6          |
|2           |Daniel Rodriguez	|2230        |1             |7          |
|3           |Olivia Smith     |2000        |1             |8          |

Query:

     SELECT
       salary AS second_highest_salary
     FROM
       (SELECT
         RANK() OVER(ORDER BY salary DESC) AS rank,
         salary
       FROM employee) a
     WHERE a.rank = 2;

Output:

|second_highest_salary|
|---------------------|
|12500                |

## 3. Sending vs. Opening Snaps
Assume you're given tables with information on Snapchat users, including their ages and time spent sending and opening snaps.

Write a query to obtain a breakdown of the time spent sending vs. opening snaps as a percentage of total time spent on these activities grouped by age group. Round the percentage to 2 decimal places in the output.

Notes:

Calculate the following percentages:
- time spent sending / (Time spent sending + Time spent opening)
- Time spent opening / (Time spent sending + Time spent opening)
- To avoid integer division in percentages, multiply by 100.0 and not 100.

`activities` table:
|Column Name        |Type                          |
|-------------------|------------------------------|
|activity_id        |integer                       |
|user_id            |integer                       |
|activity_type      |string ('send', 'open', 'chat')|
|time_spent         |float                         |
|activity_date      |datetime                      |

`age_breakdown` table:
|Column Name   |Type                               |
|--------------|-----------------------------------|
|user_id       |integer                            |
|age_bucket    |string ('21-25', '26-30', '31-25') |

Query:

    SELECT
      age_bucket,
      ROUND((sum_send / total * 100), 2) AS send_perc,
      ROUND((sum_open / total * 100), 2) AS open_perc
    FROM
      (SELECT
        tbl_send.age_bucket,
        tbl_send.sum_send,
        tbl_open.sum_open,
        (tbl_send.sum_send + tbl_open.sum_open) AS total
      FROM
        (SELECT *
        FROM
          (SELECT
            age_bucket,
            activity_type,
            SUM(time_spent) AS sum_send
          FROM
            (SELECT
              a.*,
              ab.age_bucket
            FROM activities AS a
            LEFT JOIN age_breakdown AS ab 
            ON a.user_id = ab.user_id) AS tbl
          GROUP BY age_bucket, activity_type
          ORDER BY age_bucket, activity_type) AS tbl2
        WHERE activity_type = 'send') AS tbl_send,
        (SELECT *
        FROM
          (SELECT
            age_bucket,
            activity_type,
            SUM(time_spent) AS sum_open
          FROM
            (SELECT
              a.*,
              ab.age_bucket
            FROM activities AS a
            LEFT JOIN age_breakdown AS ab 
            ON a.user_id = ab.user_id) AS tbl
          GROUP BY age_bucket, activity_type
          ORDER BY age_bucket, activity_type) AS tbl2
        WHERE activity_type = 'open') AS tbl_open
      WHERE tbl_send.age_bucket = tbl_open.age_bucket) AS tbl3;

Output:

|age_bucket |send_perc  |open_perc  |
|-----------|-----------|-----------|
|21-25      |54.31      |45.69      |
|26-30      |82.26      |17.74      |
|31-35      |37.84      |62.16      |

## 4. Highest-Grossing Items
Assume you're given a table containing data on Amazon customers and their spending on products in different category, write a query to identify the top two highest-grossing products within each category in the year 2022. The output should include the category, product, and total spend.

`product_spend` table:

|Column Name  |Type     |
|-------------|---------|
|category     |string   |
|product      |string   |
|user_id      |integer  |
|spend        |decimal  |
|transaction_date|timestamp|

Query:

    SELECT
      category,
      product,
      total_spend
    FROM
      (SELECT
        category,
        product,
        SUM(spend) AS total_spend,
        RANK() 
          OVER(PARTITION BY category ORDER BY SUM(spend) DESC) AS rank
      FROM product_spend
      WHERE EXTRACT(YEAR FROM transaction_date) = 2022
      GROUP BY category, product
      ORDER BY category, rank) AS temp
    WHERE rank BETWEEN 1 AND 2;

Output:

|category   |product          |total_spend   |
|-----------|-----------------|--------------|
|appliance  |washing machine  |439.80        |
|appliance  |refrigerator     |299.99        |
|electronics|vacuum           |486.66        |
|electronics|wireless headset |467.89        |

## 5. Top Three Salaries
As part of an ongoing analysis of salary distribution within the company, your manager has requested a report identifying high earners in each department. A 'high earner' within a department is defined as an employee with a salary ranking among the top three salaries within that department.

You're tasked with identifying these high earners across all departments. Write a query to display the employee's name along with their department name and salary. In case of duplicates, sort the results of department name in ascending order, then by salary in descending order. If multiple employees have the same salary, then order them alphabetically.

Note: Ensure to utilize the appropriate ranking window function to handle duplicate salaries effectively.

`employee` schema:

|column_name   |type         |description                            |
|--------------|-------------|---------------------------------------|
|employee_id   |integer      |The unique ID of the employee.         |
|name          |string       |The name of the employee.              |
|salary        |integer      |The salary of the employee.            |
|department_id |integer      |The department ID of the employee.     |
|manager_id    |integer      |The manager ID of the employee.        |

`department` schema:

|column_name    |type      |description                      |
|---------------|----------|---------------------------------|
|department_id  |integer   |The department ID of the employee.|
|department_name|string    |The name of the department.      |

Query:

    SELECT
      department_name,
      name,
      salary
    FROM
      (SELECT
        e.*,
        d.*,
        DENSE_RANK()
          OVER(PARTITION BY d.department_name ORDER BY e.salary DESC) AS rank
      FROM employee AS e
      LEFT JOIN department AS d
      ON e.department_id = d.department_id
      ORDER BY d.department_name, e.salary DESC, e.name) AS temp
    WHERE rank BETWEEN 1 AND 3;

 Output (first 5 row):

|department_name     |name            |salary     |
|--------------------|----------------|-----------|
|Data Analytics      |Olivia Smith    |7000       |
|Data Analytics      |Amelia Lee      |4000       |
|Data Analytics      |James Anderson  |4000       |
|Data Analytics      |Emma Thompson   |3800       |
|Data Engineering    |Liam Brown      |13000      |
