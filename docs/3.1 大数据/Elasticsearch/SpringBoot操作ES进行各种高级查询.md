创建SpringBoot项目，导入 ES 6.2.1 的 RestClient 依赖和 ES 依赖。在项目中直接引用 es-starter 的话会报容器初始化异常错误，导致项目无法启动。如果有读者解决了这个问题，欢迎留言交流

```
<!-- ES 客户端 -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>${elasticsearch.version}</version>
</dependency>
        <!-- ES 版本 -->
<dependency>
<groupId>org.elasticsearch</groupId>
<artifactId>elasticsearch</artifactId>
<version>${elasticsearch.version}</version>
</dependency>
```

为容器定义 RestClient 对象

```
/**
 * 在Spring容器中定义 RestClient 对象
 * @Author: keats_coder
 * @Date: 2019/8/9
 * @Version 1.0
 * */
@Configuration
public class ESConfig {
    @Value("${yunshangxue.elasticsearch.hostlist}")
    private String hostlist; // 127.0.0.1:9200

    @Bean // 高版本客户端
    public RestHighLevelClient restHighLevelClient() {
        // 解析 hostlist 配置信息。假如以后有多个，则需要用 ， 分开
        String[] split = hostlist.split(",");
        // 创建 HttpHost 数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for (int i = 0; i < split.length; i++) {
            String item = split[i];
            httpHostArray[i] = new HttpHost(item.split(":")[0], Integer.parseInt(item.split(":")[1]), "http");
        }
        // 创建RestHighLevelClient客户端
        return new RestHighLevelClient(RestClient.builder(httpHostArray));
    }

    // 项目主要使用 RestHighLevelClient，对于低级的客户端暂时不用
    @Bean
    public RestClient restClient() {
        // 解析hostlist配置信息
        String[] split = hostlist.split(",");
        // 创建HttpHost数组，其中存放es主机和端口的配置信息
        HttpHost[] httpHostArray = new HttpHost[split.length];
        for (int i = 0; i < split.length; i++) {
            String item = split[i];
            httpHostArray[i] = new HttpHost(item.split(":")[0], Integer.parseInt(item.split(":")[1]), "http");
        }
        return RestClient.builder(httpHostArray).build();
    }
}
```

在 yml 文件中配置 eshost

```
yunshangxue:
  elasticsearch:
    hostlist: ${eshostlist:127.0.0.1:9200}
```

调用相关 API 执行操作

1. 创建操作索引的对象
2. 构建操作索引的请求
3. 调用对象的相关API发送请求
4. 获取响应消息

```
/**
 * 删除索引库
 */
@Test
public void testDelIndex() throws IOException {
        // 操作索引的对象
        IndicesClient indices = client.indices();
        // 删除索引的请求
        DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("ysx_course");
        // 删除索引
        DeleteIndexResponse response = indices.delete(deleteIndexRequest);
        // 得到响应
        boolean b = response.isAcknowledged();
        System.out.println(b);
        }
```

创建索引, 步骤和删除类似，需要注意的是删除的时候需要指定 ES 库分片的数量和副本的数量，并且在创建索引的时候可以将映射一起指定了。代码如下

```
    public void testAddIndex() throws IOException {
        // 操作索引的对象
        IndicesClient indices = client.indices();
        // 创建索引的请求
        CreateIndexRequest request = new CreateIndexRequest("ysx_course");
        request.settings(Settings.builder().put("number_of_shards", "1").put("number_of_replicas", "0"));
        // 创建映射
        request.mapping("doc", "{\n" +
        "                \"properties\": {\n" +
        "                    \"description\": {\n" +
        "                        \"type\": \"text\",\n" +
        "                        \"analyzer\": \"ik_max_word\",\n" +
        "                        \"search_analyzer\": \"ik_smart\"\n" +
        "                    },\n" +
        "                    \"name\": {\n" +
        "                        \"type\": \"text\",\n" +
        "                        \"analyzer\": \"ik_max_word\",\n" +
        "                        \"search_analyzer\": \"ik_smart\"\n" +
        "                    },\n" +
        "\"pic\":{                    \n" +
        "\"type\":\"text\",                        \n" +
        "\"index\":false                        \n" +
        "},                    \n" +
        "                    \"price\": {\n" +
        "                        \"type\": \"float\"\n" +
        "                    },\n" +
        "                    \"studymodel\": {\n" +
        "                        \"type\": \"keyword\"\n" +
        "                    },\n" +
        "                    \"timestamp\": {\n" +
        "                        \"type\": \"date\",\n" +
        "                        \"format\": \"yyyy-MM‐dd HH:mm:ss||yyyy‐MM‐dd||epoch_millis\"\n" +
        "                    }\n" +
        "                }\n" +
        "            }", XContentType.JSON);


        // 执行创建操作
        CreateIndexResponse response = indices.create(request);
        // 得到响应
        boolean b = response.isAcknowledged();
        System.out.println(b);
        }
```

