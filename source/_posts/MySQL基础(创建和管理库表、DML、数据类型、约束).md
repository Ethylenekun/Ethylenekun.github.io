---
title: MySQL基础(创建和管理库表、DML、数据类型、约束)
tags:
  - SQL
categories: SQL
description: MySQL基础(创建和管理库表、DML、数据类型、约束)
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/mysql.jpg'
abbrlink: 18858fac
---
# 创建和管理数据库

## 创建数据库

> `DATABASE`不能改名。一些可视化工具可以改名，是建新库，把所有表复制到新库，再删旧库

```sql
-- 如果不存在,则创建数据库,设定字符集为UTF-8
CREATE DATABASE IF NOT EXISTS project CHARACTER SET "UTF8";
```

## 使用数据库

```sql
-- 展示所有的数据库
SHOW DATABASES;

-- 使用数据库
USE project;

-- 查看所处的数据库
SELECT DATABASE();

-- 查看数据库创建语句
SHOW CREATE DATABASE project;

-- 查看数据库下的表
SHOW TABLES FROM project;
```

## 修改数据库

```sql
 -- 修改数据库的字符集
 ALTER DATABASE project CHARACTER SET "GBK";
```

## 删除数据库

```sql
-- 删除数据库
DROP DATABASE IF EXISTS project;
```

---

# 创建和管理数据表

## 创建表

```sql
-- 创建方式1,声明表引擎为MYISAM
CREATE TABLE t(
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20)
) ENGINE = MYISAM;

-- 修改表引擎为INNODB
ALTER TABLE t ENGINE = INNODB; 

-- 查看表结构
SHOW CREATE TABLE t;
DESC t;

-- 创建方式2,不会复制约束、索引和触发器等
CREATE TABLE emp1 AS (SELECT * FROM employees); -- 复制表的结构和数据
CREATE TABLE emp2 AS (SELECT * FROM employees WHERE 0); -- 创建的emp2是空表

-- 创建方式3,从一个表以及表的所有从属对象复制数据
CREATE TABLE t2 LIKE t1;
INSERT INTO t2 (SELECT * FROM t1);
```

## 修改表

```sql
-- 增加表字段：ALTER TABLE 表名 ADD 【COLUMN】 字段名 字段类型 【FIRST|AFTER 字段名】;
ALTER TABLE employees ADD salary DECIMAL(10,2) FIRST;
ALTER TABLE employees ADD salary DECIMAL(10,2) AFTER emp_name;

-- 删除表字段：ALTER TABLE 表名 DROP 【COLUMN】字段名
ALTER TABLE employees DROP salary;

-- 重命名字段：ALTER TABLE 表名 CHANGE 【column】 列名 新列名 新数据类型;
ALTER TABLE employees CHANGE salary emp_sal DECIMAL(10,2);

-- 修改表字段：ALTER TABLE 表名 MODIFY 【COLUMN】 字段名1 字段类型 【DEFAULT 默认值】【FIRST|AFTER 字段名2】;
ALTER TABLE employees MODIFY salary DECIMAL(10,2) NOT NULL FIRST;

-- 重命名表
ALTER TABLE t RENAME TO tmp;
RENAME TABLE tmp TO t;

RENAME TABLE a TO a1,b TO b1; -- 重命名多个表, 若其中一个表出现错误则全部回滚
```

> 视图重命名：`RENAME TABLE v1 TO v2;`

## 删除表

```sql
DROP TABLE IF EXISTS emp;
```

## 清空表

```sql
TRUNCATE TABLE emp;
```

- `TRUNCATE`语句不能回滚，而使用 `DELETE` 语句删除数据，可以回滚

```sql
SELECT @@GLOBAL.AUTOCOMMIT; -- 全局的自动提交默认为1

SET autocommit = FALSE;

DELETE FROM emp2;
-- TRUNCATE TABLE emp2;

SELECT * FROM emp2;

ROLLBACK;
SELECT * FROM emp2;
```

---

# 数据的增删改

## 插入INSERT

### VALUES的方式添加

