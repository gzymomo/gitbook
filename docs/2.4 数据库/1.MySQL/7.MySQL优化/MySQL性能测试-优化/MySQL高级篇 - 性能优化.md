# MySQL高级篇 - 性能优化          

- 生产过程中优化的过程

  - 观察，至少跑一天，看看生产的慢SQL情况
  - 开启慢查询日志，设置阈值，比如超过5秒钟的就是慢SQL，并将它抓取出来
  - Explain+慢SQL分析
  - show profile
  - 运维经理 or DBA，进行SQL数据库服务器的参数调优

- 目标

  - 慢查询的开启并捕获
  - explain+慢SQL分析
  - show profile查询SQL在MySQL服务器里面的执行细节和生命周期情况
  - SQL数据库服务器的参数调优

  ### 索引优化 （*）

#### 索引分析

##### 单表

- 建表SQL

  - ```mysql
    //建表
    CREATE TABLE IF NOT EXISTS `article` (
    `id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
    `author_id` INT(10) UNSIGNED NOT NULL,
    `category_id` INT(10) UNSIGNED NOT NULL,
    `views` INT(10) UNSIGNED NOT NULL,
    `comments` INT(10) UNSIGNED NOT NULL,
    `title` VARBINARY(255) NOT NULL,
    `content` TEXT NOT NULL
    );
    //插入数据
    INSERT INTO `article`(`author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES
    (1, 1, 1, 1, '1', '1'),
    (2, 2, 2, 2, '2', '2'),
    (1, 1, 3, 3, '3', '3');
    //查看表数据
    SELECT * FROM article;
    
    复制代码
    ```

    ![image-20200908105638509](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b6cd28ac6b84eb9a560adcac6e2ba59~tplv-k3u1fbpfcp-zoom-1.image)

- 案例分析

  - 查询 category_id 为1 且  comments 大于 1 的情况下,views 最多的 article_id

    - ```mysql
      SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
      复制代码
      ```

      ![image-20200908105741514](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fd62ed09f3d4d12a3e04139853913d9~tplv-k3u1fbpfcp-zoom-1.image)

    - Explain分析

      ![image-20200908105821004](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a0a9ab4324a44e0aa6f95603ce0f8f0~tplv-k3u1fbpfcp-zoom-1.image)

      结论：很显然，type是ALL，即最坏的情况，Rxtra里还出现了Using filesort，也是最坏的情况，优化是必须的

    - 查看文章表已有的索引

      ```mysql
      show index from article;
      复制代码
      ```

      ![image-20200908105925547](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f7f288975de4ff280a8d0ed69ef8d3d~tplv-k3u1fbpfcp-zoom-1.image)

    - 开始优化

      1. 第一次优化：创建复合索引

         ```mysql
         create index idx_article_ccv on article(category_id,comments,views);
         复制代码
         ```

         ![image-20200908110417751](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/533a02aecbf84252a7e7cb75261c4b35~tplv-k3u1fbpfcp-zoom-1.image)

         第二次执行Explain

         ![image-20200908110705812](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fba856c7ab9c4e489929a481cfc35eb2~tplv-k3u1fbpfcp-zoom-1.image)

         结论： type 变成了 range,这是可以忍受的。但是 extra 里使用 Using filesort 仍是无法接受的。 但是我们已经建立了索引,为啥没用呢? 这是因为按照 BTree 索引的工作原理,先排序 category_id,如果遇到相同的 category_id 则再排序 comments,如果遇到相同的 comments 则再排序 views，当 comments 字段在联合索引里处于中间位置时,因comments > 1 条件是一个范围值(所谓 range)，MySQL 无法利用索引再对后面的 views 部分进行检索,**即 range 类型查询字段后面的索引无效**

      2. 第二次优化：删除第一次优化建立的索引，重建索引

         ```mysql
         //删除索引
         DROP INDEX idx_article_ccv ON article;
         //重新创建索引
         #ALTER TABLE `article` ADD INDEX idx_article_cv ( `category_id` , `views` ) ;
         create index idx_article_cv on article(category_id,views);
         
         复制代码
         ```

         ![image-20200908111610172](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af1fc747dc6c47dd85d57240f2eab092~tplv-k3u1fbpfcp-zoom-1.image)

         第三次执行Explain

         ![image-20200908111900342](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5530e6555aac4d3ca29b85b69e9cddd2~tplv-k3u1fbpfcp-zoom-1.image)

    - 结论：可以看到,type 变为了 ref,Extra 中的 Using filesort 也消失了,结果非常理想

##### 两表(关联查询)

- 建表SQL

  - ```mysql
    CREATE TABLE IF NOT EXISTS `class` (
    `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
    `card` INT(10) UNSIGNED NOT NULL,
    PRIMARY KEY (`id`)
    );
    CREATE TABLE IF NOT EXISTS `book` (
    `bookid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
    `card` INT(10) UNSIGNED NOT NULL,
    PRIMARY KEY (`bookid`)
    );
     
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
     
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
    复制代码
    ```

- 案例分析

  - ```mysql
    Explain select * from class c left join book b on c.card=b.card;
    复制代码
    ```

    第一次执行Explain![image-20200908113345526](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - 第一次优化 ，book表（右表）card字段 添加索引

    ```mysql
    ALTER TABLE `book` ADD INDEX Y (`card`);
    复制代码
    ```

    ![image-20200908113748153](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    加索引后，第二次执行Explain

    ![image-20200908113936575](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    结论：可以看到第二行的 type 变为了 ref,rows 也变成了**优化比较明显**

  - 第二次优化，删除book表的索引，在class表（左表）的card字段，创建索引

    ```mysql
    drop index Y on book;
    ALTER TABLE class ADD INDEX X (`card`);
    show index from class;
    复制代码
    ```

    ![image-20200908114627413](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    第三次执行Explain

    ![image-20200908114836689](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    优化效果不明显

  - **结论**：由左连接特性决定的，LEFT JOIN 条件用于确定如何从右表搜索行,左边一定都有,所以**右边是我们的关键点,一定需要建立索引**；同理右连接，左边表一定要建立索引

##### 三表

- 建表SQL

  - ```mysql
    CREATE TABLE IF NOT EXISTS `phone` (
    `phoneid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
    `card` INT(10) UNSIGNED NOT NULL,
    PRIMARY KEY (`phoneid`)
    )ENGINE=INNODB;
    
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
    复制代码
    ```

- 案例分析

  - ```mysql
    Explain select * from class inner join book on class.card=book.card inner join phone on book.card=phone.card;
    复制代码
    ```

    三表关联，第一次Explain![image-20200908133931153](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - 跟phone表和book表的 card字段 创建索引

    ```mysql
    ALTER TABLE phone ADD INDEX Z (`card`);
    ALTER TABLE book ADD INDEX Y (`card`);
    复制代码
    ```

    ![image-20200908134438178](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    创建完索引后，第二次Explain![image-20200908134559319](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    优化明显，后两行的type都是ref且总的必须检查的记录数rows优化很好，效果不错。因此**索引最好设置在需要经常查询的字段中**

  - 结论：join语句的优化

    - 尽可能减少Join语句中的NestedLoop的循环总次数：**永远用小的结果集驱动大的结果集**
    - **优先**优化NestedLoop的**内层循环**
    - **保证Join语句中被驱动表上Join条件字段已经被索引**
    - 当无法保证被驱动表的Join条件字段被索引且内存资源充足的情况下，不要太吝惜**JoinBuffe**r的设置

#### 索引失效（应该避免）

- 建表SQL

  - ```mysql
    CREATE TABLE IF NOT EXISTS `staffs` (
    `id` INT(10) Primary key  AUTO_INCREMENT,
    name varchar(24) not null default ' ' comment '姓名',
    age INT not NULL default 0 comment '年龄',
    pos VARCHAR(20) not null default ' ' comment '职位',
     add_time TimeStamp not null default current_timestamp comment '入职时间'
    )charset utf8 comment '员工记录表';
    
    INSERT INTO staffs(NAME,age,pos,add_time)values('z3',22,'manager',now());
    INSERT INTO staffs(NAME,age,pos,add_time)values('july',23,'dev',now());
    INSERT INTO staffs(NAME,age,pos,add_time)values('2000',23,'dev',now());
    
    select * from staffs;
    
    //创建复合索引
    ALTER TABLE staffs ADD INDEX idx_staffs_nameAgePos(name,age,pos);
    复制代码
    ```

    ![image-20200908143352651](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 案例（索引失效）

1. 全值匹配最喜欢看到

   - ```mysql
      Explain Select * from staffs where name='july' and age=25 and pos='manager';
     复制代码
     ```

     ![image-20200908155852496](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

2. 最佳左前缀法则：如果索引了**多列**，要遵守最左前缀法则，指的是查询**从索引的最左前列开始**并且**不跳过索引中的列**

   - 当使用**覆盖索引**的方式时，(select name,age,pos from staffs where age=10 (后面没有其他没有索引的字段条件))，即使不是以 name 开头，也会使用 idx_nameAgePos 索引

   - 如果中间有跳过的列 name、pos,则只会**部分使用**索引

   - 索引  idx_staffs_nameAgePos 建立索引时 以 name ， age ，pos 的顺序建立的。全值匹配表示 按顺序匹配的

     ```mysql
     Explain select * from staffs where name='july';
     Explain select * from staffs where name='july' AND age=23;
     Explain select * from staffs where name='july' AND age=23 AND pos='dev';
     复制代码
     ```

   ![image-20200908143856734](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

   - 改变查询语句

     ```mysql
     Explain select * from staffs where age=23 AND pos='dev';
     Explain select * from staffs where pos='dev';
     复制代码
     ```

   ![image-20200908144138957](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

   未遵循最佳左前缀法则，导致了索引失效

3. 不要在索引列上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描

   - 索引列 name 上做了函数操作![image-20200908155049042](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

4. 存储引擎不能使用索引中范围条件右边的列，**范围之后全失效**

   - ```mysql
     Explain Select * from staffs where name='july' and age=25 and pos='manager';
     Explain Select * from staffs where name='july' and age>25 and pos='manager';
     复制代码
     ```

     ![image-20200908160325007](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

   - 范围 若有索引则能使用到索引，**范围条件右边的索引会失效**(范围条件右边与范围条件使用的同一个组合索引，右边的才会失效。若是不同索引则不会失效)

5. 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select *

   ![image-20200908161321041](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

6. mysql 在使用不等于(!= 或者<>)的时候无法使用索引会导致全表扫描

   ![image-20200908162939596](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

7. is not null 也无法使用索引,但是is null是可以使用索引的

   ![image-20200908170126343](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

8. like以通配符**开头**（’%abc‘）mysql索引失效会变成全表扫描的操作

   - % 写在右边可以避免索引失效

     ![image-20200908170639510](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

   - 问题：解决like ‘%字符串%’时索引不被使用的方法？？------**覆盖索引**（完全重合或包含，但不能超过）

     - **覆盖索引**：建的索引和查询的字段，最好完全一致或者包含，但查询字段不能超出索引列

     - 建表SQL

       ```mysql
       CREATE TABLE `tbl_user` (
        `id` INT(11) NOT NULL AUTO_INCREMENT,
        `NAME` VARCHAR(20) DEFAULT NULL,
        `age` INT(11) DEFAULT NULL,
        email VARCHAR(20) DEFAULT NULL,
        PRIMARY KEY (`id`)
       ) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
       
       INSERT INTO tbl_user(NAME,age,email) VALUES('1aa1',21,'b@163.com');
       INSERT INTO tbl_user(NAME,age,email) VALUES('2aa2',222,'a@163.com');
       INSERT INTO tbl_user(NAME,age,email) VALUES('3aa3',265,'c@163.com');
       INSERT INTO tbl_user(NAME,age,email) VALUES('4aa4',21,'d@163.com');
       INSERT INTO tbl_user(NAME,age,email) VALUES('aa',121,'e@163.com');
       复制代码
       ```

     - before index  未创建索引

       ```mysql
       EXPLAIN SELECT NAME,age    FROM tbl_user WHERE NAME LIKE '%aa%';
        
       EXPLAIN SELECT id    FROM tbl_user WHERE NAME LIKE '%aa%';
       EXPLAIN SELECT NAME     FROM tbl_user WHERE NAME LIKE '%aa%';
       EXPLAIN SELECT age   FROM tbl_user WHERE NAME LIKE '%aa%';
       EXPLAIN SELECT id,NAME    FROM tbl_user WHERE NAME LIKE '%aa%';
       EXPLAIN SELECT id,NAME,age FROM tbl_user WHERE NAME LIKE '%aa%';
       EXPLAIN SELECT NAME,age FROM tbl_user WHERE NAME LIKE '%aa%';
       
       复制代码
       ```

       ![image-20200908171856221](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     - 创建索引后，再观察变化（使用**覆盖索引**来解决like 导致索引失效的问题）

       ```mysql
       CREATE INDEX idx_user_nameAge ON tbl_user(NAME,age);
        
       #DROP INDEX idx_user_nameAge ON tbl_user
       
       EXPLAIN SELECT name,age FROM tbl_user WHERE NAME like '%aa%';
       EXPLAIN SELECT name FROM tbl_user WHERE NAME like '%aa%';
       EXPLAIN SELECT age  FROM tbl_user WHERE NAME like '%aa%';
       EXPLAIN SELECT id,name  FROM tbl_user WHERE NAME like '%aa%';
       EXPLAIN SELECT id,name,age  FROM tbl_user WHERE NAME like '%aa%';
       EXPLAIN SELECT name,age  FROM tbl_user WHERE NAME like '%aa%';
       复制代码
       ```

       ![image-20200908172216943](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

       完全一致或者包含的情况，成功使用覆盖索引（match匹配）![image-20200908172847457](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

       查询字段，不一致或者超出建立的索引列![image-20200908173437343](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

9. 字符串不加单引号索引失效

   - mysql 优化分析，会将int类型的777 **自动转化**为String类型，但违背了不能再索引列上进行手动或自动的转换（索引失效--案例3），导致索引失效
   - ![image-20200908173702985](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

10. 少用or，用它来连接时会索引失效

    - ![image-20200908173856916](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

11. 小总结

    - ![image-20200908174229766](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 面试题讲解（*）

- 题目SQL

  - ```mysql
    create table test03(
     id int primary key not null auto_increment,
     c1 char(10),
     c2 char(10),
     c3 char(10),
     c4 char(10),
     c5 char(10)
    );
     
    insert into test03(c1,c2,c3,c4,c5) values('a1','a2','a3','a4','a5');
    insert into test03(c1,c2,c3,c4,c5) values('b1','b2','b3','b4','b5');
    insert into test03(c1,c2,c3,c4,c5) values('c1','c2','c3','c4','c5');
    insert into test03(c1,c2,c3,c4,c5) values('d1','d2','d3','d4','d5');
    insert into test03(c1,c2,c3,c4,c5) values('e1','e2','e3','e4','e5');
     
    select * from test03;
    复制代码
    ```

    ![image-20200909104337018](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - ```mysql
    //创建复合索引
    create index idx_test03_c1234 on test03(c1,c2,c3,c4);
    show index from test03;
    复制代码
    ```

    ![image-20200909104517859](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - 问题：创建了复合索引`idx_test03_c1234` ,根据以下SQL分析索引使用情况？

    - ```mysql
      #基础
      explain select * from test03 where c1='a1';
      explain select * from test03 where c1='a1' and c2='a2';
      explain select * from test03 where c1='a1' and c2='a2' and c3='a3';
      explain select * from test03 where c1='a1' and c2='a2' and c3='a3' and c4='a4';
      复制代码
      ```

      1.基础使用![image-20200909104854866](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select * from test03 where c1='a1' and c2='a2' and c4='a4' and c3='a3'; 
      复制代码
      ```

      2.效果与 c1、c2、c3、c4按顺序使用一样，mysql底层优化器会自动优化语句，尽量保持顺序一致，可避免底层做一次翻译![image-20200909105521154](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
       explain select * from test03 where c1='a1' and c2='a2' and c3>'a3' and c4='a4';
      复制代码
      ```

      3.索引只用到部分c1、c2、c3（只用来排序，无法查找）。范围之后全失效，c4完全没有用到![image-20200909110720517](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select * from test03 where c1='a1' and c2='a2' and c4>'a4' and c3='a3';
      复制代码
      ```

      4.同理2，mysql底层优化器会自动调整语句顺序，因此索引c1、2、3、4全起效![image-20200909110245254](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select * from test03 where c1='a1' and c2='a2' and c4='a4' order by c3;
      复制代码
      ```

      5.c3作用在排序而不是查找（因此没有统计到ref）  c1、c2、c3![image-20200909110900400](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select * from test03 where c1='a1' and c2='a2' order by c3;
      复制代码
      ```

      6.同理5 c1、c2、c3![image-20200909111139157](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select * from test03 where c1='a1' and c2='a2' order by c4; 
      复制代码
      ```

      7.出现filesort 文件排序 c1、c2![image-20200909111358756](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select * from test03 where c1='a1' and c5='a5' order by c2,c3; 
      复制代码
      ```

      8.1只用c1一个字段索引查找，但是c2、c3用于排序,无filesort![image-20200909111732039](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

      ```mysql
      explain select * from test03 where c1='a1' and c5='a5' order by c3,c2;
      复制代码
      ```

      8.2出现了filesort，我们建的索引是1234，它没有按照顺序来，3 2 颠倒了![image-20200909111845428](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select * from test03 where c1='a1' and c2='a2' order by c2,c3;
      复制代码
      ```

      9.用c1、c2两个字段索引，但是c2、c3用于排序,无filesort![image-20200909112054301](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select * from test03 where c1='a1' and c2='a2' and c5='a5' order by c3,c2;  
      复制代码
      ```

      10.本例对比8.2 多了c2常量（无需排序），不会导致filesort

      ![image-20200909112606114](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    - ```mysql
      explain select c2,c3  from test03 where c1='a1' and c4='a4' group by c2,c3;
      explain select c2,c3  from test03 where c1='a1' and c4='a4' group by c3,c2;
      复制代码
      ```

      11.**group by 分组之前必排序**![image-20200909113433673](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- > 定值（常量）、范围（范围之后皆失效）还是排序（**索引包含查找排序两部分**），一般order by是给个范围

- group by 基本上是需要进行排序，会有临时表产生

#### 一般性建议

- 对于单键索引，尽量选择针对当前query过滤性更好的索引
- 在选择组合索引的时候，当前Query中过滤性最好的字段在索引字段顺序中，位置越靠前（左）越好
- 在选择组合索引的时候，尽量选择能够包含当前query中的where字句更多字段的索引
- 尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的

## 查看截取分析

- 生产过程中优化的过程
  - 观察，至少跑一天，看看生产的慢SQL情况
  - 开启慢查询日志，设置阈值，比如超过5秒钟的就是慢SQL，并将它抓取出来
  - Explain+慢SQL分析
  - show profile
  - 运维经理 or DBA，进行SQL数据库服务器的参数调优
- 总结
  - 慢查询的开启并捕获
  - explain+慢SQL分析
  - show profile查询SQL在MySQL服务器里面的执行细节和生命周期情况
  - SQL数据库服务器的参数调优

### 查询优化

#### 永远小表驱动大表（子查询）

- 案例

  - 优化原则：小表驱动大表，即小的数据集驱动大的数据集

    - 原理 `in`

      ```mysql
      select * from A where id in(select id From B);
      ##等价于(嵌套循环)
      for select id from B
      for select * from A where A.id=B.id
      复制代码
      ```

    - 当B表的数据集必须小于A表的数据集时，用`in`优于`exists`

    - 原理 `exists`

      ```mysql
      select ...from table where exists(subQuery)
      复制代码
      ```

      该语法可以理解为：**将主查询的数据，放到子查询中做条件验证，根据验证结果（true或false）来决定主查询的数据结果是否得以保留**

    - 提示：

      1. Exists(subquery)只返回TRUE或FALSE，因此子查询中的SELECT * 也可以是SELECT 1 或其他，官方说法是实际执行时会忽略掉SELECT清单，因此没有区别
      2. Exists子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比，如果担忧效率问题，可进行实际校验以确定是否有效率问题
      3. Exists子查询往往也可以用条件表达式、其他子查询或者JOIN来替代，何种最优需要计提问题具体分析

    - 当A表的数据集小于B表的数据集时，用`exists`优于`in`

      ```mysql
      select * from A where exists (select 1 from B where B.id=A.id)
      ##等价于
      for select * from A
      for select * from B where B.id=A.id		
      复制代码
      ```

    - 注意：A表与B表的ID字段应建立索引

#### ORDER BY 关键字优化

- ORDER BY子句，尽量使用Index方式排序,避免使用FileSort方式排序

  - 建表SQL

    ```mysql
    CREATE TABLE tblA(
      id int primary key not null auto_increment,
      age INT,
      birth TIMESTAMP NOT NULL,
      name varchar(200)
    );
     
    INSERT INTO tblA(age,birth,name) VALUES(22,NOW(),'abc');
    INSERT INTO tblA(age,birth,name) VALUES(23,NOW(),'bcd');
    INSERT INTO tblA(age,birth,name) VALUES(24,NOW(),'def');
     
    CREATE INDEX idx_A_ageBirth ON tblA(age,birth,name);
     
    SELECT * FROM tblA; 
    
    复制代码
    ```

    ![image-20200909161345508](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - Case

    ```mysql
    Explain Select * From tblA where age>20 order by age;
    Explain Select * From tblA where age>20 order by age,birth;
    #是否产生filesort
    Explain Select * From tblA where age>20 order by birth;
    Explain Select * From tblA where age>20 order by birth,age;
    复制代码
    ```

    ![image-20200909161938105](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    ```mysql
    explain select * From tblA order by birth;
    explain select * From tblA Where birth >'2020-9-09 00:00:00' order by birth;
    
    explain select * From tblA Where birth >'2020-9-09 00:00:00' order by age;
    
    explain select * From tblA  order by age ASC,birth DESC; #mysql默认升序
    
    复制代码
    ```

    ![image-20200909162616690](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - MySQL支持两种方式排序，FileSort和Index，Index效率高，它指MySQL扫描索引本身完成排序，FileSort方式效率较低

  - ORDER BY满足两种情况，会使用Index方式排序：

    - ORDER BY语句使用索引最左前列
    - 使用WHERE子句与ORDER BY子句条件列组合满足索引最左前列

- 尽可能在索引列上完成排序操作，遵循索引建的**最佳左前缀**

- 如果不在索引列上，filesort有两种算法：

  - 双路排序
    - MySQL 4.1之前是使用双路排序,字面意思就是两次扫描磁盘，最终得到数据，读取行指针和orderby列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出
    - 从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段
    - 取一批数据，要对磁盘进行了**两次扫描**，众所周知，I/O是很耗时的，所以在mysql4.1之后，出现了第二种改进的算法，就是单路排序
  - 单路排序
    - 从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO,但是它会使用更多的空间，因为它把每一行都保存在内存中了
  - 结论及引申出的问题
    - 由于单路是后出的，总体而言好过双路
    - 但是用单路有问题： 在sort_buffer中，方法B比方法A要多占用很多空间，因为方法B是把所有字段都取出, 所以有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并），排完再取取sort_buffer容量大小，再排……从而多次I/O，本来想省一次I/O操作，反而导致了大量的I/O操作，反而得不偿失

- 优化策略

  - 增大sort_buffer_size参数的设置
    - 用于单路排序的内存大小
  - 增大max_length_for_sort_data参数的设置
    - 单次排序字段大小(单次排序请求)
  - 提高ORDER BY的速度
    - Order by时select * 是一个大忌只Query需要的字段， 这点非常重要。在这里的影响是：
      - 当Query的字段大小总和小于max_length_for_sort_data 而且排序字段不是 TEXT|BLOB 类型时，会用改进后的算法——单路排序， 否则用老算法——多路排序
      - 两种算法的数据都有可能超出sort_buffer的容量，超出之后，会创建tmp文件进行合并排序，导致多次I/O，但是用单路排序算法的风险会更大一些,所以要提高sort_buffer_size
    - 尝试提高 sort_buffer_size。不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对每个进程的
    - 尝试提高 max_length_for_sort_data。提高这个参数，会增加用改进算法的概率。但是如果设的太高，数据总容量超出sort_buffer_size的概率就增大，明显症状是高的磁盘I/O活动和低的处理器使用率

- 小总结（*）

  - 为排序使用索引

    - MySQL两种排序方式：文件排序或扫描有序索引排序

    - MySQL能为排序与查询使用相同的索引

    - KEY a_b_c(a,b,c)

      - order by 能使用索引最左前缀

        - ORDER BY a
        - ORDER BY a,b
        - ORDER BY a,b,c
        - ORDER BY a DESC,b DESC,c DESC

      - 如果WHERE使用索引的最左前缀定义为常量，则ORDER BY能使用索引

        - WHERE a = const ORDER BY b,c
        - WHERE a = const AND b=const ORDER BY c
        - WHERE a = const AND b > const ORDER BY b,c

      - 不能

        使用索引进行排序

        - ORDER BY a ASC,b DESC,c DESC  //排序不一致
        - WHERE g = const ORDER BY b,c  // 丢失a索引
        - WHERE a = const ORDER BY c   //丢失b索引
        - WHERE a = const ORDER BY a,d  //d不是索引的一部分

#### GROUP BY 关键字优化

- GROUP BY 实质是先排序后进行分组，遵照索引建的最佳左前缀（其他大致同 ORDER BY）
- 当无法使用索引列，增大max_length_for_sort_data 参数的设置 ＋ 增大sort_buffer_size参数的设置
- WHERE高于HAVING，能写在WHERE限定的条件就不要去HAVING限定了

### 慢查询日志

#### 是什么

- MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中**响应时间超过阀值**的语句，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中
- 具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time的**默认值为10**，意思是运行10秒以上的语句
- 由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析

#### 怎么玩

##### 说明

- 默认情况下，MySQL数据库**没有开启慢查询日志**，需要我们手动来设置这个参数。当然，**如果不是调优需要的话，一般不建议启动该参数**，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支持将日志记录写入文件

##### 查看是否开启及如何开启

- 默认

  - ```mysql
    SHOW VARIABLES LIKE '%slow_query_log%';
    复制代码
    ```

  - 默认情况下slow_query_log的值为OFF,表示慢查询日志是禁用的，可以通过设置slow_query_log的值来开启

    ![image-20200910092846735](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 开启

  - ```mysql
    set global slow_query_log=1;
    复制代码
    ```

  - 使用该命令开启了慢查询日志只对当前数据库生效，如果MySQL**重启后则会失效**

    ![image-20200910093127655](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - 如果要永久生效，就必须修改配置文件my.cnf（其它系统变量也是如此）

    修改my.cnf文件，[mysqld]下增加或修改参数`slow_query_log`和`slow_query_log_file`后，重启mysql服务器

    ```c
    slow_query_log=1
    slow_query_log_file=/var/lib/mysql/touchair-slow.log
    复制代码
    ```

  - 关于慢查询的参数 slow_query_log_file, 它指定慢查询日志文件的存放路径，**系统默认会给一个缺省的文件 host_name-slow,log** （如果没有指定参数 slow_query_log_file 的话）

##### 查看慢查询内容

- 这个是**由参数long_query_time控制**，默认情况下long_query_time的值为**10秒**；

  - 命令：

    ```mysql
    SHOW VARIABLES LIKE 'long_query_time%';
    复制代码
    ```

    ![image-20200910094442577](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

    假如运行时间正好等于long_query_time的情况，并不会被记录下来。也就是说，在mysql源码里是判断**大于long_query_time，而非大于等于**

- 使用命令设置阙值超过3秒钟就是慢SQL

  - ```mysql
    set global long_query_time=3;
    //再次查看
    SHOW VARIABLES LIKE 'long_query_time%';
    复制代码
    ```

- 会发现查看 long_query_time 的值并没有改变？原因：

  - 需要重新连接或者新开一个会话才能看到修改值

  - 或修改查看命令

    ```mysql
    SHOW global VARIABLES LIKE 'long_query_time%';
    复制代码
    ```

    ![image-20200910094852252](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### Case

- 记录慢SQL，并后续分析

  - 执行一条休眠4秒的SQL

    ```mysql
    SELECT SLEEP(4);
    复制代码
    ```

  - 查看日志文件

    前面配置的日志文件路径或者默认路径

    ```shell
    cat /var/lib/mysql/localhost-slow.log 
    复制代码
    ```

    ![image-20200910100054048](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 查询当前系统中有多少条慢查询记录

  - ```mysql
    show global status like '%Slow_queries%';
    复制代码
    ```

    ![image-20200910100526099](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 配置版

- 【mysqld】下配置：

  ```c
  slow_query_log=1;
  slow_query_log_file=/var/lib/mysql/touchair-slow.log
  long_query_time=3;
  log_output=FILE
  复制代码
  ```

#### 日志分析工具mysqldumpslow（*）

- 查看mysqldumpslow的帮助信息

  - mysqldumpslow --help;

    ![image-20200910101515571](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - s: 是表示按照何种方式排序；

  - c: 访问次数

  - l: 锁定时间

  - r: 返回记录

  - t: 查询行数

  - al:平均锁定时间

  - ar:平均返回记录数

  - at:平均查询时间

  - t:即为返回前面多少条的数据；

  - g:后边搭配一个正则匹配模式，大小写不敏感的；

- 工作常用参考

  - 得到返回记录集最多的10个SQL

    ```shell
    mysqldumpslow -s r -t 10 /var/lib/mysql/localhost-slow.log
    复制代码
    ```

  - 得到访问次数最多的10个SQL

    ```shell
    mysqldumpslow -s c -t 10 /var/lib/mysql/localhost-slow.log
    复制代码
    ```

  - 得到安装时间排序的前10条里面含有左连接的查询语句

    ```mysql
    mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/localhost-slow.log
    复制代码
    ```

  - 另外建议在使用这些命令时，结合 | 和 more使用，否则有可能出现爆屏情况

    ```shell
    mysqldumpslow -s r -t 10 /var/lib/mysql/localhost-slow.log | more
    复制代码
    ```

### 批量数据库脚本（模拟大批量数据）

- 往数据库表里插1000w条数据

  1. 建表SQL

     ```mysql
     # 新建库
     create database bigData;
     use bigData;
     
     #1 建表dept
     CREATE TABLE dept(  
     id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  
     deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,   
     dname VARCHAR(20) NOT NULL DEFAULT "",  
     loc VARCHAR(13) NOT NULL DEFAULT ""  
     ) ENGINE=INNODB DEFAULT CHARSET=UTF8 ;  
     
     #2 建表emp
     CREATE TABLE emp  
     (  
     id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  
     empno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0, /*编号*/  
     ename VARCHAR(20) NOT NULL DEFAULT "", /*名字*/  
     job VARCHAR(9) NOT NULL DEFAULT "",/*工作*/  
     mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,/*上级编号*/  
     hiredate DATE NOT NULL,/*入职时间*/  
     sal DECIMAL(7,2) NOT NULL,/*薪水*/  
     comm DECIMAL(7,2) NOT NULL,/*红利*/  
     deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 /*部门编号*/  
     )ENGINE=INNODB DEFAULT CHARSET=UTF8 ; 
     复制代码
     ```

  2. 设置参数log_bin_trust_function_creators

     - 创建函数，假如报错：This function has none of DETERMINISTIC......，由于开启过慢查询日志，因为我们开启了 bin-log, 我们就必须为我们的function指定一个参数

     - ```mysql
       show variables like 'log_bin_trust_function_creators';
       set global log_bin_trust_function_creators=1;
       复制代码
       ```

       ![image-20200910104106990](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

     - 这样添加了参数以后，如果mysqld重启，上述参数又会消失，永久方法：

       - windows下my.ini[mysqld]加上log_bin_trust_function_creators=1
       - linux下    /etc/my.cnf下my.cnf[mysqld]加上log_bin_trust_function_creators=1

  3. 创建函数，保证每条数据都不同

     - 随机产生字符串

       ```mysql
       DELIMITER $$
       CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
       BEGIN    ##方法开始
        DECLARE chars_str VARCHAR(100) DEFAULT   'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ'; 
        ##声明一个 字符窜长度为 100 的变量 chars_str ,默认值 
        DECLARE return_str VARCHAR(255) DEFAULT '';
        DECLARE i INT DEFAULT 0;
       ##循环开始
        WHILE i < n DO  
        SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
       ##concat 连接函数  ，substring(a,index,length) 从index处开始截取
        SET i = i + 1;
        END WHILE;
        RETURN return_str;
       END $$
        
       #假如要删除
       #drop function rand_string;
       复制代码
       ```

     - 随机产生部门编号

       ```mysql
       #用于随机产生部门编号
       DELIMITER $$
       CREATE FUNCTION rand_num( ) 
       RETURNS INT(5)  
       BEGIN   
        DECLARE i INT DEFAULT 0;  
        SET i = FLOOR(100+RAND()*10);  
       RETURN i;  
        END $$
        
        
       #假如要删除
       #drop function rand_num;
       复制代码
       ```

     1. 创建存储过程

        - 创建往emp表中插入数据的存储过程

          ```mysql
          DELIMITER $$
          CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))  
          BEGIN  
          DECLARE i INT DEFAULT 0;   
          #set autocommit =0 把autocommit设置成0  ；提高执行效率
           SET autocommit = 0;    
           REPEAT  ##重复
           SET i = i + 1;  
           INSERT INTO emp(empno, ename ,job ,mgr ,hiredate ,sal ,comm ,deptno ) VALUES ((START+i) ,rand_string(6),'SALESMAN',0001,CURDATE(),FLOOR(1+RAND()*20000),FLOOR(1+RAND()*1000),rand_num());  
           UNTIL i = max_num   ##直到  上面也是一个循环
           END REPEAT;  ##满足条件后结束循环
           COMMIT;   ##执行完成后一起提交
           END $$
           
          #删除
          # DELIMITER ;
          # drop PROCEDURE insert_emp;
          复制代码
          ```

        - 创建往dept表中插入数据的存储过程

          ```mysql
          #执行存储过程，往dept表添加随机数据
          DELIMITER $$
          CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))  
          BEGIN  
          DECLARE i INT DEFAULT 0;   
           SET autocommit = 0;    
           REPEAT  
           SET i = i + 1;  
           INSERT INTO dept (deptno ,dname,loc ) VALUES (START +i ,rand_string(10),rand_string(8));  
           UNTIL i = max_num  
           END REPEAT;  
           COMMIT;  
           END $$ 
           
          #删除
          # DELIMITER ;
          # drop PROCEDURE insert_dept;
          复制代码
          ```

        1. 调用存储过程

           - dept

             ```mysql
             DELIMITER ;
             CALL insert_dept(100,10); 
             复制代码
             ```

           - emp

             ```mysql
             #执行存储过程，往emp表添加50万条数据
             DELIMITER ;    #将 结束标志换回 ;
             CALL insert_emp(100001,500000); 
             复制代码
             ```

             插入50w条数据，耗时约24s![image-20200910110608068](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

             查询50w条数据，耗时约0.67s![image-20200910110438640](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

### Show Profile（生命周期）

#### 是什么

- 是mysql提供可以用来分析当前会话中语句执行的**资源消耗情况**。可以用于SQL的调优的测量
- [官网](https://dev.mysql.com/doc/refman/5.7/en/preface.html)

#### 默认情况下，参数处于关闭状态，并保存最近15次的运行结果

#### 分析步骤

##### 1.是否支持，看看当前的mysql版本是否支持

- 默认关闭，使用前需要开启

- 查看状态

  ```mysql
  SHOW VARIABLES LIKE 'profiling';
  复制代码
  ```

  ![image-20200910112854968](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 2.开启功能，默认是关闭，使用前需要开启

- 开启命令

  ```mysql
  set profiling=1;
  复制代码
  ```

  ![image-20200910113003706](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 3.运行SQL

- select * from emp group by id%10 limit 150000;

  - 执行不通过，原因：

    - SQL 标准中不允许 SELECT 列表，HAVING 条件语句，或 ORDER BY 语句中出现 GROUP BY 中未列表的可聚合列。而 MySQL 中有一个状态 ONLY_FULL_GROUP_BY 来标识是否遵从这一标准，默认为开启状态

    - 关闭命令

      ```mysql
      SET SESSION sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY,',''));
      复制代码
      ```

  - ![image-20200910114442250](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- select * from emp group by id%20  order by 5;

  - ![image-20200910114602169](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 4.查看结果，show profiles

- 命令

  ```mysql
  show profiles;
  复制代码
  ```

  ![image-20200910133856680](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 5.诊断SQL

- show profile cpu,block io for query  n  (n为上一步前面的问题SQL数字号码)

- 参数说明

  | TYPE               | 解释说明                                                     |
  | ------------------ | ------------------------------------------------------------ |
  | \|ALL              | 显示所有的开销信息                                           |
  | \|BLOCK IO         | 显示块IO相关开销                                             |
  | \|CONTEXT SWITCHES | 上下文切换相关开销                                           |
  | \|CPU              | 显示CPU相关开销信息                                          |
  | \|IPC              | 显示发送和接收相关开销信息                                   |
  | \|MEMORY           | 显示内存相关开销信息                                         |
  | \|PAGE FAULTS      | 显示页面错误相关开销信息                                     |
  | \|SOURCE           | 显示和Source_function，Source_file，Source_line相关的开销信息 |
  | \|SWAPS            | 显示交换次数相关开销的信息                                   |

- ```mysql
  show profile cpu,block io for query  17;
  复制代码
  ```

  ![image-20200910133815297](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 6.日常开发**需要注意**的结论

- 出现 `converting HEAP to MyISAM` ：查询结果太大，内存都不够用了往磁盘上搬了

- 出现 

  ```
  Creating tmp table
  ```

  ：创建临时表

  - 拷贝数据到临时表
  - 用完再删除

- 出现 `Copying to tmp table on disk`：把内存中临时表复制到磁盘，危险！！！

- 出现 `locked`

### 全局日志查询

#### 配置启用

- 在mysql的my.cnf中，设置如下：

  ```c
  #开启
  general_log=1   
  # 记录日志文件的路径
  general_log_file=/path/logfile
  #输出格式
  log_output=FILE
  复制代码
  ```

#### 编码启用

- 开启命令

  ```mysql
  set global general_log=1;
  复制代码
  ```

- 全局日志可以存放到日志文件中，也可以存放到Mysql系统表中。存放到日志中性能更好一些，存储到表中

  ```mysql
  set global log_output='TABLE';
  复制代码
  ```

- 此后 ，你所编写的sql语句，将会记录到mysql库里的general_log表，可以用下面的命令查看

  ```mysql
  select * from mysql.general_log;
  复制代码
  ```

  ![image-20200910135010674](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  ![image-20200910135021407](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

#### 尽量不要在生产环境开启这个功能

## MySQL锁机制

### 概述

#### 定义

- 锁是计算机**协调**多个进程或线程并发访问某一资源的机制
- 在数据库中，除传统的计算资源（如CPU、RAM、I/O等）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂

#### 锁的分类

##### 从对数据操作的类型分（读/写）

- 读锁(**共享锁**)：针对同一份数据，多个读操作可以同时进行而不会互相影响
- 写锁（**排它锁**）：当前写操作没有完成前，它会阻断其他写锁和读锁

##### 从对数据操作的粒度分

- 为了尽可能提高数据库的并发度，每次锁定的数据范围越小越好，理论上每次只锁定当前操作的数据的方案会得到最大的并发度，但是管理锁是很耗资源的事情（涉及获取，检查，释放锁等动作），因此数据库系统需要在高并发响应和系统性能两方面进行平衡，这样就产生了“锁粒度（Lock granularity）”的概念
- 一种提高共享资源并发发性的方式是让锁定对象更有选择性。尽量只锁定需要修改的部分数据，而不是所有的资源。更理想的方式是，只对会修改的数据片进行精确的锁定。任何时候，在给定的资源上，锁定的数据量越少，则系统的并发程度越高，只要相互之间不发生冲突即可
- **表锁**
- **行锁**

### 三锁

- 开销、加锁速度、死锁、粒度、并发性能；只能就具体应用的特点来说哪种锁更合适

#### 表锁（偏读）

##### 特点

- 偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低

##### 案例分析

- 建表SQL

  ```mysql
  create table mylock(
   id int not null primary key auto_increment,
   name varchar(20)
  )engine myisam;
   
  insert into mylock(name) values('a');
  insert into mylock(name) values('b');
  insert into mylock(name) values('c');
  insert into mylock(name) values('d');
  insert into mylock(name) values('e');
   
  select * from mylock;
  复制代码
  ```

  ![image-20200910140755801](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 手动增加表锁

  ```mysql
  lock table 表名字1 read(write),表名字2 read(write),其它;
  复制代码
  ```

- 查看表上加过的锁

  ```mysql
  show open tables;
  复制代码
  ```

  IN_USE 0 代表当前没有锁![image-20200910141334740](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 释放表锁

  ```mysql
  unlock tables;
  复制代码
  ```

- 给`mylock`、`dept`表分别添加读锁和写锁

  ```mysql
  lock table mylock read,dept write;
  复制代码
  ```

  ![image-20200910142014886](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  再次查看表上加过的锁

  ![image-20200910142041863](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  ![image-20200910142110811](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  ![image-20200910142131876](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  释放锁

  ![image-20200910142241609](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 加读锁

  - 新建一个MySQL会话，方便测试

  - | session1                                                     | session2                                                     |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | session1可以读![image-20200910142937434](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) | session2可以读![image-20200910142911311](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) |
    | session1无法查询其它没有锁定的表![image-20200910143231126](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) | session2查询其它表不受影响![image-20200910143302934](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) |
    | 当前session1插入或者更新读锁锁定的表，会直接报错![image-20200910143414509](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) | session2插入或更新会一直等待获取锁![image-20200910143501469](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) |
    | 当前session1释放锁![image-20200910143646421](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) | session2获得锁资源：完成上一步一直等待的更新操作![image-20200910143627432](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) |

- 加写锁

  - **mylockwrite(MyISAM)**

  - ```mysql
    lock table mylock write;
    复制代码
    ```

    ![image-20200910152552489](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  | session1                                                     | session2                                                     |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | 当前session1对锁定的表的查询+更新+插入操作都可以执行![image-20200910153707795](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) | 其他session对锁定的表的查询被阻塞，需等待锁被释放![image-20200910153758366](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) |
  |                                                              | 在锁表前，如果session2有数据缓存，锁表以后，在锁住的表不发生改变的情况下session2可以读出缓存数据，一旦数据发生改变，缓存将失效，操作将被阻塞住，最好使用不同的id进行测试 |
  | session1释放锁![image-20200910154022340](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) | session2获得锁，返回查询结果![image-20200910154105524](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>) |

##### 案例结论

- MyISAM在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁，在执行增删改操作前，会自动给涉及的表加写锁

- MySQL的表级锁有两种模式：

  - 表共享读锁（Table Read Lock）
  - 表独占写锁（Table Write Lock）

  | 锁类型 | 可否兼容 | 读操作 | 写操作 |
  | ------ | -------- | ------ | ------ |
  | 读锁   | 是       | 是     | 否     |
  | 写锁   | 是       | 否     | 否     |

- 结论：

  结合上表，所有对MyISAM表进行操作，会有以下情况：

  - 对MyISAM表的读操作（加读锁），不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求，只有当读锁释放后，才会执行其他进程的写操作
  - 对MyISAM表的写操作（加写锁），会阻塞其他进程读同一表的读和写操作，只有当写锁释放后，才会执行其他进程的读写操作
  - 简而言之，就是读锁会阻塞写，但不阻塞读操作。而写操作则会把读和写都阻塞

##### 表锁分析

- 看看哪些表被加锁了

  ```mysql
  show open tables;
  复制代码
  ```

- 如何分析表锁定

  ```mysql
  show status like 'table%';
  复制代码
  ```

  通过检查table_locks_waited 和 table_locks_immediate 状态变量来分析系统上的表锁定

  ![image-20200910155628663](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 这里有两个状态变更记录MySQL内部表级锁定的情况，两个变量说明如下：

  - `Table_locks_immediate`:产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1
  - `Table_locks_waited`:出现表级锁定争用而发生等待次数（不能立即获取锁的次数，每等待一次锁值加1），此值高则说明存在着较严重的表级锁争用情况

- 此外，MyISAM的读写锁调度是写优先，**这也是MyISAM不适合做写为主的表的引擎**，因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成**永久阻塞**

#### 行锁（偏写）

##### 特点

- 偏向InnoDB存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高
- InnoDB与MyISAM的最大不同有两点：
  - 一是**支持事务**（TRANSACTION）
  - 二是采用了**行级锁**

##### 复习老知识点

- 事务及其ACID属性

  - 原子性（Atomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行
  - 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的
  - 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然
  - 持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持

- 并发事务处理带来的问题

  - 更新丢失(Lost Update)
    - 当两个或多个事务选择同一行，然后基于最初选定的值更新该行时，由于每个事务都不知道其他事务的存在，就会发生丢失更新问题－－最后的更新覆盖了由其他事务所做的更新
  - 脏读(Dirty Reads)
    - 一个事务正在对一条记录做修改，在这个事务完成并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做”脏读”
    - 一句话：事务A读取到了事务B已修改但尚未提交的的数据，还在这个数据基础上做了操作。此时，如果B事务回滚，A读取的数据无效，不符合一致性要求
  - 不可重复读(Non-Repeatable Reads)
    - 在一个事务内，多次读同一个数据。在这个事务还没有结束时，另一个事务也访问该同一数据。那么，在第一个事务的两次读数据之间。由于第二个事务的修改，那么第一个事务读到的数据可能不一样，这样就发生了在一个事务内两次读到的数据是不一样的，因此称为不可重复读，即原始读取不可重复
    - 句话：一个事务范围内两个**相同的查询**却返回了**不同数据**
  - 幻读(Phantom Reads)
    - 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读”
    - 一句话：事务A 读取到了事务B提交的新增数据，不符合隔离性

- 事务隔离级别

  - 脏读”、“不可重复读”和“幻读”，其实都是数据库**读一致性**问题，必须由数据库提供一定的**事务隔离机制**来解决

  - | 读数据一致性及允许的并发副作用隔离级别 | 读数据一致性                       | 脏读 | 不可重复读 | 幻读 |
    | -------------------------------------- | ---------------------------------- | ---- | ---------- | ---- |
    | 未提交读（Read uncommitted）           | 最低级别，只能保证物理上损坏的数据 | 是   | 是         | 是   |
    | 以提交读（Read committed）             | 语句级                             | 否   | 是         | 是   |
    | 可重复读（Repeatable read）            | 事务级                             | 否   | 否         | 是   |
    | 可序列化（Serializable）               | 最高级别，事务级                   | 否   | 否         | 否   |

  - 数据库的事务隔离越严格，并发副作用越小，但付出的代价也就越大，因为事务隔离实质上就是使事务在一定程度上 “串行化”进行，这显然与“并发”是矛盾的。同时，不同的应用对读一致性和事务隔离程度的要求也是不同的，比如许多应用对“不可重复读”和“幻读”并不敏感，可能更关心数据并发访问的能力

  - **差看当前数据库的事务隔离级别**：show variables like 'tx_isolation';

##### 案例分析

- 建表SQL

  ```mysql
  create table test_innodb_lock (a int(11),b varchar(16))engine=innodb;
   
  insert into test_innodb_lock values(1,'b2');
  insert into test_innodb_lock values(3,'3');
  insert into test_innodb_lock values(4,'4000');
  insert into test_innodb_lock values(5,'5000');
  insert into test_innodb_lock values(6,'6000');
  insert into test_innodb_lock values(7,'7000');
  insert into test_innodb_lock values(8,'8000');
  insert into test_innodb_lock values(9,'9000');
  insert into test_innodb_lock values(1,'b1');
   
  create index test_innodb_a_ind on test_innodb_lock(a);
   
  create index test_innodb_lock_b_ind on test_innodb_lock(b);
   
  select * from test_innodb_lock;
  复制代码
  ```

  ![image-20200910163316885](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

- 行锁定基本演示

  - 关闭自动提交 session1、session2

    ```mysql
    set autocommit=0;
    复制代码
    ```

    ![image-20200910163643444](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0931dbc0eb244e9b4ee348cd0760219~tplv-k3u1fbpfcp-zoom-1.image)

  - 正常情况，各自**锁定各自的行**，互相不影响，一个2000另一个3000

- **无索引行锁升级为表锁**

  - 正常情况，各自锁定**各自的行**，**互相不影响**
  - 由于在column字段b上面建了索引，如果没有正常使用，会导致行锁变表锁
    - 比如没加单引号导致索引失效，行锁变表锁
    - 被阻塞，等待。只到Session_1提交后才阻塞解除，完成更新

| session1                                                     | session2                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 更新session1中的一条记录，未手动commit，故意写错b的类型![image-20200911092824894](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eec88318bf8e4b97a0d7a06ea938c6fd~tplv-k3u1fbpfcp-zoom-1.image) |                                                              |
| session1,commit提交![image-20200911093141141](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3a70cf32e644011b23cf134de75c056~tplv-k3u1fbpfcp-zoom-1.image) | 更新session2，阻塞等待锁释放![image-20200911093043823](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29e41dbdb50b4ffa9aab0f58b500a864~tplv-k3u1fbpfcp-zoom-1.image) |
|                                                              | session2完成update                                           |

- 间隙锁危害
  - 什么是间隙锁
    - 当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（GAP Lock）
  - 危害
    - 因为Query执行过程中通过过范围查找的话，他会**锁定整个范围内所有的索引键值，即使这个键值并不存在**
    - 间隙锁有一个比较**致命的弱点**，就是当锁定一个范围键值之后，即使某些**不存在**的键值**也会被无辜的锁定**，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害

##### 面试题

- 常考如何锁定某一行

  ![image-20200911094557313](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 案例结论

- Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势了
- 但是，Innodb的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让Innodb的整体性能表现不仅不能比MyISAM高，甚至可能会更差

##### 行锁分析

- 如何分析行锁定

  - 通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况

    ```mysql
    show status like 'innodb_row_lock%';
    复制代码
    ```

    ![image-20200911095119419](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4078c6cb5d634609ab6f7f6e75b075c9~tplv-k3u1fbpfcp-zoom-1.image)

- 各个状态量说明

  - Innodb_row_lock_current_waits：当前正在等待锁定的数量
  - Innodb_row_lock_time：从系统启动到现在锁定总时间长度
  - Innodb_row_lock_time_avg：每次等待所花平均时间
  - Innodb_row_lock_time_max：从系统启动到现在等待最常的一次所花的时间
  - Innodb_row_lock_waits：系统启动后到现在总共等待的次数

- 对于这5个状态变量，**比较重要的主要是**：

  - **Innodb_row_lock_time_avg（等待平均时长）**
  - **Innodb_row_lock_waits（等待总次数）**
  - **Innodb_row_lock_time（等待总时长）**
  - **尤其是当等待次数很高**，而且每次等待时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手指定优化计划

- **查询正在被锁阻塞的sql语句**

  ```mysql
  SELECT * FROM information_schema.INNODB_TRX\G;
  复制代码
  ```

  ![image-20200911095622273](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

##### 优化建议

- 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁
- 尽可能较少检索条件，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间长度
- 锁住某行后，尽量不要去调别的行或表，赶紧处理被锁住的行然后释放掉锁
- 涉及相同表的事务，对于调用表的顺序尽量保持一致
- 在业务环境允许的情况下,尽可能低级别事务隔离

#### 页锁

- 开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般
- 了解一下即可，目前使用较少

## 主从复制

### 复制的基本原理

#### slave会从master读取binlong来进行数据同步

#### 三步骤＋原理图

![image-20200911100510799](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

MySQL复制过程分成三步：

1. master将改变记录到**二进制日志**（binary log），这些记录过程叫做二进制日志事件，binary log events
2. slave将master的binary log events拷贝到它的**中继日志**（relay log）
3. slave重做中继日志中的事件，将改变应用到自己的数据库中，MySQL复制是**异步的且串行化的**

### 复制的基本原则

#### 每个slave只有一个Master

#### 每个slave只能有一个唯一的服务器ID

#### 每个Master可以有多个Slave

### 复制的最大问题

#### 延时

### 一主一从常见配置

#### 主从配置

- 1.mysql版本一致且后台以服务运行 （同5.7，本机win和虚拟机centos）

- 2.主从都配置在[mysqld]结点下，都是小写

- 3.主机（win）修改my.ini配置文件

  - windows+r --- services.msc 找到mysql服务 右击属性查看配置文件`my.ini`的位置

    ![image-20200911114926051](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abcbab32f6bb4de5a294085faade2609~tplv-k3u1fbpfcp-zoom-1.image)

  - 1.[必须]主服务器唯一ID

    - server-id=1

  - 2.[必须]启用二进制日志

    - log-bin=自己本地的路径/data/mysqlbin
    - log-bin=D:\Mysql\mysql-5.7.28-winx64\mysql-5.7.28-winx64\data\mysqlbin

  - 3.[可选]启用错误日志

    - log-err=D:\Mysql\mysql-5.7.28-winx64\mysql-5.7.28-winx64\data\mysqlerr

  - 4.[可选]根目录

    - basedir=D:\Mysql\mysql-5.7.28-winx64\mysql-5.7.28-winx64

  - 5.[可选]临时目录

    - tmpdir=D:\Mysql\mysql-5.7.28-winx64\mysql-5.7.28-winx64

  - 6.[可选]数据目录

    - datadir=D:\Mysql\mysql-5.7.28-winx64\mysql-5.7.28-winx64\data

  - 7.read-only=0

    - 主机，读写都可以

  - 8.[可选]设置不要复制的数据库

    - binlog-ignore-db=mysql

  - 9.[可选]设置需要复制的数据库

    - binlog-do-db=需要复制的主数据库名字

    ![image-20200911131817659](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

  - 10.配置完成后重启mysql服务

- 4.从机修改my.cnf配置文件

  - [必须]从服务器唯一ID

  - [可选]启用二进制日志

  - vim /etc/my.cnf

    ![image-20200911105241412](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8337af339fb4431f98b2f4250c0b5ce8~tplv-k3u1fbpfcp-zoom-1.image)

  - 重启mysql服务

    ```shell
    service mysqld restart
    复制代码
    ```

- 5.主机从机都关闭防火墙

  - windows手动关闭
  - 关闭虚拟机linux防火墙    service iptables stop

- 6.在Windows主机上建立帐户并授权slave

  - 授权命令

    ```mysql
    GRANT REPLICATION SLAVE ON *.* TO 'zhangsan'@'从机器数据库IP' IDENTIFIED BY '123456';
    复制代码
    ```

    ```mysql
    GRANT REPLICATION SLAVE ON *.* TO 'root'@'192.168.83.133' IDENTIFIED BY '123456';
    复制代码
    ```

    ![image-20200911110354863](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/958f98573ca548eaa4d9cffcbe9feb7a~tplv-k3u1fbpfcp-zoom-1.image)

  - flush privileges;     （刷新）

    ![image-20200911110611897](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/861fe64fae104d5c858a3e3afeef70fd~tplv-k3u1fbpfcp-zoom-1.image)

  - 查询master的状态

    - ```mysql
      show master status;
      复制代码
      ```

      ![image-20200911135427088](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4d2d940cb2c401f88e8cd01fd831846~tplv-k3u1fbpfcp-zoom-1.image)

      记下File 和 Position 的值

  - 执行完此步骤后不要再操作主服务器MYSQL，防止主服务器状态值变化

- 7.在Linux从机上配置需要复制的主机

  - 从机命令（如果之前同步过，先停止（stop slave;）再次授权）

    ```mysql
    CHANGE MASTER TO MASTER_HOST='192.168.1.8', 
    MASTER_USER='root',
    MASTER_PASSWORD='123456',
    #MASTER_LOG_FILE='mysqlbin.具体数字',MASTER_LOG_POS=具体值;
    MASTER_LOG_FILE='mysqlbin.000002',MASTER_LOG_POS=154;
    复制代码
    ```

    ![image-20200911135453266](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/933f4af1a4b54b5eb55acea6a821a9bb~tplv-k3u1fbpfcp-zoom-1.image)

  - 启动从服务器复制功能

    ```mysql
    start slave;
    复制代码
    ```

  - 查看slave状态

    ```mysql
    show slave status\G;
    复制代码
    ```

    ![image-20200911135552224](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/979bd06932d94889a73ecee98a41a3a4~tplv-k3u1fbpfcp-zoom-1.image)

  `Slave_IO_Running: Yes`

  `Slave_SQL_Running:Yes`

  两个参数都是Yes,说明主从配置成功！

#### 测试主从效果

- 主机新建库、新建表、insert记录，从机复制

  ```mysql
  #建库
  create database mydb77;
  #建表
  create table touchair(id int not null,name varchar(20));
  #插入数据
  insert into touchair valies(1,'a');
  insert into touchair values(2,'b');
  复制代码
  ```

  ![image-20200911140232776](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2114d28db384b68a19ddeedb279c41c~tplv-k3u1fbpfcp-zoom-1.image)

  - 从机上查看是否有库、表、数据

    ```mysql
    use mydb77;
    select * from touchair;
    复制代码
    ```

    ![image-20200911140341601](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0803a7ce29484502826e89522b8eba3b~tplv-k3u1fbpfcp-zoom-1.image)

- 主从复制成功！

- 如何停止从服务复制功能

  ```mysql
  stop slave;
  ```


作者：几个你_
链接：https://juejin.cn/post/6921495541897510919