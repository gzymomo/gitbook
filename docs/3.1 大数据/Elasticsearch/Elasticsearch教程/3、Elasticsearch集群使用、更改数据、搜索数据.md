# 一、使用集群

Elasticsearch提供了一套全面和强大的REST API，你可以通过它们与集群进行交互。使用该API可以但不限于:

- 检查集群、节点和索引的健康状况、状态和统计数据
- 管理集群、节点、索引数据和元数据
- 对索引执行CRUD(创建、读取、更新和删除)和搜索操作
- 执行高级搜索操作，如分页、排序、过滤、脚本、聚合等

## 1.1 Elasticsearch 健康检查

让我们从一个基本的集群健康检查开始。集群健康检查用于查看集群的运行情况。

我们将使用curl命令工具来访问Elasticsearch服务，你也可以使用任何其他HTTP/REST调试工具，例如[POSTMAN](https://www.getpostman.com/)。

可以使用`_cat`API进行集群健康检查：

API格式：

```shell
GET /_cat/health?v
```

curl访问API：

```shell
curl -X GET "localhost:9200/_cat/health?v"
```

响应如下：

```shell
epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1558609221 11:00:21  docker-cluster green           1         1      0   0    0    0        0             0                  -                100.0%
```

可以看到名为“docker-cluster”（使用docker安装）的集群处于绿色状态。

集群的健康状态有红、黄、绿三个状态：

- 绿 – 一切正常(集群功能齐全)
- 黄 – 所有数据可用，但有些副本尚未分配(集群功能完全)
- 红 – 有些数据不可用(集群部分功能)

​        **注意**: 当集群处于红色状态时，正常的分片将继续提供搜索服务，但你可能要尽快修复它。    

同样，从上面的响应中，可以看到有1个节点，0个分片，因为其中还没有数据。默认的Elasticsearch集群下，有时可能会有多个节点。

可以通过以下API，获取集群中的节点列表：

```shell
GET /_cat/nodes?v
```

curl访问API：

```shell
curl -X GET "localhost:9200/_cat/nodes?v"
```

响应如下：

```shell
[root@qikegu elasticsearch]# curl -X GET "localhost:9200/_cat/nodes?v"
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.18.0.2            8          94   2    0.04    0.03     0.10 mdi       *      86b8dd1cd964
```

这里，可以看到一个名为“86b8dd1cd964”的节点，是当前集群中的唯一一个节点。

## 1.2 列出索引

集群中的索引:

API格式：

```shell
GET /_cat/indices?v
```

curl访问API：

```shell
curl -X GET "localhost:9200/_cat/indices?v"
```

响应：

```shell
[root@qikegu elasticsearch]# curl -X GET "localhost:9200/_cat/indices?v"
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

可以看到，集群中还没有索引。

## 1.3 Elasticsearch 创建索引

创建一个名为“customer”的索引，然后再次列出所有索引:

API格式：

```shell
PUT /customer?pretty
GET /_cat/indices?v
```

第一个命令使用`PUT`创建名为“customer”的索引。末尾追加`pretty`，可以漂亮地打印JSON响应(如果有的话)。

curl命令访问API：

```shell
curl -X PUT "localhost:9200/customer?pretty"
curl -X GET "localhost:9200/_cat/indices?v"
```

响应：

```shell
[root@qikegu elasticsearch]# curl -X PUT "localhost:9200/customer?pretty"
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "customer"
}
[root@qikegu elasticsearch]# curl -X GET "localhost:9200/_cat/indices?v"
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer hGMH4nPSQjWhZ-uScEKk5w   1   1          0            0       230b           230b
```

第二个命令的结果告诉我们，现在有一个名为customer的索引，它有一个主分片和一个副本(默认值)，没有包含任何文档。

customer索引上标记了一个黄色健康状态，回想一下前面的讨论，黄色表示还没有分配一些副本。发生这种情况的原因是，默认情况下，Elasticsearch为该索引创建了一个副本，但是目前只有一个节点，要等另一个节点加入集群后，才可以分配该副本(用于高可用性)。一旦将该副本分配到第二个节点，该索引的健康状态将变为绿色。

## 1.4 Elasticsearch 创建和查询文档

现在把一些内容放入`customer`索引中。我们将一个简单的客户文档放到`customer`索引中，ID为1，如下所示:

API:

```shell
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
```

curl命令访问API：

```shell
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```

响应：

```shell
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

