---
title: MySQL基础(视图、存储过程、函数、流程控制、变量)
tags:
  - SQL
categories: SQL
description: MySQL基础(视图、存储过程、函数、流程控制、变量)
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/mysql.jpg'
swiper_index: 7
abbrlink: 6b9632bf
---
# 视图

## 创建视图

```sql
CREATE VIEW v1 AS (SELECT 1 AS "value");
-- 可在VIEW内定义字段名称
CREATE VIEW v1(value) AS (SELECT 1);

-- 基于视图创建视图
CREATE VIEW v2 AS (SELECT value + 1 FROM v1);
```

## 查看视图

```sql
SHOW TABLES; --  可以查看所有的表和视图,但不能区分

SHOW FULL TABLES; -- 可以看到table_type
```

## 修改视图

```sql
CREATE VIEW v1 AS (SELECT 1 AS "value");

CREATE OR REPLACE VIEW v1 AS (SELECT 2 AS "value");

ALTER VIEW v1 AS (SELECT 3 AS "value");
```

## 删除视图

```sql
DROP VIEW IF EXISTS v1,v2;
```

---

# 存储过程

## 参数

- `IN`：当前参数为输入参数，也就是表示入参；存储过程只是读取这个参数的值。如果没有定义参数种类， 默认就是 IN ，表示输入参数
- `OUT`：当前参数为输出参数，也就是表示出参；执行完成之后，调用这个存储过程的客户端或者应用程序就可以读取这个参数返回的值了
- `INOUT`：当前参数既可以为输入参数，也可以为输出参数

## 调用存储过程

```sql
-- 调用IN模式参数
CALL proc(IN_VALUE);

-- 调用OUT模式参数
CALL proc(@OUT_VALUE);
SELECT @OUT_VALUE;

-- 调用INOUT模式参数
SET @INOUT_VALUE = 1;
CALL proc(@INOUT_VALUE);
SELECT @INOUT_VALUE;
```

> 示例：创建存储过程，实现累加运算，计算 1+2+…+n 等于多少

```sql
DROP PROCEDURE IF EXISTS proc;

DELIMITER $

CREATE PROCEDURE proc(IN n INT,OUT m INT)
BEGIN

	DECLARE i INT DEFAULT 1;
	DECLARE result INT DEFAULT 0;
	
	WHILE i <= n DO
		SET result = result + i;
		SET i = i + 1;
	END WHILE;
	
	SELECT result INTO m;

END $
DELIMITER ;

CALL proc(100,@result);
SELECT @result; -- 5050
```

# 函数

> `you might want to use the less safe log_bin_trust_function_creators variable`报错时需要添加`READS SQL DATA`

```sql
CREATE FUNCTION 函数名(参数名 参数类型,...)
RETURNS 返回值类型
[characteristics ...]
BEGIN
函数体 --函数体中肯定有 RETURN 语句
END
```

> 示例：创建函数，实现累加运算，计算 1+2+…+n 等于多少

```sql
DROP FUNCTION IF EXISTS func;

DELIMITER $

CREATE FUNCTION func(n INT)
RETURNS INT
READS SQL DATA
BEGIN

	DECLARE i INT DEFAULT 1;
	DECLARE result INT DEFAULT 0;
	
	WHILE i <= n DO
		SET result = result + i;
		SET i = i + 1;
	END WHILE;
	
	RETURN result;

END $
DELIMITER ;

SELECT func(100); -- 5050
```

---

# 变量

## 系统变量

```sql
-- 查看所有的全局系统变量
SHOW GLOBAL VARIABLES;

-- 查看所有的会话系统变量
SHOW SESSION VARIABLES;
SHOW VARIABLES;
```

### 查看系统变量

```sql
-- 查看满足条件的部分系统变量。
SHOW GLOBAL VARIABLES LIKE '%标识符%';

-- 查看指定系统变量
SELECT @@GLOBAL.变量名;

SELECT @@SESSION.变量名;
SELECT @@变量名;
```