```sql
-- 1.为表的所有字段按默认顺序插入数据
INSERT INTO emp VALUES(2,"B",3),(3,"C",4);

-- 2.为表的指定字段插入数据
INSERT INTO emp(id,name,hire_date) VALUES(4,"D",CURDATE()),(5,"E","2000-01-01");

-- 3.若字段设置有默认值
INSERT INTO emp(dept) VALUES(DEFAULT);
```

### 将查询结果插入

```sql
INSERT INTO emp(id,name,hire_date) (SELECT 7,"G",CURDATE());
```

###  INSERT IGNORE：忽略会导致错误的行

```sql
-- id字段有UNIQUE约束
INSERT IGNORE INTO t(id) VALUES(1),(1); -- 只会插入一条记录

-- name字段类型为VARCHAR(5)
INSERT IGNORE INTO t(name) VALUES("123456") -- 截断数据,会插入12345

-- id字段类型为INT UNSIGNED
INSERT IGNORE INTO t(id) VALUES(-10) -- 会插入0
```

## 更新UPDATE

```sql
-- 更新多个字段
UPDATE emp SET dept = 20,salary = salary + 100 WHERE id = 1

-- UPDATE和JOIN语法
UPDATE employees INNER JOIN merits USING (performance) SET salary = salary + salary * percentage; 

-- 等同于下方相关子查询
UPDATE employees e SET salary = salary + salary * (SELECT percentage FROM merits m WHERE m.performance = e.performance)

UPDATE employees LEFT JOIN merits USING(performance) SET salary = salary + salary * 0.015 WHERE merits.percentage IS NULL;
```

## 删除DELETE

```sql
DELETE FROM emp WHERE dept = 20;

-- 对表排序后删除前两行
DELETE FROM employees ORDER BY performance DESC LIMIT 2;

DELETE T1,T2 FROM T1 INNER JOIN T2 ON T1.KEY=T2.KEY WHERE CONDITION;
```

> 扩展：计算列（MySQL8.0以上)

```sql
CREATE TABLE t(
	a INT,
	b INT,
	c INT GENERATED ALWAYS AS (a + b) VIRTUAL
);

ALTER TABLE emp ADD deptadd1 INT GENERATED ALWAYS AS (dept + 1) VIRTUAL; -- 添加计算列字段
```

---

# 数据类型

| 类型             | 类型举例                                                     |
| ---------------- | :----------------------------------------------------------- |
| 整数类型         | TINYINT、SMALLINT、MEDIUMINT、INT(或INTEGER)、BIGINT         |
| 浮点类型         | FLOAT、DOUBLE                                                |
| 定点数类型       | DECIMAL                                                      |
| 位类型           | BIT                                                          |
| 日期时间类型     | YEAR、TIME、DATE、DATETIME、TIMESTAMP                        |
| 文本字符串类型   | CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT          |
| 枚举类型         | ENUM                                                         |
| 集合类型         | SET                                                          |
| 二进制字符串类型 | BINARY、VARBINARY、TINYBLOB、BLOB、MEDIUMBLOB、LONGBLOB      |
| JSON类型         | JSON对象、JSON数组                                           |
| 空间数据类型     | 单值：GEOMETRY、POINT、LINESTRING、POLYGON；集合：MULTIPOINT、MULTILINESTRING、MULTIPOLYGON、GEOMETRYCOLLECTION |

## 整数类型

| 整数类型    | 字节 | 有符号数取值范围                         | 无符号数取值范围       |
| ----------- | ---- | ---------------------------------------- | ---------------------- |
| `TINYINT`   | 1    | -128~127                                 | 0~255                  |
| `SMALLINT`  | 2    | -32768~32767                             | 0~65535                |
| `MEDIUMINT` | 3    | -8388608~8388607                         | 0~16777215             |
| `INT`       | 4    | -2147483648~2147483647                   | 0~4294967295           |
| `BIGINT`    | 8    | -9223372036854775808~9223372036854775807 | 0~18446744073709551615 |

`UNSIGNED`：无符号类型（非负），所有的整数类型都有一个可选的属性`UNSIGNED`（无符号属性），无符号整数类型的最小取值为0。所以，如果需要在MySQL数据库中保存非负整数值时，可以将整数类型设置为无符号类型

