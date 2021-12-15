- [力扣Mysql练手](https://blog.noheart.cn/archives/sqlshell)



## 组合两个表

- 题目描述
   给定一个 salary 表，如下所示，有 m = 男性 和 f = 女性 的值 。交换所有的 f 和 m 值（例如，将所有 f 值更改为 m，反之亦然）。要求使用一个更新（Update）语句，并且没有中间临时表。

请注意，你必须编写一个 Update 语句，不要编写任何 Select 语句。

例如:

| id   | name | sex  | salary |
| ---- | ---- | ---- | ------ |
| 1    | A    | m    | 2500   |
| 2    | B    | f    | 1500   |
| 3    | C    | m    | 5500   |
| 4    | D    | f    | 500    |

运行你所编写的更新语句之后，将会得到以下表:

| id   | name | sex  | salary |
| ---- | ---- | ---- | ------ |
| 1    | A    | f    | 2500   |
| 2    | B    | m    | 1500   |
| 3    | C    | f    | 5500   |
| 4    | D    | m    | 500    |

MySQL脚本：

```sql
-- ----------------------------
-- Table structure for `salary`
-- ----------------------------
DROP TABLE IF EXISTS `salary`;
CREATE TABLE `salary` (
  `id`int(11) NOT NULL,
  `name`varchar(10) NOT NULL,
  `sex`varchar(10) NOT NULL,
 `salary` int(11) NOT NULL,
  PRIMARYKEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
-- ----------------------------
-- Records of salary
-- ----------------------------
INSERT INTO `salary` VALUES ('1', 'A', 'm','2500');
INSERT INTO `salary` VALUES ('2', 'B', 'f','1500');
INSERT INTO `salary` VALUES ('3', 'C', 'm','5500');
INSERT INTO `salary` VALUES ('4', 'D', 'f','500');
```

SQL

### 本题解法：

方法一：使用 if 函数

IF(expr1, expr2, expr3) 如果 expr1 为 TRUE，则 IF () 的返回值为 expr2；否则返回值为 expr3。

```sql
# Write your MySQL query statement below
UPDATE salary
SET sex = IF(sex = "f","m","f");
```

SQL

方法二：使用 case…when..then..else..end

```sql
UPDATE salary 
SET sex  = (CASE WHEN sex = 'm' THEN 'f' ELSE 'm' END);
```

## 第二高的薪水

- 题目描述
   编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary）。

| Id   | Salary |
| ---- | ------ |
| 1    | 100    |
| 2    | 200    |
| 3    | 300    |

例如上述 Employee 表，SQL 查询应该返回 200 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null。

| SecondHighestSalary |
| ------------------- |
| 200                 |

### 本题解法

方法 1 简便方法，倒序排序，limit(1,1)即可

```sql
SELECT IFNULL((SELECT DISTINCT(Salary) 
FROM Employee
ORDER BY Salary DESC
LIMIT 1,1),null) AS SecondHighestSalary
```

SQL

方法 2
 先查询出最高的身高值，然后查询身高小于该值的最高身高。

```sql
# Write your MySQL query statement below
SELECT Max(Salary) AS SecondHighestSalary
FROM Employee
WHERE Salary < (SELECT Max(Salary) FROM Employee);
```

SQL

注：IFNULL() 方法介绍：

```sql
IFNULL(expr1,expr2)
```

SQL

如果 expr1 不是 NULL，IFNULL() 返回 expr1，否则它返回 expr2。IFNULL() 返回一个数字或字符串值，取决于它被使用的上下文环境。

## 第 n 高的薪水

- 题目描述
   编写一个 SQL 查询，获取 Employee 表中第 n 高的薪水（Salary）。

| Id   | Salary |
| ---- | ------ |
| 1    | 100    |
| 2    | 200    |
| 3    | 300    |

例如上述 Employee 表，n = 2 时，应返回第二高的薪水 200。如果不存在第 n 高的薪水，那么查询应返回 null。

| getNthHighestSalary(2) |
| ---------------------- |
| 200                    |

MySQL 脚本

```sql
Create table If Not Exists Employee (Idint, Salary int);
Truncate table Employee;
insert into Employee (Id, Salary) values('1', '100');
insert into Employee (Id, Salary) values('2', '200');
insert into Employee (Id, Salary) values('3', '300');
```

SQL

### 本题解法

查询条件：

- 1）返回第 N 高的薪水
- 2）如果不存在第 N 高的薪水，那么查询应返回 null

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  SET N = N-1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT(
      SELECT DISTINCT Salary
      FROM Employee
      ORDER BY Salary DESC
      LIMIT 1 OFFSET N
      )
  );
END
```

SQL

注意：LIMIT 子句后面不能做运算。

或者可以新定义一个变量 x：

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  DECLARE x int;
  SET x = N-1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT(
      SELECT DISTINCT Salary
      FROM Employee
      ORDER BY Salary DESC
      LIMIT 1 OFFSET x
      )
  );
END
```

## 分数排名