# [Java API操作ES](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

## [准备数据环境](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

创建索引：ysx_course

创建映射：

```
PUT http://localhost:9200/ysx_course/doc/_mapping
{
    "properties": {
        "description": { // 课程描述
            "type": "text", // String text 类型
            "analyzer": "ik_max_word", // 存入的分词模式：细粒度
            "search_analyzer": "ik_smart" // 查询的分词模式：粗粒度
        },
        "name": { // 课程名称
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
        },
        "pic":{ // 图片地址
            "type":"text",
            "index":false // 地址不用来搜索，因此不为它构建索引
        },
        "price": { // 价格
        	"type": "scaled_float", // 有比例浮点
        	"scaling_factor": 100 // 比例因子 100
        },
        "studymodel": {
            "type": "keyword" // 不分词，全关键字匹配
        },
        "timestamp": {
            "type": "date",
            "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
    }
}
```

加入原始数据:

```
POST http://localhost:9200/ysx_course/doc/1
{
	"name": "Bootstrap开发",
	"description": "Bootstrap是由Twitter推出的一个前台页面开发框架，是一个非常流行的开发框架，此框架集成了多种页面效果。此开发框架包含了大量的CSS、JS程序代码，可以帮助开发者（尤其是不擅长页面开发的程序人员）轻松的实现一个不受浏览器限制的精美界面效果。",
	"studymodel": "201002",
	"price":38.6,
	"timestamp":"2018-04-25 19:11:35",
	"pic":"group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
}
```

## [DSL搜索](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

DSL(Domain Specific Language)是ES提出的基于json的搜索方式，在搜索时传入特定的json格式的数据来完成不 同的搜索需求。DSL比URI搜索方式功能强大，在项目中建议使用DSL方式来完成搜索。

### [查询全部](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

原本我们想要查询全部的话，需要使用 GET 请求发送 _search 命令，如今使用 DSL 方式搜索，可以使用 POST 请求，并在请求体中设置 JSON 字符串来构建查询条件

```
POST http://localhost:9200/ysx_course/doc/_search
```

请求体 JSON

```
{
    "query": {
        "match_all": {} // 查询全部
    },
    "_source" : ["name","studymodel"] // 查询结果包括 课程名 + 学习模式两个映射
}
```

具体的测试方法如下：过程比较繁琐，好在条理还比较清晰

```
// 搜索全部记录
@Test
public void testSearchAll() throws IOException, ParseException {
    // 搜索请求对象
    SearchRequest searchRequest = new SearchRequest("ysx_course");
    // 指定类型
    searchRequest.types("doc");
    // 搜索源构建对象
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    // 搜索方式
    // matchAllQuery搜索全部
    searchSourceBuilder.query(QueryBuilders.matchAllQuery());
    // 设置源字段过虑,第一个参数结果集包括哪些字段，第二个参数表示结果集不包括哪些字段
    searchSourceBuilder.fetchSource(new String[]{"name","studymodel","price","timestamp"},new String[]{});
    // 向搜索请求对象中设置搜索源
    searchRequest.source(searchSourceBuilder);
    // 执行搜索,向ES发起http请求
    SearchResponse searchResponse = client.search(searchRequest);
    // 搜索结果
    SearchHits hits = searchResponse.getHits();
    // 匹配到的总记录数
    long totalHits = hits.getTotalHits();
    // 得到匹配度高的文档
    SearchHit[] searchHits = hits.getHits();
    // 日期格式化对象
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    for(SearchHit hit:searchHits){
        // 文档的主键
        String id = hit.getId();
        // 源文档内容
        Map<String, Object> sourceAsMap = hit.getSourceAsMap();
        String name = (String) sourceAsMap.get("name");
        // 由于前边设置了源文档字段过虑，这时description是取不到的
        String description = (String) sourceAsMap.get("description");
        // 学习模式
        String studymodel = (String) sourceAsMap.get("studymodel");
        // 价格
        Double price = (Double) sourceAsMap.get("price");
        // 日期
        Date timestamp = dateFormat.parse((String) sourceAsMap.get("timestamp"));
        System.out.println(name);
        System.out.println(studymodel);
        System.out.println("你看不见我，看不见我~" + description);
        System.out.println(price);
    }

}
```

