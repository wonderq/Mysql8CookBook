> 以下都基于mysql8.0

------



we can alter the definition of the table any time,but full table will be rebuilt while doing so.



#### show 命令

show Engines/G; 查看引擎

show tables; 查看当前库下的表

show create table xxx\G;   G表示 ，graph   看xxx表的创建语句

show warnings; 看报警

SHOW FUNCTION STATUS\G; 查看库里所有的函数

SHOW CREATE FUNCTION <function_name>\G. 看function的创建语句

SHOW FULL TABLES WHERE TABLE_TYPE LIKE 'VIEW'; 查看所有的视图

SHOW CREATE VIEW salary_view\G；查看创建视图语句

SHOW EVENTS\G；查看所有事件

SHOW CREATE EVENT purge_salary_audit\G；查看创建event的语句



#### 如果原数据存在，但继续执行其他操作

insert ignore into values... 如果不加ignore，且原数据存在，那么会报错不继续执行。

replace into xxx values (1,'bob','man');

insert into xxx on duplicate key update payment = payment + values(payment)

truncating table preserve the table structure while delete does not.                                                                                 

schema = databases + tables



#### mysql 导入数据

 mysql -u root -p <xxx.sql       

 mysql -u root -p  xxx  -A   xxx 是指定数据库



#### 查看表结构表示

Desc tablename 

#### 正则表达式

```sql
SELECT COUNT(*) FROM employees WHERE first_name RLIKE '^christ';  
--或者
SELECT COUNT(*) FROM employees WHERE last_name REGEXP 'ba$';
```

```sql
SELECT COUNT(*) FROM employees WHERE last_name NOT REGEXP '[aeiou]';
--lastname中不包含aeiou
```

order by  可以 ＋数字 表示 对第几个字段（从1开始）排序，与limit 可以合起来用 ，

```sql
SELECT emp_no,salary FROM salaries ORDER BY 2 DESC LIMIT 5;
```



#### 创建用户，并带有一定的权限

  ip可以指定 10.148.%.% ，其中%代表any

```sql
	CREATE USER IF NOT EXISTS  'company_read_only'@'localhost' 
	IDENTIFIED WITH mysql_native_password 
	BY 'company_pass' 
    WITH MAX_QUERIES_PER_HOUR 500 
	MAX_UPDATES_PER_HOUR 100;
```



#### 指定权限，只有部分列

```sql
	GRANT SELECT(salary) ON 
	employees.salaries TO 'employees_ro'@'%';
```



#### 授权superUser

​	

```sql
mysql> GRANT ALL ON *.* TO 'dbadmin'@'%';
```



#### 撤权

```sql
mysql> REVOKE DELETE ON company.* FROM 'company_write'@'%';

mysql> REVOKE SELECT(salary) ON employees.salaries FROM 'employees_ro'@'%';
```



#### 设值密码过期

```sql
ALTER USER 'developer'@'%' PASSWORD EXPIRE INTERVAL 90 DAY;
```



#### 给用户上锁、解锁

```sql
ALTER USER 'developer'@'%' ACCOUNT LOCK;

ALTER USER 'developer'@'%' ACCOUNT UNLOCK;
```



#### 创建角色role

```sql
mysql> CREATE ROLE 'app_read_only','app_writes', 'app_developer';

mysql> GRANT SELECT ON employees.* TO 'app_read_only';

mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO 'app_writes';

mysql> GRANT ALL ON employees.* TO 'app_developer';

mysql> GRANT 'app_writes' TO 'emp_writes'@'%'; --指定host.
```



#### 将数据导出为文件或表

- 文件

  ```sql
  mysql> SELECT first_name, last_name INTO OUTFILE 'result.csv' 
  FIELDS TERMINATED BY ',' 
  OPTIONALLY ENCLOSED BY '"' 
  LINES TERMINATED BY '\n' 
  FROM employees WHERE hire_date<'1986-01-01'
  LIMIT 10;
  ```



