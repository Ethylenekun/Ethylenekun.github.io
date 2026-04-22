---
title: MySQL基础(多表查询、函数、子查询)
tags:
  - SQL
categories: SQL
description: MySQL基础(多表查询、函数、子查询)
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/mysql.jpg'
swiper_index: 7
abbrlink: be626d7e
---
# MySQL基础(多表查询、函数、子查询)

## 多表查询

### CROSS JOIN

```sql
-- 以上两种结果相同，以上两种语法用来产生笛卡尔积，不常用

SELECT * FROM t1,t2; -- 隐式

SELECT * FROM t1 CROSS JOIN t2; -- 显式
```

### UNION

基本规则：

1. 首先，所有`SELECT`语句中出现的列的数量和顺序必须相同。

2. 其次，列的数据类型必须相同或可转换。

`UNION`会删除重复的数据，`UNION ALL`则不会，后者的效率较高

```sql
-- 新的列名为value1
SELECT 1 AS "value1" UNION ALL SELECT 2 AS "value2";

-- 要对union的结果进行排序，在最后一个SELECT语句中使用ORDER BY，还可以基于列位置使用
SELECT 10,20 UNION ALL SELECT 30,15 ORDER BY 1 DESC;
```

### NATRUAL JOIN

> 自动查询两张连接表中所有相同的字段，然后进行等值连接

```sql
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
ON e.`department_id` = d.`department_id`
AND e.`manager_id` = d.`manager_id`;

-- 等同于下方写法
SELECT employee_id,last_name,department_name
FROM employees e NATURAL JOIN departments d;
```

### USING

> 指定数据表里的同名字段进行等值连接
>
> 注意点：若是`left join` / `right join`，连接条件为`id`，`select id` 为主表的id

```sql
WITH 
  t1(id,value) AS (SELECT 1,10 UNION ALL SELECT 2,20),
  t2(id,value) AS (SELECT 2,20 UNION ALL SELECT 3,30)

SELECT * FROM t1 INNER JOIN t2 USING(id,value);
```

---

## 函数

### 单行函数

#### 数值函数

