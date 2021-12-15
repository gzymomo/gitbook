> 字符串类型是MySQL使用最广泛的数据类型之一，主要包括固定长度字符串，变长字符串，大文本字符串，二进制字符串，枚举，集合等。

##### 字符串类型：

- CHAR
- VARCHAR
- BINARY
- VARBINARY
- TINYTEXT
- TEXT
- MEDIUMTEXT
- LONGTEXT
- TINYBLOB
- BLOB
- MEDIUMBLOB
- LONGBLOB
- ENUM
- SET

**CHAR(M)：**
固定长度字符串，M为字符串长度，范围为0~255。

**VARCHAR(M)：**
变长字符串，M为字符串长度，范围为0~65535，如果使用utf8，一个字符占用多个字节，那么最大长度将小于65535。

**BINARY(M)：**
固定长度二进制字符串，M为字节长度。

**VARBINARY(M)：**
可变长度的二进制字符串，M为字节长度。

**TINYTEXT：**
字符串，最大255个字节。

**TEXT(M)：**
字符串，最大65535个字符，M为字符串长度。

**MEDIUMTEXT：**
字符串，最大16777215个字符。

**LONGTEXT：**
字符串，最大4294967295个字符。

**TINYBLOB：**
二进制字符串，最大255个字节。

**BLOB(M)：**
二进制字符串，最大65535个字节，M为字节长度。

**MEDIUMBLOB：**
二进制字符串，最大16777215个字节。

**LONGBLOB：**
二进制字符串，最大4294967295个字节。

**ENUM：**
枚举类型，最大可以有65535个不同的元素。
示例：
create table t8(id enum('green','blue','red'));
insert into t8 select 'red';

**SET：**
集合类型，最大可以有64个不同的元素。
create table t9(id set('red','blue','green'));
insert into t9 select 'red,green';

##### 字符校验规则：

CHAR,VARCHAR,TEXT,ENUM,SET能够指定字符集和校验规则，这些类型定义的长度是字符长度，并不是所占的字节长度。

字符比较和排序依赖于校验规则，比如是否大小写敏感，从校验规则的名称就能看出。

- cs 表示大小写敏感，case-sensitive
- ci 表示大小写不敏感，case-insensitive
- bin 表示二进制形式校验

**示例：**
指定表的字符集和校验规则：
create table t12(id varchar(50)) CHARACTER SET utf8mb4, COLLATE utf8mb4_bin;

指定字段的字符集和校验规则：
create table t13(id varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin);