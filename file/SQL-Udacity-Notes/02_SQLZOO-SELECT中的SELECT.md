# SQLZOO 刷题宝典

## Using nested SELECT

![1538272924759](E:\数据分析\数据分析\习题\SQL\assets\1538272924759.png)

### 1.如何使用SELECT中的SELECT

See SELECT FROM SELECT for how to use a [derived table](http://sqlzoo.net/w/index.php/SELECT_.._SELECT).

SELECT查詢的結果可以當作一個值，用在另一個查詢當中，例如 **SELECT continent FROM world WHERE name = 'Brazil'** 得到結果 `'South America'` ，所以我們可以用此值來找出與巴西'Brazil'同一洲份的全部國家

```plsql
SELECT name,population FROM world WHERE continent = 
(SELECT continent FROM world WHERE name = 'Brazil'
```

![1538273178698](E:\数据分析\数据分析\习题\SQL\assets\1538273178698.png)

### 2. 多於一個結果

子查詢有時會得出多於一個結果-如出現，而你又對此結果進行值的比較，會出現錯誤。 可能的話，使用IN會更加安全。

查詢`(SELECT continent FROM world WHERE name = 'Brazil' OR name='Mexico')` 會得到兩個值('North America' 和 'South America'). 你應使用:

```plsql
SELECT name, continent FROM world
WHERE continent IN
 (SELECT continent FROM world WHERE name='Brazil'OR name='Mexico')
```

![1538273979145](E:\数据分析\数据分析\习题\SQL\assets\1538273979145.png)

### 3.SELECT語句中的子查詢

如你保證子查詢只會有一個結果，你可以在SELECT語句中使用子查詢。 

顯示中國人口是英國人口的多少倍。 

```plsql
SELECT
 population/(SELECT population FROM world
             WHERE name='United Kingdom')
  FROM world
WHERE name = 'China'
```

![1538274067345](E:\数据分析\数据分析\习题\SQL\assets\1538274067345.png)

### 4.對一個集合使用運算子

這些運算子是二元的，即它要兩個數值運作: 

```
=     等於
>     大於
<     小於
>=    大於或等於
<=    小於或等於
```

你可以使用 ALL 或 ANY ，當運算子的右面有多於一個值的時 

找出哪些國家的人口是高於歐洲每一國的人口。

> 留意，我們是找高於歐洲每一個單一國家的人口，不是歐洲各國合計的全部總人口。

```plsql
SELECT name FROM world
 WHERE population > ALL
      (SELECT population FROM world
        WHERE continent='Europe')
```

![15381190342371](E:\数据分析\数据分析\习题\SQL\assets\1538274340346.png)

## SELECT within SELECT Tutorial

![15381191077196](E:\数据分析\数据分析\习题\SQL\assets\1538441588009.png)

### 1.找出人口多于俄罗斯的國家。 



```plsql
SELECT name FROM world
WHERE population > 
	(SELECT population FROM world
     WHERE name = 'Russia');
```

![15381193259634](E:\数据分析\数据分析\习题\SQL\assets\1538441735161.png)

### 2.找出高于英国GDP的欧洲国家。 

```plsql
SELECT name FROM world
WHERE gdp/population >
	(SELECT gdp/population FROM world
    WHERE name = 'United Kingdom') AND continent = 'Europe'
ORDER by gdp/population DESC
```

![15381193420762](E:\数据分析\数据分析\习题\SQL\assets\1538442140311.png)

### 3.找出所有國家,其名字以 C 作開始,ia 作結尾。 

> 注意：因为要在已经筛选好的表中直接IN，所以注意只能有一列

```plsql
SELECT name,continent FROM world
WHERE continent IN 
	(SELECT continent FROM world
    WHERE name = 'Argentina' OR name = 'Australia')
ORDER BY name;
```

![15381119421016](E:\数据分析\数据分析\习题\SQL\assets\1538442571350.png)

### 4.找出人口大于加拿大但是少于波兰的国家。 

```plsql
SELECT name,population FROM world
WHERE population > 
	(SELECT population FROM world WHERE name = 'Canada')
AND population <
        (SELECT population FROM world WHERE name = 'Poland')
```

![1538443074655](E:\数据分析\数据分析\习题\SQL\assets\1538443074655.png)

### 5.显示欧洲各个国家人口占比德国人口多少百分比。 

> 您可以使用函數[ROUND](http://zh.sqlzoo.net/wiki/ROUND) 刪除小數。 您可以使用函數 [CONCAT](http://zh.sqlzoo.net/wiki/CONCAT) 增加的百分比符號。 

```plsql
SELECT name, CONCAT(ROUND(population/
                           (SELECT population FROM world
                           WHERE name = 'Germany')*100,0),'%') 
FROM world
WHERE continent = 'Europe'
```

![1538444174175](E:\数据分析\数据分析\习题\SQL\assets\1538444174175.png)

### 6.寻找人口最多的国家。 

> 我們可以用`ALL` 這個詞對一個列表進行>=或>或<或<=充當比較。 
>
> 在子查詢的條件中使用 **population>0**，因為有些國家的記錄中，人口是沒有填入，只有 **null**值。 

```plsql
SELECT name
FROM world
WHERE population >= ALL(SELECT population
                           FROM world
                          WHERE population>0)
```

### 7.顯示gdp大于全部欧洲国家的国家。 

> 有些國家的記錄中，GDP是NULL，沒有填入資料的。 

```plsql
SELECT name
FROM world
WHERE gdp > ALL(SELECT gdp FROM world 
                WHERE gdp>0 AND continent = 'Europe')
```

![1538444884963](E:\数据分析\数据分析\习题\SQL\assets\1538444884963.png)

### 8.找出每一個州中最大面積的國家 

> 有些國家的記錄中，AREA是NULL，沒有填入資料的。 
>
> 可以在子查詢，參閱外部查詢的數值。我們為表格再命名，便可以分別內外兩個不同的表格。 

```plsql
SELECT continent, name, area FROM world t1
WHERE area >= ALL
	(SELECT area FROM world t2 
     WHERE t2.continent=t1.continent AND area>0)
```

![15382121519966](E:\数据分析\数据分析\习题\SQL\assets\1538445326054.png)

### 9.找出一个州中所有国家人数均少于一个值。 

找出洲份，當中全部國家都有少於或等於 25000000 人口. 在這些洲份中，列出國家名字**name**，**continent** 洲份和**population**人口。 

```plsql
SELECT name, continent, population FROM world t1
WHERE 25000000>= ALL
	(SELECT population FROM world t2 
     WHERE t2.continent=t1.continent AND population>0 )
```

![1538446592079](E:\数据分析\数据分析\习题\SQL\assets\1538446592079.png)

### 10.3倍人口。

有些國家的人口是同洲份的所有其他國的3倍或以上。列出 國家名字name 和 洲份 continent。 

```plsql
SELECT name, continent FROM world t1
WHERE t1.population/3 >= ALL
	(SELECT population FROM world t2 
     WHERE t2.continent=t1.continent AND population>0 
     AND t1.name != t2.name)
```

![15138122046335](E:\数据分析\数据分析\习题\SQL\assets\1538446806508.png)