可以看到，`customer`索引中成功创建了一个新文档。文档的内部id也是1，在创建文档时指定。

注意，Elasticsearch并不要求，先要有索引，才能将文档编入索引。创建文档时，如果指定索引不存在，将自动创建。

现在检索刚才编入索引的文档:

```shell
GET /customer/_doc/1?pretty
```

curl命令访问API：

```shell
curl -X GET "localhost:9200/customer/_doc/1?pretty"
```

响应：

```shell
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "John Doe"
  }
}
```

这里有一个字段`found`为真，表示找到了一个ID为1的文档，另一个字段`_source`，该字段返回完整JSON文档。

## 1.5 Elasticsearch 删除索引

现在让我们删除刚才创建的索引，然后再列出所有索引:

API:

```shell
DELETE /customer?pretty
GET /_cat/indices?v
```

curl命令访问API：

```shell
curl -X DELETE "localhost:9200/customer?pretty"
curl -X GET "localhost:9200/_cat/indices?v"
```

响应：

```shell
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

可以看到，索引被成功删除，现在回到了集群中什么都没有的状态。

在继续之前，让我们再仔细看看，目前学过的一些API命令:

API:

```shell
PUT /customer
PUT /customer/_doc/1
{
  "name": "John Doe"
}
GET /customer/_doc/1
DELETE /customer
```

如果仔细研究上面的命令，可以看出Elasticsearch中访问数据的模式。这种模式可以概括如下:

```shell
<HTTP Verb> /<Index>/<Endpoint>/<ID>
```

这种REST访问模式在API命令中是很常见的。



# 二、修改数据

Elasticsearch提供近实时的数据操作和搜索功能。默认情况下，从索引中更新、删除数据，到这些操作在搜索结果中反映出来，会有一秒钟的延迟(刷新间隔)。这是与其他平台(如SQL)的一个重要区别，SQL中，事务完成后立即生效。

**替换文档**

前面已经介绍了如何把单个文档编入索引。让我们再回忆一下这个命令:

**API**

```shell
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
```

**CURL**

```shell
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```

上面命令将把指定的文档编入到`customer`索引中，ID为1。如果我们用一个不同的(或相同的)文档，再次执行上面的命令，Elasticsearch将替换现有文档:

**API**

```shell
PUT /customer/_doc/1?pretty
{
  "name": "Jane Doe"
}
```

**CURL**

```shell
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Jane Doe"
}
'
```

上面将ID为1的文档的客户名称，从“John Doe”更改为“Jane Doe”。另一方面，如果使用不同的ID，则将创建一个新文档，索引中已有的文档保持不变。

**API**

```shell
PUT /customer/_doc/2?pretty
{
  "name": "Jane Doe"
}
```

**CURL**

```shell
curl -X PUT "localhost:9200/customer/_doc/2?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Jane Doe"
}
'
```

上面的命令，在`customer`索引中，创建一个ID为2的新文档。

创建文档时，ID部分是可选的。如果没有指定，Elasticsearch将生成一个随机ID，然后使用它来引用文档。

这个例子展示了，如何创建一个没有显式ID的文档:

**API**

```shell
POST /customer/_doc?pretty
{
  "name": "Jane Doe"
}
```

**CURL**

```shell
curl -X POST "localhost:9200/customer/_doc?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Jane Doe"
}
'
```

## 2.1 更新文档

Elasticsearch实际上并没有在底层执行就地更新，而是先删除旧文档，再添加新文档。

这个例子展示了，把文档(ID为1)中的name字段更改为“Jane Doe”:

**API**

```shell
POST /customer/_update/1?pretty
{
  "doc": { "name": "Jane Doe" }
}
```

**CURL**

```shell
curl -X POST "localhost:9200/customer/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe" }
}
'
```

这个例子展示了，把文档(ID为1)中的name字段更改为“Jane Doe”，再添加一个年龄字段:

**API**

```shell
POST /customer/_update/1?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
```

**CURL**

```shell
curl -X POST "localhost:9200/customer/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
'
```

还可以使用简单的脚本执行更新。这个例子使用脚本将年龄增加5岁:

**API**

```shell
POST /customer/_update/1?pretty
{
  "script" : "ctx._source.age += 5"
}
```

**CURL**

```shell
curl -X POST "localhost:9200/customer/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "script" : "ctx._source.age += 5"
}
'
```

在上面的例子中，`ctx._source`引用源文档。

Elasticsearch提供了根据查询条件更新文档的能力(类似`SQL update - where`语句)。详情参考官网：[docs-update-by-query API](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/docs-update-by-query.html)

## 2.2 删除文档

删除文档相当简单。

这个例子展示了如何删除ID为2的客户:

**API**

```shell
DELETE /customer/_doc/2?pretty
```

**CURL**

```shell
curl -X DELETE "localhost:9200/customer/_doc/2?pretty"
```

请参阅[delete by query API](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/docs-delete-by-query.html)来删除与特定查询匹配的所有文档。注意，删除整个索引比使用delete By Query API删除所有文档要快得多。

## 2.3 批处理

除了对单个文档执行创建、更新和删除之外，Elasticsearch还提供了使用`_bulk API`批量执行上述操作的能力。

下面的调用，在一个批量操作中，创建两个文档(ID 1 – John Doe和ID 2 – Jane Doe):

**API**

```shell
POST /customer/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```

**CURL**

```shell
curl -X POST "localhost:9200/customer/_bulk?pretty" -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
'
```

下面的例子，在一个批量操作中，先更新第一个文档(ID为1)，再删除第二个文档(ID为2):

**API**

```shell
POST /customer/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```

**CURL**

```shell
curl -X POST "localhost:9200/customer/_bulk?pretty" -H 'Content-Type: application/json' -d'
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
'
```

注意，对于delete操作，只需提供被删除文档的ID即可。

某个操作失败不会导致批量API执行中断，剩下的操作将继续执行。当`_bulk API`返回时，它将为每个操作提供一个状态(与发送操作的顺序相同)，以便检查某个特定操作是否失败。

# 三、搜索数据

### 样本数据

现在我们已经了解了基本知识，让我们尝试使用更真实的数据。

我们提供了一些虚构的客户银行账户信息，格式如下所示：

```json
{
    "account_number": 0,
    "balance": 39988,
    "firstname": "Kevin",
    "lastname": "Wu",
    "age": 35,
    "gender": "F",
    "address": "525 Xixi Road",
    "employer": "Qikegu",
    "email": "kevinwu@qikegu.com",
    "city": "Hangzhou",
    "province": "Zhejiang"
}
```

### 加载样本数据

可以从这里下载样本数据([accounts.json](https://raw.githubusercontent.com/kevinhwu/qikegu-demo/master/elasticsearch/accounts.json))。将其放到当前目录，加载到集群中，如下所示:

```shell
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"
curl "localhost:9200/_cat/indices?v"
```

响应：

```shell
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank  YBca-2v7SbKGBrnv222USQ   1   1       1000            0      438kb          438kb
```

可以看到，1000个文档已经批量存储到`bank`索引中。

## 3.1 Elasticsearch 搜索API

现在让我们从一些简单的搜索开始。

搜索参数传递有2种方法:

- URI发送搜索参数
- 请求体(request body)发送搜索参数

搜索相关的REST API可以从`_search`端点访问。下面的例子返回`bank`索引中的所有文档：

**API**

```shell
GET /bank/_search?q=*&sort=account_number:asc&pretty
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty"
```

本例采用uri方式传递搜索参数：

- `q=*` 搜索索引中的所有文档
- `sort=account_number:asc` 搜索结果以字段`account_number`升序排列
- `pretty` 返回结果以漂亮的JSON格式打印

响应：

```json
{
  "took" : 55,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "0",
        "_score" : null,
        "_source" : {
          "account_number" : 0,
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
        "sort" : [
          0
        ]
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        },
        "sort" : [
          1
        ]
      },

      ...

    ]
  }
}
```

看一下响应中的重要字段:

- `took` – 搜索时间(毫秒)

- `timed_out` – 搜索是否超时

- `_shards` – 搜索了多少分片，搜索分片的成功/失败计数

- `hits` – 搜索结果

- ```
  hits.total
  ```

   – 搜索命中总数信息

  - `hits.total.value` – 命中总数
  - `hits.total.relation` – 取值eq(等于)/gte(大于等于)，表示`hits.total.value`与实际的搜索命中数量的关系。

- `hits.hits` – 实际的搜索结果数组(默认为前10个文档)

- `hits.sort` – 结果排序键(如果按分数排序，则忽略)

- `hits._score` 与 `max_score` – 分数是衡量文档与搜索条件匹配程度的一个指标。分数越高，文档越相关，分数越低，文档越不相关。并不总是需要生成分数，需不需要Elasticsearch会自动判断，以避免计算无用的分数。

如果搜索结果很多，超过一定数量后，通常就不再统计，只是笼统地表示为：搜索结果超过XXXX个。`hits.total`的准确性由请求参数`track_total_hits`控制，当`track_total_hits`为`true`时，搜索时将精确地跟踪总命中数(“relationship”:“eq”)。`track_total_hits`默认值为10,000，意味着总命中数可以精确地跟踪到10000个文档，如果超过10000，会表示为超过10000个结果，如下所示：

```json
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
```

通过将`track_total_hits`显式地设置为`true`，可以强制进行准确计数。详细信息，请参阅[request body](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-track-total-hits.html)文档。

同样的例子，使用请求体(request body)发送搜索参数：

**API**

```shell
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
```

可以看到，没有在URI中传递`q=*`，而是向`_search API`传递json风格的请求体，下一节中会详细讨论。

注意，一旦获得了搜索结果，Elasticsearch就会结束这次搜索，不会再维护任何服务端资源，也没有结果游标，这与其他很多平台，如SQL，不一样。

## 3.2 Elasticsearch Query DSL(查询语言)

Elasticsearch提供了一种json风格的查询语言，称为Query DSL（Query domain-specific language）。查询语言功能很全面，让我们从几个基本示例开始。

回到上一章例子，我们执行了这个查询:

**API**

```shell
GET /bank/_search
{
  "query": { "match_all": {} }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} }
}
'
```

`query`字段表示这次查询的定义，其中`match_all`字段表示查询类型 – 匹配所有文档。

除了`query`参数，还可以传递其他参数。下面例子中，我们传入一个`size`参数，设置返回条目数量:

**API**

```shell
GET /bank/_search
{
  "query": { "match_all": {} },
  "size": 1
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "size": 1
}
'
```

注意，如果没有指定`size`，默认为10。

下面例子执行`match_all`，返回文档10到19:

**API**

```shell
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
'
```

`from`参数(基于0)指定从哪个文档序号开始，`size`参数指定返回多少个文档，这两个参数对于搜索结果分页非常有用。注意，如果没有指定`from`，则默认值为0。

下面例子执行`match_all`操作，对结果按帐户余额降序排序，返回前10个(默认)文档。

**API**

```shell
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
'
```

## 3.3 Elasticsearch 搜索

让我们深入研究Query DSL。

### 文档字段

首先看看返回的文档字段。默认情况下，搜索结果中包含了完整的JSON文档(`_source`字段)，如果不希望返回源文档全部内容，可以设置要返回的字段。

下面的例子，返回`_source`中的两个字段`account_number`和`balance`:

**API**

```shell
GET /bank/_search
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
'
```

### 匹配查询

前面已经介绍过，使用`match_all`查询匹配所有文档。下面介绍一个新的查询类型：`match`查询，可以对某个字段进行搜索。

下面的例子，返回编号为20的帐户:

**API**

```shell
GET /bank/_search
{
  "query": { "match": { "account_number": 20 } }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "account_number": 20 } }
}
'
```

下面的例子，返回地址中包含“mill”的所有帐户:

**API**

```shell
GET /bank/_search
{
  "query": { "match": { "address": "mill" } }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill" } }
}
'
```

下面的例子，返回地址中包含“mill”或“lane”的所有帐户:

**API**

```shell
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'
```

下面的例子，是`match`的一个变体`match_phrase`，`match_phrase`匹配整个短语，它返回地址中包含短语“mill lane”的所有帐户:

**API**

```shell
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'
```

### 布尔查询

布尔查询使用布尔逻辑，将小查询组合成大查询。

下面的例子，`bool must`子句下包含两个匹配查询，返回地址中包含“mill”且也包含“lane”的帐户:

**API**

```shell
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