- 题目描述
   编写一个 SQL 查询来实现分数排名。如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有 “间隔”。

| Id   | Score |
| ---- | ----- |
| 1    | 3.50  |
| 2    | 3.65  |
| 3    | 4.00  |
| 4    | 3.85  |
| 5    | 4.00  |
| 6    | 3.65  |

例如，根据上述给定的 Scores 表，你的查询应该返回（按分数从高到低排列）：

| Score | Rank |
| ----- | ---- |
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |

MySQL脚本

```sql
Create table If Not Exists Scores (Id int,Score DECIMAL(3,2));
Truncate table Scores;
insert into Scores (Id, Score) values ('1','3.5');
insert into Scores (Id, Score) values ('2','3.65');
insert into Scores (Id, Score) values ('3','4.0');
insert into Scores (Id, Score) values ('4','3.85');
insert into Scores (Id, Score) values ('5','4.0');
insert into Scores (Id, Score) values ('6','3.65');
```

SQL

### 本题解法：

- 方法 1
   对于每一个分数，从表中找出有多少个大于或等于该分数的不重复分数，然后降序排列。

```sql
# Write your MySQL query statement below
SELECT Score,(
    SELECT COUNT(DISTINCT s.Score)
    FROM Scores s
    WHERE s.Score >= Scores.Score) AS Rank
FROM Scores
ORDER BY Score DESC;
```

SQL

这种方式虽然很简单，但是在 select 语句中使用子查询来生成 rank，相当于对每一条记录都需要查询一次 scores 表，查询次数为 （记录条数 ^2），会很慢。

- 方法2
   在 select 语句中创建两个变量，其中 @PreScore 用来记录前一个分数，@Ranker 用来记录当前排名。然后判断当前的分数是否等于前面的分数 @PreScore，如果等于的话，排名值就为 @Ranker，否则，排名值为 @Ranker+1。

这里用了 CAST() 函数进行类型转换，其语法为：

```sql
CAST(字段名 as 转换的类型 )
```

SQL

其中类型可以为：

- CHAR [(N)] 字符型
- DATE 日期型
- DATETIME 日期和时间型
- DECIMAL float 型
- SIGNED int
- TIME 时间型

```sql
SELECT Score,CAST(
    CASE 
    WHEN @PreScore = Score THEN @Ranker
    WHEN @PreScore := Score THEN @Ranker := @Ranker + 1
    ELSE @Ranker := @Ranker + 1
  END AS SIGNED) AS Rank
FROM Scores a,(SELECT @PreScore := NULL, @Ranker := 0) r
ORDER BY Score DESC;
```

## 连续出现的数字

- 题目描述
   编写一个 SQL 查询，查找所有至少连续出现三次的数字。

| Id   | Num  |
| ---- | ---- |
| 1    | 1    |
| 2    | 1    |
| 3    | 1    |
| 4    | 2    |
| 5    | 1    |
| 6    | 2    |
| 7    | 2    |

例如，给定上面的 Logs 表， 1 是唯一连续出现至少三次的数字。

| ConsecutiveNums |
| --------------- |
| 1               |

- MySQL 脚本

```sql
Create table If Not Exists Logs (Id int,Num int);
Truncate table Logs;
insert into Logs (Id, Num) values ('1','1');
insert into Logs (Id, Num) values ('2','1');
insert into Logs (Id, Num) values ('3', '1');
insert into Logs (Id, Num) values ('4','2');
insert into Logs (Id, Num) values ('5','1');
insert into Logs (Id, Num) values ('6','2');
insert into Logs (Id, Num) values ('7','2');
```

SQL

### 本题解法：

由于需要找三次相同数字，所以我们需要建立三个表的实例，我们可以用 l1 分别和 l2, l3 内交，l1 和 l2 的 Id 下一个位置比，l1 和 l3 的下两个位置比，然后将 Num 都相同的数字返回。

**注意：** SELECT DISTINCT l1.Num AS ConsecutiveNums 保证当连续的数字大于 3 个的时候，返回的还是一个 Num。

```sql
# Write your MySQL query statement below
SELECT DISTINCT l1.Num AS ConsecutiveNums
FROM Logs l1 
LEFT JOIN Logs l2
ON l1.Id = l2.Id - 1
LEFT JOIN Logs l3
ON l1.Id = l3.Id - 2
WHERE l1.Num = l2.Num AND l1.Num = l3.Num;
```

## 超过经理收入的员工

- 题目描述
   Employee 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。

| Id   | Name  | Salary | ManagerId |
| ---- | ----- | ------ | --------- |
| 1    | Joe   | 70000  | 3         |
| 2    | Henry | 80000  | 4         |
| 3    | Sam   | 60000  | NULL      |
| 4    | Max   | 90000  | NULL      |

给定 Employee 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。

| Employee |
| -------- |
| Joe      |

- MySQL 脚本

