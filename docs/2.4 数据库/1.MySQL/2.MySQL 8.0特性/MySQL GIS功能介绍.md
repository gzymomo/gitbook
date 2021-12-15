# MySQL · 功能介绍 · GIS功能介绍

从MySQL4.1开始，MySQL就支持了基本空间数据类型以及一部分的空间对象函数，但是对GIS功能的支持非常有限；随着不断发展，MySQL8对GIS功能的支持已经比较丰富了，本文将基于MySQL8.0.18版本对MySQL的GIS功能进行介绍。PolarDB MySQL8.0最新版本即将支持包含GIS函数的并行查询，可以显著提高GIS查询的性能。

## MySQL支持的GIS数据

### GIS数据类型

MySQL的GIS功能遵守OGC的OpenGIS Geometry Model，支持其定义的空间数据类型的一个子集，包括以下空间数据类型:

- GEOMETRY：不可实例化的数据类型，但是可以作为一个列的类型，存储任何一种其他类型的数据
- POINT：点
- LINESTRING：线
- POLYGON：多边形，由多条闭合的线构成的图形
- MULTIPOINT：点集合
- MULTILINESTRING：线集合
- MULTIPOLYGON：多边形集合
- GEOMCOLLECTION：空间对象集合

其中GEOEMTRY、POINT、LINESTRING、POLYGON用于保存单个空间数据，并且GEOMETRY可以存储其它任意单个空间数据类型，即如果一个字段定义是GEOMETRY类型，那么该字段可以存储其它类型（不包括集合）的数据，而其它类型必须存储特定类型的数据，例如：

```
mysql> CREATE TABLE test(a INT primary key, geom GEOMETRY SRID 4326) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Query OK, 0 rows affected, 1 warning (0.25 sec)

mysql> INSERT INTO test VALUES(1, ST_SRID(Point(1,1), 4326));
Query OK, 1 row affected (0.02 sec)

mysql> INSERT INTO test VALUES(2, ST_SRID(LineString(Point(1,1), Point(2,2)), 4326));
Query OK, 1 row affected (0.00 sec)

mysql> select a, ST_AsText(geom) from test;
+---+---------------------+
| a | ST_AsText(geom)     |
+---+---------------------+
| 1 | POINT(1 1)          |
| 2 | LINESTRING(1 1,2 2) |
+---+---------------------+
mysql>  CREATE TABLE test(a INT primary key, geom POINT SRID 4326) ENGINE=InnoDB DEFAULT CHARSET=utf8;
ERROR 1050 (42S01): Table 'test' already exists
mysql> drop table test;
Query OK, 0 rows affected (0.06 sec)

mysql>  CREATE TABLE test(a INT primary key, geom POINT SRID 4326) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Query OK, 0 rows affected, 1 warning (0.22 sec)

mysql> INSERT INTO test VALUES(1, ST_SRID(Point(1,1), 4326));
Query OK, 1 row affected (0.02 sec)

mysql> INSERT INTO test VALUES(2, ST_SRID(LineString(Point(1,1), Point(2,2)), 4326));
ERROR 1416 (22003): Cannot get geometry object from data you send to the GEOMETRY field
mysql> INSERT INTO test VALUES(2, ST_SRID(Point(1,1), 4326));
Query OK, 1 row affected (0.01 sec)
```

列geom中的类型为GEOMETRY时插入POINT类型数据和LINESTRING类型数据都可以，而类型为POINT时插入LINESTRING类型数据会失败。 MULTIPOINT、MULTILINESTRING、MULTIPOLYGON、GEOMCOLLECTION用于保存空间数据集合，GEOMCOLLECTION可以存储任意空间数据的集合，即GEOEMTRY类型数据的集合，其他的空间数据集合只能存储特定类型数据的集合。

### GIS对象属性

MySQL中每个GIS对象都具有一些属性，MySQL中有一系列函数来获取GIS对象的属性，目前MySQL中GIS对象具有以下属性：