在上面的示例中，`bool must`子句包含的所有匹配条件为真，文档才能被视为匹配，类似逻辑与。

下面例子中，`bool should`子句下包含两个匹配查询，返回地址中包含“mill”或“lane”的帐户:

**API**

```shell
GET /bank/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

在上面的示例中，`bool should`子句包含的匹配条件有一个为真，文档将被视为匹配，类似逻辑或。

下面例子中，`bool must_not`子句包含两个匹配查询，返回地址中既不包含“mill”也不包含“lane”的帐户:

**API**

```shell
GET /bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

在上面的示例中，`bool must_not`子句包含的匹配条件全部为假，文档将被视为匹配，类似逻辑与非。

可以在布尔查询中同时组合`must`、`should`和`must_not`子句，可以在这些布尔子句中组合布尔查询，以模拟任何复杂的多级布尔逻辑。

下面例子中，返回所有40岁，但不居住在ID(aho)的人的账户:

**API**

```shell
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'
```

## 3.4 Elasticsearch 过滤

`_score`(分数)字段是衡量文档与搜索条件匹配程度的一个指标。分数越高，文档越相关，分数越低，文档越不相关。并不总是需要生成分数，需不需要Elasticsearch会自动判断，以避免计算无用的分数。

布尔查询还支持`filter`子句，用于设置过滤条件。过滤条件不影响文档的相关性分数。

