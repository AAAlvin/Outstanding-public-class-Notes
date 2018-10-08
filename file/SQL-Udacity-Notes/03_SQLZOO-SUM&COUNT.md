# SQLZOO 刷题宝典

## SUM & COUNT

### 全球統計:群組函數

此教程是有關群組函數，例如COUNT, SUM 和 AVG。群組函數把多個數值運算，得出結果只有一個數值。例如SUM函數會把數值2,4,和5運算成結果11。

![15138116148752](E:\数据分析\数据分析\习题\SQL\assets\1538529236166.png)

### 1.列出所有的洲份, 每個只有一次。 

```plsql
SELECT continent 
FROM world
GROUP BY continent
```

![15381162310081](E:\数据分析\数据分析\习题\SQL\assets\1538529490436.png)

### 2. 找出非洲(Africa)的GDP總和 

```plsql
SELECT SUM(gdp) AS sum_Africa_GDP
FROM world
WHERE continent = 'Africa'
```

![1538116585167](E:\数据分析\数据分析\习题\SQL\assets\1538529673744.png)

### 3.有多少個國家具有至少百萬(1000000)的面積。 

```plsql
SELECT COUNT(*) AS Count
FROM world
WHERE area >= 1000000
```

### 4.Just the right size

法國”，“德國”，“西班牙”的總人口是多少？ 

```plsql
SELECT SUM(population) AS sum_population
FROM world
WHERE name IN('France','Germany','Spain')
```

### 5.對於每一個洲份，顯示洲份和國家的數量。  

```plsql
SELECT continent, COUNT(name)
FROM world
GROUP BY 1
```

![1538119259634](E:\数据分析\数据分析\习题\SQL\assets\1538530324033.png)

## 諾貝爾獎:群組函數

![1538267664493](E:\数据分析\数据分析\习题\SQL\assets\1538267664493.png)

### 1.找出總共有多少個獎頒發了 。 

```plsql
SELECT COUNT(*)
FROM nobel
```

![1538119340762](E:\数据分析\数据分析\习题\SQL\assets\1538531892259.png)

### 3.找出物理獎的總頒發次數。  

```plsql
SELECT COUNT(*) 
FROM nobel
WHERE subject= 'Physics'
```

![1538119421016](E:\数据分析\数据分析\习题\SQL\assets\1538119421016.png)

### 4.每一個獎項(Subject),列出首次頒發的年份。  

```plsql
SELECT subject,MIN(yr) 
FROM nobel
GROUP BY 1
```

![1538532363045](E:\数据分析\数据分析\习题\SQL\assets\1538532363045.png)

### 5.對每一個獎項(Subject),列出有多少個不同的得獎者。  

```plsql
SELECT subject,COUNT(DISTINCT winner) 
FROM nobel
GROUP BY 1
```

![1538532786295](E:\数据分析\数据分析\习题\SQL\assets\1538532786295.png)

### 6.列出哪年曾同年有3個物理獎Physics得獎者。  

```plsql
SELECT yr
FROM nobel
WHERE subject = 'Physics'
GROUP BY yr
HAVING COUNT(winner) = 3
```

![1538119956041](E:\数据分析\数据分析\习题\SQL\assets\1538533096642.png)

### 7.列出誰得獎多於一次。 

```plsql
SELECT winner
FROM nobel
GROUP BY winner
HAVING COUNT(winner) >1
```

![1538533362931](E:\数据分析\数据分析\习题\SQL\assets\1538533362931.png)

### 8.列出誰獲得多於一個獎項(Subject)  

> 注意如果这里没有DISTINCT，会取到同一个奖领多次的人，比如领两次物理的John Bardeen 

```plsql
SELECT winner
FROM nobel
GROUP BY winner
HAVING COUNT(DISTINCT subject) >1
```

![1538121519966](E:\数据分析\数据分析\习题\SQL\assets\1538533611193.png)

### 9.同一獎項(subject)頒發給3個人  

哪年哪獎項，是同一獎項(subject)頒發給3個人。只列出2000年及之後的資料。

```plsql
SELECT yr, subject
FROM nobel
WHERE yr >= 2000
GROUP BY yr, subject 
HAVING COUNT(winner) = 3
```

![1538121807356](E:\数据分析\数据分析\习题\SQL\assets\1538533804167.png)