| 函数名                             | 描述                                                         | 实例                                                         |
| :--------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ABS(x)`                           | 返回 x 的绝对值                                              | 返回 -1 的绝对值：`SELECT ABS(-1)` – 返回1                   |
| `SIGN(x)`                          | 返回 x 的符号，x 是负数、0、正数分别返回 -1、0 和 1          | `SELECT SIGN(-10)` – (-1)                                    |
| `PI()`                             | 返回圆周率(3.141593）                                        | `SELECT PI() `–3.141593                                      |
| `CEIL(x)`                          | 返回大于或等于 x 的最小整数                                  | `SELECT CEIL(1.5)` – 返回2                                   |
| `FLOOR(x)`                         | 返回小于或等于 x 的最大整数                                  | 小于或等于1.5的整数`SELECT FLOOR(1.5)` – 返回1               |
| `MOD(x,y)`                         | 返回 x 除以 y 以后的余数                                     | 5除于2的余数：`SELECT MOD(5,2)` – 1                          |
| `GREATEST(expr1, expr2, expr3, …)` | 返回列表中的最大值                                           | 返回数字列表中的最大值：`SELECT GREATEST(3,12,34,8,25);` – 34 |
| `LEAST(expr1, expr2, expr3, …)`    | 返回列表中的最小值                                           | 返回以下数字列表中的最小值：`SELECT LEAST(3, 12, 34, 8, 25);` – 3 |
| `RAND() / RAND(x)`                 | 返回 0 到 1 的随机数 / 其中x的值用作种子值，相同的x值会产生相同的随机数 | `SELECT RAND()` –0.93099315644334                            |
| `n DIV m`                          | 整除，n 为被除数，m 为除数                                   | 计算 10 除于 5：`SELECT 10 DIV 5;` – 2                       |
| `POWER(x,y)`                       | 返回 x 的 y 次方                                             | 2 的 3 次方：`SELECT POWER(2,3)`– 8                          |
| `ROUND(x) / ROUND(x,y)`            | 返回一个对x的值进行四舍五入后最接近X的整数 / 并保留到小数点后面Y位 | `SELECT ROUND(1.23456) `–1                                   |
| `SQRT(x)`                          | 返回x的平方根                                                | 25 的平方根：`SELECT SQRT(25)` – 5                           |
| `TRUNCATE(x,y)`                    | 返回数值 x 保留到小数点后 y 位的值（与 ROUND 最大的区别是不会进行四舍五入） | `SELECT TRUNCATE(1.23456,3)` – 1.234                         |

#### 文本函数

| 函数                                                      | 描述                                                         | 实例                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `LENGTH(s)`                                               | 返回字符串s的字节数                                          | `SELECT LENGTH(‘阿’);` – 3                                   |
| `CHAR_LENGTH(s)`                                          | 返回字符串s的字符数                                          | 返回字符串 RUNOOB 的字符数`SELECT CHAR_LENGTH(“RUNOOB”) AS LengthOfString;` |
| `CONCAT(s1,s2…sn)`                                        | 字符串s1,s2等多个字符串合并为一个字符串                      | 合并多个字符串`SELECT CONCAT(“SQL”, “Runoob”, “Gooogle”, “Facebook”) AS String;` |
| `CONCAT_WS(x, s1,s2…sn)`                                  | 同 CONCAT(s1,s2,…) 函数，但是每个字符串之间要加上x，x可以是分隔符 | 合并多个字符串并添加分隔符：`SELECT CONCAT_WS(“-”, “SQL”, “Tutorial”, “is”, “fun!”) AS ConcatenatedString;` |
| `LEFT(s,n)`                                               | 返回字符串s的前n个字符                                       | 返回字符串 runoob 中的前两个字符：`SELECT LEFT(‘runoob’,2) `– ru |
| `MID(s,n,len)`                                            | 从字符串s的n位置截取长度为len的子字符串，同 SUBSTRING(s,n,len) | 字符串RUNOOB中第2个位置截取3个字符：`SELECT MID(“RUNOOB”,2,3) AS ExtractString;` – UNO |
| `RIGHT(s,n)`                                              | 返回字符串s的后n个字符                                       | 返回字符串 runoob 的后两个字符：`SELECT RIGHT(‘runoob’,2)` – ob |
| **`SUBSTRING_INDEX(s,delimiter,n)`**                      | 返回第n个分隔符前的字符串，正负控制方向                      | `SELECT SUBSTRING_INDEX(SUBSTRING_INDEX("1@2@3","@",2),"@",-1)` # 2 |
| `SUBSTR(s, start, length)`                                | 从字符串s的start位置截取长度为length的子字符串               | 从字符串 RUNOOB 中的第2个位置截取3个字符：`SELECT SUBSTR(“RUNOOB”,2,3) AS ExtractString;` – UNO |
| `INSERT(s1,x,len,s2)`                                     | 字符串 s2 替换 s1 的 x 位置开始长度为 len 的字符串           | 从字符串第一个位置开始的6个字符替换为runoob：                |
| `REPLACE(s,s1,s2)`                                        | 将字符串 s2 替代字符串 s 中的字符串 s1                       | 将字符串 abc 中的字符 a 替换为字符 x：`SELECT REPLACE("1@2@3","@","!")` # 1!2!3 |
| `UPPER(s)`                                                | 将字符串转换为大写                                           | 将字符串 runoob 转换为大写：`SELECT UPPER(“runoob”);` – RUNOOB |
| `LOWER(s)`                                                | 将字符串s的所有字母变成小写字母                              | 字符串 RUNOOB 转换为小写：`SELECT LOWER(‘RUNOOB’)` – runoob  |
| `LPAD(s1,len,s2)`                                         | 在字符串 s1 的开始处填充字符串 s2，使字符串长度达到 len      | 将字符串 xx 填充到 abc 字符串的开始处：`SELECT LPAD(‘abc’,5,‘xx’)` – xxabc |
| `RPAD(s1,len,s2)`                                         | 在字符串 s1 的结尾处添加字符串 s2，使字符串的长度达到 len    | 将字符串 xx 填充到 abc 字符串的结尾处：`SELECT RPAD(‘abc’,5,‘xx’)` – abcxx |
| `TRIM(s)`                                                 | 去掉字符串 s 开始和结尾处的空格                              | `SELECT TRIM("!" FROM LPAD("AS",5,"!"))`  - AS               |
| `TRIM(LEADING s1 FROM s)`<br />`TRIM(TRAILING s1 FROM s)` | 去掉字符串s开始处的s1 / 去掉字符串s结尾处的s1                | `SELECT TRIM(LEADING "@" FROM "@AS@"),TRIM(TRAILING "@" FROM "@AS@");` -- AS@ @AS |
| `LTRIM(s)`                                                | 去掉字符串 s 开始处的空格                                    | 去掉RUNOOB开始处的空格：`SELECT LTRIM(” RUNOOB”) AS LeftTrimmedString;`– RUNOOB |
| `RTRIM(s)`                                                | 去掉字符串 s 结尾处的空格                                    | 去掉RUNOOB 末尾空格：`SELECT RTRIM(“RUNOOB”) AS RightTrimmedString;` –RUNOOB |
| `REPEAT(s,n)`                                             | 将字符串 s 重复 n 次                                         | 将字符串 runoob 重复三次：`SELECT REPEAT(‘runoob’,3)` – runoobrunoobrunoob |
| `SPACE(n)`                                                | 返回 n 个空格                                                | 返回 10 个空格：`SELECT SPACE(10);`                          |
| `ELT(N,str1,str2…)`                                       | 返回指定位置的字符串                                         | `SELECT ELT(2,‘a’,‘b’,‘c’) `# b                              |
| `FIELD(s,s1,s2…)`                                         | 返回第一个字符串s在字符串列表(s1,s2…)中的位置                | 返回字符串 c 在列表值中的位置：`SELECT FIELD(“c”, “a”, “b”, “c”, “d”, “e”);` |
| `LOCATE(s1,s)`                                            | 从字符串 s 中获取 s1 的开始位置                              | 获取 b 在字符串 abc 中的位置：`SELECT LOCATE(‘st’,‘myteststring’); `– 5 |
| `POSITION(s1 IN s)`                                       | 从字符串 s 中获取 s1 的开始位置                              | 返回字符串 abc 中 b 的位置：`SELECT POSITION(‘b’ in ‘abc’)` – 2 |
| `FIND_IN_SET(s1,s2)`                                      | 返回在字符串s2中与s1匹配的字符串的位置,s2是一个`,`连接的字符串 | 返回字符串 c 在指定字符串中的位置：`SELECT FIND_IN_SET(“c”, “a,b,c,d,e”);` # 3 |
| `REVERSE(s)`                                              | 将字符串s的顺序反过来                                        | 将字符串 abc 的顺序反过来：`SELECT REVERSE(‘abc’) `– cba     |

#### 日期函数

| 函数                               | 描述                                                         | 实例                                                         |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `NOW()`                            | 返回当前日期和时间                                           |                                                              |
| `CURDATE()`                        | 返回当前日期                                                 |                                                              |
| `CURTIME()`                        | 返回当前时间                                                 |                                                              |
| `UNIX_TIMESTAMP(date)`             | 空参数返回当前时间的时间戳，否则返回输入时间的时间戳         |                                                              |
| `FROM_UNIXTIME(timestamp)`         | 返回时间戳对应的时间                                         |                                                              |
| `MONTHNAME(date)`                  | 返回日期的对应的月份                                         | SELECT MONTHNAME(NOW()); # June                              |
| `DAYNAME(date)`                    | 返回日期的星期几                                             | SELECT DAYNAME(NOW()); # Sunday                              |
| `WEEKDAY(date)`                    | 返回周几，注意，周1是0，周2是1，。。。周日是6                |                                                              |
| `DAYOFWEEK(date)`                  | 返回周几，注意：周日是1，周一是2，。。。周六是7              |                                                              |
| `WEEKOFYEAR(date)`                 | 返回是一年中的第几周                                         |                                                              |
| `DAYOFYEAR(date)`                  | 返回是一年中的第几天                                         |                                                              |
| `DAYOFMONTH(date)`                 | 返回是一月中的第几天                                         |                                                              |
| `ADDDATE(date,INTERVAL expr type)` | 返回与给定日期时间相差INTERVAL时间段的日期时间               | SELECT ADDDATE(NOW(),INTERVAL 2 MONTH)                       |
| `DATEDIFF(date1,date2)`            | 返回date1 - date2的日期间隔天数                              |                                                              |
| `TIMESTAMPDIFF(unit，dt1，dt2)`    | 返回dt2 - dt1的对应单位的间隔                                | `SET @dt = "2025-01-01 0:00:00";`<br />`SET @dt2 = "2025-01-02 3:00:00";`<br />`SELECT TIMESTAMPDIFF(MINUTE,@dt,@dt2);` # 1620 |
| `LAST_DAY(date)`                   | 返回date所在月份的最后一天的日期                             |                                                              |
| `TIME_TO_SEC(time)`                | 将 time 转化为秒并返回结果值。转化的公式为： 小时*3600+分钟*60+秒 |                                                              |
| `SEC_TO_TIME(seconds)`             | 将 seconds 描述转化为包含小时、分钟和秒的时间                |                                                              |
| `DATE_FORMAT(date,format)`         | 按照字符串格式化日期date值                                   | `SELECT DATE_FORMAT(NOW(),"%Y/%m/%d %H:%i:%s"); -- 2026/03/25 12:30:12` |
| `STR_TO_DATE(str, format)`         | 按照字符串format对str进行解析，解析为一个日期                | `SELECT STR_TO_DATE("2026年3月10日 12:23:34","%Y年%m月%d日 %H:%i:%s"); -- 2026-03-10 12:23:34` |

#### 流程控制函数

| 函数名                                                       | 描述                                            |
| ------------------------------------------------------------ | ----------------------------------------------- |
| `IF(value,value1,value2)`                                    | 如果value的值为TRUE，返回value1，否则返回value2 |
| `IFNULL(value1, value2)`                                     | 如果value1不为NULL，返回value1，否则返回value2  |
| `CASE WHEN 条件1 THEN 结果1 WHEN 条件2 THEN 结果2 …. [ELSE resultn] END` |                                                 |
| `CASE expr WHEN 常量值1 THEN 值1 WHEN 常量值1 THEN 值1 …. [ELSE 值n] END` |                                                 |

### 聚合函数

- 聚合函数忽略`null`值
- `COUNT(1)` / `COUNT(*)` 计算表的所有记录数 , `COUNT(字段)`会忽略null值MyISAM引擎中速度 `COUNT(*)` = `COUNT(1)` = `COUNT(字段)` , InnoDB引擎中速度 `COUNT(*)`= `COUNT(1)` > `COUNT(字段)`

## GROUP BY

```sql
-- 当使用ROLLUP时，不能同时使用ORDER BY子句进行结果排序，即ROLLUP和ORDER BY是互相排斥
SELECT
    `department_id`,
    `job_id`,
    COUNT(*) AS "num"