## 浮点类型

```sql
-- 浮点数存在精度误差，二进制不一定能精确表达小数
CREATE TABLE IF NOT EXISTS emp(
    value FLOAT
 );

INSERT INTO emp(value) VALUES(0.47),(0.44),(0.19);

SELECT SUM(value) FROM emp; -- 1.0999999940395355

CREATE TABLE IF NOT EXISTS emp(
    value DOUBLE
);

SELECT SUM(value) FROM emp; -- 1.0999999999999999
```

## 定点类型

| 数据类型       | 字节数  | 含义               |
| -------------- | ------- | ------------------ |
| `DECIMAL(M,D)` | M+2字节 | 有效范围由M和D决定 |

- 使用 `DECIMAL(M,D)`的方式表示高精度小数。其中，`M`被称为精度，`D`被称为标度。0<=`M`<=65，0<=`D`<=30，`D`<`M`。例如，定义`DECIMAL(5,2)`的类型，表示该列取值范围是-999.99~999.99
- `DECIMAL` 的存储空间并不是固定的，由精度值M决定，总共占用的存储空间为M+2个字节
- 定点数在MySQL内部是以字符串的形式进行存储，这就决定了它一定是精准的
- 当`DECIMAL`类型不指定精度和标度时，其默认为`DECIMAL(10,0)`。当数据的精度超出了定点数类型的精度范围时，则MySQL同样会进行四舍五入处理

## 日期与时间类型

| 类型名称    | 名称     | 字节 | 日期格式            | 最小值                  | 最大值                 |
| ----------- | -------- | ---- | ------------------- | ----------------------- | ---------------------- |
| `YEAR`      | 年       | 1    | YYYY或YY            | 1901                    | 2155                   |
| `TIME`      | 时间     | 3    | HH:MM:SS            | -838:59:59              | 838:59:59              |
| `DATE`      | 日期     | 3    | YYYY-MM-DD          | 1000-01-01              | 9999-12-03             |
| `DATETIME`  | 日期时间 | 8    | YYYY-MM-DD HH:MM:SS | 1000-01-01 00:00:00     | 9999-12-31 23:59:59    |
| `TIMESTAMP` | 日期时间 | 4    | YYYY-MM-DD HH:MM:SS | 1970-01-01 00:00:00 UTC | 2038-01-19 03:14:07UTC |

### 设置日期格式的默认值

> `MySQL 8.0.13` 及以上版本可行

```sql
CREATE TABLE emp(
	id INT PRIMARY KEY,
	name VARCHAR(20),
	hire_date DATE DEFAULT (CURRENT_DATE) -- DEFAULT (CURDATE())也可以
);

INSERT INTO emp(id,name,hire_date) VALUES(1,"A",DEFAULT);
```

> 更新数据后时间也进行更新

```sql
CREATE TABLE emp(
	id INT,
	dt1 TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	dt2 DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

INSERT INTO emp(id,dt1,dt2) VALUES(1,DEFAULT,DEFAULT);
```

> 使用触发器设置默认时间

```sql
CREATE TABLE emp(
	id INT,
	dt DATETIME
);

DELIMITER $
CREATE TRIGGER trig BEFORE INSERT ON emp FOR EACH ROW
BEGIN

	IF NEW.dt IS NULL 
	THEN SET NEW.dt = NOW();
	END IF;

END $
DELIMITER ;
```

## 文本类型

| 字符串(文本)类型 | 特点     | 长度 | 长度范围        |
| ---------------- | -------- | ---- | --------------- |
| `CHAR(M)`        | 固定长度 | M    | 0 <= M <= 255   |
| `VARCHAR(M)`     | 可变长度 | M    | 0 <= M <= 65535 |

`CHAR`类型：

- `CHAR(M)`类型一般需要预先定义字符串长度。如果不指定(M)，则表示长度默认是1个字符
- 如果保存时，数据的实际长度比`CHAR`类型声明的长度小，则会在右侧填充空格以达到指定的长度。当MySQL检索CHAR类型的数据时，CHAR类型的字段会去除尾部的空格(**若插入字符串本身右侧就有空格，检索时则会去除空格**)
- 定义`CHAR`类型字段时，声明的字段长度即为`CHAR`类型字段所占的存储空间的字节数