```sql
Create table If Not Exists Employee (Id int, Name varchar(255), Salary int, ManagerId int);
Truncate table Employee;
insert into Employee (Id, Name, Salary, ManagerId) values ('1', 'Joe', '70000', '3');
insert into Employee (Id, Name, Salary, ManagerId) values ('2', 'Henry', '80000', '4');
insert into Employee (Id, Name, Salary, ManagerId) values ('3', 'Sam', '60000', null);
insert into Employee (Id, Name, Salary, ManagerId) values ('4', 'Max', '90000', null);
```

## 查找重复的电子邮箱

- 题目描述
   编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。
   示例：

| Id   | Email                     |
| ---- | ------------------------- |
| 1    | [a@b.com](mailto:a@b.com) |
| 2    | [c@d.com](mailto:c@d.com) |
| 3    | [a@b.com](mailto:a@b.com) |

根据以上输入，你的查询应返回以下结果：

| Email                     |
| ------------------------- |
| [a@b.com](mailto:a@b.com) |

> 说明：所有电子邮箱都是小写字母。

- MySQL 脚本

```sql
Create table If Not Exists Person (Id int,Email varchar(255));
Truncate table Person;
insert into Person (Id, Email) values ('1','a@b.com');
insert into Person (Id, Email) values ('2','c@d.com');
insert into Person (Id, Email) values ('3','a@b.com');
```

SQL

### 本题解法：

寻找重复的数据：用 group by Email 分组后 数据个数大于 1 的就是重复的。

```sql
# Write your MySQL query statement below
SELECT Email
FROM Person
GROUP BY Email
HAVING Count(*) >= 2;
```

## 从不订购的客户

- 题目描述
   某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

| Id   | Name  |
| ---- | ----- |
| 1    | Joe   |
| 2    | Henry |
| 3    | Sam   |
| 4    | Max   |

Orders 表：

| Id   | CustomerId |
| ---- | ---------- |
| 1    | 3          |
| 2    | 1          |

例如给定上述表格，你的查询应返回：

| Customers |
| --------- |
| Henry     |
| Max       |

- MySQL 脚本

```sql
Create table If Not Exists Customers (Idint, Name varchar(255));
Create table If Not Exists Orders (Id int,CustomerId int);
Truncate table Customers;
insert into Customers (Id, Name) values('1', 'Joe');
insert into Customers (Id, Name) values('2', 'Henry');
insert into Customers (Id, Name) values('3', 'Sam');
insert into Customers (Id, Name) values('4', 'Max');
Truncate table Orders;
insert into Orders (Id, CustomerId) values('1', '3');
insert into Orders (Id, CustomerId) values('2', '1');
```

SQL

### 本题解法

- 方法 1——LEFT JOIN
   直接让两个表左外连接，然后只要找出右边的 CustomerId 为 Null 的顾客就是没有下订单的顾客。

```sql
SELECT Customers.Name AS Customers
FROM Customers LEFT JOIN Orders
ON Customers.Id = Orders.CustomerId
WHERE Orders.CustomerId IS null;
```

SQL

- 方法2——NOT IN

```sql
SELECT c.Name AS Customers
FROM Customers AS c
WHERE c.Id NOT IN (
    SELECT DISTINCT CustomerId
    FROM Orders
);
```

## 部门最高工资

- 题目描述
   Employee 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

| Id   | Name  | Salary | DepartmentId |
| ---- | ----- | ------ | ------------ |
| 1    | Joe   | 70000  | 1            |
| 2    | Henry | 80000  | 2            |
| 3    | Sam   | 60000  | 2            |
| 4    | Max   | 90000  | 1            |

Department 表包含公司所有部门的信息。

| Id   | Name  |
| ---- | ----- |
| 1    | IT    |
| 2    | Sales |

编写一个 SQL 查询，找出每个部门工资最高的员工。例如，根据上述给定的表格，Max 在 IT 部门有最高工资，Henry 在 Sales 部门有最高工资。

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| IT         | Max      | 90000  |
| Sales      | Henry    | 80000  |

- MySQL 脚本

```sql
Create table If Not Exists Employee (Idint, Name varchar(255), Salary int, DepartmentId int);
Create table If Not Exists Department (Idint, Name varchar(255));
Truncate table Employee;
insert into Employee (Id, Name, Salary,DepartmentId) values ('1', 'Joe', '70000', '1');
insert into Employee (Id, Name, Salary,DepartmentId) values ('2', 'Henry', '80000', '2');
insert into Employee (Id, Name, Salary,DepartmentId) values ('3', 'Sam', '60000', '2');
insert into Employee (Id, Name, Salary,DepartmentId) values ('4', 'Max', '90000', '1');
Truncate table Department;
insert into Department (Id, Name) values('1', 'IT');
insert into Department (Id, Name) values('2', 'Sales');
```

SQL

### 本题解法

通过 GROUP BY 把每个部门的最高工资找出来，再通过联结表，找到相应的员工名字和部门名字。

```sql
# Write your MySQL query statement below
SELECT d.name AS Department, e.name AS Employee, e.Salary AS Salary
FROM Employee e,Department d
WHERE e.DepartmentId = d.Id 
AND ((e.DepartmentId,e.Salary) IN (SELECT DepartmentId,Max(Salary) AS MSalary
                                   FROM Employee
                                   GROUP BY DepartmentId));
```

