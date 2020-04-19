# 题目

## 存储过程的编写， IN,OUT等参数的使用。

#### 创建数据表

`CREATE TABLE t_user(`

`USER_ID INT NOT NULL AUTO_INCREMENT,`

`USER_NAME CHAR(30) NOT NULL,`

`USER_PASSWORD CHAR(10) NOT NULL,`

`USER_EMAIL CHAR(30) NOT NULL,PRIMARY KEY (USER_ID),`

`INDEX IDX_NAME (USER_NAME))`

`ENGINE=InnoDB DEFAULT CHARSET=utf8;`

#### 带in的存储过程

`delimiter //
CREATE PROCEDURE SP_SEARCH(IN p_name CHAR(20)) 
BEGIN
IF p_name is null or p_name='' THEN
SELECT * FROM t_user; 
ELSE
SELECT * FROM t_user WHERE USER_NAME LIKE p_name; 
END IF; 
END //`

`delimiter ;
CALL SP_SEARCH('林炳文');`

##### 输出结果为：

+---------+-----------+---------------+----------------------+
| USER_ID | USER_NAME | USER_PASSWORD | USER_EMAIL           |
+---------+-----------+---------------+----------------------+
|       1 | 林炳文    | 1234567       | 1ing200810050126.com |
+---------+-----------+---------------+----------------------+
1 row in set (0.001 sec)

Query OK, 0 rows affected (0.006 sec)

#### 带OUT的存储过程

`delimiter //`
`CREATE PROCEDURE SP_SEARCH2(IN p_name CHAR(20),OUT p_int INT)` 
`BEGIN`
`IF p_name is null or p_name='' THEN`
`SELECT * FROM t_user;` 
`ELSE`
`SELECT * FROM t_user WHERE USER_NAME LIKE p_name;` 
`END IF;` 
`SELECT FOUND_ROWS() INTO p_int;` 
`END //`

`delimiter ;`
`CALL SP_SEARCH2('林%',@p_num);` 

##### 输出结果为：

+---------+------------+---------------+----------------------+
| USER_ID | USER_NAME  | USER_PASSWORD | USER_EMAIL           |
+---------+------------+---------------+----------------------+
|       8 | 林林之家   | 123456        | 1in@126.com          |
|       1 | 林炳文     | 1234567       | 1ing200810050126.com |
|       9 | 林炳文Evan | 123456        | 1ing20080126.com     |
+---------+------------+---------------+----------------------+
3 rows in set (0.003 sec)

Query OK, 1 row affected (0.040 sec)

`SELECT @p_num;`

+--------+
| @p_num |
+--------+
|      3 |
+--------+
1 row in set (0.000 sec)

#### 带INOUT的存储过程

`delimiter //`
`CREATE PROCEDURE sp_inout(INOUT p_num INT)` 
`BEGIN`
`SET p_num=p_num*10;` 
`END //`

`delimiter ;`
`SET @p_num=2;` 
`call sp_inout(@p_num);` 

##### 输出结果

Query OK, 0 rows affected (0.002 sec)

`SELECT @p_num;`

##### 输出结果

+--------+
| @p_num |
+--------+
|     20 |
+--------+
1 row in set (0.000 sec)

## 局部变量的定义+循环控制的使用（while）

`DELIMITER //`
`CREATE PROCEDURE test_while_001(IN in_count INT)`
`BEGIN`
    `DECLARE COUNT INT DEFAULT 0;//定义count&sum`
    `DECLARE SUM INT DEFAULT 0;`
    `WHILE COUNT < in_count DO`
        `SET SUM = SUM + COUNT;`
        `SET COUNT = COUNT + 1;//赋值`
    `END WHILE;`
    `SELECT SUM;`
`END //`
`DELIMITER ;`

##### 调用

`CALL test_while_001(10)；`

##### 输出

+------+
| SUM  |
+------+
|   45 |
+------+
1 row in set (0.001 sec)

## 条件控制的使用(IF)

`delimiter //`
`CREATE PROCEDURE SP_SCHOLARSHIP_LEVEL(IN p_level char(1))` 
`BEGIN`
`IF p_level ='A' THEN`
`SELECT * FROM t_grade WHERE STU_SCORE >=90;` 
`ELSEIF p_level ='B' THEN`
`SELECT * FROM t_grade WHERE STU_SCORE <90 AND STU_SCORE>=80;` 
`ELSEIF p_level ='C' THEN`
`SELECT * FROM t_grade WHERE STU_SCORE <80 AND STU_SCORE>=70;` 
`ELSEIF p_level ='D' THEN`
`SELECT * FROM t_grade WHERE STU_SCORE <60;` 
`ELSE`
`SELECT * FROM t_grade;` 
`END IF;` 
`END //`

