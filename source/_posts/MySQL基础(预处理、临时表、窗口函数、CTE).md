---
title: MySQL基础(预处理、临时表、窗口函数、CTE)
tags:
  - SQL
categories: SQL
description: MySQL基础(预处理、临时表、窗口函数、CTE)
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/mysql.jpg'
abbrlink: 2c97f835
---
# PREPARE预处理

- `PREPARE` - 准备要执行的语句
- `EXECUTE` - 执行由`PREPARE`语句准备的预准备语句
- `DEALLOCATE PREPARE` - 解除声明

```sql
-- 在 PREPARE 语句中，占位符 ? 只能用于数据值，不能用于表名、列名等标识符

PREPARE stmt FROM 'SELECT ? + ? * ?';
 
SET @a = 2;
SET @b = 3;
SET @c = 4;

-- 等同于SELECT 1,2,3 INTO @a,@b,@c;

EXECUTE stmt USING @a,@b,@c;
 
DEALLOCATE PREPARE stmt; 
```

> 示例：根据匹配模式批量删除表

```sql
--  根据匹配模式删除对应数据库的表

DELIMITER $

CREATE PROCEDURE drop_tables_by_pattern(
    IN pattern VARCHAR(255),
    IN db_name VARCHAR(255)
)
BEGIN
    DECLARE table_count INT;

    -- 检查是否存在匹配表
    SELECT COUNT(*) INTO table_count 
    FROM information_schema.TABLES 
    WHERE TABLE_SCHEMA = db_name 
    AND TABLE_NAME LIKE pattern;

    IF table_count > 0 THEN 
        SELECT CONCAT("DROP TABLE ", 
            GROUP_CONCAT(CONCAT(TABLE_SCHEMA, ".", TABLE_NAME)), ";") 
        INTO @dynamic_sql
        FROM information_schema.TABLES 
        WHERE TABLE_SCHEMA = db_name 
        AND TABLE_NAME LIKE pattern;
        
        PREPARE stmt FROM @dynamic_sql;
        EXECUTE stmt;
        DEALLOCATE PREPARE stmt;
    ELSE
        SELECT 'No tables matched' AS result;
    END IF;
END $

DELIMITER ;
```

---

# TEMPORARY TABLE临时表

```sql
CREATE TEMPORARY TABLE t AS (SELECT 1);

DROP TEMPORARY TABLE t;
```

- 会话结束或连接终止时，MySQL会自动删除临时表。当然，您可以使用 `DROP TEMPORARY TABLE`语句在不再使用它时显式删除临时表
- 临时表仅可供创建它的客户端访问。不同的客户端可以创建具有相同名称的临时表而不会导致错误，因为只有创建临时表的客户端才能看到它。但是，在同一会话中，两个临时表不能共享相同的名称
- 临时表可以与数据库中的普通表具有相同的名称。例如，如果创建一个名为`employees`临时表为示例数据库，现有的`employees`表变得不可访问。您针对`employees`表发出的每个查询现在都引用临时 `employees` 表。删除`employees`临时表时，`employees`表可用并可再次访问

|   **特性**   |               **CTE**                |             **临时表**             |
| :----------: | :----------------------------------: | :--------------------------------: |
| **存储方式** |         逻辑结构，不物化数据         |  物理存储于tempdb或数据库临时空间  |
| **生命周期** |       仅在定义它的查询块内有效       |        会话级或批处理级存在        |
| **索引支持** |            不支持显式索引            |  支持创建聚集/非聚集索引优化查询   |
| **性能特性** | 适合单次引用的小数据，递归计算更高效 | 适合大数据多次引用，统计信息更准确 |
| **事务支持** |            受外层事务影响            |     完全支持事务（如回滚操作）     |
| **典型应用** |        递归查询、查询逻辑分层        |  跨多步骤的数据加工、复杂ETL流程   |

---

# CTE公共表达式

```sql
WITH 
	t1(n1,n2) AS (SELECT 1,2),
	t2(n1,n2) AS (SELECT n1 + 1 ,n2 + 1 FROM t1)
SELECT * FROM t1 UNION ALL SELECT * FROM t2;

-- 1. 涉及两个CTE表t1和t2,t2又是由t1产生的
-- 2. cte表中声明表的字段名
```

> 递归CTE：生成一整个月的日期表

```sql
WITH RECURSIVE t(d) AS (
	SELECT "2025-01-01"
	UNION ALL
	SELECT ADDDATE(d,INTERVAL 1 DAY) FROM t WHERE d < LAST_DAY(d)
) 
SELECT * FROM t;
```

# 窗口函数