#### [坑：red>](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

执行过程中遇到的问题：不能对这个值进行初始化，导致 Spring 容器无法初始化

```
Caused by: java.lang.IllegalArgumentException: Could not resolve placeholder 'yunshangxue.elasticsearch.hostlist' in value "${yunshangxue.elasticsearch.hostlist}"
```

通过检查 target 目录发现，生成的 target 文件包中没有将 yml 配置文件带过来... 仔细对比发现，我的项目竟然变成了一个不是 Maven 的项目。重新使用 IDEA 导入 Mavaen 工程之后便能正常运行了

### [分页查询](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

我们来 look 一下 ES 的分页查询参数：

```
{
    // from 起始索引
    // size 每页显示的条数
    "from" : 0, "size" : 1,
    "query": {
       "match_all": {}
     },
    "_source" : ["name","studymodel"]
}
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/6mychickmupUicRleSua8yBiaQMibMM7aDKaPU0iahPXxaXD6BiaeNMoM03gaEzSjT9dHKWnjRT2KtLVPq0IB1FeYD6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)1565524349684

通过查询结果可以发现，我们设置了分页参数之后， hits.total 仍然是 3，表示它找到了 3 条数据，而按照分页规则，它只会返回一条数据，因此 hits.hits 里面只有一条数据。这也符合我们的业务规则，在查询前端页面显示总共的条数和当前的数据。

由此，我们就可以通过 Java API 来构建查询条件了：对上面查询全部的代码进行如下改造：

```
// 搜索源构建对象
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
int page = 2; // 页码
int size = 1; // 每页显示的条数
int index = (page - 1) * size;
searchSourceBuilder.from(index);
searchSourceBuilder.size(1);
// 搜索方式
// matchAllQuery搜索全部
searchSourceBuilder.query(QueryBuilders.matchAllQuery());
```

### [精确查询 TermQuery](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

Term Query为精确查询，在搜索时会整体匹配关键字，不再将关键字分词

例如：

```
{
    "query": {
     "term": { // 查询的方式为 term 精确查询
      "name": "spring" // 查询的字段为 name 关键字是 spring
     }
    },
    "_source": [
        "name",
        "studymodel"
    ]
}
```

此时查询的结果是：

```
 "hits": [
     {
         "_index": "ysx_course",
         "_type": "doc",
         "_id": "3",
         "_score": 0.9331132,
         "_source": {
             "studymodel": "201001",
             "name": "spring开发基础"
         }
     }
 ]
