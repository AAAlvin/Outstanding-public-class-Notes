## 了解SQL中的聚合函数

这个MD主要是记录在优达上学习SQL聚合函数时解决问题的语句，下图是EDR，根据这个解决问题：

![ERD.png](https://i.loli.net/2018/07/25/5b582587e07b7.png)

> 解决COUNT、SUM、MAX等比较简单的聚合函数问题的代码就不写出来了

## **GROUP BY**

```sql
/*哪个客户（按照名称）下的订单最早？你的答案应该包含订单的客户名称和日期。*/
SELECT a.name, occurred_at
FROM accounts a
JOIN orders o
ON a.id = o.account_id
ORDER BY occurred_at
LIMIT 1;
```

```SQL
/* 最近的 web_event 是通过哪个渠道发生的，与此 web_event 相关的客户是哪个？你的查询应该仅返回三个值：日期、渠道和客户名称。*/
SELECT a.name, w.channel, w.occurred_at
FROM accounts a
JOIN web_events w
ON a.id = w.account_id
ORDER BY w.occurred_at DESC
LIMIT 1;
```

```SQL
/*与最早的 web_event 相关的主要联系人是谁？*/
SELECT a.primary_poc
FROM web_events w
JOIN accounts a
ON a.id = w.account_id
ORDER BY occurred_at
LIMIT 1;
```

```SQL
/*算出每个区域的销售代表人数。最早表格应该包含两列：区域和 sales_reps 数量。从最少到最多的代表人数排序*/
SELECT r.name, COUNT(*) AS sales_num
FROM region r
JOIN sales_reps s
ON r.id = s.region_id
GROUP BY r.name
ORDER BY sales_num;
```

```sql
/*对于每个客户，确定他们在订单中购买的每种纸张的平均数额。结果应该有四列：客户名称一列，每种纸张类型的平均数额一列。*/
SELECT a.name account_name, AVG(standard_qty) standard_avg, AVG(gloss_qty) gloss_avg, AVG(poster_qty) poster_avg
FROM orders o
JOIN accounts a
ON a.id = o.account_id
GROUP BY account_name
Order BY account_name;
```

```sql
/*确定在 web_events 表格中针对每个地区特定渠道的使用次数。最终表格应该有三列：区域名称、渠道和发生次数。按照最高的发生次数在最上面对表格排序。*/
SELECT r.name, w.channel, COUNT(*) num_events
FROM accounts a
JOIN web_events w
ON a.id = w.account_id
JOIN sales_reps s
ON s.id = a.sales_rep_id
JOIN region r
ON r.id = s.region_id
GROUP BY r.name, w.channel
ORDER BY num_events DESC
```

**HAVING**

```SQL
/*有多少位销售代表需要管理超过 5 个客户？*/
SELECT s.name sales_name, COUNT(a.id) account_count
FROM sales_reps s
JOIN accounts a
ON s.id = a.sales_rep_id
GROUP BY sales_name
HAVING COUNT(a.id) > 5
ORDER BY account_count DESC;
```

```SQL
/*哪个客户的订单最多？*/
SELECT a.name, COUNT(o.id) order_num
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY a.name
ORDER BY order_num DESC
LIMIT 1;
```

```SQL
/*哪个客户消费的最少？*/
SELECT a.name, SUM(o.total_amt_usd) total_spent
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY a.name
ORDER BY total_spent
LIMIT 1;
```

```SQL
/*哪些客户使用 facebook 作为与消费者沟通的渠道超过 6 次？*/
SELECT a.name
FROM web_events w
JOIN accounts a
ON a.id = w.account_id 
WHERE channel = 'facebook'
GROUP BY a.name, w.channel
HAVING COUNT(w.channel) > 6
ORDER BY w.channel DESC
LIMIT 1;
```

```SQL
/*哪个渠道是客户最常用的渠道？*/
SELECT a.name, w.channel, COUNT(w.channel) count_channel
FROM web_events w
JOIN accounts a
ON a.id = w.account_id 
GROUP BY w.channel, a.name
ORDER BY count_channel DESC;
```

**DATE**

```SQL
/*Parch & Posey 在哪一年的总订单量最多？数据集中的所有年份保持均匀分布吗？*/
SELECT DATE_PART('year', occurred_at) order_year, SUM(total) total_sales
FROM orders
GROUP BY 1
ORDER BY 2 DESC;
```

```SQL
/*Walmart 在哪一年的哪一个月在铜版纸上的消费最多？*/
SELECT a.name account_name, DATE_TRUNC('month', occurred_at) order_time, SUM(o.gloss_amt_usd) tot_spent
FROM accounts a
JOIN orders o
ON a.id = o.account_id
WHERE name = 'Walmart'
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 1;
```

**CASE**

```SQL
/*我们想要根据相关的消费量了解三组不同的客户。最高的一组是终身价值（所有订单的总销售额）大于 200,000 美元的客户。第二组是在 200,000 到 100,000 美元之间的客户。最低的一组是低于 under 100,000 美元的客户。请提供一个表格，其中包含与每个客户相关的级别。你应该提供客户的名称、所有订单的总销售额和级别。消费最高的客户列在最上面。*/
SELECT a.name, SUM(o.total_amt_usd) total_spent, CASE WHEN SUM(total_amt_usd) >= 200000 THEN 'top' WHEN SUM(total_amt_usd) < 200000 AND SUM(total_amt_usd) >= 100000 THEN 'middle' ELSE 'low' END AS customer_level
FROM orders o
JOIN accounts a
ON a.id = o.account_id
GROUP BY 1
ORDER BY 2 DESC;
```

```SQL
/*我们想要找出绩效最高的销售代表，也就是有超过 200 个订单的销售代表。创建一个包含以下列的表格：销售代表名称、订单总量和标为 top 或 not 的列（取决于是否拥有超过 200 个订单）。销售量最高的销售代表列在最上面。*/
SELECT a.name, SUM(o.total_amt_usd) total_spent, CASE WHEN SUM(total_amt_usd) SELECT s.name, COUNT(o.id) order_count, CASE WHEN COUNT(o.id) > 200 THEN 'top' ELSE 'not' END AS sale_rep_level
FROM orders o
JOIN accounts a
ON a.id = o.account_id
JOIN sales_reps s
ON s.id = a.sales_rep_id
GROUP BY s.name
ORDER BY 2 DESC;
```

