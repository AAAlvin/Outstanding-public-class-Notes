# SQLZOO 刷题宝典

## SELECT basics

![1538116148752](E:\数据分析\数据分析\习题\SQL\assets\1538116148752.png)

### 1.顯示德國 Germany 的人口。

```plsql
SELECT population 
FROM world
WHERE name = 'Germany'
```

![1538116230081](E:\数据分析\数据分析\习题\SQL\assets\1538116230081.png)

### 2. Per Capita GDP

查詢面積為 5,000,000 以上平方公里的國家,對每個國家顯示她的名字和人均國內生產總值(`gdp/population`) 

```plsql
SELECT name, gdp/population AS Per_Capita_GDP
FROM world
WHERE area > 5000000
```

![1538116585167](E:\数据分析\数据分析\习题\SQL\assets\1538116585167.png)

### 3.Scandinavia

顯示“Ireland 愛爾蘭”,“Iceland 冰島”,“Denmark 丹麥”的國家名稱和人口。 

```plsql
SELECT name, population 
FROM world
WHERE name IN ('Ireland', 'Iceland', 'Denmark');
```

![1538117422215](E:\数据分析\数据分析\习题\SQL\assets\1538117422215.png)

### 4.Just the right size

顯示面積為 200,000 及 250,000 之間的國家名稱和該國面積。 

```plsql
SELECT name, area 
FROM world
WHERE area BETWEEN 200000 AND 250000
```

![1538119034237](E:\数据分析\数据分析\习题\SQL\assets\1538119034237.png)

## SELECT name

![1538119107796](E:\数据分析\数据分析\习题\SQL\assets\1538119107796.png)

### 1.找出以 **Y** 為開首的國家。 

> 你可以用`WHERE name LIKE 'B%'`來找出以 B 為首的國家。 %是萬用字元,可以用代表任何字符。 

```plsql
SELECT name 
FROM world
WHERE name LIKE 'Y%'
```

![1538119259634](E:\数据分析\数据分析\习题\SQL\assets\1538119259634.png)

### 2.找出所有國家,其名字包括字母x。 

```plsql
SELECT name 
FROM world
WHERE name LIKE '%X%'
```

![1538119340762](E:\数据分析\数据分析\习题\SQL\assets\1538119340762.png)

### 3.找出所有國家,其名字以 C 作開始,ia 作結尾。 

```plsql
SELECT name 
FROM world
WHERE name LIKE 'C%ia'
```

![1538119421016](E:\数据分析\数据分析\习题\SQL\assets\1538119421016.png)

### 4.找出所有國家,其名字包括三個或以上的a。 

```plsql
SELECT name 
FROM world
WHERE name LIKE '%a%a%a%'
```

### 5.找出所有國家,其名字以t作第二個字母。 

可以用底線符`_`當作單一個字母的萬用字元。 

```plsql
SELECT name 
FROM world
WHERE name LIKE '_t%'
ORDER BY name
```

### 6.找出所有國家,其名字都有兩個字母 o,且每个字母o都被另外兩個字母相隔着。 

```plsql
SELECT name 
FROM world
WHERE name LIKE '%o__o%'
```

![1538119956041](E:\数据分析\数据分析\习题\SQL\assets\1538119956041.png)

### 7.顯示所有國家名字,其首都和國家名字是相同的。 

> CONCAT函数用于将两个字符串连接起来，形成一个单一的字符串。 

```plsql
SELECT name 
FROM world
WHERE name = concat(capital, '')
```

### 8.顯示所有國家名字,其首都是國家名字加上”City” 

> 注意引号内的空格

```plsql
SELECT name
FROM world
WHERE capital = concat(name, ' City')
```

![1538121519966](E:\数据分析\数据分析\习题\SQL\assets\1538121519966.png)

### 9.找出所有首都和其國家名字,而首都要有國家名字出現。 