```

查询到了上面这条数据，因为 spring开发基础 分完词后是 spring 开发 基础 ，而查询关键字是 spring  不分词，这样当然可以匹配到这条记录，但是当我们修改关键字为 spring开发，按照往常的查询方法，也是可以查询到的。但是 term  不一样，它不会对关键字分词。结果可想而知是查询不到的

JavaAPI如下：

```
// 搜索方式
// termQuery 精确查询
searchSourceBuilder.query(QueryBuilders.termQuery("studymodel", "201002"));
```

#### [根据 ID 查询：](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

根据 ID 精确查询和根据其他条件精确查询是一样的，不同的是 id 字段前面有一个下划线注意写上

```
searchSourceBuilder.query(QueryBuilders.termQuery("_id", "1"));
```

但是，当**一次查询多个 ID 时** ，相应的 API 也应该改变，使用 termsQuery 而不是 termQuery。多了一个 s

### [全文检索 MatchQuery](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

MatchQuery 即全文检索，会对关键字进行分词后匹配词条。

query：搜索的关键字，对于英文关键字如果有多个单词则中间要用半角逗号分隔，而对于中文关键字中间可以用 逗号分隔也可以不用

operator：设置查询的结果取交集还是并集，并集用 or， 交集用 and

```
{
    "query": {
        "match": {
            "description": {
                "query": "spring开发",
                "operator": "or"
            }
        }
    }
}
```

有时，我们需要设定一个量化的表达方式，例如查询 spring开发基础，这三个词条。我们需求是至少匹配两个词条，这时 operator 属性就不能满足要求了，ES  还提供了另外一个属性：minimum_should_match 用一个百分数来设定应该有多少个词条满足要求。例如查询：

“spring开发框架”会被分为三个词：spring、开发、框架 设置"minimum_should_match": "80%"表示，三个词在文档的匹配占比为80%，即3*0.8=2.4，**向下取整** 得2，表 示至少有两个词在文档中要匹配成功。

#### [JavaAPI](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

通过 matchQuery.minimumShouldMatch 的方式来设置条件

```
// matchQuery全文检索
        searchSourceBuilder.query(QueryBuilders.matchQuery("description", "Spring开发框架").minimumShouldMatch("70%"));