## 部门工资前三高的员工

- 题目描述
   Employee 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id 。

| Id   | Name  | Salary | DepartmentId |
| ---- | ----- | ------ | ------------ |
| 1    | Joe   | 70000  | 1            |
| 2    | Henry | 80000  | 2            |
| 3    | Sam   | 60000  | 2            |
| 4    | Max   | 90000  | 1            |
| 5    | Janet | 69000  | 1            |
| 6    | Randy | 85000  | 1            |

Department 表包含公司所有部门的信息。

| Id   | Name  |
| ---- | ----- |
| 1    | IT    |
| 2    | Sales |

编写一个 SQL 查询，找出每个部门工资前三高的员工。例如，根据上述给定的表格，查询结果应返回：

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |

- MySQL 脚本

```sql
Create table If Not Exists Employee (Id int, Name varchar(255), Salary int, DepartmentId int);
Create table If Not Exists Department (Id int, Name varchar(255));
Truncate table Employee;
insert into Employee (Id, Name, Salary, DepartmentId) values ('1', 'Joe', '85000', '1');
insert into Employee (Id, Name, Salary, DepartmentId) values ('2', 'Henry', '80000', '2');
insert into Employee (Id, Name, Salary, DepartmentId) values ('3', 'Sam', '60000', '2');
insert into Employee (Id, Name, Salary, DepartmentId) values ('4', 'Max', '90000', '1');
insert into Employee (Id, Name, Salary, DepartmentId) values ('5', 'Janet', '69000', '1');
insert into Employee (Id, Name, Salary, DepartmentId) values ('6', 'Randy', '85000', '1');
insert into Employee (Id, Name, Salary, DepartmentId) values ('7', 'Will', '70000', '1');
Truncate table Department;
insert into Department (Id, Name) values ('1', 'IT');
insert into Department (Id, Name) values ('2', 'Sales');
```

SQL

### 本题解法

- 方法 1

```sql
SELECT d.Name AS Department,e1.Name AS Employee,e1.Salary AS Salary
FROM Employee AS e1 INNER JOIN Department AS d
ON e1.DepartmentId = d.Id
WHERE (
    SELECT COUNT(DISTINCT e2.Salary)
    FROM Employee AS e2
    WHERE e2.DepartmentId = d.Id AND e2.Salary >= e1.Salary
      ) <= 3
ORDER BY e1.DepartmentId,e1.Salary DESC;
```

SQL

- 方法2，一个注意点，部门薪水前三高包含了相同薪水下排名相同的意思
- 查询出每个部门 薪水前三高 的员工。我们可以分解查询步骤，再组合。
- 根据 部门 (升)，薪水 (降) 顺序查询出每个部门的员工 (Department, Employee, Salary)

```sql
SELECT dep.Name AS Department, emp.Name AS Employee, emp.Salary
FROM Employee AS emp INNER JOIN Department AS dep 
ON emp.DepartmentId = dep.Id
ORDER BY emp.DepartmentId, emp.Salary DESC
```

SQL

- 每个部门的员工根据薪水进行排序

```sql
SELECT te.DepartmentId, te.Salary,
       CASE 
            WHEN @pre = DepartmentId THEN @rank:= @rank + 1
            WHEN @pre := DepartmentId THEN @rank:= 1
       END AS RANK
FROM (SELECT @pre:=null, @rank:=0)tt,
     (
         SELECT DepartmentId,Salary
         FROM Employee
         GROUP BY DepartmentId,Salary
         ORDER BY DepartmentId,Salary DESC
     ) te
```

SQL

得出

| DepartmentId | Salary | RANK |
| ------------ | ------ | ---- |
| 1            | 90000  | 1    |
| 1            | 85000  | 2    |
| 1            | 70000  | 3    |
| 1            | 69000  | 4    |
| 2            | 80000  | 1    |
| 2            | 60000  | 2    |

- 组合步骤,将每个步骤变成一个结果集（没有二次查询），再将所有步骤的 结果集进行关联，从而提高性能。

```sql
SELECT dep.Name Department, emp.Name Employee, emp.Salary
FROM (
        SELECT te.DepartmentId, te.Salary,
               CASE 
                    WHEN @pre = DepartmentId THEN @rank:= @rank + 1
                    WHEN @pre := DepartmentId THEN @rank:= 1
               END AS RANK
        FROM (SELECT @pre:=null, @rank:=0)tt,
             (
                 SELECT DepartmentId,Salary
                 FROM Employee
                 GROUP BY DepartmentId,Salary
                 ORDER BY DepartmentId,Salary DESC
             )te
       )t
INNER JOIN Department dep 
ON t.DepartmentId = dep.Id
INNER JOIN Employee emp 
ON t.DepartmentId = emp.DepartmentId and t.Salary = emp.Salary and t.RANK <= 3
ORDER BY t.DepartmentId, t.Salary DESC
```

SQL

