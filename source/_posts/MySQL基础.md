---
title: MySQL基础
tags:
  - SQL
categories: SQL
description: MySQL基础部分
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/mysql.jpg'
abbrlink: bce68613
---
# MySQL基础

[MySQL教程](https://www.begtut.com/mysql/mysql-tutorial.html)

## 命令行操作

```powershell
# 查看mysql版本
mysql -V
mysql --version
# C:\Program Files\MySQL\MySQL Server 8.4\bin\mysql.exe  Ver 8.4.8 for Win64 on x86_64 (MySQL Community Server - GPL)

# 启动/终止mysql进程(管理员权限下)
net stop mysql84
net start mysql84

# 运行sql文件(进入mysql后)
SOURCE demo.sql;

# 登陆/退出mysql
mysql -h localhost -P 3308 -u root -p "mysql" -e "SHOW TABLES";
quit;
exit;
```

## SQL分类

- DDL（`Data Definition Languages`数据定义语言）：这些语句定义了不同的数据库、表、视图、索引等数据库对象，还可以用来创建、删除、修改数据库和数据表的结构
  
  - 主要的语句关键字包括`CREATE` 、`DROP` 、`ALTER` 等

- DML（`
Data Manipulation Language
  `数据操作语言）：用于添加、删除、更新和查询数据库记录，并检查数据完整性
  - 主要的语句关键字包括`INSERT` 、`DELETE` 、`UPDATE` 、`SELECT` 等。SELECT是SQL语言的基础，最为重要
  
- DCL（`Data Control Language`数据控制语言）：用于定义数据库、表、字段、用户的访问权限和安全级别

  - 主要的语句关键字包括`GRANT` 、`REVOKE` 、`COMMIT` 、`ROLLBACK` 、`SAVEPOINT` 等

## 注释

```sql
-- 单行注释

/*
多行
注释
*/
```

---

# 基础语法

## 执行顺序

`FROM`  > `JOIN | ON` > `WHERE` > `GROUP BY | HAVING` > `SELECT`  > `DISTINCT` > `ORDER BY` > `LIMIT`

> 注意点
>
> 1. 因`WHERE`在`SELECT`前运行，无法在`WHERE`中使用别名
> 2. 标准SQL不允许您在`GROUP BY`子句中使用别名，但MySQL支持此功能

## SELECT

在实例使用中不推荐使用`SELECT *`:

1. `*`返回您可能不使用的列中的数据。这会占用和浪费MySQL数据库服务器和应用程序之间产生不必要的I/O磁盘和网络流量
2. 明确指定列，则结果集更易于预测且更易于管理。想象一下，当您使用`*`并且有人通过添加更多列来更改表时，您将得到与您预期的结果集不同的结果集
3. 可能会将敏感信息暴露给未经授权的用户
4. 可能会导致索引失效

## DISTINCT

`DISTINCT`对于`NULL`的处理：仅保留一个`NULL`值

```sql
WITH t(v) AS (SELECT NULL UNION ALL SELECT NULL UNION ALL SELECT 1)
SELECT DISTINCT v from t -- 结果为NULL和1
```

`GROUP BY`也可以实现去重效果

```sql
-- 以下两种方法结果相同
SELECT DISTINCT col FROM t;

SELECT col FROM t GROUP BY col;
```

`DISTINCT`和聚合函数

```sql
WITH T1 AS (
    SELECT 1 AS `VALUE` UNION ALL
    SELECT 1 UNION ALL
    SELECT 2 UNION ALL
    SELECT 2) -- with 语句结尾不能加 ;

SELECT COUNT(DISTINCT `VALUE`) FROM T1 -- 2
```

## ORDER BY

> 默认为`ASC`升序

```sql
SELECT
 ordernumber,
 orderlinenumber,
 quantityOrdered * priceEach AS subtotal -- 取别名增加可读性
FROM
    orderdetails
ORDER BY
    ordernumber,
    orderLineNumber,
    subtotal; -- ORDER BY运行在SELECT之后,可以使用别名

-- 可以使用表达式
ORDER BY ABS(value) -- 在列名前添加ABS函数
```

> 使用`FIELD`自定义排序

`FIELD`逻辑

```sql
SELECT FIELD("s","s","q","l"); -- 1

SELECT FIELD("s","S","q","l"); -- 1 不区分大小写

SELECT FIELD("s","s","q","l","s"); -- 1 返回第一出现的位置

SELECT FIELD('a','s','q','l') -- 0 不存在返回0
```

```sql
SELECT
	orderNumber, status
FROM
	orders
ORDER BY FIELD(status,
	'In Process',
	'On Hold',
	'Cancelled',
	'Resolved',
	'Disputed','Shipped'); 
```

> 排序字段简写为数字：按照`SELECT`出现字段的顺序对应

```sql
SELECT
 ordernumber,
 orderlinenumber
FROM
    orderdetails
ORDER BY
	1 ASC,
	2 DESC;
```

## LIMIT

```sql
LIMIT 2

-- 等效方法
LIMIT 2 OFFSET 0

LIMIT 0,2
```

---

# 运算符

## NULL值

```sql
SELECT 
  TRUE AND NULL,   -- NULL
  FALSE AND NULL,  -- 0
  TRUE OR NULL,    -- 1
  NOT NULL;        -- NULL
  
SELECT 1 FROM DUAL WHERE 1 = NULL; -- NULL

SELECT 10 + NULL, 0 * NULL, 8 / NULL; -- 结果均为 NULL

-- 使用IS NULL 或者 安全等于 <=>进行NULL判断
SELECT NULL IS NULL,NULL <=> NULL; -- 1 1

-- NULL值相关函数
SELECT 
  IFNULL(NULL,1), -- 字段为 NULL 时返回替代值，否则返回字段值
  COALESCE(NULL,NULL,1,NULL), -- 返回第一个非 NULL 的值
  NULLIF(1,1), -- 值 1 = 值 2 时返回 NULL，否则返回值1
  ISNULL(NULL); -- 1
```

## 算数运算符

```sql
SELECT
    10 DIV 2, -- 5 DIV为整除,结果取整(不四舍五入)

    10 / 2, -- 5.0000 一个数除以整数后，不管是否能除尽，结果都为一个浮点数

    10 + '1', -- 11

    10 + '1A2', -- 11

    10 + 'A2', -- 10

    "A" + "B", -- 0

    11 % 3 + 11 DIV 3 * 3, -- 11 DIV为结果取整(DIV除数为浮点数结果仍为整数)
    
    1 / 0 -- NULL
```

## 比较运算符

```sql
SELECT 1 = '1A2' ,0 = 'A1' FROM DUAL; -- 1 1

-- 如果等号两边的值一个是整数，另一个是字符串，则MySQL会将字符串转化为数字进行比较

SELECT "A" < "B";  -- 1 字符串的比较

SELECT LEAST('a','b','c'); -- 'a'
```

### BETWEEN运算符

```sql
SELECT
    1 BETWEEN 1 AND 10, -- 1

    10 BETWEEN 1 AND 10, -- 1 包含边界值

    5 BETWEEN 10 AND 1, -- 0

    NULL BETWEEN 1 AND 10, -- NULL

    "M" BETWEEN "A" AND "Z", -- 1

    "m" BETWEEN "A" AND "Z", -- 1

    "2023-06-20" BETWEEN CAST("2023-06-01" AS DATE) AND CONVERT("2023-06-30",DATE) -- 1
```

> `CAST()`函数将任何类型的值转换为具有指定类型的值。目标类型可以是以下类型之一：`BINARY`，`CHAR`，`DATE`，`DATETIME`，`TIME`，`DECIMAL`，`SIGNED`，`UNSIGNED`

### LIKE / RLIKE运算符

- `LIKE`：
  - `%`：匹配0个或多个字符
  - `_`：只能匹配一个字符

```sql
SELECT
"ABC" LIKE "%C", -- 1
"ABC" LIKE "A_C"; -- 1

-- ESCAPE \ 转义符
SELECT "A__B" LIKE "A\_\_B","A__B" LIKE "A@_@_B" ESCAPE "@"; -- 1 1
```

- `REGEXP ` / `RLIKE`

```sql
"^[1-9]\\d{5}(19|20)\\d{2}(0[1-9]|1[0-2])(0[1-9]|[12]\\d|3[01])\\d{3}[0-9Xx]$" -- 身份证正则判断
```

## 逻辑运算符

> `AND`运算优先级高于`OR`

```sql
SELECT true OR false AND false; -- 1
```

---