### 修改系统变量

```sql
-- 方式1：
SET @@global.变量名=变量值;
-- 方式2：
SET GLOBAL 变量名=变量值;

-- 方式1：
SET @@session.变量名=变量值;
-- 方式2：
SET SESSION 变量名=变量值;
```

## 用户变量

```sql
-- 会话用户变量
SET @num = 10;
SELECT @num ; -- 10

SELECT NOW() INTO @dt FROM DUAL;
SELECT @dt;

-- 局部变量一般用于PROCEDURE和FUNCTION
DELIMITER //
CREATE PROCEDURE add_value()
BEGIN
    -- 局部变量
    DECLARE m INT DEFAULT 1;
    DECLARE n INT DEFAULT 3;
    DECLARE SUM INT;
    SET SUM = m+n;
    SELECT SUM;
END //
DELIMITER ;
```

# 定义条件与处理程序

定义：**定义条件**是事先定义程序执行过程中可能遇到的问题， **处理程序**定义了在遇到问题时应当采取的处理方式，并且保证存储过程或函数在遇到警告或错误时能继续执行

## 定义条件

- `MySQL_error_code`和`sqlstate_value` 都可以表示MySQL的错误
  - `MySQL_error_code`是数值类型错误代码
  - `sqlstate_value`是长度为5的字符串类型错误代码
- 例如，在`ERROR 1418 (HY000)`中，1418是`MySQL_error_code`，’HY000’是`sqlstate_value`。
- 例如，在`ERROR 1142（42000）`中，1142是`MySQL_error_code`，’42000’是`sqlstate_value`。

```sql
-- 定义语法
DECLARE 错误名称 CONDITION FOR 错误码（或错误条件）

-- 示例
-- 使用MySQL_error_code
DECLARE Field_Not_Be_NULL CONDITION FOR 1048;
-- 使用sqlstate_value
DECLARE Field_Not_Be_NULL CONDITION FOR SQLSTATE '23000';
```

### 处理程序

- 处理方式：处理方式有3个取值：`CONTINUE`、`EXIT`
  - `CONTINUE`：表示遇到错误不处理，继续执行。
  - `EXIT` ：表示遇到错误马上退出。
- 错误类型（即条件）可以有如下取值：
  - `SQLSTATE '字符串错误码'` ：表示长度为5的sqlstate_value类型的错误代码；
  - `MySQL_error_code` ：匹配数值类型错误代码；
  - 错误名称：表示`DECLARE ... CONDITION`定义的错误条件名称。
  - `SQLWARNING` ：匹配所有以01开头的`SQLSTATE`错误代码；
  - `NOT FOUND` ：匹配所有以02开头的`SQLSTATE`错误代码；
  - `SQLEXCEPTION` ：匹配所有没有被`SQLWARNING`或`NOT FOUND`捕获的`SQLSTATE`错误代码；
- 处理语句：如果出现上述条件之一，则采用对应的处理方式，并执行指定的处理语句。语句可以是 像`SET 变量 = 值`这样的简单语句，也可以是使用`BEGIN ... END` 编写的复合语句。

```sql
-- 方法1：捕获sqlstate_value
DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02' SET @info = 'NO_SUCH_TABLE';

-- 方法2：捕获mysql_error_value
DECLARE CONTINUE HANDLER FOR 1146 SET @info = 'NO_SUCH_TABLE';

-- 方法3：先定义条件，再调用
DECLARE no_such_table CONDITION FOR 1146;
DECLARE CONTINUE HANDLER FOR NO_SUCH_TABLE SET @info = 'NO_SUCH_TABLE';

-- 方法4：使用SQLWARNING
DECLARE EXIT HANDLER FOR SQLWARNING SET @info = 'ERROR';

-- 方法5：使用NOT FOUND
DECLARE EXIT HANDLER FOR NOT FOUND SET @info = 'NO_SUCH_TABLE';

-- 方法6：使用SQLEXCEPTION
DECLARE EXIT HANDLER FOR SQLEXCEPTION SET @info = 'ERROR';
```