得出

| Department | Employee | Salary |
| ---------- | -------- | ------ |
| IT         | Max      | 90000  |
| IT         | Joe      | 85000  |
| IT         | Randy    | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |

## 删除重复的电子邮箱

- 题目描述
   编写一个 SQL 查询，来删除 Person 表中所有重复的电子邮箱，重复的邮箱里只保留 Id 最小 的那个。

| Id   | Email                                       |
| ---- | ------------------------------------------- |
| 1    | [john@example.com](mailto:john@example.com) |
| 2    | [bob@example.com](mailto:bob@example.com)   |
| 3    | [john@example.com](mailto:john@example.com) |

Id 是这个表的主键。

例如，在运行你的查询语句之后，上面的 Person 表应返回以下几行:

| Id   | Email                                       |
| ---- | ------------------------------------------- |
| 1    | [john@example.com](mailto:john@example.com) |
| 2    | [bob@example.com](mailto:bob@example.com)   |

- MySQL 脚本

```sql
-- ----------------------------
-- Table structure for `person`
-- ----------------------------
DROP TABLE IF EXISTS `person`;
CREATE TABLE `person` (
 `Id` int(11) DEFAULT NULL,
 `Email` varchar(255) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
-- ----------------------------
-- Records of person
-- ----------------------------
INSERT INTO `person` VALUES ('1','john@example.com');
INSERT INTO `person` VALUES ('2','bob@example.com');
INSERT INTO `person` VALUES ('3','john@example.com');
```

SQL

### 本题解法

- 查询目标：删除一条记录
- 查询范围：Person 表
- 查询条件：删除所有重复的电子邮箱 ，重复的邮箱里只保留 Id 最小的哪个。

显然，通过这个查询条件可以提取出来两条 and 关系的条件：

- 找出所有重复的电子邮箱
- 删除 Id 大的重复邮箱
- 对于条件（1），需要判断出所有重复的电子邮箱，即 p1.Email = p2.Email；
- 对于条件（2），需要判断重复邮箱中 Id 较大的：p1.Id > p2.Id

```sql
# Write your MySQL query statement below
DELETE p1
FROM Person p1,Person p2
WHERE (p1.Email = p2.Email) AND (p1.Id > p2.Id);
```

## 上升的温度

- 题目描述
   给定一个 Weather 表，编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 Id。

| Id(INT) | RecordDate(DATE) | Temperature(INT) |
| ------- | ---------------- | ---------------- |
| 1       | 2015-01-01       | 10               |
| 2       | 2015-01-02       | 25               |
| 3       | 2015-01-03       | 20               |
| 4       | 2015-01-04       | 30               |

- MySQL 脚本

```sql
-- ----------------------------
-- Table structure for `weather`
-- ----------------------------
DROP TABLE IF EXISTS `weather`;
CREATE TABLE `weather` (
 `Id` int(11) DEFAULT NULL,
 `RecordDate` date DEFAULT NULL,
 `Temperature` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
-- ----------------------------
-- Records of weather
-- ----------------------------
INSERT INTO `weather` VALUES ('1','2015-01-01', '10');
INSERT INTO `weather` VALUES ('2','2015-01-02', '25');
INSERT INTO `weather` VALUES ('3','2015-01-03', '20');
INSERT INTO `weather` VALUES ('4','2015-01-04', '30');
```

SQL

### 本题解法

我们可以使用 MySQL 的函数 DATEDIFF 来计算两个日期的差值，我们的限制条件是温度高且日期差 1。

```sql
# Write your MySQL query statement below
SELECT a.Id
FROM Weather a INNER JOIN Weather b
WHERE DATEDIFF(a.RecordDate,b.RecordDate) = 1 AND a.Temperature > b.Temperature;
```

## 行程和用户

- 本题描述
   Trips 表中存所有出租车的行程信息。每段行程有唯一键 Id，Client_Id 和 Driver_Id 是 Users 表中  Users_Id 的外键。Status 是枚举类型，枚举成员为 (‘completed’, ‘cancelled_by_driver’,  ‘cancelled_by_client’)。

| Id   | Client_Id | Driver_Id | City_Id | Status              | Request_at |
| ---- | --------- | --------- | ------- | ------------------- | ---------- |
| 1    | 1         | 10        | 1       | completed           | 2013-10-01 |
| 2    | 2         | 11        | 1       | cancelled_by_driver | 2013-10-01 |
| 3    | 3         | 12        | 6       | completed           | 2013-10-01 |
| 4    | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 |
| 5    | 1         | 10        | 1       | completed           | 2013-10-02 |
| 6    | 2         | 11        | 6       | completed           | 2013-10-02 |
| 7    | 3         | 12        | 6       | completed           | 2013-10-02 |
| 8    | 2         | 12        | 12      | completed           | 2013-10-03 |
| 9    | 3         | 10        | 12      | completed           | 2013-10-03 |
| 10   | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 |

Users 表存所有用户。每个用户有唯一键 Users_Id。Banned 表示这个用户是否被禁止，Role 则是一个表示（‘client’, ‘driver’, ‘partner’）的枚举类型。

