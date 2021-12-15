# sql

1. 只有 limit，没有 offset。
2. explain plan for (select)
3. 聚合函数可以加过滤，sum(xx) filter(where expr)
4. 使用 group by 之后，select 中只能查询 grouped 字段、聚合函数，除非使用 ANY_VALUE()。
5. 在 sql 中，字符串使用 `''` 包裹，字段名使用 `""` 包裹。

# 函数

官网：http://druid.apache.org/docs/latest/querying/sql.html

中文：https://www.apache-druid.cn/Querying/druidsql.html

## 数值函数

## 字符串函数

```sql
--拼接
1. a || b。
2. concat(a,b,c)。

length(a)
lower(a)--转为小写
replace(field,pattern,replacement) --在expr中用replacement替换pattern
substr(expr,index,[length]) --同 java
parse_long(field, radix) --radix是进制
```

## 时间函数

```sql
CURRENT_TIMESTAMP
CURRENT_DATE
time_parse('2020-05-04 11:11:11') --字符串转时间
MILLIS_TO_TIMESTAMP(1597804165626); --时间戳转时间
TIME_EXTRACT(__time, 'unit') --抽取时间，unit 可以是 EPOCH、SECOND、MINUTE、HOUR、DAY（月的日）、DOW（周的日）、DOY（年的日）、WEEK（年周）、MONTH（1到12）、QUARTER（1到4）或YEAR。
TIMESTAMPADD(unit,count,timestamp) --对 timestamp 以 unti 为单位增加 count，unit 同上，但是不需要用引号括起来
TIMESTAMPDIFF(unit,count,timestamp) --同上，但是是减
```

## 其他函数

```sql
case(field as type) --类型转换，type 是 druid 的类型
case ... when
nvl(field, value) --如果 field 是空的，则返回 value
```