- 表

  表不存在； 

  ```sql
  mysql> CREATE TABLE titles_only AS SELECT DISTINCT title FROM titles;
  ```

  表存在； 

  ```sql
  mysql> INSERT INTO titles_only SELECT DISTINCT title FROM titles;
  ```

  

#### 将文件导入表

```sql
mysql> LOAD DATA INFILE 'result.csv' INTO TABLE
employee_names
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES; --忽略第一行
```

如果有重复，要覆盖

```sql
LOAD DATA INFILE 'result.csv' REPLACE
INTO TABLE employee_names FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"' LINES TERMINATED
BY '\n';
```

如果有重复，要忽略

```sql
mysql> LOAD DATA INFILE 'result.csv' IGNORE INTO
TABLE employee_names FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"' LINES TERMINATED
BY '\n';
```



#### 自连接

```sql
mysql> SELECT
emp1.* FROM
employees emp1
JOIN employees emp2
ON emp1.first_name=emp2.first_name
AND emp1.last_name=emp2.last_name
AND emp1.gender=emp2.gender
AND emp1.hire_date=emp2.hire_date
AND emp1.emp_no!=emp2.emp_no
ORDER BY first_name, last_name;
```



#### 子查询

```sql
SELECT
first_name,
last_name
FROM
employees
WHERE
emp_no
IN (SELECT emp_no FROM titles WHERE title="Senior
Engineer" AND from_date="1986-06-26");
```



#### 如何查询两张表的不同数据行

使用not in或者outer join（left join）

正常的join是A表与B表的交集，而outer join则不仅仅给了交集的部分，而且以Null的结果对A表不匹配的记录进行连接。所以可以利用这个特性

```sql
SELECT l1.* FROM employees_list1 l1 LEFT OUTER
JOIN employees_list2 l2 ON l1.emp_no=l2.emp_no WHERE
l2.emp_no IS NULL;
```



如果使用right join，则左表为null。

#### 存储过程

封装所有的sql语句到一个简单的程序中，不需要返回值。以begin开头和end结尾。

可以通过call语句调用

https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html

举个例子：

```sql
/* DROP the existing procedure if any with the same name before creating */
DROP PROCEDURE IF EXISTS create_employee;
/* Change the delimiter to $$ */
DELIMITER $$
/* IN specifies the variables taken as arguments,INOUT specifies the output variable*/
CREATE PROCEDURE create_employee (OUT new_emp_no INT,
IN first_name varchar(20), IN last_name varchar(20),
IN gender enum('M','F'), IN birth_date date, IN
emp_dept_name varchar(40), IN title varchar(50))
BEGIN
/* Declare variables for emp_dept_no and salary */
DECLARE emp_dept_no char(4);
DECLARE salary int DEFAULT 60000;
/* Select the maximum employee number into the variable new_emp_no */
SELECT max(emp_no) INTO new_emp_no FROM employees;
/* Increment the new_emp_no */
SET new_emp_no = new_emp_no + 1;
/* INSERT the data into employees table */
/* The function CURDATE() gives the current date) */
INSERT INTO employees VALUES(new_emp_no,birth_date, first_name, last_name, gender,CURDATE());
/* Find out the dept_no for dept_name */
SELECT emp_dept_name;
SELECT dept_no INTO emp_dept_no FROM departments
WHERE dept_name=emp_dept_name;
SELECT emp_dept_no;
/* Insert into dept_emp */
INSERT INTO dept_emp VALUES(new_emp_no,emp_dept_no, CURDATE(), '9999-01-01');
/* Insert into titles */
INSERT INTO titles VALUES(new_emp_no, title,CURDATE(), '9999-01-01');
/* Find salary based on title */
IF title = 'Staff'
THEN SET salary = 100000;
ELSEIF title = 'Senior Staff'
THEN SET salary = 120000;
END IF;
/* Insert into salaries */
INSERT INTO salaries VALUES(new_emp_no, salary,CURDATE(), '9999-01-01');
END
$$
/* Change the delimiter back to ; */
DELIMITER ;
```



调用

