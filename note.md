以下都基于mysql8.0

------



we can alter the definition of the table any time,but full table will be rebuilt while doing so.



#### show 命令

show Engines/G; 查看引擎

show tables; 查看当前库下的表

show create table xxx\G;   G表示 ，graph   看xxx表的表结构

show warnings; 看报警



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

SELECT COUNT(*) FROM employees WHERE first_name RLIKE '^christ';  或者SELECT COUNT(*) FROM employees WHERE last_name REGEXP 'ba$';

SELECT COUNT(*) FROM employees WHERE last_name NOT REGEXP '[aeiou]';————lastname中不包含aeiou

order by  可以 ＋数字 表示 对第几个字段（从1开始）排序，与limit 可以合起来用 ，SELECT emp_no,salary FROM salaries ORDER BY 2 DESC LIMIT 5;



#### 创建用户，并带有一定的权限

  ip可以指定 10.148.%.% ，其中%代表any

​	CREATE USER IF NOT EXISTS  'company_read_only'@'localhost' 

​	IDENTIFIED WITH mysql_native_password 

​	BY 'company_pass' 

​    WITH MAX_QUERIES_PER_HOUR 500 

​	MAX_UPDATES_PER_HOUR 100;



#### 指定权限，只有部分列

​	GRANT SELECT(salary) ON 

​	employees.salaries TO 'employees_ro'@'%';



#### 授权superUser

​	mysql> GRANT ALL ON *.* TO 'dbadmin'@'%';



#### 撤权

mysql> REVOKE DELETE ON company.* FROM 'company_write'@'%';

mysql> REVOKE SELECT(salary) ON employees.salaries FROM 'employees_ro'@'%';



#### 设值密码过期

ALTER USER 'developer'@'%' PASSWORD EXPIRE INTERVAL 90 DAY;



#### 给用户上锁、解锁

ALTER USER 'developer'@'%' ACCOUNT LOCK;

ALTER USER 'developer'@'%' ACCOUNT UNLOCK;



#### 创建角色role

mysql> CREATE ROLE 'app_read_only','app_writes', 'app_developer';

mysql> GRANT SELECT ON employees.* TO 'app_read_only';

mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO 'app_writes';

mysql> GRANT ALL ON employees.* TO 'app_developer';

mysql> GRANT 'app_writes' TO 'emp_writes'@'%'; ——指定host.



#### 将数据导出为文件或表

- 文件

  mysql> SELECT first_name, last_name INTO OUTFILE 'result.csv' FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n' 

FROM employees WHERE hire_date<'1986-01-01'
LIMIT 10;

- 表

mysql> CREATE TABLE titles_only AS SELECT DISTINCT title FROM titles;