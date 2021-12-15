> 数值类型是MySQL最基本的数据类型，也是使用最多的类型，主要包括整型、浮点型、精确数值型等。

##### 数值类型：

- BIT
- TINYINT
- BOOL/BOOLEAN
- SMALLINT
- MEDIUMINT
- INT/INTEGER
- BIGINT
- DECIMAL/DEC/FIXED
- FLOAT
- DOUBLE/REAL

##### BIT：

位类型，位长度为1~64，默认为1。
示例：
create table t(bitmap bit(10));
insert into t value(b'1101001011');
select bin(bitmap) from t;

##### TINYINT：

1个字节的整型，取值范围为-128~127，TINYINT UNSIGNED 取值范围为0~255。
示例：
create table t(id tinyint);
insert into t values(1);

##### BOOL/BOOLEAN：

BOOL/BOOLEAN实际上是一个TINYINT，0为FALSE，非0为TRUE。
create table t(id bool);
insert into t values(true);

##### SMALLINT：

2个字节的整型，取值范围为-32768~32767，SMALLINT UNSIGNED取值范围为0 ~ 65535。
示例：
create table t(id smallint unsigned);

##### MEDIUMINT：

3个字节的整型，取值范围为-8388608~8388607，MEDIUMINT UNSIGNED 取值范围为
0~16777215。
示例：
create table t(id mediumint);

##### INT/INTEGER：

4个字节的整型，取值范围为-2147483648~2147483647，INT UNSIGNED 取值范围为0~4294967295。
示例：
create table t(id integer unsigned);

##### BIGINT：

8字节的整型，取值范围为-9223372036854775808~9223372036854775807，BIGINT UNSIGNED取值范围为0~18446744073709551615。

SERIAL 类型实际上是BIGINT的一个别名，相当于 BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE

示例：
create table t(id bigint unsigned);

##### DECIMAL(M,D)：

精确数值类型，能够定义小数点前后的位数，M表示小数点前后总的位数，最大为65，D为小数点后的位数，最大为30。常用于金融货币，需要精确计算的场合。DEC/FIXED都是DECIMAL的别名。
示例：
create table t(id decimal(65,30));

##### FLOAT(M,D)：

单精度浮点数类型，占用4个字节。
示例：
create table t(id float(6,2));

##### DOUBLE(M,D)：

双精度浮点数精英，占用8个字节。REAL类型是DOUBLE类型的一个别名。
示例：
create table t(id double(10,4));