- 类型(type)：每个GIS对象都会有一个类型，表示了该对象是上述八种类型中的哪一种。
- 空间参考标识符(SRID)：每个GIS对象都对应一个SRID，该值标识了当前对象属于哪个空间参考系(SRS)。目前MySQL支持地理和投影两类共5152个空间参考系，MySQL也支持用户自定义空间参考系，可以通过查询表information_schema.st_spatial_reference_systems获取系统中所有空间参考系的详细定义。MySQL中比较常用空间参考系的有SRID 0以及SRID 4326，SRID 0是一个无限大的平面笛卡尔坐标系，如果一个对象没有定义SRID，则默认使用SRID 0；SRID  4326是GPS系统使用的地理空间参考系。在MySQL中对GIS对象进行计算时各个GIS对象必须在同一坐标系中，不在同一参考系的话会报错，如果需要对不同坐标系中的对象计算可以使用MySQL提供的坐标系转换函数在不同的空间坐标系中进行转换。
- 坐标(coordinates)：每一非空对象都会有一个坐标，至少有一对(X,Y)坐标，坐标加上SRID可以定义一个空间对象。需要注意的是如果是在不同的空间参考系中，有相同坐标的对象之间的距离也可能是不一样的。
- 最小边界矩形(MBR, minimum bounding rectangle)：由空间对象最外部顶点为界构造出的矩形，具体定义如下：

```
((MINX MINY, MAXX MINY, MAXX MAXY, MINX MAXY, MINX MINY))
```

- 简单(simple)：表示一个对象是否是简单的，只有LineString、MultiPoint、MultiLineString具备这一属性，没有交叉点的线是简单的，没有相同点的点集合是简单的，集合中所有的线都是简单的线集合是简单的。
- 闭合(Closed)：表示一个对象是否是闭合的，只有LineString、MultiLineString有这一属性，Start  Point和End  Point相同的LineString是闭合的，如果MultiLineString中所有的LineString是闭合的，那么它也是闭合的。
- 边界(boundary)：每一个对象都会在空间中占据一部分位置，对象外部是指对象未占据的空间、对象内部是指占据的空间、而对象边界在两者之间。特别的：点的边界为空，闭合的线的边界为空，非闭合的线的边界为两个端点，点集合的边界也为空，线集合的边界为其中所有奇数次顶点的集合。
- 空(empty)：表示一个对象是否为空，没有任何点的对象就是空的，目前MySQL中只有空的GEOMRTRYCOLLECTION是有效的。
- 维度(dimension)：空间对象的维度，目前MySQL最多只支持二维空间对象，因此该值有-1、0、1、2四种，-1是空对象具有的维度、0是点的维度、1是线的维度、2是多边形的维度，集合的维度和其中元素的最大维度相同。

虽然MySQL支持上述八种空间数据类型，但是在Server层都是使用的Field_geom类来处理空间数据列的，Server层统一的类型是MYSQL_TYPE_GEOMETRY， 在`sql/spatial.cc`中定义了GIS数据类型及相关操作。目前MySQL支持GIS数据的存储引擎有MyASIM、InnoDB、NDB以及ARCHIVE，其中MyASIM和InnoDB还支持建立空间索引。

### GIS数据格式

MySQL中支持WKT以及WKB两种格式来操作GIS数据，同时在内部以另外一种格式存储GIS数据。

#### WKT

即文本格式，在用户操作GIS类型的数据时可以使用直观的文本进行插入或查询，MySQL支持OpenGIS定义的语法来写WKT数据，示例如下：

- Point：`POINT(15 20)`
- LineString：`LINESTRING(0 0, 10 10, 20 25, 50 60)`
- Polygon ：`POLYGON((0 0,10 0,10 10,0 10,0 0),(5 5,7 5,7 7,5 7, 5 5))`
- MultiPoint：`MULTIPOINT(0 0, 20 20, 60 60)`
- MultiLineString：`MULTILINESTRING((10 10, 20 20), (15 15, 30 15))`
- MultiPolygon：`MULTIPOLYGON(((0 0,10 0,10 10,0 10,0 0)),((5 5,7 5,7 7,5 7, 5 5)))`
- GeometryCollection：`GEOMETRYCOLLECTION(POINT(1 -1), POINT(10 10), POINT(30 30), LINESTRING(15 15, 20 20))`

