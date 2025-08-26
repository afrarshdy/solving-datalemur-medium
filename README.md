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

Output as expected:
|  user_id  |  spend  |  transaction_date  |
|-----------|---------|--------------------|
|111        |89.60    |02/05/2022 12:00:00  |
|121        |67.90    |04/03/2022 12:00:00  |
|263        |100.00    |07/12/2022 12:00:00  |
