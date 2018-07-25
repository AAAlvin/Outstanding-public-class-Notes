## 用SQL做数据清理

这个MD主要是记录在优达上学习用SQL做数据清理时解决问题的语句，下图是EDR，根据这个解决问题：

![ERD.png](https://i.loli.net/2018/07/25/5b582587e07b7.png)

**LEFT 和 RIGHT**

```sql
/*在 accounts 表格中，有一个列存储的是每个公司的网站。最后三个数字表示他们使用的是什么类型的网址。此处给出了扩展（和价格）列表。请获取这些扩展并得出 accounts 表格中每个网址类型的存在数量。*/
SELECT RIGHT(website, 3) AS web_type, COUNT(id)
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;
```

```sql
/*对于公司名称（甚至名称的第一个字母）的作用存在颇多争议。请从accounts表格中获取每个公司名称的第一个字母，看看以每个字母（数字）开头的公司名称分布情况。*/
SELECT LEFT(name, 1) AS initial, COUNT(id)
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;
```

```sql
/*使用 accounts 表格和 CASE 语句创建两个群组：一个是以数字开头的公司名称群组，另一个是以字母开头的公司名称群组。以字母开头的公司名称所占的比例是多少？*/
SELECT CASE WHEN LEFT(name, 1) IN ('1','2','3','4','5','6','7','8','9','0') THEN 'nums' ELSE 'letter' END AS initial, COUNT(id) tot_count
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;
```

**POSITION 和 STRPOS**

```SQL
/*使用 accounts 表格创建一个名字和姓氏列，用于存储 primary_poc 的名字和姓氏。*/
SELECT name, primary_poc, LEFT(primary_poc, POSITION(' ' IN primary_poc) - 1) AS first_name, RIGHT(primary_poc, LENGTH(primary_poc) - POSITION(' ' IN primary_poc)) AS last_name
FROM accounts
```

```SQL
/*现在创建一个包含 sales_rep 表格中每个销售代表姓名的列，同样，需要提供名字和姓氏列。*/
SELECT name, LEFT(name, POSITION(' ' IN name) - 1) AS first_name, RIGHT(name, LENGTH(name) - POSITION(' ' IN name)) AS last_name
FROM sales_reps
```

**CONCAT**

```SQL
/*accounts 表格中的每个客户都想为每个 primary_poc 创建一个电子邮箱。邮箱应该是 primary_poc 的名字.primary_poc的姓氏@公司名称.com。*/
WITH t1 AS(
SELECT name, primary_poc, LEFT(primary_poc, POSITION(' ' IN primary_poc) - 1) AS first_name, RIGHT(primary_poc, LENGTH(primary_poc) - POSITION(' ' IN primary_poc)) AS last_name
FROM accounts)

SELECT primary_poc, first_name, last_name, CONCAT(first_name, '.', last_name, '@', name, '.com')
FROM t1
```

```SQL
/*你可能注意到了，在上一个答案中，有些公司名称存在空格，肯定不适合作为邮箱地址。看看你能否通过删掉客户名称中的所有空格来创建合适的邮箱地址，否则你的答案就和问题 1. 的一样。此处是一些实用的文档。*/
WITH t1 AS(
SELECT name, primary_poc, LEFT(primary_poc, POSITION(' ' IN primary_poc) - 1) AS first_name, RIGHT(primary_poc, LENGTH(primary_poc) - POSITION(' ' IN primary_poc)) AS last_name
FROM accounts)

SELECT primary_poc, first_name, last_name, CONCAT(first_name, '.', last_name, '@', REPLACE(name, ' ', ''), '.com')
FROM t1;
/*补充链接：https://www.postgresql.org/docs/8.1/static/functions-string.html*/
```

```sql
/*我们还需要创建初始密码，在用户第一次登录时将更改。初始密码将是 primary_poc 的名字的第一个字母（小写），然后依次是名字的最后一个字母（小写）、姓氏的第一个字母（小写）、姓氏的最后一个字母（小写）、名字的字母数量、姓氏的字母数量，然后是合作的公司名称（全大写，没有空格）*/
WITH t1 AS(
SELECT name, primary_poc, LEFT(primary_poc, POSITION(' ' IN primary_poc) - 1) AS first_name, RIGHT(primary_poc, LENGTH(primary_poc) - POSITION(' ' IN primary_poc)) AS last_name
FROM accounts)

SELECT name, first_name, last_name, LEFT(LOWER(first_name), 1) || RIGHT(LOWER(first_name), 1) || LEFT(LOWER(last_name), 1) || RIGHT(LOWER(last_name), 1) || LENGTH(first_name) || LENGTH(last_name) || REPLACE(UPPER(name), ' ', '')
FROM t1;

SELECT *,(year || '-' || DATE_PART('month', TO_DATE(month, 'month')) || '-' || day)::date AS formatted_date
FROM ad_clicks
```

**CAST**

新的数据集，该数据集与 Parch & Posey 数据集不同，因为后者中的所有数据类型都已清理。 1

```SQL
/*编写一个查询，将日期修改成正确的格式,你至少需要用到SUBSTR和CONCAT才能执行这一操作，然后转化成日期格式*/
SELECT date AS old_date, (SUBSTR(date, 7, 4) || '-' || LEFT(date, 2) || '-' || SUBSTR(date,4,2))::date AS cor_date
from sf_crime_data
limit 10;
```

**COALESCE**

```sql
/*使用COALESCE将accounts.id列中的NULL值用accounts.id进行填充*/
SELECT COALESCE(a.id, a.id) AS filled_id, a.name, a.website, a.lat, a.long, a.primary_poc, a.sales_rep_id, o.*
FROM accounts a
LEFT JOIN orders o
ON a.id = o.account_id
WHERE o.total IS NULL; 
```

```SQL
/*使用COALESCE将orders.accounts_id列中的NULL值用accounts.id进行填充*/
SELECT COALESCE(a.id, a.id) AS filled_id, a.name, a.website, a.lat, a.long, a.primary_poc, a.sales_rep_id, COALESCE(o.account_id, a.id) account_id, o.occurred_at, o.standard_qty, o.gloss_qty, o.poster_qty, o.total, o.standard_amt_usd, o.gloss_amt_usd, o.poster_amt_usd, o.total_amt_usd
FROM accounts a
LEFT JOIN orders o
ON a.id = o.account_id
WHERE o.total IS NULL; 
```

```sql
/*使用COALESCE将qty&usd列分别填充为0*/
SELECT COALESCE(a.id, a.id) filled_id, a.name, a.website, a.lat, a.long, a.primary_poc, a.sales_rep_id, COALESCE(o.account_id, a.id) account_id, o.occurred_at, COALESCE(o.standard_qty, 0) standard_qty, COALESCE(o.gloss_qty,0) gloss_qty, COALESCE(o.poster_qty,0) poster_qty, COALESCE(o.total,0) total, COALESCE(o.standard_amt_usd,0) standard_amt_usd, COALESCE(o.gloss_amt_usd,0) gloss_amt_usd, COALESCE(o.poster_amt_usd,0) poster_amt_usd, COALESCE(o.total_amt_usd,0) total_amt_usd
FROM accounts a
LEFT JOIN orders o
ON a.id = o.account_id
WHERE o.total IS NULL;
```

