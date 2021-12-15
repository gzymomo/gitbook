[TOC]

# MySQL 查看相关地址信息：

```mysql
show variables where Variable_name in ('log_error','general_log','general_log_file','slow_query_log','slow_query_log_file','datadir','basedir','innodb_buffer_pool_size','performance_schema','version','character_set_database')
```

# 一、增加数据
```sql
insert into table_name
( column_name1,column_name2,.....column_nameN )
values
( value1,value2,....valueN );
```
# 二、删除数据
```sql
delete from table_name
[where conditions];
```
# 三、更新数据
```sql
update table_name
set column_name=value
[where conditions];
```
# 四、查询数据
```sql
select column_name
from table_name
[where conditions]
[limit N][ offset M];

// limit  设定返回的记录条数
// offset 指定select语句开始查询的数据偏移量。默认情况下偏移量为0。
```
# 五、模糊查询
```sql
select * from position where name like 'java%';//%匹配任意长度字符，匹配中文用%%
select * from position where name like 'java_';//_匹配任意单个字符
select * from position where name like 'java[?]';//匹配满足？条件的单个字符，[^?]匹配不满足条件的单个字符
```
# 六、交集查询
```sql
select column_name
from tables
[where conditions]

union [all | distinct]  //默认返回交集，不含重复数据，ALL返回所有交集数据

select column_name
from tables
[where conditions];
```
# 七、排序查询
```sql
select column_name
from table_name
order by field1 [asc [desc]]; 

//默认升序排列，desc为降序排列。
```
# 八、分组查询
```sql
select column_name
from table_name
[where conditions]
group by column_name;

//按colume_name进行分组，不含重复数据
```
# 九、连接查询
```sql
select *
from table_name1
inner
join table_name2
on conditions;

// inner join（内连接,或等值连接）：获取两个表中字段匹配关系的记录。
// left join（左连接）：获取左表所有记录，即使右表没有对应匹配的记录。
// right join（右连接）：用于获取右表所有记录，即使左表没有对应匹配的记录。
```
## 查询tb_dept表特有的地方
```sql
select t1.*,t2.empName,t2.deptId 
from tb_dept t1 LEFT JOIN tb_emp t2 on t1.id=t2.deptId
WHERE t2.deptId IS NULL;
```
## 全连接
```sql
select t1.*,t2.empName,t2.deptId 
from tb_dept t1 LEFT JOIN tb_emp t2 on t1.id=t2.deptId
UNION
select t1.*,t2.empName,t2.deptId 
from tb_dept t1 RIGHT JOIN tb_emp t2 on t1.id=t2.deptId
```

## 全不连接
查询两张表互不关联到的数据。
```sql
select t1.*,t2.empName,t2.deptId 
from tb_dept t1 RIGHT JOIN tb_emp t2 on t1.id=t2.deptId
WHERE t1.id IS NULL
UNION
select t1.*,t2.empName,t2.deptId 
from tb_dept t1 LEFT JOIN tb_emp t2 on t1.id=t2.deptId
WHERE t2.deptId IS NULL
```

# 建表语句
部门和员工关系表：
```sql
CREATE TABLE `tb_dept` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `deptName` varchar(30) DEFAULT NULL COMMENT '部门名称',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;
CREATE TABLE `tb_emp` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `empName` varchar(20) DEFAULT NULL COMMENT '员工名称',
  `deptId` int(11) DEFAULT '0' COMMENT '部门ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;
```

# 时间日期查询
## 日期范围内首条数据
场景：产品日常运营活动中，经常见到这样规则：活动时间内，首笔消费满多少，优惠多少。
```sql
SELECT * FROM
    (
        SELECT * FROM ms_consume
        WHERE
            create_time 
        BETWEEN '2019-12-10 00:00:00' AND '2019-12-18 23:59:59'
        ORDER BY create_time
    ) t1
GROUP BY t1.user_id ;
```
## 日期之间时差
场景：常用的倒计时场景
```sql
SELECT t1.*,
       timestampdiff(SECOND,NOW(),t1.create_time) second_diff 
FROM ms_consume t1 WHERE t1.id='9' ;
```
## 查询今日数据
-- 方式一
```sql
SELECT * FROM ms_consume 
WHERE DATE_FORMAT(NOW(),'%Y-%m-%d')=DATE_FORMAT(create_time,'%Y-%m-%d');
```
-- 方式二
```sql
SELECT * FROM ms_consume 
WHERE TO_DAYS(now())=TO_DAYS(create_time) ;
```
## 时间范围统计
场景：统计近七日内，消费次数大于两次的用户。
```sql
SELECT user_id,user_name,COUNT(user_id) userIdSum 
FROM ms_consume WHERE create_time>date_sub(NOW(), interval '7' DAY) 
GROUP BY user_id  HAVING userIdSum>1;
```
## 日期范围内平均值
场景：指定日期范围内的平均消费，并排序。
```sql
SELECT * FROM
    (
        SELECT user_id,user_name,
            AVG(consume_money) avg_money
        FROM ms_consume t
        WHERE t.create_time BETWEEN '2019-12-10 00:00:00' 
                            AND '2019-12-18 23:59:59'
        GROUP BY user_id
    ) t1
ORDER BY t1.avg_money DESC;
```

# 函数查询
##查询父级名称
```sql
DROP FUNCTION IF EXISTS get_city_parent_name;
CREATE FUNCTION `get_city_parent_name`(pid INT)
RETURNS varchar(50) CHARSET utf8
begin
    declare parentName VARCHAR(50) DEFAULT NULL;
    SELECT city_name FROM ms_city_sort WHERE id=pid into parentName;
    return parentName;
end

SELECT t1.*,get_city_parent_name(t1.parent_id) parentName FROM ms_city_sort t1 ;
```
## 查询根节点子级
```sql
DROP FUNCTION IF EXISTS get_root_child;
CREATE FUNCTION `get_root_child`(rootId INT)
    RETURNS VARCHAR(1000) CHARSET utf8
    BEGIN
        DECLARE resultIds VARCHAR(500);
        DECLARE nodeId VARCHAR(500);
        SET resultIds = '%';
        SET nodeId = cast(rootId as CHAR);
        WHILE nodeId IS NOT NULL DO
            SET resultIds = concat(resultIds,',',nodeId);
            SELECT group_concat(id) INTO nodeId
            FROM ms_city_sort WHERE FIND_IN_SET(parent_id,nodeId)>0;
        END WHILE;
        RETURN resultIds;
END  ;

SELECT * FROM ms_city_sort WHERE FIND_IN_SET(id,get_root_child(5)) ORDER BY id ;
```