# SQLZOO 刷题宝典

## NULL, INNER JOIN, LEFT JOIN, RIGHT JOIN

![1538877531835](E:\数据分析\数据分析\习题\SQL\assets\1538877531835.png)

### 1.List the teachers who have NULL for their department. 

> You might think that the phrase dept=NULL would work here but it doesn't - you can use the phrase `dept IS NULL` 

```plsql
SELECT name
FROM teacher 
WHERE dept IS NULL
```

![1538116230081](E:\数据分析\数据分析\习题\SQL\assets\1538877671744.png)

### 2. 用INNER JOIN 来忽略掉NULL部分。

Note the INNER JOIN misses the teachers with no department and the departments with no teacher. 

```plsql
SELECT t1.name teacher_name, t2.name dept_name
FROM teacher t1
INNER JOIN dept t2
ON t1.dept = t2.id
```

![1538877885896](E:\数据分析\数据分析\习题\SQL\assets\1538877885896.png)

### 3.LEFT JOIN 选取teacher表所有老师数据

列出全部Star Trek星空奇遇記系列的電影，包括**id**, **title** 和 **yr**(此系統電影都以Star Trek為電影名稱的開首)。按年份順序排列。 

```plsql
SELECT t1.name teacher_name, t2.name dept_name
FROM teacher t1
LEFT JOIN dept t2
ON t1.dept = t2.id
```

![1538117422215](E:\数据分析\数据分析\习题\SQL\assets\1538877947774.png)

### 4. 'Using the [COALESCE](http://sqlzoo.net/wiki/COALESCE) function 

Use COALESCE to print the mobile number. Use the number '07986 444 2266' if there is no number given.

> **coalesce** 有点像CASE-THEN，该函数返回参数中第一个非NULL的字段值，可用于替换NULL为特定字符串

```plsql
SELECT name, COALESCE(mobile,'07986 444 2266') AS monile
FROM teacher
```

![1538119034237](E:\数据分析\数据分析\习题\SQL\assets\1538878750765.png)

### 5.显示没有公寓的为None

Use the COALESCE function and a LEFT JOIN to print the teacher **name** and department name. Use the string 'None' where there is no department.  

```plsql
SELECT t1.name teacher_name, 
	COALESCE(t2.name,'None') AS dept_name
FROM teacher t1
LEFT JOIN dept t2
ON t1.dept = t2.id
```

![1538549653448](E:\数据分析\数据分析\习题\SQL\assets\1538878873035.png)

### 6.show the number of teachers and the number of mobile phones 。

```plsql
SELECT COUNT(DISTINCT name) num_name,
	COUNT(DISTINCT mobile) num_mob
FROM teacher
```

![1538119956041](E:\数据分析\数据分析\习题\SQL\assets\1538879092029.png)

### 7.显示所有公寓住的人

Use COUNT and GROUP BY **dept.name** to show each department and the number of staff. Use a RIGHT JOIN to ensure that the Engineering department is listed.

```plsql
SELECT t2.name dept_name, COUNT(t1.dept)
FROM teacher t1
RIGHT JOIN dept t2
ON t1.dept = t2.id
GROUP BY 1
```

![1538706557230](E:\数据分析\数据分析\习题\SQL\assets\1538879340203.png)

### 8.用CASE分类表示

Use CASE to show the **name** of each teacher followed by 'Sci' if the teacher is in **dept** 1 or 2 and 'Art' otherwise. 

```plsql
SELECT name,
	CASE WHEN dept = 1 OR dept = 2 THEN 'Sci'
	ELSE 'Art' END dept
FROM teacher
```

![1538121519966](E:\数据分析\数据分析\习题\SQL\assets\1538879773676.png)

## SELF JOIN

## stops

This is a list of areas served by buses. The detail does not really include each actual bus stop - just areas within Edinburgh and whole towns near Edinburgh.

![1538880482089](E:\数据分析\数据分析\习题\SQL\assets\1538880482089.png)

## route

A route is the path through town taken by a bus.

![1538880518182](E:\数据分析\数据分析\习题\SQL\assets\1538880518182.png)

>  pos:这表示路线中的停靠顺序 

### 1.How many **stops** are in the database. 

列出 賽事編號*matchid* 和球員名 *player* ,該球員代表德國隊Germany入球的。

```plsql
SELECT COUNT(DISTINCT(id)) num_stops 
FROM stops
```

![1538116230081](E:\数据分析\数据分析\习题\SQL\assets\1538880774008.png)

### 2.Give the **id** and the **name** for the **stops** on the '4' 'LRT' service.

```plsql
SELECT t1.id, t1.name
FROM stops t1
JOIN route t2
ON t1.id = t2.stop
WHERE t2.num = 4 AND company = 'LRT'
```

![1538116585167](E:\数据分析\数据分析\习题\SQL\assets\1538881364562.png)