`VARCHAR`类型：

- `VARCHAR(M)`定义时， 必须指定长度M，否则报错
- MySQL4.0版本以下，`varchar(20)`：指的是20字节，如果存放UTF8汉字时，只能存6个（每个汉字3字节） ；MySQL5.0版本以上，`varchar(20)`：指的是20字符
- 检索`VARCHAR`类型的字段数据时，会保留数据尾部的空格。`VARCHAR`类型的字段所占用的存储空间为字符串实际长度加1个字节

### ENUM类型

| 文本字符串类型 | 长度 | 长度范围       |
| -------------- | ---- | -------------- |
| ENUM           | L    | 1<= L <= 65535 |

- 当`ENUM`类型包含1～255个成员时，需要1个字节的存储空间；
- 当`ENUM`类型包含256～65535个成员时，需要2个字节的存储空间。
- `ENUM`类型的成员个数的上限为65535个

```sql
CREATE TABLE IF NOT EXISTS t(
    e ENUM("3","4","1","2")
);

INSERT INTO t(e) VALUES(1),('1');

SELECT e FROM t; # "3" , "1" -- 按照索引获取枚举值
```

### SET类型

> `SET`表示一个字符串对象，可以包含0个或多个成员，但成员个数的上限为64 。设置字段值时，可以取取值范围内的 0 个或多个值。
>
> - 如果插入 `SET` 字段中的列值有重复，则 MySQL 自动删除重复的值；
> - 插入 `SET`字段的值的顺序并不重要，MySQL 会在存入数据库时，按照定义的顺序显示；

```sql
CREATE TABLE IF NOT EXISTS t(
    e SET('A','B','C')
);
```

## JSON类型

```sql
CREATE TABLE test_json(
    js json
);

INSERT INTO test_json (js) VALUES ('{"name":"songhk", "age":18, "address":{"province":"beijing","city":"beijing"}}');

SELECT js -> '$.name' AS NAME,js -> '$.age' AS age ,js -> '$.address.province' AS province, js -> '$.address.city' AS city FROM test_json;
```

---

# 约束

根据约束起的作用，约束可分为：

- `NOT NULL` 非空约束，规定某个字段不能为空
- `UNIQUE` 唯一约束，规定某个字段在整个表中是唯一的
- `PRIMARY KEY` 主键(非空且唯一)约束
- `FOREIGN KEY` 外键约束
- `CHECK` 检查约束
- `DEFAULT` 默认值约束

```sql
-- 查看表具有的约束
SELECT * FROM information_schema.TABLE_CONSTRAINTS WHERE TABLE_NAME = "employees" ;
```

## 非空约束(NOT NULL)

```sql
-- 建表时
CREATE TABLE 表名称(
字段名 数据类型,
字段名 数据类型 NOT NULL,
字段名 数据类型 NOT NULL);

-- 建表后
Alter table 表名称 MODIFY 字段名 数据类型 NOT NULL;

-- 删除非空约束
Alter table 表名称 MODIFY 字段名 数据类型 NULL; -- 去掉not null，相当于修改某个非注解字段，该字段允许为空

Alter table 表名称 MODIFY 字段名 数据类型; -- 去掉not null，相当于修改某个非注解字段，该字段允许为空
```

## 唯一约束(UNIQUE)

- 同一个表可以有多个唯一约束
- 唯一约束可以是某一个列的值唯一，也可以多个列组合的值唯一
- 唯一性约束允许列值为空（**多个NULL值不会触发唯一约束**）
- 在创建唯一约束的时候，如果不给唯一约束命名，就默认和列名相同（**若为多列，约束名为第一个列**）
- MySQL会给唯一约束的列上默认创建一个唯一索引

> 创建唯一约束