> 示例

```sql
-- 定义处理程序，捕获sqlstate_value值，当遇到sqlstate_value值为23000时，执行EXIT操作，并且将@proc_value的值设置为-1。

DELIMITER //
CREATE PROCEDURE InsertDataWithCondition()
    BEGIN
        DECLARE duplicate_entry CONDITION FOR SQLSTATE '23000' ;
        DECLARE EXIT HANDLER FOR duplicate_entry SET @proc_value = -1;

        SET @x = 1;
        INSERT INTO departments(department_name) VALUES('测试');
        SET @x = 2;
        INSERT INTO departments(department_name) VALUES('测试');
        SET @x = 3;
    END //
DELIMITER ;

mysql> CALL InsertDataWithCondition();
Query OK, 0 rows affected (0.01 sec)
mysql> SELECT @x,@proc_value;
+------+-------------+
| @x | @proc_value |
+------+-------------+
| 2  | -1 |
+------+-------------+
1 row in set (0.00 sec)
```

> 手动报错

```sql
SIGNAL SQLSTATE "HY000" SET MESSAGE_TEXT = "My Error";
```

---

# 流程控制

## 分支结构

### IF

```sql
IF 表达式1 THEN 操作1
[ELSEIF 表达式2 THEN 操作2]……
[ELSE 操作N]
END IF
```

### CASE

```sql
-- 情况一：类似于switch
CASE 表达式
WHEN 值1 THEN 结果1或语句1(如果是语句，需要加分号)
WHEN 值2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）

-- 情况二：类似于多重if
CASE
WHEN 条件1 THEN 结果1或语句1(如果是语句，需要加分号)
WHEN 条件2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）
```

## 循环结构

```sql
/*[loop_label:] LOOP
循环执行的语句
END LOOP [loop_label]*/

DECLARE id INT DEFAULT 0;
add_loop:LOOP
    SET id = id +1;
    IF id >= 10 THEN LEAVE add_loop;
    END IF;
END LOOP add_loop;

/*[while_label:] WHILE 循环条件 DO
循环体
END WHILE [while_label];*/

DECLARE i INT DEFAULT 0;
    WHILE i < 10 DO
    SET i = i + 1;
    END WHILE;
SELECT i;

/*[repeat_label:] REPEAT
　　　　循环体的语句
UNTIL 结束循环的条件表达式
END REPEAT [repeat_label]*/

DECLARE i INT DEFAULT 0;
    REPEAT
    SET i = i + 1;
    UNTIL i >= 10
    END REPEAT;
    SELECT i;
```

## 跳转语句

```sql
-- LEAVE
DELIMITER //
CREATE PROCEDURE leave_begin(IN num INT)
begin_label: BEGIN
    IF num<=0
        THEN LEAVE begin_label;
    ELSEIF num=1
        THEN SELECT AVG(salary) FROM employees;
    ELSEIF num=2
        THEN SELECT MIN(salary) FROM employees;
    ELSE SELECT MAX(salary) FROM employees;
    END IF;

    SELECT COUNT(*) FROM employees;
END //
DELIMITER ;

-- ITERATE
DELIMITER //
CREATE PROCEDURE test_iterate()

BEGIN
    DECLARE num INT DEFAULT 0;
    my_loop:LOOP
        SET num = num + 1;
        IF num < 10
            THEN ITERATE my_loop;
        ELSEIF num > 15
            THEN LEAVE my_loop;
        END IF;

    SELECT '尚硅谷：让天下没有难学的技术';
    END LOOP my_loop;
END //
DELIMITER ;
```

---

# 游标