| 姓名             | 描述                                   |
| :--------------- | :------------------------------------- |
| `CUME_DIST()`    | 累计分配值(小于或等于当前价格的比例)   |
| `DENSE_RANK()`   | 当前行在其分区内的排名，没有间隙       |
| `FIRST_VALUE()`  | 窗口框架第一行的参数值                 |
| `LAG()`          | 分区内滞后当前行的行的参数值           |
| `LAST_VALUE()`   | 窗口框架最后一行的参数值               |
| `LEAD()`         | 分区内当前行前导行的参数值             |
| `NTH_VALUE()`    | 来自窗口框架第 N 行的参数值            |
| `NTILE()`        | 当前行在其分区内的桶号。               |
| `PERCENT_RANK()` | 百分比排名值 `(rank - 1) / (rows - 1)` |
| `RANK()`         | 当前行在其分区内的排名，有间隙         |
| `ROW_NUMBER()`   | 其分区内的当前行数                     |

> `ORDER BY`的影响
>
> - **当使用`ORDER BY`时**：
>
>   默认窗口框架为 **`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`**
>
>   窗口包含从分区开始到当前行的所有行，按`ORDER BY`排序，并包括与当前行具有相同排序值的所有行
>
> - **当不使用`ORDER BY`时**：
>
>   默认窗口框架为 **`ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`**
>
>   窗口包含整个分区的所有行

```sql
WITH sales(sale_date,amount) AS (
	SELECT CAST("2023-01-01" AS DATE),100 
	UNION ALL 
	SELECT CAST("2023-01-01" AS DATE),100  
	UNION ALL 
	SELECT CAST("2023-01-02" AS DATE),300 
	UNION ALL 
	SELECT CAST("2023-01-03" AS DATE),200 
	UNION ALL 
	SELECT CAST("2023-01-04" AS DATE),400
)

SELECT *,SUM(amount) OVER(ORDER BY sale_date),SUM(amount) OVER() FROM sales;

/*
2023-01-01	100	200	 1100
2023-01-01	100	200	 1100
2023-01-02	300	500	 1100
2023-01-03	200	700	 1100
2023-01-04	400	1100 1100
*/
```

## 窗口框架

**以下函数受到当前框架的影响**

- 聚合函数
- `FIRST_VALUE()`
- `LAST_VALUE()`
- `NTH_VALUE()`

**单位**

- `CURRENT ROW`：当前行
- `n PRECEDING`：往前n行数据
- `n FOLLOWING`：往后n行数据
- `UNBOUNDED PRECEDING`： 表示从前面的起点
- `UNBOUNDED FOLLOWING`：表示到后面的终点

### ROWS和RANGE的区别

| **模式**  | **原理**       | **相同排序值的处理**         | **适用场景**                |
| --------- | -------------- | ---------------------------- | --------------------------- |
| **ROWS**  | 基于物理行偏移 | 逐行计算，不合并相同值       | 需要明确的行范围（如前N行） |
| **RANGE** | 基于逻辑值范围 | 合并相同排序值的行，统一计算 | 按值分组计算（如相同日期）  |

`ROWS`：基于行号的窗口框架，可以是固定的行数（如 `ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING`），也可以是相对于当前行的位置（如 `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`）

`RANGE`：基于值的范围的窗口框架，它允许你基于列的值来定义窗口的大小。例如，`RANGE BETWEEN 0 PRECEDING AND 0 FOLLOWING` 会创建一个窗口，其中包含当前行和与当前行具有相同值的行

```sql
WITH sales(sale_date,amount) AS (
	SELECT CAST("2023-01-01" AS DATE),100 
	UNION ALL 
	SELECT CAST("2023-01-01" AS DATE),100  
	UNION ALL 
	SELECT CAST("2023-01-02" AS DATE),300 
	UNION ALL 
	SELECT CAST("2023-01-03" AS DATE),200 
	UNION ALL 
	SELECT CAST("2023-01-04" AS DATE),400
)

SELECT 
  sale_date,
  amount,
  SUM(amount) OVER (
    ORDER BY sale_date
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total_range,
	SUM(amount) OVER (
    ORDER BY sale_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total_rows
FROM sales;

/*
2023-01-01	100	200	 100
2023-01-01	100	200	 200
2023-01-02	300	500	 500
2023-01-03	200	700	 700
2023-01-04	400	1100 1100
*/
```

> `RANGE`可以对日期进行移动，根据`ORDER BY`的字段

```sql
SELECT 
	sale_date, 
	amount,
	SUM(amount) OVER(ORDER BY sale_date RANGE BETWEEN INTERVAL 1 DAY PRECEDING AND INTERVAL 1 DAY FOLLOWING) AS sum_range,
	SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING) AS sum_rows
FROM sales;		
/*
2023-01-01	100	500	200
2023-01-01	100	500	500
2023-01-02	300	700	600
2023-01-03	200	900	900
2023-01-04	400	600	600
*/
```

---

## 窗口别名

```sql
SELECT id, category, NAME, price,LAG(price,1) OVER w AS pre_price
FROM goods
WINDOW w AS (PARTITION BY category_id ORDER BY price);
```

---