| Users_Id | Banned | Role   |
| -------- | ------ | ------ |
| 1        | No     | client |
| 2        | Yes    | client |
| 3        | No     | client |
| 4        | No     | client |
| 10       | No     | driver |
| 11       | No     | driver |
| 12       | No     | driver |
| 13       | No     | driver |

- Mysql脚本

```sql
Create table If Not Exists Trips (Id int, Client_Id int, Driver_Id int, City_Id int, Status ENUM('completed', 'cancelled_by_driver', 'cancelled_by_client'), Request_at varchar(50))
Create table If Not Exists Users (Users_Id int, Banned varchar(50), Role ENUM('client', 'driver', 'partner'))
Truncate table Trips
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('1', '1', '10', '1', 'completed', '2013-10-01')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('2', '2', '11', '1', 'cancelled_by_driver', '2013-10-01')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('3', '3', '12', '6', 'completed', '2013-10-01')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('4', '4', '13', '6', 'cancelled_by_client', '2013-10-01')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('5', '1', '10', '1', 'completed', '2013-10-02')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('6', '2', '11', '6', 'completed', '2013-10-02')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('7', '3', '12', '6', 'completed', '2013-10-02')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('8', '2', '12', '12', 'completed', '2013-10-03')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('9', '3', '10', '12', 'completed', '2013-10-03')
insert into Trips (Id, Client_Id, Driver_Id, City_Id, Status, Request_at) values ('10', '4', '13', '12', 'cancelled_by_driver', '2013-10-03')
Truncate table Users
insert into Users (Users_Id, Banned, Role) values ('1', 'No', 'client')
insert into Users (Users_Id, Banned, Role) values ('2', 'Yes', 'client')
insert into Users (Users_Id, Banned, Role) values ('3', 'No', 'client')
insert into Users (Users_Id, Banned, Role) values ('4', 'No', 'client')
insert into Users (Users_Id, Banned, Role) values ('10', 'No', 'driver')
insert into Users (Users_Id, Banned, Role) values ('11', 'No', 'driver')
insert into Users (Users_Id, Banned, Role) values ('12', 'No', 'driver')
insert into Users (Users_Id, Banned, Role) values ('13', 'No', 'driver')
```

SQL

写一段 SQL 语句查出 2013年10月1日 至 2013年10月3日 期间非禁止用户的取消率。基于上表，你的 SQL 语句应返回如下结果，取消率（Cancellation Rate）保留两位小数。

取消率的计算方式如下：(被司机或乘客取消的非禁止用户生成的订单数量) / (非禁止用户生成的订单总数)

| Day        | Cancellation Rate |
| ---------- | ----------------- |
| 2013-10-01 | 0.33              |
| 2013-10-02 | 0.00              |
| 2013-10-03 | 0.50              |

### 本题解法：

- 方法1：
- 对trips表和users表连接，连接条件是行程对应的乘客非禁止且司机非禁止
- 筛选订单日期在目标日期之间
- 用日期进行分组
- 分别统计所有订单数和被取消的订单数，其中取消订单数用一个bool条件来得到0或1，再用avg求均值
- 对订单取消率保留两位小数，对输出列名改名

```sql
# Write your MySQL query statement below
SELECT
    request_at as 'Day', round(avg(Status!='completed'), 2) as 'Cancellation Rate'
FROM 
    trips t JOIN users u1 ON (t.client_id = u1.users_id AND u1.banned = 'No')
    JOIN users u2 ON (t.driver_id = u2.users_id AND u2.banned = 'No')
WHERE	
    request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY 
    request_at
```

SQL

- 方法2：
   非禁止用户生成的订单意思是非禁止乘客的订单，主要结合日常生活，打车订单都是由乘客发起生成的，因此换个思路，GROUP，然后俩COUNT除一下

```sql
SELECT
	t.Request_at 'Day',ROUND(
    1 - COUNT(IF(t.`Status` = 'completed', 1, NULL)) / COUNT(*) , 2) 'Cancellation Rate'
FROM	Trips t
INNER JOIN Users u ON t.Client_Id = u.Users_Id AND u.Banned = 'No' 
WHERE
  t.Request_at >= '2013-10-01' AND t.Request_at < '2013-10-04'
GROUP BY t.Request_at
```

## 大的国家

- 题目描述
   这里有张 World 表

| name        | continent | area    | population | gdp       |
| ----------- | --------- | ------- | ---------- | --------- |
| Afghanistan | Asia      | 652230  | 25500100   | 20343000  |
| Albania     | Europe    | 28748   | 2831741    | 12960000  |
| Algeria     | Africa    | 2381741 | 37100000   | 188681000 |
| Andorra     | Europe    | 468     | 78115      | 3712000   |
| Angola      | Africa    | 1246700 | 20609294   | 100990000 |

如果一个国家的面积超过 300 万平方公里，或者人口超过 2500 万，那么这个国家就是大国家。