在用户进行插入时可以使用ST_GeomFromText等函数来将WKT格式的GIS数据转换成内部格式进行插入，在进行查询时可以使用ST_AsText函数来将内部数据转换为更直观的WKT结果格式。

#### WKB

该格式时OpenGIS定义的一个格式，使用二进制数据来保存信息，例如平面坐标系下的点(1,-1)的WKB格式为：

```
0101000000000000000000F03F000000000000F0BF`
```

其中第一个字节标识了字节序，0是大端，1是小端；紧接着的四个字节表示数据类型，目前MySQL只支持上述七种数据类型（不包括GEOMETRY），因此只使用了1到7来分别标识七种数据；因为点只有x坐标和y坐标，所以紧接着的16字节中前8字节是x坐标，后8字节是y坐标。 在WKB格式中，前五个字节是所有类型的GIS数据都具有的，而后续具体表示数据的内容就因类型而异了，复杂的数据类型有复杂的表达方式，具体可以参考[OpenGIS标准](https://www.ogc.org/standards/sfs)。用户可以通过ST_GeomFromWKB等函数来将WKB格式的GIS数据转换为内部格式插入到系统中，在进行查询时可以使用ST_AsWKB等函数将内部数据转换成WKB结果格式。

#### 内部存储格式

MySQL内部存储格式与WKB格式类似，也是使用二进制格式来保存GIS数据，所以MySQL内部存储GIS数据实际上使用的是blob类型。MySQL内部存储的数据格式是四字节的SRID加上WKB格式的数据，同样以平面坐标系下的点(1,-1)为例，点(1,-1)在内部存储的数据为：

```
000000000101000000000000000000F03F000000000000F0BF
```

其中前四个字节标识了对象的SRID，例中为SRID 0；接下来的21个字节就是WKB格式表示了，需要注意的是在MySQL中都是按小端序来存储GIS数据的，因此字节序标记一直为1。

## GIS函数

MySQL提供了一系列的空间数据分析计算函数，大部分空间数据计算函数是基于Boost.Geometry库实现的，选择Boost的原因可以参考Manyi Lu的[博客](http://mysqlserverteam.com/why-boost-geometry-in-mysql/)。空间数据计算函数主要分为空间对象构造、空间数据格式转换、空间对象属性、空间对象关系计算，空间对象生成等等几大类，MySQL中GIS函数的实现定义在`sql/item_geofunc*`等文件中，其中大部分定义在`sql/item_geofunc.h`和`sql/item_geofunc.cc`文件中。

### 空间对象构造函数

这类函数用于构造空间对象，其中点是由两个坐标构成的，其他的对象都是基于点生成的。

| 函数名                                               | 描述                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| GeomCollection(g [, g] …)，GeomCollection(g [, g] …) | 构造一个地理空间集合，比如一个包含多个点的集合，参数可为空   |
| Point(x, y)                                          | 构造一个点对象，参数是两个double型数据，可以是x、y轴值也可以是经纬度值 |
| LineString(pt [, pt] …)                              | 根据多个点构造一条线，可以不是直线，参数至少是两个点         |
| Polygon(ls [, ls] …)                                 | 构造一个多边形对象，参数是一条或多条线                       |
| MultiLineString(ls [, ls] …)                         | 根据多条线构造一个线的集合，参数是一条或多条线               |
| MultiPoint(pt [, pt2] …)                             | 根据多个点构造一个点的集合，参数是一个或多个点               |
| MultiPolygon(poly [, poly] …)                        | 根据多个多边形构造一个多边形的集合，参数是一个或多个多边形   |

### 空间数据格式转换函数

这类函数用于将空间对象从一种格式转化成另一种格式，主要是内部格式和WKB、WKT格式的相互转换，可用于空间数据的导入导出。

| 函数名                                                       | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ST_GeomCollFromText(wkt [, srid [, options]])，ST_GeometryCollectionFromText(wkt [, srid [, options]])，ST_GeomCollFromTxt(wkt [, srid [, options]]) | 根据wkt格式的输入参数返回几何集合                            |
| ST_GeomFromText(wkt [, srid [, options]])，ST_GeometryFromText(wkt [, srid [, options]]) | 根据wkt格式的参数返回几何对象，与上一函数不同之处在于该函数接受单个对象为输入，而上一函数接受的空间对象集合 |
| ST_LineFromText(wkt [, srid [, options]])，ST_LineStringFromText(wkt [, srid [, options]]) | 根据wkt格式的输入参数构造一个linestring对象                  |
| ST_MLineFromText(wkt [, srid [, options]])，ST_MultiLineStringFromText(wkt [, srid [, options]]) | 根据wkt格式的输入参数构造一个multilinestring对象             |
| ST_MPointFromText(wkt [, srid [, options]])，ST_MultiPointFromText(wkt [, srid [, options]]) | 根据wkt格式的输入参数构造一个multipoint对象                  |
| ST_MPolyFromText(wkt [, srid [, options]])，ST_MultiPolygonFromText(wkt [, srid [, options]]) | 根据wkt格式的输入参数构造一个multipolygon对象                |
| ST_PolyFromText(wkt [, srid [, options]])，ST_PolygonFromText(wkt [, srid [, options]]) | 根据wkt格式的输入参数构造一个polygon对象                     |
| ST_PointFromText(wkt [, srid [, options]])                   | 根据wkt格式的输入参数构造一个point对象                       |
| ST_GeomCollFromWKB(wkb [, srid [, options]])，ST_GeometryCollectionFromWKB(wkb [, srid [, options]]) | 从wkb格式的参数返回几何集合                                  |
| ST_GeomFromWKB(wkb [, srid [, options]]), ST_GeometryFromWKB(wkb [, srid [, options]]) | 根据wkb格式的参数返回几何对象，与上一函数不同之处在于该函数接受单个对象为输入，而上一函数接受的空间对象集合 |
| ST_LineFromWKB(wkb [, srid [, options]]), ST_LineStringFromWKB(wkb [, srid [, options]]) | 根据wkb格式的输入参数构造一个linestring对象                  |
| ST_MLineFromWKB(wkb [, srid [, options]]), ST_MultiLineStringFromWKB(wkb [, srid [, options]]) | 根据wkb格式的输入参数构造一个multilinestring对象             |
| ST_MPointFromWKB(wkb [, srid [, options]]), ST_MultiPointFromWKB(wkb [, srid [, options]]) | 根据wkb格式的输入参数构造一个multipoint对象                  |
| ST_MPolyFromWKB(wkb [, srid [, options]]), ST_MultiPolygonFromWKB(wkb [, srid [, options]]) | 根据wkb格式的输入参数构造一个multipolygon对象                |
| ST_PointFromWKB(wkb [, srid [, options]])                    | 根据wkb格式的输入参数构造一个point对象                       |
| ST_PolyFromWKB(wkb [, srid [, options]]), ST_PolygonFromWKB(wkb [, srid [, options]]) | 根据wkb格式的输入参数构造一个polygon对象                     |
| ST_AsBinary(g [, options]),ST_AsWKB(g [, options])           | 将内部几何格式参数转换为WKB形式，并返回二进制结果            |
| ST_AsText(g [, options]), ST_AsWKT(g [, options])            | 将内部几何格式的参数转换为其WKT表示形式，并返回字符串结果    |
| ST_SwapXY(g)                                                 | 交换空间对象中所有顶点坐标的x轴和y轴值                       |
| ST_AsGeoJSON(g [, max_dec_digits [, options]])               | 将一个空间对象转换成一个GeoJSON形式输出                      |
| ST_GeomFromGeoJSON(str [, options [, srid]])                 | 读取一个以geojson格式字符串输入的空间对象                    |

### 空间对象属性函数

| 函数名                                              | 描述                                                         |                                                              |
| --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ST_Dimension(g)                                     | 返回空间对象的维度，维度有-1、0, 1, 2，-1是空对象的维度，0是点的维度 |                                                              |
| ST_Envelope(g)                                      | 返回对象的MBR（minimum bounding rectangle），如果参数是点或者垂直或水平的直线，则返回参数本身 |                                                              |
| ST_GeometryType(g)                                  | 返回空间对象的类型，参数可以是任意一个空间对象实例           |                                                              |
| ST_IsEmpty(g)                                       | 返回一个对象集合是否是空的                                   |                                                              |
| ST_IsSimple(g)                                      | 返回一个对象是否是简单的，简单对象不会发生自相交和自相切     |                                                              |
| ST_SRID(g [, srid])                                 | 返回一个空间对象的SRS号（一个参数时）或者为一个空间对象设置SRS（两个参数时） |                                                              |
| ST_Latitude(p [, new_latitude_val])                 | 返回一个点的纬度值（一个参数时）或者设置一个点的纬度值（两参数时） |                                                              |
| ST_Longitude(p [, new_longitude_val])               | 返回一个点的经度值（一个参数时）或者设置一个点的经度值（两参数时） |                                                              |
| ST_X(p [, new_x_val])                               | 返回一个点的x坐标（一个参数时）或设置一个点的x坐标（两参数时） |                                                              |
| ST_Y(p [, new_y_val])                               | 返回一个点的y坐标（一个参数时）或设置一个点的y坐标（两参数时） |                                                              |
| ST_EndPoint(ls)                                     | 返回线段的终点，参数只可是lingstring对象，否则会返回NULL值，正常返回值是point |                                                              |
| ST_IsClosed(ls)                                     | 返回一个lingstring是否是闭环（头顶点和尾顶点是同一个），返回值为0或1，参数可以是linestring和multilinestring（multilinestring需要所有的linestring都是闭环才会返回1） |                                                              |
| ST_Length(ls [, unit])                              | 返回linestring或者multilinestring的长度，multilinestring的长度是其中所有linestring的长度和 |                                                              |
| ST_NumPoints(ls)                                    | 返回linestring上点的个数，参数只能是lingstring否则返回NULL值 |                                                              |
| ST_PointN(ls, N)                                    | 返回linestring中第N（第二个参数决定）个顶点                  |                                                              |
| ST_StartPoint(ls)                                   | 返回linestring的开始顶点（第一个点）                         |                                                              |
| ST_Area({poly                                       | mpoly})                                                      | 计算多边形的面积，参数可以是Polygon或者MultiPolygon，MultiPolygon的面积是其中所有多边形面积的和 |
| ST_Centroid({poly                                   | mpoly})                                                      | 返回多边形的质心，参数是polygon或者multipolygon，返回值是一个点 |
| ST_ExteriorRing(poly)                               | 返回一个多边形对象的外圈，返回值是linestring（一条线），参数只能是polygon，否则报错或返回NULL |                                                              |
| ST_InteriorRingN(poly, N)                           | 返回多边形的第N（第二个参数）个的内环，只支持polygon，返回值是linestring |                                                              |
| ST_NumInteriorRing(poly)，ST_NumInteriorRings(poly) | 返回多边形内环的个数，参数只能是polygon类型对象              |                                                              |
| ST_GeometryN(gc, N)                                 | 读取空间对象集合中的第N（第二个参数）个对象，第一个参数需要是GeometryCollection |                                                              |
| ST_NumGeometries(gc)                                | 返回空间对象集合中对象的个数，参数只能是geometrycollection类型对象 |                                                              |

### 空间对象关系计算函数

这类函数主要是计算两个空间对象之间的关系，比如包含、相交、距离等等。

| 函数名                       | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| ST_Contains(g1, g2)          | 测试第一个参数的对象是否包含第二个参数的对象，与ST_Within正好相反 |
| ST_Crosses(g1, g2)           | 测试两个空间对象是否交叉，非一维对象指的是两个对象有相同的点，但不是完全相同，一维对象指有交叉点而无公共线段，但是如果参数一是polygon或者multipolygon或参数二是point或者multipoint时会返回NULL值 |
| ST_Disjoint(g1, g2)          | 测试两个空间对象是否不相交，是ST_Crossess的反函数            |
| ST_Distance(g1, g2 [, unit]) | 返回两个空间对象的距离，可用于任何空间对象，第三个参数可用于指定距离单位，比如米（metre）、英尺（foot）等 |
| ST_Equals(g1, g2)            | 两个空间对象是否在空间上相同                                 |
| ST_Intersects(g1, g2)        | 返回两个对象是否相交，与ST_Crosses不同的是它可以处理各个空间对象，没有polygon和point的限制 |
| ST_Overlaps(g1, g2)          | 返回两个对象是否重叠,跟ST_Intersects的区别在与该函数只在相交部分和对象的维度相同是才返回1 |
| ST_Touches(g1, g2)           | 返回两个空间对象是否相邻，即边界挨着，但不相交或重叠         |
| ST_Within(g1, g2)            | 返回参数一是否在参数二内部，与ST_Contains相反                |
| MBRContains(g1, g2)          | 测试第一个参数的MBR是否包含第二个参数的MBR，与MBRWithin正好相反 |
| MBRCoveredBy(g1, g2)         | 测试第二个参数的MBR是否覆盖了第一个参数的MBR，与MBRCovers相反 |
| MBRCovers(g1, g2)            | 测试第一个参数的MBR是否覆盖了第二个参数的MBR，与MBRCoveredBy相反 |
| MBRDisjoint(g1, g2)          | 测试两个参数的MBR是否不相交                                  |
| MBREquals(g1, g2)            | 测试两个参数的MBR是否相同                                    |
| MBRIntersects(g1, g2)        | 测试两个参数的MBR是否相交                                    |
| MBROverlaps(g1, g2)          | 测试两个参数的MBR是否重叠                                    |
| MBRTouches(g1, g2)           | 测试两个参数的MBR是否相邻，边界挨着而不重叠                  |
| MBRWithin(g1, g2)            | 测试第二个参数的MBR是否包含第一个参数的MBR，与MBRWithin正好相反 |

### 空间对象生成函数

这类函数依据某种规则从一个已有的空间对象生成另一个空间对象，比如生成凸包、坐标系转换等等。

| 函数名                                                    | 描述                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| ST_Buffer(g, d [, strategy1 [, strategy2 [, strategy3]]]) | 返回一个空间对象，返回的空间对象中的所有的点距离参数一表示的对象都小于等于参数二指定的长度，参数三指定计算策略 |
| ST_Buffer_Strategy(strategy [, points_per_circle])        | 根据输入的参数返回一个值用于指定ST_Buffer函数使用的策略（参数三） |
| ST_ConvexHull(g)                                          | 生成参数对象的凸包，凸包是指包含给定集合的凸集的交集，不严谨的说是最外层的点连接起来构成的凸多边形 |
| ST_Difference(g1, g2)                                     | 返回参数一减去参数二的对象                                   |
| ST_Intersection(g1, g2)                                   | 返回两个空间对象的交集，返回值可能是各种类型，取决于输入     |
| ST_SymDifference(g1, g2)                                  | 返回两个对象的对称差集，即两对象的并集减去两对象的交集       |
| ST_Transform(g, target_srid)                              | 将一个空间对象转换到另一个坐标系中，第二个参数决定了目标坐标系 |
| ST_Union(g1, g2)                                          | 返回两个空间对象的并集，目前只支持平面坐标系                 |

### 其他函数

这类函数主要是一些提供其他操作以提高应用灵活性的函数。

| 函数名                                                       | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ST_GeoHash(longitude, latitude, max_length)，ST_GeoHash(point, max_length) | 返回一个点的geohash值（字符串），第二或三个参数定义了hash值的长度，可以通过[比较geohash值的相似度来判断点的距离](https://www.cnblogs.com/LBSer/p/3310455.html) |
| ST_LatFromGeoHash(geohash_str)                               | 从geohash中读取纬度值                                        |
| ST_LongFromGeoHash(geohash_str)                              | 从geohash中读取经度值                                        |
| ST_PointFromGeoHash(geohash_str, srid)                       | 从geohash值中解析出point点，point点是以经纬度形式输出的      |
| ST_Distance_Sphere(g1, g2 [, radius])                        | 返回两个空间对象的球面距离(以米为单位)，目前只可用于point以及multipoint对象，第三个参数指定了球的半径 |
| ST_IsValid(g)                                                | 返回一个空间对象是否是有效的，空对象只有对象集合有效，其余看实际对象，比如linestring(0 0, -0 0, 0 0)就不是有效的 |
| ST_MakeEnvelope(pt1, pt2)                                    | 返回两个点的MBR，与ST_Envelope不同之处在于该函数只支持点作为参数，返回值可能是点（两个参数相同），线（两个参数构成水平或垂直的线）或者多边形（其他情况） |
| ST_Simplify(g, max_distance)                                 | 使用Douglas-Peucker算法对曲线进行简化，第二参数（必须是正数）决定了距离阈值，第一个参数可以是任意类型，但是理论上Douglas-Peucker算法只能处理曲线 |
| ST_Validate(g)                                               | 检查一个空间对象是否有效，比如顶点全是(0 0)的多边形或者曲线就是无效的，例外空的geometry collection是有效的，如果对象有效，返回对象本身，否则返回NULL值 |

上面是MySQL-8.0.18所支持的所有空间函数的简单介绍，各个GIS函数的详细信息和用法可以通过[官方文档](https://dev.mysql.com/doc/refman/8.0/en/spatial-analysis-functions.html)了解。在MySQL-8.0.18中常用的空间操作函数都已经支持了，但是还有一些GIS函数并不支持，比如空间聚集函数ST_Collect。

## 总结

在MySQL8中对GIS功能的支持已经比较完善了，支持了主要的空间数据类型，还在InnoDB引擎上支持了GIS，使得GIS数据支持完整的MVCC和事务特性，并且支持InnoDB上的R树索引，此外还支持了空间投影，支持大量的空间坐标系，这大大扩展了MySQL中GIS功能的应用场景。 下面是MySQL和开源空间数据库插件PostGIS在部分功能上的对比。

| 功能           | MySQL                                | PostGIS                              |
| -------------- | ------------------------------------ | ------------------------------------ |
| 空间索引       | MyIASM以及InnoDB都支持R树索引        | GIST树索引                           |
| 支持的空间数据 | 二维数据                             | 三维数据                             |
| 空间操作函数   | 大部分OGC标准定义的空间操作函数      | 基本实现OGC标准定义的空间操作函数    |
| 空间投影       | 支持多种投影坐标系，支持空间坐标转换 | 支持多种投影坐标系，支持空间坐标转换 |
| 事务支持       | 在InnoDB引擎上支持完整的事务特性     | 提供了一系列的长事务支持             |

MySQL8对GIS的功能已经比较完善了，但是在某些方面还是有缺失，比如MySQL只能支持二维数据，并且支持的空间操作函数也不够完备。总体而言MySQL的GIS功能相比于PostGIS还有差距，但是MySQL的GIS功能是集成在MySQL服务中的，不需要加载插件，使用起来更方便，MySQL也一直在对GIS功能进行改进，已经能够满足基本的应用场景。

## 参考文档

[MySQL · 引擎特性 · 初识 MySQL GIS 及 InnoDB R-TREE](https://developer.aliyun.com/article/50625)

[Spatial Data Types](https://dev.mysql.com/doc/refman/8.0/en/spatial-types.html)

[Spatial Function Reference](https://dev.mysql.com/doc/refman/8.0/en/spatial-function-reference.html)

[Why Boost.Geometry in MySQL?](http://mysqlserverteam.com/why-boost-geometry-in-mysql/)

[PgSQL · 功能分析 · PostGIS 在 O2O应用中的优势](http://mysql.taobao.org/monthly/2015/07/04/)