```sql
CALL create_employee(@new_emp_no, 'John','Smith', 'M', '1984-06-19', 'Research', 'Staff');
```



#### 函数

函数和存储过程的区别在于，函数有返回值，而且可以通过select语句调用。一般函数用于建华复杂的计算。

举个例子：

```sql
DROP FUNCTION IF EXISTS get_sal_level;
DELIMITER $$
CREATE FUNCTION get_sal_level(emp int) RETURNS
VARCHAR(10)
DETERMINISTIC
BEGIN
DECLARE sal_level varchar(10);
DECLARE avg_sal FLOAT;
SELECT AVG(salary) INTO avg_sal FROM salaries WHERE
emp_no=emp;
IF avg_sal < 50000 THEN
SET sal_level = 'BRONZE';
ELSEIF (avg_sal >= 50000 AND avg_sal < 70000) THEN
SET sal_level = 'SILVER';
ELSEIF (avg_sal >= 70000 AND avg_sal < 90000) THEN
SET sal_level = 'GOLD';
ELSEIF (avg_sal >= 90000) THEN
SET sal_level = 'PLATINUM';
ELSE
SET sal_level = 'NOT FOUND';
END IF;
RETURN (sal_level);
END
$$
DELIMITER ;
```

调用

```sql
SELECT get_sal_level(10002);
```

https://dev.mysql.com/doc/refman/8.0/en/func-op-summary-ref.html.

#### 内建函数



mysql提供了很多内建函数。比如CURDATE()，就是当前日期。连接字符串的concat函数

```sql
SELECT CONCAT(first_name, ' ', last_name)
```



#### 触发器

触发器被用来激活某件事情，在要触发的的事件前或者后。例如：每当插入一行，或者更新一行，或者删除一行，你可以有一个触发器。

从mysql5.7开始，一个表可以同时有多个触发器。例如，一个表可以有两个 before insert 的触发器。但是你必须要指定哪个触发在前，哪个在后。可以通过FOLLOWS或PRECEDES关键字来解决。

例如：

```sql
DROP TRIGGER IF EXISTS salary_round;
DELIMITER $$
CREATE TRIGGER salary_round BEFORE INSERT ON salaries
FOR EACH ROW
BEGIN
SET NEW.salary=ROUND(NEW.salary);
END
$$
DELIMITER ;
```

https://dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html,



#### 视图

视图是一个基于sql语句结果集的虚拟表。视图隐藏了sql的复杂性，当然，更重要的一点，也提供了安全性。

例如：

创建

```sql
CREATE ALGORITHM=UNDEFINED
DEFINER=`root`@`localhost`
SQL SECURITY DEFINER VIEW salary_view
AS
SELECT emp_no, salary FROM salaries WHERE from_date > '2002-01-01';
```

使用

```sql
SELECT emp_no, AVG(salary) as avg FROM
salary_view GROUP BY emp_no ORDER BY avg DESC LIMIT 5;
```





#### 事件

就像linux系统的cron表达式一样，mysql有events来处理调度任务。mysql使用一个特殊的线程，唤做 event 调度线程，来执行所有的调度事件。默认的，这个event 调度线程不开启，如果要开启，需要执行

```sql
SET GLOBAL event_scheduler = ON;
```

例如：

```sql
DROP EVENT IF EXISTS purge_salary_audit;
DELIMITER $$
CREATE EVENT IF NOT EXISTS purge_salary_audit
ON SCHEDULE
EVERY 1 WEEK
STARTS CURRENT_DATE
DO BEGIN
DELETE FROM salary_audit WHERE date_modified
< DATE_ADD(CURDATE(), INTERVAL -7 day);
END $$
DELIMITER ;
```

一旦event 被创建，那么它就会自动执行。

让event失效/生效，可以执行如下语句：

```sql
mysql> ALTER EVENT purge_salary_audit DISABLE;
mysql> ALTER EVENT purge_salary_audit ENABLE;
```



所有的存储过程、函数、触发器、事件、视图都有一个definer。如果definer没有指定，那么创建者将被选择为definer。

