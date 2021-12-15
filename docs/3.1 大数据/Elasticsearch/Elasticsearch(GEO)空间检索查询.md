[TOC]

腾讯云：[Elasticsearch(GEO)空间检索查询](https://cloud.tencent.com/developer/article/1050321)

# 1、ES GEO空间检索
空间检索顾名思义提供了通过空间距离和位置关系进行检索的能力。有很多空间索引算法和类库可供选择。
ES内置了这种索引方式。下面详细介绍。
## 1.1 创建索引
```python
def create_index():
    mapping = {
        "mappings": {
            "poi": {
                "_routing": {
                    "required": "true",
                    "path": "city_id"
                },
                "properties": {
                    "id": {
                        "type": "integer"
                    },
                    "geofence_type": {
                        "type": "integer"
                    },
                    "city_id": {
                        "type": "integer"
                    },
                    "city_name": {
                        "type": "string",
                        "index": "not_analyzed"
                    },
                    "activity_id": {
                        "type": "integer"
                    },
                    "post_date": {
                        "type": "date"
                    },
                    "rank": {
                        "type": "float"
                    },
                    # 不管是point还是任意shape, 都用geo_shape,通过type来设置
                    # type在数据里
                    "location_point": {
                        "type": "geo_shape"
                    },
                    "location_shape": {
                        "type": "geo_shape"
                    },
                    # 在计算点间距离的时候, 需要geo_point类型变量
                    "point": {
                        "type": "geo_point"
                    }
                }
            }
        }
    }
    # 创建索引的时候可以不 mapping
    es.create_index(index='mapapp', body=mapping)
    # set_mapping = es_dsl.set_mapping('mapapp', 'poi', body=mapping)
```
创建了一个名叫mapapp的索引，映射的设置如mapping所示。

## 1.2 批量插入数据bulk
```python
def bulk():
    # actions 是一个可迭代对象就行, 不一定是list
    workbooks = xlrd.open_workbook('./geo_data.xlsx')
    table = workbooks.sheets()[1]
    colname = list()
    actions = list()
    for i in range(table.nrows):
        if i == 0:
            colname = table.row_values(i)
            continue
        geo_shape_point = json.loads(table.row_values(i)[7])
        geo_shape_shape = json.loads(table.row_values(i)[8])
        geo_point = json.loads(table.row_values(i)[9])
        raw_data = table.row_values(i)[:7]
        raw_data.extend([geo_shape_point, geo_shape_shape, geo_point])
        source = dict(zip(colname, raw_data))
        geo = GEODocument(**source)
        action = {
                "_index": "mapapp",
                "_type": "poi",
                "_id": table.row_values(i)[0],
                "_routing": geo.city_id,
                #"_source": source,
                "_source": geo.to_json(),
            }
        actions.append(action)
    es.bulk(index='mapapp', actions=actions, es=es_handler, max=25)
```
刷入测试数据，geo_data数据形如：
```sql
id    geofence_type    city_id    city_name    activity_id    post_date    rank    location_point    location_shape    point
1    1    1    北京    100301    2016/10/20    100.30     {"type":"point","coordinates":[55.75,37.616667]}    {"type":"polygon","coordinates":[[[22,22],[4.87463,52.37254],[4.87875,52.36369],[22,22]]]}    {"lat":55.75,"lon":37.616667}
2    1    1    北京    100302    2016/10/21    12.00     {"type":"point","coordinates":[55.75,37.616668]}    {"type":"polygon","coordinates":[[[0,0],[4.87463,52.37254],[4.87875,52.36369],[0,0]]]}    {"lat":48.8567,"lon":2.3508}
3    1    1    北京    100303    2016/10/22    3432.23     {"type":"point","coordinates":[55.75,37.616669]}    {"type":"polygon","coordinates":[[[4.8833,52.38617],[4.87463,52.37254],[4.87875,52.36369],[4.8833,52.38617]]]}    {"lat":32.75,"lon":37.616668}
4    1    1    北京    100304    2016/10/23    246.80     {"type":"point","coordinates":[52.4796, 2.3508]}    {"type":"polygon","coordinates":[[[4.8833,52.38617],[4.87463,52.37254],[4.87875,52.36369],[4.8833,52.38617]]]}    {"lat":11.56,"lon":37.616669}
```
## 1.3 GEO查询：两点间距离
```python
# 点与点之间的距离
# 按照距离升序排列,如果size取1个,就是最近的
def sort_by_distance():
    body = {
        "from": 0,
        "size": 1,
        "query": {
            "bool": {
                "must": [{
                    "term": {
                        "geofence_type": 1
                    }
                }, {
                    "term": {
                        "city_id": 1
                    }
                }]
            }
        },
        "sort": [{
            "_geo_distance": {
                "point": {
                    "lat": 8.75,
                    "lon": 37.616
                },
                "unit": "km",
                "order": "asc"
            }
        }]
    }
    for i in es.search(index='mapapp', doc_type='poi', body=body)['hits']['hits']:
        print type(i), i
```
## 1.4 GEO查询：边界框过滤
ES的过滤是会生成缓存的，所以在优化查询的时候，常常需要将频繁用到的查询提取出来作为过滤呈现，但不幸的是，对于GEO过滤不会生成缓存，所以没有必要考虑，这里为了做出区分，使用post_filter，查询后再过滤，下面的都类似。
```python
# 边界框过滤:用框去圈选点和形状
# 这里实现了矩形框选中
# post_filter后置filter, 对查询结果再过滤; aggs常用后置filter
def bounding_filter():
    body = {
        "from": 0,
        "size": 1,
        "query": {
            "bool": {
                "must": [{
                    "term": {
                        "geofence_type": 1
                    }
                }, {
                    "term": {
                        "city_id": 1
                    }
                }]
            }
        },
        "post_filter": {
            "geo_shape": {
                "location_point": {
                    "shape": {
                        "type": "envelope",
                        "coordinates": [[52.4796, 2.3508], [48.8567, -1.903]]
                    },
                    "relation": "within"
                }
            }
        }
    }
    for i in es.search(index='mapapp', doc_type='poi', body=body)['hits']['hits']:
        print type(i), i
```
## 1.5 GEO查询：圆形圈选
```python
# 边界框过滤: 圆形圈选
# post_filter后置filter, 对查询结果再过滤; aggs常用后置filter
def circle_filter():
    body = {
        "from": 0,
        "size": 1,
        "query": {
            "bool": {
                "must": [{
                    "term": {
                        "geofence_type": 1
                    }
                }, {
                    "term": {
                        "city_id": 1
                    }
                }]
            }
        },
        "post_filter": {
            "geo_shape": {
                "location_point": {
                    "shape": {
                        "type": "circle",
                        "radius": "10000km",
                        "coordinates": [22, 45]
                    },
                    "relation": "within"
                }
            }
        }
    }
    for i in es.search(index='mapapp', doc_type='poi', body=body)['hits']['hits']:
        print type(i), i
```
## 1.6 GEO查询：反选
```python
# 边界框反选:点落在框中,框被查询出来
# post_filter后置filter, 对查询结果再过滤; aggs常用后置filter
# 包含正则匹配regexp
def intersects():
    body = {
       "from": 0,
       "size": 1,
       "query": {
            "bool": {
                "must": [{
                    "term": {
                        "geofence_type": 1
                    }
                }, {
                    "regexp": {
                        "city_name": u".*北京.*"
                    }
                }, {
                    "term": {
                        "city_id": 1
                    }
                }]
            }
       },
       "post_filter": {
            "geo_shape": {
                "location_shape": {
                    "shape": {
                        "type": "point",
                        "coordinates": [22,22]
                    },
                    "relation": "intersects"
                }
            }
       }
    }
    for i in es.search(index='mapapp', doc_type='poi', body=body)['hits']['hits']:
        print type(i), i
```
## 1.7 两个空间聚合的例子
```python
# 空间聚合
# 按照与中心点距离聚合
def aggs_geo_distance():
    body = {
        "aggs": {
            "aggs_geopoint": {
                "geo_distance": {
                    "field": "point",
                    "origin": {
                        "lat": 51.5072222,
                        "lon": -0.1275
                    },
                    "unit": "km",
                    "ranges": [
                        {
                            "to": 1000
                        },
                        {
                            "from": 1000,
                            "to": 3000
                        },
                        {
                            "from": 3000
                        }
                    ]
                }
            }
        }
    }
    for i in es.search(index='mapapp', doc_type='poi', body=body)['aggregations']['aggs_geopoint']['buckets']:
        print type(i), i


# 空间聚合
# geo_hash算法, 网格聚合grid
# 两次聚合
def aggs_geohash_grid():
    body = {
        "aggs": {
            "new_york": {
                "geohash_grid": {
                    "field":     "point",
                    "precision": 5
                }
            },
            "map_zoom": {
                "geo_bounds": {
                    "field": "point"
              }
            }
          }
    }
    for i in es.search(index='mapapp', doc_type='poi', body=body)['aggregations']['new_york']['buckets']:
        print type(i), i
```