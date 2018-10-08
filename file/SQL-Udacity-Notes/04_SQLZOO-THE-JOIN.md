# SQLZOO 刷题宝典

## THE JOIN

![1538116148752](E:\数据分析\数据分析\习题\SQL\assets\1538547177752.png)

### 1.

列出 賽事編號*matchid* 和球員名 *player* ,該球員代表德國隊Germany入球的。

```plsql
SELECT matchid, player 
FROM goal
WHERE teamid = 'GER'
```

![1538116230081](E:\数据分析\数据分析\习题\SQL\assets\1538548173153.png)

### 2. 

由以上查詢，你可見Lars Bender's 於賽事 1012入球。.現在我們想知道此賽事的對賽隊伍是哪一隊。

留意在 `goal` 表格中的欄位 `matchid` ，是對應表格`game`的欄位`id`。我們可以在表格 **game**中找出賽事1012的資料。

```plsql
SELECT id,stadium,team1,team2
FROM game
WHERE id = 1012
```

![1538116585167](E:\数据分析\数据分析\习题\SQL\assets\1538548373087.png)

### 3.利用`JOIN`來同時進行以上兩個步驟 

顯示每一個德國入球的球員名，隊伍名，場館和日期。

```plsql
SELECT t2.player, t2.teamid, t1.stadium, t1.mdate 
FROM game t1
JOIN goal t2
ON t1.id = t2.goal
WHERE t2.teamid = 'GER'
```

![1538117422215](E:\数据分析\数据分析\习题\SQL\assets\1538548782418.png)

### 4.球員 Mario 

列出球員名字叫Mario 有入球的 隊伍1 team1, 隊伍2 team2 和 球員名 player 。 

```plsql
SELECT t1.team1, t1.team2, t2.player 
FROM game t1
JOIN goal t2
ON t1.id=t2.matchid
WHERE t2.player LIKE ('Mario%')
```

![1538119034237](E:\数据分析\数据分析\习题\SQL\assets\1538549009142.png)

### 5.JOIN eteam表和goal表 

列出每場球賽中首10分鐘`gtime<=10`有入球的球員 `player`, 隊伍`teamid`, 教練`coach`, 入球時間`gtime` 

```plsql
SELECT t2.player, t2.teamid, t3.coach, t2.gtime 
FROM goal t2
JOIN eteam t3
ON t2.teamid = t3.id
WHERE t2.gtime <= 10 
```

![1538549653448](E:\数据分析\数据分析\习题\SQL\assets\1538549653448.png)

### 6.JOIN eteam表和game表 

列出'Fernando Santos'作為隊伍1 team1 的教練的賽事日期，和隊伍名。 

```plsql
SELECT t1.mdate, t3.teamname 
FROM game t1
JOIN eteam t3
ON t1.team1 = t3.id
WHERE t3.coach = 'Fernando Santos'
```

![1538119956041](E:\数据分析\数据分析\习题\SQL\assets\1538549875383.png)

### 7.列出場館 'National Stadium, Warsaw'的入球球員。  

```plsql
SELECT t2.player
FROM game t1
JOIN goal t2
ON t1.id = t2.matchid
WHERE t1.stadium= 'National Stadium, Warsaw'
```

![1538550178874](E:\数据分析\数据分析\习题\SQL\assets\1538550178874.png)

### 8.列出全部賽事，射入德國龍門的球員名字。  

> 后面有个一人进了两个球，所以需要GROUP BY一下

```plsql
SELECT t2.player
FROM game t1
JOIN goal t2
ON t1.id = t2.matchid
WHERE (t1.team1='GER' OR t1.team2='GER') AND t2.teamid != 'GER'
GROUP BY 1
```

![1538121519966](E:\数据分析\数据分析\习题\SQL\assets\1538551443412.png)

### 9.列出隊伍名稱 **teamname** 和該隊入球總數 。 

```plsql
SELECT t3.teamname,COUNT(t3.id)
FROM goal t2
JOIN eteam t3
ON t2.teamid = t3.id
GROUP BY 1
```

![1538121807356](E:\数据分析\数据分析\习题\SQL\assets\1538702285047.png)

### 10.列出場館名和在該場館的入球數字。。

```plsql
SELECT t1.stadium,COUNT(t2.matchid)
FROM goal t2
JOIN game t1
ON t2.matchid = t1.id
GROUP BY 1
```

![1538122046335](E:\数据分析\数据分析\习题\SQL\assets\1538702516220.png)