https://dev.mysql.com/doc/refman/8.0/en/event-scheduler.html



#### information_schema数据库

这个information_schema数据库，是一个包含数据库所有对象的元数据。

https://mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/



##### TABLES表

这个表包含了所有有关表的inxi，例如这个表属于哪个数据库，表的行数，使用的引擎，数据的长度，索引的长度等。

例如：

```sql
SELECT SUM(DATA_LENGTH)/1024/1024 AS
DATA_SIZE_MB, SUM(INDEX_LENGTH)/1024/1024 AS
INDEX_SIZE_MB, SUM(DATA_FREE)/1024/1024 AS
DATA_FREE_MB FROM INFORMATION_SCHEMA.TABLES WHERE
TABLE_SCHEMA='employees';
+--------------+---------------+--------------+
| DATA_SIZE_MB | INDEX_SIZE_MB | DATA_FREE_MB |
+--------------+---------------+--------------+
| 17.39062500 | 14.62500000 | 11.00000000 |
+--------------+---------------+--------------+
1 row in set (0.01 sec)
```

##### COLUMNS表

这个表列出了每个表所有的列，还有列的定义

```sql
SELECT * FROM COLUMNS WHERE TABLE_NAME='employees'\G
```

##### FILES表

我们知道，mysql存储数据到innoDb的数据在.ibd文件中(与数据库一个名字，的data 目录)。比如

```sql
SELECT * FROM FILES WHERE FILE_NAME LIKE
'./employees/employees.ibd'\G
~~~
EXTENT_SIZE: 1048576
AUTOEXTEND_SIZE: 4194304
DATA_FREE: 13631488
~~~
```



##### INNODB_SYS_TABLESPACES表

像file表一样，也可查询文件的大小

```sql
mysql> SELECT * FROM INNODB_TABLESPACES WHERE
NAME='employees/employees'\G
*************************** 1. row
***************************
SPACE: 118
NAME: employees/employees
FLAG: 16417
ROW_FORMAT: Dynamic
PAGE_SIZE: 16384
ZIP_PAGE_SIZE: 0
SPACE_TYPE: Single
FS_BLOCK_SIZE: 4096
FILE_SIZE: 32505856
ALLOCATED_SIZE: 32509952
1 row in set (0.00 sec)
```

在linux操作系统中也可核实一下

```shell
shell> sudo ls -ltr
/var/lib/mysql/employees/employees.ibd
-rw-r----- 1 mysql mysql 32505856 Jun 20 16:50
/var/lib/mysql/employees/employees.ibd
```

##### INNODB_TABLESPACES表

索引的大小，以及大概的行数，对，这里是大概的。

```sql
mysql> SELECT * FROM INNODB_TABLESTATS WHERE
NAME='employees/employees'\G
*************************** 1. row
***************************
TABLE_ID: 128
NAME: employees/employees
STATS_INITIALIZED: Initialized
NUM_ROWS: 299468
CLUST_INDEX_SIZE: 1057
OTHER_INDEX_SIZE: 545
MODIFIED_COUNTER: 0
AUTOINC: 0
REF_COUNT: 1
1 row in set (0.00 sec)
```

##### PROCESSLIST表

一个最常使用的进程视图展示就是进程表。它列出了服务器上所有的正在运行的查询语句。

```sql
mysql> SELECT * FROM PROCESSLIST\G
*************************** 1. row
***************************
ID: 85
USER: event_scheduler
HOST: localhost
DB: NULL
COMMAND: Daemon
TIME: 44
STATE: Waiting for next activation
INFO: NULL
*************************** 2. row
***************************
ID: 26231
USER: root
HOST: localhost
DB: information_schema
COMMAND: Query
TIME: 0
STATE: executing
INFO: SELECT * FROM PROCESSLIST
2 rows in set (0.00 sec
```

或者你可以使用

```sql
SHOW PROCESSLIST;
```

得到同样的输出。



##### 其他的表

**ROUTINES**：包含了函数和存储的规则。

**TRIGGERS**：包含了触发器的定义。