```sql
-- 1.声明游标：游标声明必须在任何变量声明之后
DECLARE cursor_name CURSOR FOR select_statement;
DECLARE cur_emp CURSOR FOR SELECT employee_id,salary FROM employees;

-- 2.打开游标
OPEN cursor_name;

-- 3.使用游标
FETCH cursor_name INTO var_name [, var_name] ...
FETCH　cur_emp INTO emp_id, emp_sal ;

-- 4.关闭游标
CLOSE cursor_name

-- 5.声明NOT FOUND处理程序以在光标找不到任何行时处理情况
DECLARE CONTINUE HANDLER FOR NOT FOUND SET finished = 1; 
```

> 示例

```sql
DELIMITER $$
CREATE PROCEDURE build_email_list (INOUT email_list VARCHAR (4000))
BEGIN
	DECLARE v_finished INT DEFAULT 0;
	DECLARE v_email VARCHAR (100) DEFAULT "";
	
	DECLARE email_cursor CURSOR FOR SELECT email FROM employees;
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_finished = 1;
	
	OPEN email_cursor;
	
	get_email :LOOP
		FETCH email_cursor INTO v_email;
		
		IF v_finished = 1 THEN
			LEAVE get_email;
		END IF;
		
		SET email_list = CONCAT( v_email, ";", email_list );
	END LOOP get_email;
	
	CLOSE email_cursor;
	
END $$
DELIMITER ; 
```

---

# 触发器

```sql
CREATE TRIGGER 触发器名称
{BEFORE|AFTER} {INSERT|UPDATE|DELETE} ON 表名
FOR EACH ROW
触发器执行的语句块;
```

> 示例：数据变更日志触发器

```sql
CREATE TABLE test(
    id INT
);

CREATE TABLE test_log(
    new_id INT,
    old_id INT,
    dt DATETIME,
    type VARCHAR(255)
);

DELIMITER $
CREATE TRIGGER trig_insert BEFORE INSERT ON test FOR EACH ROW
BEGIN
    INSERT INTO test_log(new_id,dt,type) VALUES(NEW.id,NOW(),"INSERT");
END $
DELIMITER;

DELIMITER $
CREATE TRIGGER trig_update BEFORE UPDATE ON test FOR EACH ROW
BEGIN
    INSERT INTO test_log(new_id,old_id,dt,type) VALUES(NEW.id,OLD.id,NOW(),"UPDATE");
END $
DELIMITER;

DELIMITER $
CREATE TRIGGER trig_delete BEFORE DELETE ON test FOR EACH ROW
BEGIN
    INSERT INTO test_log(old_id,dt,type) VALUES(OLD.id,NOW(),"DELETE");
END $
DELIMITER;
```

> 示例：创建多个触发器

- `FOLLOWS` 选项允许在现有触发器之后激活新触发器
- `PRECEDES` 选项允许在现有触发器之前激活新触发器

```sql
DELIMITER $$
CREATE TRIGGER before_products_update 
BEFORE UPDATE ON products 
FOR EACH ROW
BEGIN
	INSERT INTO price_logs ( product_code, price )
	VALUES(old.productCode, old.msrp);
END $$
DELIMITER ; 

-- 在第一个触发器后再创建一个触发器
DELIMITER $$
CREATE TRIGGER before_products_update_2 
BEFORE UPDATE ON products 
FOR EACH ROW FOLLOWS before_products_update 
BEGIN
	INSERT INTO user_change_logs ( product_code, updated_by )
	VALUES(old.productCode, USER ());
END $$
DELIMITER ; 
```

## 查看/删除触发器

```sql
-- 查看当前数据库所有的触发器
SHOW TRIGGERS FROM 数据库;

-- 查看当前数据库中某个触发器的定义
SHOW CREATE TRIGGER 触发器名称

-- 从系统库information_schema的TRIGGERS表中查询触发器的信息
SELECT * FROM information_schema.TRIGGERS WHERE trigger_schema = 'database_name' AND trigger_name = 'trigger_name'; 

-- 删除触发器
DROP TRIGGER IF EXISTS 数据表.触发器名称
```

---