编写一个 SQL 查询，输出表中所有大国家的名称、人口和面积。

例如，根据上表，我们应该输出:

| name        | population | area    |
| ----------- | ---------- | ------- |
| Afghanistan | 25500100   | 652230  |
| Algeria     | 37100000   | 2381741 |

- Mysql脚本

```sql
-- ----------------------------
-- Table structure for `world`
-- ----------------------------
DROP TABLE IF EXISTS `world`;
CREATE TABLE `world` (
  `name`varchar(255) DEFAULT NULL,
 `continent` varchar(255) DEFAULT NULL,
  `area`int(11) DEFAULT NULL,
 `population` int(11) DEFAULT NULL,
  `gdp`varchar(20) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
-- ----------------------------
-- Records of world
-- ----------------------------
INSERT INTO `world` VALUES ('Afghanistan','Asia', '652230', '25500100', '20343000000');
INSERT INTO `world` VALUES ('Albania','Europe', '28748', '2831741', '12960000000');
INSERT INTO `world` VALUES ('Algeria','Africa', '2381741', '37100000', '188681000000');
INSERT INTO `world` VALUES ('Andorra','Europe', '468', '78115', '3712000000');
INSERT INTO `world` VALUES ('Angola', 'Africa','1246700', '20609294', '100990000000');
```

SQL

### 本题解法

需要注意的是：如果一个国家的面积超过 300 万平方公里，或者人口超过 2500 万，那么这个国家就是大国家。

```sql
# Write your MySQL query statement below
SELECT name,population,area
FROM World
WHERE area > 3000000 OR population > 25000000;
```

## 超过 5 名学生的课

- 题目描述
   有一个 courses 表 ，有: student (学生) 和 class (课程)。

请列出所有超过或等于 5 名学生的课。

例如，表:

| student | class    |
| ------- | -------- |
| A       | Math     |
| B       | English  |
| C       | Math     |
| D       | Biology  |
| E       | Math     |
| F       | Computer |
| G       | Math     |
| H       | Math     |
| I       | Math     |

应该输出:

| class |
| ----- |
| Math  |

- 注意: 学生在每个课中不应被重复计算。
- MySQL 脚本

```sql
-- ----------------------------
-- Table structure for `courses`
-- ----------------------------
DROP TABLE IF EXISTS `courses`;
CREATE TABLE `courses` (
 `student` varchar(255) DEFAULT NULL,
  `class`varchar(255) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
-- ----------------------------
-- Records of courses
-- ----------------------------
INSERT INTO `courses` VALUES ('A', 'Math');
INSERT INTO `courses` VALUES ('B', 'English');
INSERT INTO `courses` VALUES ('C', 'Math');
INSERT INTO `courses` VALUES ('D', 'Biology');
INSERT INTO `courses` VALUES ('E', 'Math');
INSERT INTO `courses` VALUES ('F', 'Computer');
INSERT INTO `courses` VALUES ('G', 'Math');
INSERT INTO `courses` VALUES ('H', 'Math');
INSERT INTO `courses` VALUES ('I', 'Math');
```

SQL

### 本题解法：

这表里有重复记录的，比如 A 学生选了 Math 的记录可以有多条，但是认为 Math 只被 A 学生选了 1 次。

```sql
# Write your MySQL query statement below
SELECT class
FROM courses
GROUP BY class
HAVING COUNT(DISTINCT student) >= 5;
```

## 体育馆的人流量

- 题目描述
   X 市建了一个新的体育馆，每日人流量信息被记录在这三列信息中：序号 (id)、日期 (date)、 人流量 (people)。

请编写一个查询语句，找出高峰期时段，要求连续三天及以上，并且每天人流量均不少于 100。

例如，表 stadium：

| id   | date       | people |
| ---- | ---------- | ------ |
| 1    | 2017-01-01 | 10     |
| 2    | 2017-01-02 | 109    |
| 3    | 2017-01-03 | 150    |
| 4    | 2017-01-04 | 99     |
| 5    | 2017-01-05 | 145    |
| 6    | 2017-01-06 | 1455   |
| 7    | 2017-01-07 | 199    |
| 8    | 2017-01-08 | 188    |

对于上面的示例数据，输出为：

| id   | date       | people |
| ---- | ---------- | ------ |
| 5    | 2017-01-05 | 145    |
| 6    | 2017-01-06 | 1455   |
| 7    | 2017-01-07 | 199    |
| 8    | 2017-01-08 | 188    |

- 注意: 每天只有一行记录，日期随着 id 的增加而增加。
- MySQL 脚本

```sql
Create table If Not Exists stadium (id int, visit_date DATE NULL, people int);
Truncate table stadium;
insert into stadium (id, visit_date, people) values ('1', '2017-01-01', '10');
insert into stadium (id, visit_date, people) values ('2', '2017-01-02', '109');
insert into stadium (id, visit_date, people) values ('3', '2017-01-03', '150');
insert into stadium (id, visit_date, people) values ('4', '2017-01-04', '99');
insert into stadium (id, visit_date, people) values ('5', '2017-01-05', '145');
insert into stadium (id, visit_date, people) values ('6', '2017-01-06', '1455');
insert into stadium (id, visit_date, people) values ('7', '2017-01-07', '199');
insert into stadium (id, visit_date, people) values ('8', '2017-01-08', '188');
```