```sql
-- 建表时
CREATE TABLE IF NOT EXISTS test(
    name VARCHAR(20),
    pwd VARCHAR(20) UNIQUE -- 列级约束
);

CREATE TABLE IF NOT EXISTS test(
    name VARCHAR(20),
    pwd VARCHAR(20),
    CONSTRAINT uk_name_pwd UNIQUE(name,pwd) -- 表级约束
);

-- 建表后
ALTER TABLE test ADD UNIQUE(id);

ALTER TABLE test MODIFY id INT UNIQUE;

ALTER TABLE test ADD CONSTRAINT uk_id_pwd UNIQUE(id,pwd);
```

> 删除唯一约束

```sql
SHOW INDEX FROM test; -- 查看表的索引

ALTER TABLE test DROP INDEX uk_id_pwd; -- 删除唯一索引

ALTER TABLE test DROP CONSTRAINT uk_id_pwd; -- 删除唯一约束
```

## 主键约束(PRIMARY)

- 一个表最多只能有一个主键约束
- 主键约束对应着表中的一列或者多列（复合主键）
- 如果是多列组合的复合主键约束，那么这些列都不允许为空值，并且组合的值不允许重复
- MySQL的主键名总是**PRIMARY**，就算自己命名了主键约束名也没用
- 当创建主键约束时，系统默认会在所在的列或列组合上建立对应的主键索引（能够根据主键查询的，就根据主键查询，效率更高）。删除主键约束，主键约束对应的索引就自动删除
- 需要注意的一点是，不要修改主键字段的值。因为主键是数据记录的唯一标识，如果修改了主键的值，就有可能会破坏数据的完整性

```sql
-- 建表时
CREATE TABLE IF NOT EXISTS test(
    id INT PRIMARY KEY, -- 列级约束
    pwd VARCHAR(20));

CREATE TABLE IF NOT EXISTS test(
    id INT,
    pwd VARCHAR(20),
    PRIMARY KEY(id,pwd) -- 表级约束);

-- 建表后
ALTER TABLE test ADD PRIMARY KEY(id,pwd);

-- 删除主键
ALTER TABLE test DROP PRIMARY KEY;
```

> 自增列(`AUTO_INCREMENT`)：MySQL 8.0将自增主键的计数器持久化到重做日志中。每次计数器发生改变，都会将其写入重做日志中。如果数据库重启，InnoDB会根据重做日志中的信息来初始化计数器的内存值
>
> - 默认情况下，`AUTO_INCREMENT` 的初始值是 1，每新增一条记录，字段值自动加 1
> - 一个表中只能有一个字段使用 `AUTO_INCREMENT`约束，且该字段必须有唯一索引，以避免序号重复（即为主键或主键的一部分）
> - `AUTO_INCREMENT` 约束的字段必须具备 `NOT NULL` 属性
> - `AUTO_INCREMENT`约束的字段只能是整数类型
> - `AUTO_INCREMENT`约束字段的最大值受该字段的数据类型约束，如果达到上限，`AUTO_INCREMENT`就会失效
> - 如果自增列指定了 0 和NULL，会在当前最大值的基础上自增；如果自增列手动指定了具体值，直接赋值为具体值

```sql
-- id为自增列
INSERT INTO t(name) VALUES("A"),("B"); -- 自动添加id 1和2

DELETE FROM t WHERE id = 2;
INSERT INTO t(name) VALUES("C"); -- 自动添加id 3

UPDATE t SET id = 10 WHERE id = 3;
INSERT INTO t(name) VALUES("D"); -- 自动添加id 11
```

## 外键约束(FOREIGN)

- 从表的外键列，必须引用/参考主表的主键或唯一约束的列
- 在创建外键约束时，如果不给外键约束命名，默认名不是列名，而是自动产生一个外键名（例如`student_ibfk_1`），也可以指定外键约束名
- 创建(`CREATE`)表时就指定外键约束的话，先创建主表，再创建从表
- 删表时，先删从表（或先删除外键约束），再删除主表
- 当主表的记录被从表参照时，主表的记录将不允许删除，如果要删除数据，需要先删除从表中依赖该记录的数据，然后才可以删除主表的数据
- 在“从表”中指定外键约束，并且一个表可以建立多个外键约束
- 从表的外键列与主表被参照的列名字可以不相同，但是数据类型必须一样，逻辑意义一致。如果类型不一样，创建子表时，就会出现错误`ERROR 1005 (HY000): Can't createtable'database.tablename'(errno: 150)`
- 当创建外键约束时，系统默认会在所在的列上建立对应的普通索引。但是索引名是外键的约束名（根据外键查询效率很高）
- 删除外键约束后，必须手动删除对应的索引