**VIEWS**：包含了视图的定义。

更多：http://mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/.





#### JSON

从mysql5.7，mysql支持了JSON的数据类型，早期版本是string 存储的。新的json数据类型提供自动校验格式功能，并优化了存储格式。json记录采用二进制格式存储。

##### 声明

```sql
CREATE TABLE emp_details(
emp_no int primary key,
details json
);
```

##### 插入json记录

```sql
INSERT INTO emp_details(emp_no, details)
VALUES ('1',
'{ "location": "IN", "phone": "+11800000000",
"email": "abc@example.com", "address": { "line1":
"abc", "line2": "xyz street", "city": "Bangalore",
"pin": "560103"} }'
);
```

##### 获取json记录

```sql
mysql> SELECT emp_no, details->'$.address.pin' pin
FROM emp_details;
+--------+----------+
| emp_no | pin |
+--------+----------+
| 1 | "560103" |
+--------+----------+
1 row in set (0.00 sec)
```

不要双引号

```sql
mysql> SELECT emp_no, details->>'$.address.pin' pin
FROM emp_details;
+--------+--------+
| emp_no | pin |
+--------+--------+
| 1 | 560103 |
+--------+--------+
1 row in set (0.00 sec)
```

##### json函数

JSON_PRETTY()；————美化json

JSON_CONTAINS()；————搜索json

JSON_SET(), JSON_INSERT(), JSON_REPLACE().JSON_REMOVE()等等

JSON_KEYS()；——获取json中所有的key

更多：https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html.

#### Common table expressions

mysql8支持Common table expressions，递归的和非递归的。

为啥需要这个玩意？

在同一个查询中不可能两次引用派生表。因此，派生表的计算次数是引用表的两倍或两倍，这表明存在严重的性能问题。使用CTE，子查询只计算一次。

##### 非递归的CTE

通用表表达式（CTE）只是像派生表一样，但是声明放在在查询块之前，而不是在FROM子句中。

```sql
mysql> SELECT
q1.year,
q2.year AS next_year,
q1.sum,
q2.sum AS next_sum,
100*(q2.sum-q1.sum)/q1.sum AS pct
FROM
(SELECT year(from_date) as year, sum(salary) as
sum FROM salaries GROUP BY year) AS q1,
(SELECT year(from_date) as year, sum(salary) as sum
FROM salaries GROUP BY year)
```

上面这个子查询会执行两边，而下面的只会执行一遍

```sql
mysql>
WITH CTE AS
(SELECT year(from_date) AS year, SUM(salary) AS
sum FROM salaries GROUP BY year)
SELECT
q1.year, q2.year as next_year, q1.sum, q2.sum as
next_sum, 100*(q2.sum-q1.sum)/q1.sum as pct FROM
CTE AS q1,
CTE AS q2
WHERE
q1.year = q2.year-1;
+------+-----------+-------------+-------------+-----
-----+
| year | next_year | sum | next_sum | pct
| +------+-----------+-------------+-------------+-----
-----+
| 1985 | 1986 | 972864875 | 2052895941 |
111.0155 |
```



派生子查询不能互相引用

```sql
SELECT ...
FROM (SELECT ... FROM ...) AS d1, (SELECT ... FROM
d1 ...) AS d2 ...
ERROR: 1146 (42S02): Table ‘db.d1’ doesn’t exist
```

而CTE可以

```sql
WITH d1 AS (SELECT ... FROM ...), d2 AS (SELECT ...
FROM d1 ...)
SELECT
FROM d1, d2 ...
```

递归的情况：

```sql
mysql> WITH RECURSIVE employee_paths (id, name, path)
AS
(
SELECT id, name, CAST(id AS CHAR(200))
FROM employees_mgr
WHERE manager_id IS NULL
UNION ALL
SELECT e.id, e.name, CONCAT(ep.path, ',', e.id)
FROM employee_paths AS ep JOIN employees_mgr AS e
ON ep.id = e.manager_id) SELECT *
FROM employee_paths ORDER BY path;
```