### 3.

The query shown gives the number of routes that visit either London Road (149) or Craiglockhart (53). Run the query and notice the two services that link these **stops**have a count of 2. Add a HAVING clause to restrict the output to these two routes. 

```plsql
SELECT company,num,COUNT(*) 
FROM route
WHERE stop = 149 OR stop = 53
GROUP BY 1,2
HAVING COUNT(*) = 2
```

![1538117422215](E:\数据分析\数据分析\习题\SQL\assets\1538893676553.png)

### 4.首次尝试自连接

Execute the self join shown and observe that b.stop gives all the places you can get to from Craiglockhart, without changing routes. Change the query so that it shows the services from Craiglockhart to London Road.  

> 前面输出了Craiglockhart为53和London Road为149可以直接采用；这次尝试了没有用JOIN ON 的形式

```plsql
SELECT a.company, a.num, a.stop, b.stop  
FROM route a, route b
WHERE a.company = b.company AND a.num = b.num
AND a.stop = (SELECT id FROM stops WHERE name = 'Craiglockhart')
AND b.stop = (SELECT id FROM stops WHERE name = 'London Road')
```

![1538119034237](E:\数据分析\数据分析\习题\SQL\assets\1538961660099.png)

### 5.两个表的自连接

The query shown is similar to the previous one, however by joining two copies of the **stops** table we can refer to **stops**by **name** rather than by number. Change the query so that the services between 'Craiglockhart' and 'London Road' are shown.  

```plsql
SELECT routea.company,routea.num,stopa.name,stopb.name 
FROM route routea, route routeb, stops stopa, stops stopb
WHERE routea.company = routeb.company AND routea.num = routeb.num
	AND routea.stop = stopa.id AND routeb.stop = stopb.id 
	AND stopa.name = 'Craiglockhart' AND stopb.name = 'London Road'
```

![1538549653448](E:\数据分析\数据分析\习题\SQL\assets\1538962471588.png)

### 6.Give a list of all the services which connect stops 115 and 137 

```plsql
SELECT a.company, a.num  
FROM route a, route b
WHERE a.company = b.company AND a.num = b.num
AND a.stop = 115
AND b.stop = 137
GROUP BY 1,2
```

![1538119956041](E:\数据分析\数据分析\习题\SQL\assets\1538964423903.png)

### 7.Give a list of the services which connect the **stops**'Craiglockhart' and 'Tollcross' 

```plsql
SELECT a.company, a.num  
FROM route a, route b
WHERE a.company = b.company AND a.num = b.num
AND a.stop = (SELECT id FROM stops WHERE name = 'Craiglockhart')
AND b.stop = (SELECT id FROM stops WHERE name = 'Tollcross')
GROUP BY 1,2
```

![1538550178874](E:\数据分析\数据分析\习题\SQL\assets\1538964481288.png)

### 8.返回到达Craiglockhart 的车信息。

Give a distinct list of the **stops** which may be reached from 'Craiglockhart' by taking one bus, including 'Craiglockhart' itself, offered by the LRT company. Include the company and bus no. of the relevant services. 

> 注意限定了stopb.name，而输出的是stopa.name

```plsql
SELECT DISTINCT stopa.name,routea.company,routea.num 
FROM route routea, route routeb, stops stopa, stops stopb
WHERE routea.company = routeb.company AND routea.num = routeb.num
	AND routea.stop = stopa.id AND routeb.stop = stopb.id 
	AND stopb.name = 'Craiglockhart'
```

![1538121519966](E:\数据分析\数据分析\习题\SQL\assets\1538964875384.png)

### 9.

Find the routes involving two buses that can go from Craiglockhart to Sighthill. Show the bus no. and company for the first bus, the name of the stop for the transfer, and the bus no. and company for the second bus. 

> Hint: Self-join twice to find buses that visit Craiglockhart and Sighthill, then join those on matching stops. 

```plsql
SELECT DISTINCT bus1.num, bus1.company, name, bus2.num, bus2.company FROM (
SELECT start1.num, start1.company, stop1.stop FROM route AS start1 JOIN route AS stop1 
ON start1.num = stop1.num 
AND start1.company = stop1.company AND start1.stop != stop1.stop WHERE start1.stop = 
(SELECT id FROM stops WHERE name = 'Craiglockhart')) AS bus1 JOIN (SELECT start2.num, start2.company, start2.stop FROM route AS start2 JOIN route AS stop2 ON start2.num = stop2.num AND start2.company = stop2.company AND start2.stop != stop2.stop WHERE stop2.stop = (SELECT id FROM stops WHERE name = 'Sighthill')) AS bus2 ON bus1.stop = bus2.stop JOIN stops ON bus1.stop = stops.id

```

![1538121807356](E:\数据分析\数据分析\习题\SQL\assets\1538965119429.png)