### 11.关于波兰的比赛 

每一場波蘭'POL'有參與的賽事中，列出賽事編號 matchid, 日期date 和入球數字。 

```plsql
SELECT t2.matchid, t1.mdate, COUNT(t2.matchid) goal_number
FROM game t1
JOIN goal t2
ON t1.id = t2.matchid
WHERE (team1 = 'POL' OR team2 = 'POL')
GROUP BY 1,2
```

![1538122502760](E:\数据分析\数据分析\习题\SQL\assets\1538702816222.png)

### 12.关于德国的比赛

每一場德國'GER'有參與的賽事中，列出賽事編號 matchid, 日期date 和**德國的入球**數字。

```plsql
SELECT t2.matchid, t1.mdate, COUNT(t2.matchid) goal_number
FROM game t1
JOIN goal t2
ON t1.id = t2.matchid
WHERE (team1 = 'GER' OR team2 = 'GER') AND t2.teamid = 'GER'
GROUP BY 1,2
```

![1538185877809](E:\数据分析\数据分析\习题\SQL\assets\1538702998759.png)

### 13.Case When - 比赛得分 

Notice in the query given every goal is listed. If it was a team1 goal then a 1 appears in score1, otherwise there is a 0. You could SUM this column to get a count of the goals scored by team1. **Sort your result by mdate, matchid, team1 and team2.** 

```plsql
SELECT t1.mdate, t1.team1,
    SUM(CASE WHEN t1.team1 = t2.teamid THEN 1 ELSE 0 END) score1,
    t1.team2,
    SUM(CASE WHEN t1.team2 = t2.teamid THEN 1 ELSE 0 END) score2
FROM game t1
JOIN goal t2
ON t1.id = t2.matchid
GROUP BY 1,2,4
ORDER BY 1,2,4
```

![1538188425781](E:\数据分析\数据分析\习题\SQL\assets\1538703973041.png)

## More JOIN(电影)

此教程練習表格合拼。數據庫有三個表格 `movie電影(id編號, title電影名稱, yr首影年份, director導演, budget製作費, gross票房收入)` 

![1538704573459](E:\数据分析\数据分析\习题\SQL\assets\1538704573459.png)

actor演員(id編號, name姓名)

![1538704619767](E:\数据分析\数据分析\习题\SQL\assets\1538704619767.png)

casting角色(movieid電影編號, actorid演員編號, ord角色次序)

![1538704638352](E:\数据分析\数据分析\习题\SQL\assets\1538704638352.png)

角色次序代表第1主角是1, 第2主角是2...如此類推. 

### 1.列出1962年首影的電影， [顯示 **id**, **title**] 

```plsql
SELECT id, title 
FROM movie 
WHERE yr = 1962
```

![1538116230081](E:\数据分析\数据分析\习题\SQL\assets\1538705341526.png)

### 2.電影大國民 'Citizen Kane' 的首影年份。 

```plsql
SELECT yr
FROM movie
WHERE title = 'Citizen Kane'
```

### 3.列出全部Star Trek系列電影 

列出全部Star Trek星空奇遇記系列的電影，包括**id**, **title** 和 **yr**(此系統電影都以Star Trek為電影名稱的開首)。按年份順序排列。 

```plsql
SELECT id, title, yr 
FROM movie
WHERE title LIKE ('Star Trek%')
ORDER BY yr
```

![1538117422215](E:\数据分析\数据分析\习题\SQL\assets\1538705529738.png)

### 4. 'Casablanca'的演員名單 

列出電影北非諜影 'Casablanca'的演員名單。 

```plsql
SELECT t2.name 
FROM movie t1
JOIN casting t3
ON t1.id=t3.movieid
JOIN actor t2
ON t2.id = t3.actorid
WHERE t1.title='Casablanca'
```

![1538119034237](E:\数据分析\数据分析\习题\SQL\assets\1538705919023.png)

### 5.'Harrison Ford' 曾演出的電影，但他不是第1主角 

列出演員夏里遜福 'Harrison Ford' 曾演出的電影，但他不是第1主角。 

```plsql
SELECT t1.title 
FROM movie t1
JOIN casting t3
ON t1.id=t3.movieid
JOIN actor t2
ON t2.id = t3.actorid
WHERE t2.name='Harrison Ford' AND t3.ord !=1
```

![1538549653448](E:\数据分析\数据分析\习题\SQL\assets\1538706055031.png)

