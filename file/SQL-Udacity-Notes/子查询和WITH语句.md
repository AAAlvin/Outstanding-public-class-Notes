## 子查询和WITH语句

这个MD主要是记录在优达上学习子查询和with语句时解决问题的语句，下图是EDR，根据这个解决问题：

![ERD.png](https://i.loli.net/2018/07/25/5b582587e07b7.png)

```sql
/*提供每个区域拥有最高销售额 (total_amt_usd) 的销售代表的姓名。*/
SELECT df2.region_name, df3.sale_rep, df2.total_amt
FROM 
    (SELECT region_name, MAX(total_amt) total_amt
    FROM
      (SELECT r.name region_name, SUM(o.total_amt_usd) total_amt, s.name sale_rep
      FROM region r
      JOIN sales_reps s
      ON r.id = s.region_id
      JOIN accounts a
      ON a.sales_rep_id = s.id
      JOIN orders o
      ON o.account_id = a.id
      GROUP BY 1,3) df1
    GROUP BY 1
    ORDER BY 2 DESC) df2
JOIN
    (SELECT r.name region_name, SUM(o.total_amt_usd) total_amt, s.name sale_rep
      FROM region r
      JOIN sales_reps s
      ON r.id = s.region_id
      JOIN accounts a
      ON a.sales_rep_id = s.id
      JOIN orders o
      ON o.account_id = a.id
      GROUP BY 1,3) df3
ON df2.region_name = df3.region_name AND df2.total_amt = df3.total_amt

/*提供每个区域拥有最高销售额 (total_amt_usd) 的销售代表的姓名。WITH版本*/
WITH df1 AS(
SELECT r.name region_name, SUM(o.total_amt_usd) total_amt, s.name sale_rep
FROM region r
JOIN sales_reps s
ON r.id = s.region_id
JOIN accounts a
ON a.sales_rep_id = s.id
JOIN orders o
ON o.account_id = a.id
GROUP BY 1,3),

df2 AS (
SELECT region_name, MAX(total_amt) total_amt
FROM df1
GROUP BY 1)

SELECT df2.region_name, df2.total_amt, df1.sale_rep
FROM df1
JOIN df2
ON df1.region_name = df2.region_name AND df1.total_amt = df2.total_amt
ORDER BY 2 DESC
```

```SQL
/* 对于具有最高销售额 (total_amt_usd) 的区域，总共下了多少个订单 （total count orders） ？*/
SELECT r.name region_name, SUM(o.total_amt_usd) total_amt, SUM(o.total) total_order
FROM region r
JOIN sales_reps s
ON r.id = s.region_id
JOIN accounts a
ON a.sales_rep_id = s.id
JOIN orders o
ON o.account_id = a.id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
/* 我觉得这样已经可以得到结果了，但是答案给出的子查询貌似有点复杂：*/
SELECT r.name, SUM(o.total) total_orders
FROM sales_reps s
JOIN accounts a
ON a.sales_rep_id = s.id
JOIN orders o
ON o.account_id = a.id
JOIN region r
ON r.id = s.region_id
GROUP BY r.name
HAVING SUM(o.total_amt_usd) = (
SELECT MAX(total_amt)
FROM (SELECT r.name region_name, SUM(o.total_amt_usd) total_amt
    FROM sales_reps s
    JOIN accounts a
    ON a.sales_rep_id = s.id
    JOIN orders o
    ON o.account_id = a.id
    JOIN region r
    ON r.id = s.region_id
    GROUP BY r.name) sub);
    
/* 对于具有最高销售额 (total_amt_usd) 的区域，总共下了多少个订单 （total count orders）？ WITH版本*/
WITH t1 AS (
SELECT r.name region_name, SUM(o.total_amt_usd) total_amt
FROM sales_reps s
JOIN accounts a
ON a.sales_rep_id = s.id
JOIN orders o
ON o.account_id = a.id
JOIN region r
ON r.id = s.region_id
GROUP BY r.name
ORDER BY 2 DESC),

t2 AS (
SELECT MAX(total_amt)
FROM t1)

SELECT r.name, SUM(o.total) total_orders
FROM sales_reps s
JOIN accounts a
ON a.sales_rep_id = s.id
JOIN orders o
ON o.account_id = a.id
JOIN region r
ON r.id = s.region_id
GROUP BY r.name
HAVING SUM(o.total_amt_usd) = (SELECT * FROM t2)
```

```SQL
/*对于购买标准纸张数量 (standard_qty) 最多的客户（在作为客户的整个时期内），有多少客户的购买总数依然更多？*/
SELECT COUNT(*)
FROM(
    SELECT a.name account_name
    FROM accounts a
    JOIN orders o
    ON a.id = o.account_id
    GROUP BY 1
    HAVING SUM(o.total) > (SELECT total
    FROM(SELECT a.name account_name, SUM(o.standard_qty) tot_std, SUM(o.total) total
        FROM accounts a
        JOIN orders o
        ON a.id = o.account_id
        GROUP BY 1
        ORDER BY 2 DESC
        LIMIT 1) df1)
) df2;

/*对于购买标准纸张数量 (standard_qty) 最多的客户（在作为客户的整个时期内），有多少客户的购买总数依然更多？ WITH版本*/
WITH t1 AS(
SELECT a.name account_name, SUM(o.standard_qty) tot_std, SUM(o.total) total
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1),

t2 AS (
SELECT total
FROM t1),

t3 AS (
SELECT a.name account_name
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY 1
HAVING SUM(o.total) > (SELECT * FROM t2))

SELECT COUNT(*)
FROM t3
```

```SQL
/*对于（在作为客户的整个时期内）总消费 (total_amt_usd) 最多的客户，他们在每个渠道上有多少 web_events？*/

SELECT a.name account_name, w.channel, COUNT(w.id) channel_count
FROM accounts a
JOIN web_events w
ON a.id = w.account_id AND a.name =
    (SELECT account_name
    FROM 
        (SELECT a.name account_name, SUM(o.total_amt_usd) tot_spent
        FROM accounts a
        JOIN orders o
        ON a.id = o.account_id
        GROUP BY 1
        ORDER BY 2 DESC
        LIMIT 1) df1)
GROUP BY 1,2
ORDER BY 3 DESC;
/*不知道为何用where取name一样的不行，用and取就可以了*/

/*对于（在作为客户的整个时期内）总消费 (total_amt_usd) 最多的客户，他们在每个渠道上有多少 web_events？ WITH版本*/
WITH t1 AS (
SELECT a.name account_name, SUM(o.total_amt_usd) tot_spent
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1),

t2 AS(
SELECT account_name
FROM t1)

SELECT a.name account_name, w.channel, COUNT(w.id) channel_count
FROM accounts a
JOIN web_events w
ON a.id = w.account_id AND a.name = (SELECT * FROM t2)
GROUP BY 1,2
ORDER BY 3 DESC;
```

```sql
/*对于总消费前十名的客户，他们的平均终身消费 (total_amt_usd) 是多少?*/
SELECT AVG(df.tot_spent)
FROM (
    SELECT a.name account_name, SUM(o.total_amt_usd) tot_spent
    FROM accounts a
    JOIN orders o
    ON a.id = o.account_id
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT 10) df
    
/*对于总消费前十名的客户，他们的平均终身消费 (total_amt_usd) 是多少? WITH版本 */
WITH t1 AS(
SELECT a.name account_name, SUM(o.total_amt_usd) tot_spent
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10)

SELECT AVG(tot_spent) avg_total_spent
FROM t1
```

```sql
/*比所有客户的平均消费高的企业平均终身消费 (total_amt_usd) 是多少？*/
SELECT AVG(avg_amt)
FROM 
    (SELECT o.account_id, AVG(o.total_amt_usd) avg_amt
    FROM orders o
    GROUP BY 1
    HAVING AVG(o.total_amt_usd) > 
        (SELECT AVG(o.total_amt_usd) avg_all
        FROM orders o
        JOIN accounts a
        ON a.id = o.account_id)
    ) temp_table;

/*比所有客户的平均消费高的企业平均终身消费 (total_amt_usd) 是多少？WITH版本*/
WITH t1 AS(
SELECT AVG(o.total_amt_usd) avg_all
FROM orders o
JOIN accounts a
ON a.id = o.account_id),

t2 AS(
SELECT o.account_id, AVG(o.total_amt_usd) avg_amt
FROM orders o
GROUP BY 1
HAVING AVG(o.total_amt_usd) > (SELECT * FROM t1))

SELECT AVG(avg_amt)
FROM t2
```