```sql
CREATE TABLE IF NOT EXISTS dept(
    id INT PRIMARY KEY,
    name VARCHAR(10)
);

CREATE TABLE IF NOT EXISTS emp(
    id INT PRIMARY KEY,
    name VARCHAR(10),
    dept_id INT,
    CONSTRAINT fk_emp_dept FOREIGN KEY(dept_id) REFERENCES dept(id)
);

-- 建表后添加外键约束
ALTER TABLE emp ADD CONSTRAINT fk_dept_id FOREIGN KEY(dept_id) REFERENCES dept(id) ON DELETE RESTRICT ON UPDATE CASCADE; 
```

### 约束等级

- `Cascade`方式：在父表上update/delete记录时，同步update/delete掉子表的匹配记录
- `Set null`方式：在父表上update/delete记录时，将子表上匹配记录的列设为`null`，但是要注意子表的外键列不能为`not null`
- `No action`方式：如果子表中有匹配的记录，则不允许对父表对应候选键进行update/delete操作
- `Restrict`方式：同`no action`， 都是立即检查外键约束
- `Set default`方式：父表有变更时，子表将外键列设置成一个默认的值，但`Innodb`不能识别

> 如果没有指定等级，就相当于Restrict方式。 对于外键约束，最好是采用: `ON UPDATE CASCADE ON DELETE RESTRICT` 的方式。

```sql
-- 第一步先查看约束名和删除外键约束
SELECT * FROM information_schema.table_constraints WHERE table_name = '表名称';
ALTER TABLE 从表名 DROP CONSTRAINT 外键约束名;

-- 第二步查看索引名和删除索引。（注意，只能手动删除）
SHOW INDEX FROM 表名称; -- 查看某个表的索引名
ALTER TABLE 从表名 DROP INDEX 索引名;
```

## 默认值约束(DEFAULT)

```sql
value INT DEFAULT 200;
```

## 检查约束(CHECK)

```sql
value INT CHECK(value > 10 and value < 20);

value CHAR(1) CHECK("男" OR "女");

-- 表级约束
CREATE TABLE IF NOT EXISTS t(
	a INT,
	b INT,
	c INT,
	CONSTRAINT ck_a_b CHECK(a+b<10 and a>0 and b>0 and a+b>c)
);
```

> 扩展：使用存储过程和触发器实现

```sql
-- price始终大于或等于cost
CREATE TABLE IF NOT EXISTS parts (
    part_no VARCHAR(18) PRIMARY KEY,
    description VARCHAR(40),
    cost DECIMAL(10 , 2 ) NOT NULL,
    price DECIMAL(10,2) NOT NULL
); 

-- 检查cost和price列中值的存储过程
DELIMITER $
CREATE PROCEDURE `check_parts`(IN cost DECIMAL(10,2), IN price DECIMAL(10,2))
BEGIN
    IF cost < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'check constraint on parts.cost failed';
    END IF;
    
    IF price < 0 THEN
 		SIGNAL SQLSTATE '45001'
    	SET MESSAGE_TEXT = 'check constraint on parts.price failed';
    END IF;
    
    IF price < cost THEN
 		SIGNAL SQLSTATE '45002'
        SET MESSAGE_TEXT = 'check constraint on parts.price & parts.cost failed';
    END IF;
END $
DELIMITER ; 

-- 使用触发器调用存储过程
-- before insert
DELIMITER $
CREATE TRIGGER `parts_before_insert` BEFORE INSERT ON `parts` FOR EACH ROW
BEGIN
    CALL check_parts(new.cost,new.price);
END$   
DELIMITER ; 

-- before update
DELIMITER $
CREATE TRIGGER `parts_before_update` BEFORE UPDATE ON `parts` FOR EACH ROW
BEGIN
    CALL check_parts(new.cost,new.price);
END$   
DELIMITER ; 
```

---
