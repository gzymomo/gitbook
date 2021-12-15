MySQL支持地理空间数据的存储，基于GIS的相关理论，MySQL提供了配套的数据类型、内部存储格式、分析函数和空间索引，能够高效地存储、查询地理空间数据。

**1、MySQL地理空间数据类型**

- POINT，存储一个位置点数据
- LINESTRING，存储一条线数据
- POLYGON，存储一个多边形数据
- MULTIPOINT，存储多个位置点数据
- MULTILINESTRING，存储多条线数据
- MULTIPOLYGON，存储多个多边形数据
- GEOMETRY，可以存储任何POINT，LINESTRING，POLYGON 的数据
- GEOMETRYCOLLECTION，可以存储任何空间数据类型的集合

创建一个带有空间数据类型的表：

CREATE TABLE geom (g GEOMETRY NOT NULL);

插入空间数据：

INSERT INTO geom VALUES (ST_GeomFromText('POINT(1 1)'));

查询空间数据：

SELECT ST_AsText(g) FROM geom;

查询时要加上ST_AsText()函数，将空间数据以WKT文本格式显示出来，如果不用ST_AsText()函数转换，那么查询出来的空间数据将是一串二进制符号，无法阅读。

**2、MySQL支持的地理空间数据格式**

地理空间数据通常以文本方式来描述，比如一个点可以描述为 POINT(15 20)，再使用 ST_GeomFromText()函数将文本转成MySQL内部的地理空间数据，下面列出常用的地理空间数据表示方法。

**POINT：**

SELECT ST_GeomFromText('POINT(15 20)');

**LINESTRING：**

SELECT ST_GeomFromText('LINESTRING(0 0, 10 10, 20 25, 50 60)');

SELECT ST_LineStringFromText('LINESTRING(0 0, 10 10, 20 25, 50 60)');

**POLYGON：**

SELECT ST_GeomFromText('POLYGON((0 0,10 0,10 10,0 10,0 0),(5 5,7 5,7 7,5 7, 5 5))');

SELECT ST_PolygonFromText('POLYGON((0 0,10 0,10 10,0 10,0 0),(5 5,7 5,7 7,5 7, 5 5))');

**MULTIPOINT：**

SELECT ST_MPointFromText('MULTIPOINT (1 1, 2 2, 3 3)');

SELECT ST_MPointFromText('MULTIPOINT ((1 1), (2 2), (3 3))');

**MULTILINESTRING：**

SELECT ST_GeomFromText('MULTILINESTRING((10 10, 20 20), (15 15, 30 15))');

**MULTIPOLYGON：**

SELECT ST_GeomFromText('MULTIPOLYGON(((0 0,10 0,10 10,0 10,0 0)),((5 5,7 5,7 7,5 7, 5 5)))');

**GEOMETRYCOLLECTION：**

SELECT ST_GeomFromText('GEOMETRYCOLLECTION(POINT(10 10), POINT(30 30), LINESTRING(15 15, 20 20))');

SELECT ST_GeomCollFromText('GEOMETRYCOLLECTION(POINT(10 10), POINT(30 30), LINESTRING(15 15, 20 20))');

**3、MySQL地理空间索引**

创建空间索引的字段必须为NOT NULL，否则创建空间索引报错。

创建表的同时创建空间索引：

CREATE TABLE geom (g GEOMETRY NOT NULL, SPATIAL INDEX(g));

ALTER TABLE方式创建空间索引：

CREATE TABLE geom (g GEOMETRY NOT NULL); 

ALTER TABLE geom ADD SPATIAL INDEX(g);

删除空间索引：

ALTER TABLE geom DROP INDEX g;

创建好索引之后，就可以在一些空间查询函数中使用索引，比如 MBRContains()， MBRWithin()

例如：

mysql> SET @poly = 'Polygon((30000 15000, 31000 15000, 31000 16000, 30000 16000, 30000 15000))';

mysql> SELECT fid,ST_AsText(g) FROM geom WHERE MBRContains(ST_GeomFromText(@poly),g);

也可以使用explain 来查看select空间查询有没有走空间索引，用法与普通索引类似，不再赘述。

**4、MySQL中常用的地理空间函数**

MySQL提供了很多地理空间数据相关的函数，包括地理空间对象构造函数，地理空间分析函数等等，种类非常丰富，能够实现各种复杂的地理空间应用。比如

查询两个地理对象之间的距离，可使用函数 ST_Distance()。

判断两个地理对象是否有包含关系，可使用函数 ST_Contains()。

判断一个地理对象是否穿过另外一个地理对象，可使用函数 ST_Crosses()。

判断两个地理对象是否相交，可使用函数 ST_Intersects()。

关于这些函数的具体使用，参与官方文档，如下：

https://dev.mysql.com/doc/refman/5.7/en/spatial-function-reference.html