FROM
    employees
GROUP BY
    `department_id`,
    `job_id`
WITH ROLLUP
```

> `GROUPING`：当NULL在聚合时，`GROUPING()`函数返回1 ，否则返回0，用来避免分组依据本身为NULL的情况

```sql
SELECT
    orderYear,
    productLine,
    SUM(orderValue) totalOrderValue,
    GROUPING(orderYear),
    GROUPING(productLine)
FROM
    sales
GROUP BY
    orderYear,
    productline
WITH ROLLUP;

/*
+-----------+------------------+-----------------+---------------------+-----------------------+
| orderYear | productLine      | totalOrderValue | GROUPING(orderYear) | GROUPING(productLine) |
+-----------+------------------+-----------------+---------------------+-----------------------+
|      2013 | Classic Cars     |         5571.80 |                   0 |                     0 |
|      2013 | Motorcycles      |         2440.50 |                   0 |                     0 |
|      2013 | Planes           |         4825.44 |                   0 |                     0 |
|      2013 | Ships            |         5072.71 |                   0 |                     0 |
|      2013 | Trains           |         2770.95 |                   0 |                     0 |
|      2013 | Trucks and Buses |         3284.28 |                   0 |                     0 |
|      2013 | Vintage Cars     |         4080.00 |                   0 |                     0 |
|      2013 | NULL             |        28045.68 |                   0 |                     1 |
|      2014 | Classic Cars     |         8124.98 |                   0 |                     0 |
|      2014 | Motorcycles      |         2598.77 |                   0 |                     0 |
|      2014 | Planes           |         2857.35 |                   0 |                     0 |
|      2014 | Ships            |         4301.15 |                   0 |                     0 |
|      2014 | Trains           |         4646.88 |                   0 |                     0 |
|      2014 | Trucks and Buses |         4615.64 |                   0 |                     0 |
|      2014 | Vintage Cars     |         2819.28 |                   0 |                     0 |
|      2014 | NULL             |        29964.05 |                   0 |                     1 |
|      2015 | Classic Cars     |         5971.35 |                   0 |                     0 |
|      2015 | Motorcycles      |         4004.88 |                   0 |                     0 |
|      2015 | Planes           |         4018.00 |                   0 |                     0 |
|      2015 | Ships            |         3774.00 |                   0 |                     0 |
|      2015 | Trains           |         1603.20 |                   0 |                     0 |
|      2015 | Trucks and Buses |         6295.03 |                   0 |                     0 |
|      2015 | Vintage Cars     |         5346.50 |                   0 |                     0 |
|      2015 | NULL             |        31012.96 |                   0 |                     1 |
|      NULL | NULL             |        89022.69 |                   1 |                     1 |
+-----------+------------------+-----------------+---------------------+-----------------------+
*/
```

> `GROUP_CONCAT`：`GROUP_CONCAT( DISTINCT expression1 ORDER BY expression2 SEPARATOR sep)`

```sql
SELECT department_id,GROUP_CONCAT(DISTINCT job_id) FROM employees GROUP BY department_id

/*
+---------------+-------------------------------+
| department_id | GROUP_CONCAT(DISTINCT job_id) |
+---------------+-------------------------------+
|          NULL | SA_REP                        |
|            10 | AD_ASST                       |
|            20 | MK_MAN,MK_REP                 |
|            30 | PU_CLERK,PU_MAN               |
|            40 | HR_REP                        |
|            50 | SH_CLERK,ST_CLERK,ST_MAN      |
|            60 | IT_PROG                       |
|            70 | PR_REP                        |
|            80 | SA_MAN,SA_REP                 |
|            90 | AD_PRES,AD_VP                 |
|           100 | FI_ACCOUNT,FI_MGR             |
|           110 | AC_ACCOUNT,AC_MGR             |
+---------------+-------------------------------+
*/
```

---

## 子查询

### EXISTS

```sql
SELECT 
    customerNumber, customerName
FROM
    customers
WHERE
    EXISTS( SELECT 
            1
        FROM
            orders
        WHERE
            orders.customernumber = customers.customernumber);
```

---