```plsql
SELECT capital, name
FROM world
WHERE capital LIKE concat('%',name,'%')
```

![1538121807356](E:\数据分析\数据分析\习题\SQL\assets\1538121807356.png)

### 10.找出所有首都和其國家名字,而首都是國家名字的延伸。

```plsql
SELECT name,capital
FROM world
WHERE capital LIKE concat(name,'_','%')
```

![1538122046335](E:\数据分析\数据分析\习题\SQL\assets\1538122046335.png)

### 11.顯示國家名字，及其延伸詞 

> ```plsql
>  REPLACE('vessel','e','a') -> 'vassal'
> ```

```plsql
SELECT name, replace(capital, name, '')  extension
FROM world
WHERE capital LIKE concat(name,'_','%')
```

![1538122502760](E:\数据分析\数据分析\习题\SQL\assets\1538122502760.png)

## ELECT from WORLD Tutorial

![1538185756077](E:\数据分析\数据分析\习题\SQL\assets\1538185756077.png)

### 1.substitute Australasia for Oceania

Show the name - but substitute Australasia for Oceania - for countries beginning with N.

```plsql
SELECT name, 
CASE WHEN continent='Oceania ' THEN 'Australasia' 
ELSE continent END AS continent
FROM world
WHERE name LIKE 'N%'
```

![1538185877809](E:\数据分析\数据分析\习题\SQL\assets\1538185877809.png)

### 2.Case When - substitute 

Show the name and the continent - but substitute **Eurasia** for Europe and Asia; substitute **America** - for each country in **North America** or **South America** or **Caribbean**. Show countries beginning with A or B 

```plsql
SELECT name,
    CASE WHEN continent = 'Europe' OR continent = 'Asia' THEN 'Eurasia'
    WHEN continent = 'North America' OR continent = 'South America' OR continent = 'Caribbean' THEN 'America' 
    ELSE continent END AS continent
FROM world
WHERE name LIKE 'A%' OR name LIKE 'B%'
```

![1538188425781](E:\数据分析\数据分析\习题\SQL\assets\1538188425781.png)

### 3.New Continent

Put the continents right...

- Oceania becomes Australasia
- Countries in Eurasia and Turkey go to **Europe/Asia**
- Caribbean islands starting with 'B' go to **North America**, other Caribbean islands go to **South America**

Show the name, the original continent and the new continent of all countries.

```plsql
SELECT name, continent AS old_continent,
	CASE WHEN continent = 'Oceania' THEN 'Australasia'
	WHEN continent = 'Eurasia' OR continent = 'Turkey' THEN 'Europe/Asia'
	WHEN continent = 'Caribbean' THEN
		CASE WHEN name LIKE 'B%' THEN 'North America'
		ELSE 'South America' END
	ELSE continent END AS new_continent
FROM world
ORDER BY name;
		
```

![1538189385666](E:\数据分析\数据分析\习题\SQL\assets\1538189385666.png)

## SELECT from Nobel Tutorial

![1538267664493](E:\数据分析\数据分析\习题\SQL\assets\1538267664493.png)

### 2.顯示誰贏得了1962年文學獎(Literature)。

```plsql
SELECT winner
FROM nobel
WHERE yr = 1962
   AND subject = 'Literature'
```

![1538267758077](E:\数据分析\数据分析\习题\SQL\assets\1538267758077.png)

### 3.顯示“愛因斯坦”('Albert Einstein') 的獲獎年份和獎項。 

```plsql
SELECT yr, subject
FROM nobel
WHERE winner = 'Albert Einstein'
```

![1538267947191](E:\数据分析\数据分析\习题\SQL\assets\1538267947191.png)

### 4.顯示2000年及以後的和平獎(‘Peace’)得獎者。

```plsql
SELECT winner
FROM nobel
WHERE yr >= 2000 AND subject = 'Peace'
```

![1538268199847](E:\数据分析\数据分析\习题\SQL\assets\1538268199847.png)