下面的例子，使用布尔查询，返回余额在20000到30000之间的所有帐户。

**API**

```shell
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'
```

上面的布尔查询中，包含一个`match_all`查询(查询部分)和一个`range`查询(筛选部分)。过滤条件中的`range`查询不影响文档的相关性分数计算。

除了`match_all`、`match`、`bool`和`range`查询，还有其他许多查询类型，工作原理大同小异，可参考相关资料。

## 3.5 Elasticsearch 聚合

聚合提供了对数据进行分组、统计的能力，类似于SQL中`GROUP by`和SQL聚合函数。在Elasticsearch中，可以同时返回搜索结果及其聚合计算结果，这是非常强大和高效的。

下面的例子，对所有帐户按所在州分组，统计每组账户数量，然后返回前10个条目，按账户数量降序排列:

**API**

```shell
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'
```

上述命令与以下SQL语句意义相同：

```sql
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC LIMIT 10;
```

响应：

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30
        },
        {
          "key" : "MD",
          "doc_count" : 28
        },
        {
          "key" : "ID",
          "doc_count" : 27
        },
        {
          "key" : "AL",
          "doc_count" : 25
        },
        {
          "key" : "ME",
          "doc_count" : 25
        },
        {
          "key" : "TN",
          "doc_count" : 25
        },
        {
          "key" : "WY",
          "doc_count" : 25
        },
        {
          "key" : "DC",
          "doc_count" : 24
        },
        {
          "key" : "MA",
          "doc_count" : 24
        },
        {
          "key" : "ND",
          "doc_count" : 24
        }
      ]
    }
  }
}
```

可以看到ID(爱达荷州)有27个帐户，其次是TX(德克萨斯州)的27个帐户，然后是AL(阿拉巴马州)的25个帐户，等等。

注意，`size=0`表示不显示搜索结果，我们只想看到聚合结果。

基于前面例子的结果，下面例子同时按州计算平均帐户余额，然后返回前10个条目，按账户数量降序排列:

**API**

```shell
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
```

注意，`average_balance`聚合嵌入在`group_by_state`聚合中，这是聚合的常见模式，可以在聚合中任意嵌套聚合。

基于前面例子的结果，对结果按平均账户余额降序排序:

**API**

```shell
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
```

下面例子按照年龄段(20-29岁，30-39岁，40-49岁)分组，然后按性别分组，统计每个年龄等级，每种性别的平均账户余额:

**API**

```shell
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
```

**CURL**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
'
```



整理自：

[奇客谷：Elasticsearch教程](https://www.qikegu.com/docs/3044)