```

### [多字段联合搜索 MultiQuery](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

上面的 MatchQuery 有一个短板，假如用户输入了某关键字，我们在查找的时候并不知道他输入的是 name 还是  description，这时我们用什么都不合适，而 MultiQuery 的出现解决了这个问题，他可以通过 fields  属性来设置多个域联合查找：具体用法如下

```
{
    "query": {
        "multi_match": {
            "query": "Spring开发",
            "minimum_should_match": "70%",
            "fields": ["name", "description"]
        }
    }
}
```

JavaAPI

```
searchSourceBuilder.query(QueryBuilders.multiMatchQuery("Spring开发框架", "name", "description").minimumShouldMatch("70%"));
```

#### [提升 boost](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

在多域联合查询的时候，可以通过 boost 来设置某个域在计算得分时候的比重，比重越高的域当他符合条件时计算的得分越高，相应的该记录也更靠前。通过在 fields 中给相应的字段用 ^权重倍数来实现

```
"fields": ["name^10", "description"]
```

上面的代码表示给 name 字段提升十倍权重，查询到的结果：

```
{
    "_index": "ysx_course",
    "_type": "doc",
    "_id": "3",
    "_score": 13.802518, // 可以清楚的发现，得分竟然是 13 了
    "_source": {
        "name": "spring开发基础",
        "description": "spring 在java领域非常流行，java程序员都在用。",
        "studymodel": "201001",
        "price": 88.6,
        "timestamp": "2018-02-24 19:11:35",
        "pic": "group1/M00/00/00/wKhlQFs6RCeAY0pHAAJx5ZjNDEM428.jpg"
    }
},
```

而在 Java 中，仍然可以通过链式编程来实现

```
searchSourceBuilder.query(QueryBuilders.multiMatchQuery("Spring开发框架", "name", "description").field("name", 10)); // 设置 name 10倍权重
```

### [布尔查询 BoolQuery](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

如果我们既要对一些字段进行分词查询，同时要对另一些字段进行精确查询，就需要使用布尔查询来实现了。布尔查询对应于Lucene的BooleanQuery查询，实现将多个查询组合起来，有三个可选的参数：

must：文档必须匹配must所包括的查询条件，相当于 “AND”

should：文档应该匹配should所包括的查询条件其中的一个或多个，相当于 "OR"

must_not：文档不能匹配must_not所包括的该查询条件，相当于“NOT”

```
{
    "query": {
        "bool": { // 布尔查询
            "must": [ // 查询条件 must 表示数组中的查询方式所规定的条件都必须满足
                {
                    "multi_match": {
                        "query": "spring框架",
                        "minimum_should_match": "50%",
                        "fields": [
                            "name^10",
                            "description"
                        ]
                    }
                },
                {
                    "term": {
                        "studymodel": "201001"
                    }
                }
            ]
        }
    }
}
```

JavaAPI

```
// 搜索方式
// 首先构造多关键字查询条件
MultiMatchQueryBuilder matchQueryBuilder = QueryBuilders.multiMatchQuery("Spring开发框架", "name", "description").field("name", 10);
// 然后构造精确匹配查询条件
TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("studymodel", "201002");
// 组合两个条件，组合方式为 must 全满足
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
boolQueryBuilder.must(matchQueryBuilder);
boolQueryBuilder.must(termQueryBuilder);
// 将查询条件封装给查询对象
searchSourceBuilder.query(boolQueryBuilder);
```

### [过滤器](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

定义过滤器查询，是在原本查询结果的基础上对数据进行筛选，因此省略了重新计算的分的步骤，效率更高。并且方便缓存。推荐尽量使用过虑器去实现查询或者过虑器和查询共同使用，过滤器在布尔查询中使用，下边是在搜索结果的基础上进行过滤：

```
{
    "query": {
        "bool": {
            "must": [
                {
                    "multi_match": {
                        "query": "spring框架",
                        "minimum_should_match": "50%",
                        "fields": [
                            "name^10",
                            "description"
                        ]
                    }
                }
            ],
            "filter": [
                {
                    // 过滤条件：studymodel 必须是 201001
                    "term": {"studymodel": "201001"}
                },
                {
                    // 过滤条件：价格 >=60 <=100
                    "range": {"price": {"gte": 60,"lte": 100}}
                }
            ]
        }
    }
}
```

**注意** ：range和term一次只能对一个Field设置范围过虑。

JavaAPI

```
// 首先构造多关键字查询条件
MultiMatchQueryBuilder matchQueryBuilder = QueryBuilders.multiMatchQuery("Spring框架", "name", "description").field("name", 10);
// 添加条件到布尔查询
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
boolQueryBuilder.must(matchQueryBuilder);
// 通过布尔查询来构造过滤查询
boolQueryBuilder.filter(QueryBuilders.termQuery("studymodel", "201001"));
boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gte(60).lte(100));
// 将查询条件封装给查询对象
searchSourceBuilder.query(boolQueryBuilder);
```

### [排序](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

我们可以在查询的结果上进行二次排序，支持对 keyword、date、float 等类型添加排序，text类型的字段不允许排序。排序使用的 JSON 格式如下：

```json
{
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "price": {
                            "gte": 0,
                            "lte": 100
                        }
                    }
                }
            ]
        }
    },
    "sort": [ // 注意这里排序是写在 query key 的外面的。这就表示它的API也不是布尔查询提供
        {
            "studymodel": "desc" // 对 studymodel(keyword)降序
        },
        {
            "price": "asc" // 对 price(double)升序
        }
    ]
}
```

由上面的 JSON 数据可以发现，排序所属的 API 是和 query 评级的，因此在调用 API 时也应该选择对应的 SearchSourceBuilder 对象

```java
// 排序查询
@Test
public void testSort() throws IOException, ParseException {
    // 搜索请求对象
    SearchRequest searchRequest = new SearchRequest("ysx_course");
    // 指定类型
    searchRequest.types("doc");
    // 搜索源构建对象
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    // 搜索方式
    // 添加条件到布尔查询
    BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
    // 通过布尔查询来构造过滤查询
    boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gte(0).lte(100));
    // 将查询条件封装给查询对象
    searchSourceBuilder.query(boolQueryBuilder);
    // 向搜索请求对象中设置搜索源
    searchRequest.source(searchSourceBuilder);

    // 设置排序规则
    searchSourceBuilder.sort("studymodel", SortOrder.DESC); // 第一排序规则
    searchSourceBuilder.sort("price", SortOrder.ASC); // 第二排序规则

    // 执行搜索,向ES发起http请求
    SearchResponse searchResponse = client.search(searchRequest);
    // 搜索结果
    SearchHits hits = searchResponse.getHits();
    // 匹配到的总记录数
    long totalHits = hits.getTotalHits();
    // 得到匹配度高的文档
    SearchHit[] searchHits = hits.getHits();
    // 日期格式化对象
    soutData(searchHits);
}
```

### [高亮显示](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

高亮显示可以将搜索结果一个或多个字突出显示，以便向用户展示匹配关键字的位置。

高亮三要素：高亮关键字、高亮前缀、高亮后缀

```json
{
    "query": {
        "bool": {
            "must": [
                {
                    "multi_match": {
                        "query": "开发框架",
                        "minimum_should_match": "50%",
                        "fields": [
                            "name^10",
                            "description"
                        ],
                        "type": "best_fields"
                    }
                }
            ]
        }
    },
    "sort": [
        {
            "price": "asc"
        }
    ],
    "highlight": {
        "pre_tags": [
            "<em>"
        ],
        "post_tags": [
            "</em>"
        ],
        "fields": {
            "name": {},
            "description": {}
        }
    }
}
```

查询结果的数据如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)1565585272091

Java 代码如下，注意到上面的 JSON 数据， highlight 和 sort 和 query 依然是同级的，所以也需要用 SearchSourceBuilder 对象来设置到搜索条件中

```java
// 高亮查询
@Test
public void testHighLight() throws IOException, ParseException {
        // 搜索请求对象
        SearchRequest searchRequest = new SearchRequest("ysx_course");
        // 指定类型
        searchRequest.types("doc");
        // 搜索源构建对象
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        // 搜索方式
        // 首先构造多关键字查询条件
        MultiMatchQueryBuilder matchQueryBuilder = QueryBuilders.multiMatchQuery("Spring框架", "name", "description").field("name", 10);
        // 添加条件到布尔查询
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(matchQueryBuilder);
        // 通过布尔查询来构造过滤查询
        boolQueryBuilder.filter(QueryBuilders.rangeQuery("price").gte(60).lte(100));
        // 将查询条件封装给查询对象
        searchSourceBuilder.query(boolQueryBuilder);
        // ***********************

        // 高亮查询
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.preTags("<em>"); // 高亮前缀
        highlightBuilder.postTags("</em>"); // 高亮后缀
        highlightBuilder.fields().add(new HighlightBuilder.Field("name")); // 高亮字段
        // 添加高亮查询条件到搜索源
        searchSourceBuilder.highlighter(highlightBuilder);

        // ***********************

        // 设置源字段过虑,第一个参数结果集包括哪些字段，第二个参数表示结果集不包括哪些字段
        searchSourceBuilder.fetchSource(new String[]{"name","studymodel","price","timestamp"},new String[]{});
        // 向搜索请求对象中设置搜索源
        searchRequest.source(searchSourceBuilder);
        // 执行搜索,向ES发起http请求
        SearchResponse searchResponse = client.search(searchRequest);
        // 搜索结果
        SearchHits hits = searchResponse.getHits();
        // 匹配到的总记录数
        long totalHits = hits.getTotalHits();
        // 得到匹配度高的文档
        SearchHit[] searchHits = hits.getHits();
        // 日期格式化对象
        soutData(searchHits);
        }