### 5.文學獎(Literature)獲獎者

顯示1980年至1989年(包含首尾)的文學獎(Literature)獲獎者所有細節（年，主題，獲獎者）。

> **注意between的用法**

```plsql
SELECT * 
FROM nobel
WHERE yr BETWEEN 1980 AND 1989 AND subject = 'Literature'
```

![1538268360092](E:\数据分析\数据分析\习题\SQL\assets\1538268360092.png)

### 6.顯示以下獲奖者的所有細節

- 西奧多•羅斯福 Theodore Roosevelt
- 伍德羅•威爾遜 Woodrow Wilson
- 吉米•卡特 Jimmy Carter

```plsql
SELECT * 
FROM nobel
WHERE winner IN ('Theodore Roosevelt',
                  'Woodrow Wilson',
                  'Jimmy Carter')
```

![1538268513311](E:\数据分析\数据分析\习题\SQL\assets\1538268513311.png)

### 7.顯示名字為John 的得獎者。

>  (注意:外國人名字(First name)在前，姓氏(Last name)在後)

```plsql
SELECT winner
FROM nobel
WHERE winner LIKE ('John%')
```

![1538268710445](E:\数据分析\数据分析\习题\SQL\assets\1538268710445.png)

### 8.顯示两年獲獎者

顯示1980年物理學(physics)獲獎者，及1984年化學獎(chemistry)獲得者。

```plsql
SELECT *
FROM nobel
WHERE yr = '1980' AND subject = 'Physics' or yr = '1984' AND subject = 'Chemistry'
```

![1538269036675](E:\数据分析\数据分析\习题\SQL\assets\1538269036675.png)

### 9.顯示不包括的獲獎者

查看1980年獲獎者，但不包括化學獎(Chemistry)和醫學獎(Medicine)。 

```plsql
SELECT *
FROM nobel
WHERE subject IN ('Economics','Literature','Peace','Physics') AND yr = '1980'
```

![1538269317251](E:\数据分析\数据分析\习题\SQL\assets\1538269317251.png)

### 10.單引號 

查找尤金•奧尼爾EUGENE O'NEILL得獎的所有細節

> 你不能把一個單引號直接的放在字符串中。但您可連續使用兩個單引號在字符串中當作一個單引號。 

```plsql
SELECT *
FROM nobel
WHERE winner = 'EUGENE O''NEILL'
```

![1538270781504](E:\数据分析\数据分析\习题\SQL\assets\1538270781504.png)

### 11.排序

列出爵士的獲獎者、年份、獎頁(爵士的名字以Sir開始)。先顯示最新獲獎者，然後同年再按名稱順序排列。 

```plsql
SELECT winner, yr, subject
FROM nobel
WHERE winner LIKE('Sir%')
ORDER BY yr DESC, winner ASC
```

![1538271387248](E:\数据分析\数据分析\习题\SQL\assets\1538271387248.png)

### 12.排序

Show the 1984 winners and subject ordered by subject and winner name; but list Chemistry and Physics last. 

```plsql
SELECT winner, subject
FROM nobel
WHERE yr=1984
ORDER BY subject IN ('Chemistry','Physics'),subject asc,winner asc
```

![1538271714743](E:\数据分析\数据分析\习题\SQL\assets\1538271714743.png)

### 13.顯示有多少年沒有頒發醫學獎。 

![1538272575217](E:\数据分析\数据分析\习题\SQL\assets\1538272575217.png)

### 14.選擇代碼以顯示哪一年沒有頒發物理獎，亦沒有頒發化學獎。 

![1538272603070](E:\数据分析\数据分析\习题\SQL\assets\1538272603070.png)

### 15.顯示哪一年有頒發醫學獎，但沒有頒發和平或文學獎。 

![1538272657799](E:\数据分析\数据分析\习题\SQL\assets\1538272657799.png)