##### 输出

Query OK, 0 rows affected (0.030 sec)

`delimiter ;`

`CALL SP_SCHOLARSHIP_LEVEL('A')；`

##### 输出

+--------+-----------+
| STU_ID | STU_SCORE |
+--------+-----------+
|   2004 |        90 |
|   2008 |        90 |
|   2012 |        90 |
|   2018 |        99 |
|   2021 |        95 |
+--------+-----------+
5 rows in set (0.007 sec)

## 条件控制的使用(CASE)

`delimiter //`
`CREATE PROCEDURE SP_SCHOLARSHIP_LEVEL3(IN p_level char(1))` 
`BEGIN`
`DECLARE p_num int DEFAULT 0;` 
`CASE p_level` 
`WHEN 'A' THEN`
`SET p_num=90;` 
`WHEN 'B' THEN`
`SET p_num=80;` 
`WHEN 'C' THEN`
`SET p_num=70;` 
`WHEN 'D' THEN`
`SET p_num=60;` 
`ELSE`
`SET p_num=0;` 
`END CASE;` 
`SELECT * FROM t_grade g, t_student s WHERE g.STU_ID=s.STU_ID AND g.STU_SCORE >= p_num ;` 
`END //`

##### 输出

Query OK, 0 rows affected (0.028 sec)

`delimiter ;`

`CALL SP_SCHOLARSHIP_LEVEL3('d');`

##### 输出结果

+--------+-----------+--------+----------+-----------+---------+---------+
| STU_ID | STU_SCORE | STU_ID | STU_NAME | STU_CLASS | STU_SEX | STU_AGE |
+--------+-----------+--------+----------+-----------+---------+---------+
|   2000 |        78 |   2000 | 小王     |         1 | 男      |      12 |
|   2003 |        67 |   2003 | dfvca    |         2 | 男      |       3 |
|   2004 |        90 |   2004 | efa      |         1 | 女      |       4 |
|   2005 |        67 |   2005 | acva     |         2 | 男      |      12 |
|   2006 |        78 |   2006 | cvsa     |         1 | 女      |      12 |
|   2007 |        79 |   2007 | csa      |         1 | 女      |      10 |
|   2008 |        90 |   2008 | 吕g      |         1 | 女      |      12 |
|   2011 |        80 |   2011 | et       |         2 | 男      |      12 |
|   2012 |        90 |   2012 | vdgv     |         2 | 男      |      11 |
|   2013 |        88 |   2013 | qwe      |         1 | 女      |      12 |
|   2014 |        88 |   2014 | hj       |         1 | 女      |      12 |
|   2015 |        67 |   2015 | fvb      |         2 | 男      |      13 |
|   2016 |        88 |   2016 | hgb      |         1 | 男      |      13 |
|   2017 |        85 |   2017 | fa       |         1 | 女      |      12 |
|   2018 |        99 |   2018 | afa      |         2 | 男      |      23 |
|   2019 |        87 |   2019 | yr       |         1 | 男      |      12 |
|   2020 |        78 |   2020 | uk       |         1 | 女      |       5 |
|   2021 |        95 |   2021 | oph      |         1 | 男      |      12 |
+--------+-----------+--------+----------+-----------+---------+---------+
18 rows in set (0.008 sec)

## 循环控制的使用（repeat)

`delimiter //`
`CREATE PROCEDURE sp_cal2(IN p_num INT,OUT p_result INT)` 
`BEGIN`
 `SET p_result=1;` 
 `REPEAT` 
  `SET p_result = p_num * p_result;` 
  `SET p_num = p_num-1;` 
  `UNTIL p_num<=1` 
 `END REPEAT;` 
`END //`

##### 输出

Query OK, 0 rows affected (0.033 sec)

`delimiter ;`

`CALL sp_cal2(5,@result);` 

##### 输出

Query OK, 0 rows affected (0.002 sec)

`SELECT @result;` 

##### 输出结果

+---------+
| @result |
+---------+
|     120 |
+---------+
1 row in set (0.000 sec)