```

根据查询结果的数据结构来获取高亮的数据，替换原有的数据：

```java
private void soutData(SearchHit[] searchHits) throws ParseException {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        for (SearchHit hit : searchHits) {
        // 文档的主键
        String id = hit.getId();
        // 源文档内容
        Map<String, Object> sourceAsMap = hit.getSourceAsMap();
        String name = (String) sourceAsMap.get("name");

        // 获取高亮查询的内容。如果存在，则替换原来的name
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        if( highlightFields != null ){
        HighlightField nameField = highlightFields.get("name");
        if(nameField!=null){
        Text[] fragments = nameField.getFragments();
        StringBuffer stringBuffer = new StringBuffer();
        for (Text str : fragments) {
        stringBuffer.append(str.string());
        }
        name = stringBuffer.toString();
        }
        }

        // 由于前边设置了源文档字段过虑，这时description是取不到的
        String description = (String) sourceAsMap.get("description");
        // 学习模式
        String studymodel = (String) sourceAsMap.get("studymodel");
        // 价格
        Double price = (Double) sourceAsMap.get("price");
        // 日期
        Date timestamp = dateFormat.parse((String) sourceAsMap.get("timestamp"));
        System.out.println(name);
        System.out.println(id);
        System.out.println(studymodel);
        System.out.println("你看不见我，看不见我~" + description);
        System.out.println(price);
        }
        }
```