### 6.列出1962年首影的電影及它的第1主角。 

```plsql
SELECT t1.title, t2.name
FROM movie t1
JOIN casting t3
ON t1.id=t3.movieid
JOIN actor t2
ON t2.id = t3.actorid
WHERE t1.yr=1962 AND t3.ord =1
```

![1538119956041](E:\数据分析\数据分析\习题\SQL\assets\1538706209555.png)

### 7.'John Travolta'最忙是哪一年? 

尊·特拉華達'John Travolta'最忙是哪一年? 顯示年份和該年的電影數目。

```plsql
SELECT t1.yr, COUNT(t1.title)
FROM movie t1
JOIN casting t3
ON t1.id=t3.movieid
JOIN actor t2
ON t2.id = t3.actorid
WHERE t2.name = 'John Travolta'
GROUP BY yr
ORDER BY 2 DESC
LIMIT 1
```

![1538706557230](E:\数据分析\数据分析\习题\SQL\assets\1538706557230.png)

```plsql
-- 答案给的语句，有点复杂
SELECT yr,COUNT(title) FROM
  movie JOIN casting ON movie.id=movieid
         JOIN actor   ON actorid=actor.id
where name='John Travolta'
GROUP BY yr
HAVING COUNT(title)=(SELECT MAX(c) FROM
(SELECT yr,COUNT(title) AS c FROM
   movie JOIN casting ON movie.id=movieid
         JOIN actor   ON actorid=actor.id
 where name='John Travolta'
 GROUP BY yr) AS t
)
```

### 8.'Julie Andrews'曾參與的電影 

列出演員茱莉·安德絲'Julie Andrews'曾參與的電影名稱及其第1主角。 

> 是否列了電影 "Little Miss Marker"兩次? 因为她於1980再參與此電影Little Miss Marker. 原作於1934年,她也有參與。 電影名稱不是獨一的。在子查詢中使用電影編號。 

```plsql
SELECT t1.title, t2.name
FROM movie t1
JOIN casting t3 ON t1.id = t3.movieid
JOIN actor t2 ON t2.id = t3.actorid
WHERE t3.ord= 1 AND t3.movieid IN
	(SELECT t1.id FROM movie t1
	JOIN casting t3 ON t1.id = t3.movieid
	JOIN actor t2 ON t2.id = t3.actorid
    WHERE t2.name = 'Julie Andrews')
```

![1538121519966](E:\数据分析\数据分析\习题\SQL\assets\1538707338190.png)

### 9.列出按字母順序，列出哪一演員至少作过30次第1主角 。

```plsql
SELECT t2.name
FROM casting t3
JOIN actor t2 ON t2.id = t3.actorid
WHERE t3.actorid IN 
	(SELECT t3.actorid FROM casting t3
     WHERE t3.ord = 1
     GROUP BY t3.actorid
     HAVING COUNT(*)>=30)
GROUP BY 1
ORDER BY 1 
```

![1538121807356](E:\数据分析\数据分析\习题\SQL\assets\1538708063230.png)

### 10.1978年首影的電影名稱及角色數目 

列出1978年首影的電影名稱及角色數目，按此數目由多至少排列。 

```plsql
--代码应该没错，但是显示结果错误
SELECT t1.title, COUNT(t3.actorid) count_act
FROM movie t1
JOIN casting t3 ON t1.id = t3.movieid
WHERE t1.id IN
	(SELECT id FROM movie
    WHERE yr = 1978)
GROUP BY 1
ORDER BY 2 DESC
```

![1538122046335](E:\数据分析\数据分析\习题\SQL\assets\1538708758058.png)

### 11.与'Art Garfunkel'合作過的演員姓名 

列出曾與演員亞特·葛芬柯'Art Garfunkel'合作過的演員姓名。 

> 注意两点：1.因为题目问与'Art Garfunkel'合作過的演員，所以需要排除'Art Garfunkel'；2.有了DISTINCT之后可以省略GROUP BY

```plsql
SELECT DISTINCT t2.name
FROM casting t3
JOIN actor t2 ON t2.id = t3.actorid
WHERE t2.name!='Art Garfunkel' AND t3.movieid IN 
	(SELECT DISTINCT t3.movieid FROM casting t3
     JOIN actor t2 ON t2.id = t3.actorid
     WHERE t2.name = 'Art Garfunkel')
```

![1538122502760](E:\数据分析\数据分析\习题\SQL\assets\1538709669534.png)

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