SQL

### 本题解法：

举例来说：(29,'2017-5-29',150),(30,'2017-6-1',150), 没有 5 月 30 日，按照日期检索就会直接跳过，但这个其实是算连续的，所以只能按照 id 来检索。

```sql
SELECT DISTINCT t1.*
FROM stadium t1, stadium t2, stadium t3
WHERE t1.people >= 100 AND t2.people >= 100 AND t3.people >= 100
AND
(
    (t1.id - t2.id = 1 AND t1.id - t3.id = 2 AND t2.id - t3.id = 1) 
    OR
    (t2.id - t1.id = 1 AND t2.id - t3.id = 2 AND t1.id - t3.id = 1) 
    OR
    (t3.id - t2.id = 1 AND t2.id - t1.id = 1 AND t3.id - t1.id = 2)
)
ORDER BY t1.id;
```

## 有趣的电影

- 题目描述
   某城市开了一家新的电影院，吸引了很多人过来看电影。该电影院特别注意用户体验，专门有个 LED 显示板做电影推荐，上面公布着影评和相关电影描述。

作为该电影院的信息部主管，您需要编写一个 SQL 查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。

例如，下表 cinema:

| id   | movie      | description | rating |
| ---- | ---------- | ----------- | ------ |
| 1    | War        | great 3D    | 8.9    |
| 2    | Science    | fiction     | 8.5    |
| 3    | irish      | boring      | 6.2    |
| 4    | Ice song   | Fantacy     | 8.6    |
| 5    | House card | Interesting | 9.1    |

对于上面的例子，则正确的输出是为：

| id   | movie      | description | rating |
| ---- | ---------- | ----------- | ------ |
| 5    | House card | Interesting | 9.1    |
| 1    | War        | great 3D    | 8.9    |

- 解题思路
- 通过MySQL来判断奇偶数
   如

```sql
num % 2 =1; num为奇数
num % 2 =0; num为偶数
```

SQL

### 本题解法

```sql
select * from cinema 
where description <> 'boring' and id % 2 =1
order by rating desc
```

## 换座位

- 题目描述
   小美是一所中学的信息科技老师，她有一张 seat 座位表，平时用来储存学生名字和与他们相对应的座位 id。

其中纵列的 id 是连续递增的

小美想改变相邻俩学生的座位。

你能不能帮她写一个 SQL query 来输出小美想要的结果呢？

示例：

| id   | student |
| ---- | ------- |
| 1    | Abbot   |
| 2    | Doris   |
| 3    | Emerson |
| 4    | Green   |
| 5    | Jeames  |

假如数据输入的是上表，则输出结果如下：

| id   | student |
| ---- | ------- |
| 1    | Doris   |
| 2    | Abbot   |
| 3    | Green   |
| 4    | Emerson |
| 5    | Jeames  |

- 注意：如果学生人数是奇数，则不需要改变最后一个同学的座位。
- MySQL 脚本

```sql
Create table If Not Exists seat(id int, student varchar(255));
Truncate table seat;
insert into seat (id, student) values ('1','Abbot');
insert into seat (id, student) values ('2','Doris');
insert into seat (id, student) values ('3','Emerson');
insert into seat (id, student) values ('4','Green');
insert into seat (id, student) values ('5','Jeames');
```

SQL

### 本题解法：

我们可以分成三块来写，第一块就是 id 为偶数的，id-1 就相当于和奇数的互换了，第二块是 id 为奇数的，id+1 就相当于和偶数的互换了，最后一块是最后一个为奇数的，不换，然后三块合并排序就出来结果了。

1. 方法1：
    需要注意的是，最后一块的过滤条件为 WHERE mod(id,2) = 1 AND id = (SELECT COUNT(*) FROM seat，需要保证 id 不为偶数。

```sql
# Write your MySQL query statement below
SELECT s.id,s.student
FROM (SELECT (id-1) AS id,student
      FROM seat
      WHERE mod(id,2) = 0
      UNION
      SELECT (id+1) AS id,student
      FROM seat
      WHERE mod(id,2) = 1 AND id != (SELECT COUNT(*) FROM seat)
      UNION
      SELECT id,student
      FROM seat
      WHERE mod(id,2) = 1 AND id = (SELECT COUNT(*) FROM seat)) s
ORDER BY s.id;
```

SQL

1. 方法2：CASE WHEN

```sql
# Write your MySQL query statement below
SELECT (CASE 
        WHEN MOD(id,2)!=0 AND id!=counts THEN id+1  
        WHEN MOD(id,2)!=0 AND id=counts THEN id  
        ELSE id-1 END) AS id,student 
        FROM seat,(SELECT COUNT(*) AS counts FROM seat) AS seat_counts  
ORDER BY id;
```