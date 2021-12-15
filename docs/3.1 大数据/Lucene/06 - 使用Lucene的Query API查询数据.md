[06 - 使用Lucene的Query API查询数据](https://www.cnblogs.com/shoufeng/p/9399691.html)



> Lucene是使用Query对象执行查询的, 由Query对象生成查询的语法. 如bookName:java, 表示搜索bookName域中包含java的文档数据.

# 1  Query对象的创建(方式一): 使用子类对象

## 1.1  常用的Query子类对象

| **子类对象**          | **说明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| **TermQuery**         | 不使用分析器, 对关键词做精确匹配搜索. 如:订单编号、身份证号  |
| **NumericRangeQuery** | 数字范围查询, 比如: 图书价格大于80, 小于100                  |
| **BooleanQuery**      | 布尔查询, 实现组合条件查询. 组合关系有:   1. MUST与MUST: 表示“与”, 即“交集”    2. MUST与MUST NOT: 包含前者, 排除后者    3. MUST   NOT与MUST NOT: 没有意义    4. SHOULD与MUST: 表示MUST, SHOULD失去意义    5. SHOULD与MUST NOT: 等于MUST与MUST   NOT    6. SHOULD与SHOULD表示“或”, 即“并集” |

## 1.2  常用的Query子类对象使用

### 1.2.1  使用TermQuery

(1) 需求: 查询图书名称中包含java的图书.

```java
/**
 * 搜索索引(封装搜索方法)
 */
private void seracher(Query query) throws Exception {
    // 打印查询语法
    System.out.println("查询语法: " + query);
    
    // 1.创建索引库目录位置对象(Directory), 指定索引库的位置
    Directory directory = FSDirectory.open(new File("/Users/healchow/Documents/index"));
    
    // 2.创建索引读取对象(IndexReader), 用于读取索引
    IndexReader reader = DirectoryReader.open(directory);
    
    // 3.创建索引搜索对象(IndexSearcher), 用于执行搜索
    IndexSearcher searcher = new IndexSearcher(reader);  

    // 4. 使用IndexSearcher对象执行搜索, 返回搜索结果集TopDocs
    // 参数一:使用的查询对象, 参数二:指定要返回的搜索结果排序后的前n个
    TopDocs topDocs = searcher.search(query, 10);

    // 5. 处理结果集
    // 5.1 打印实际查询到的结果数量
    System.out.println("实际查询到的结果数量: " + topDocs.totalHits);
    // 5.2 获取搜索的结果数组
    // ScoreDoc中有文档的id及其评分
    ScoreDoc[] scoreDocs = topDocs.scoreDocs;
    
    for (ScoreDoc scoreDoc : scoreDocs) {
        System.out.println("= = = = = = = = = = = = = = = = = = =");
        // 获取文档的id和评分
        int docId = scoreDoc.doc;
        float score = scoreDoc.score;
        System.out.println("文档id= " + docId + " , 评分= " + score);
        
        // 根据文档Id, 查询文档数据 -- 相当于关系数据库中根据主键Id查询数据
        Document doc = searcher.doc(docId);
        System.out.println("图书Id: " + doc.get("bookId"));
        System.out.println("图书名称: " + doc.get("bookName"));
        System.out.println("图书价格: " + doc.get("bookPrice"));
        System.out.println("图书图片: " + doc.get("bookPic"));
        System.out.println("图书描述: " + doc.get("bookDesc"));
    }
    
    // 6. 关闭资源
    reader.close();
}
```

(2) 测试使用TermQuery:

```java
/**
 * 测试使用TermQuery: 需求: 查询图书名称中包含java的图书
 */
@Test
public void testTermQuery() throws Exception {
    //1. 创建TermQuery对象
    TermQuery termQuery = new TermQuery(new Term("bookName", "java"));
    // 2.执行搜索
    this.seracher(termQuery);
}
```

![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731233454131-917596653.jpg)

### 1.2.2  使用NumericRangeQuery

(1) 需求: 查询图书价格在80-100之间的图书(不包含80和100):

```java
/**
 * 测试使用NumericRangeQuery: 需求: 查询图书价格在80-100之间的图书
 */
@Test
public void testNumericRangeQuery() throws Exception{
    // 1.创建NumericRangeQuery对象, 参数说明: 
    // field: 搜索的域; min: 范围最小值; max: 范围最大值
    // minInclusive: 是否包含最小值(左边界); maxInclusive: 是否包含最大值(右边界)
    NumericRangeQuery numQuery = NumericRangeQuery.newFloatRange("bookPrice", 80f, 100f, false, false);

    // 2.执行搜索
    this.seracher(numQuery); 
}
```

![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731233520895-829754029.jpg)

(2) 测试包含80和100:

```java
// 测试包含80和100
NumericRangeQuery numQuery = NumericRangeQuery.newFloatRange("bookPrice", 80f, 100f, true, true);
```

![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731233535883-1029468024.jpg)

### 1.2.3  使用BooleanQuery

- 需求: 查询图书名称中包含Lucene, 并且价格在80-100之间的图书.

```java
/**
 * 测试使用BooleanQuery: 需求: 查询图书名称中包含Lucene, 且价格在80-100之间的图书
 */
@Test
public void testBooleanQuery() throws Exception {
    // 1.创建查询条件
    // 1.1.创建查询条件一
    TermQuery query1 = new TermQuery(new Term("bookName", "lucene"));
    
    // 1.2.创建查询条件二
    NumericRangeQuery query2 = NumericRangeQuery.newFloatRange("bookPrice", 80f, 100f, true, true);
    // 2.创建组合查询条件
    BooleanQuery bq = new BooleanQuery();
    // add方法: 添加组合的查询条件
    // query参数: 查询条件对象
    // occur参数: 组合条件
    bq.add(query1, Occur.MUST);
    bq.add(query2, Occur.MUST);
    
    // 3.执行搜索
   this.seracher(bq);
}
```

查询语法中, "+"表示并且条件, "-"表示不包含后面的条件:
 ![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731233551268-970801123.jpg)

# 2  Query对象的创建(方式二): 使用QueryParser

说明: 使用QueryParser对象解析查询表达式, 实例化Query对象.

## 2.1  QueryParse表达式语法

(1) 关键词基本查询: 域名+":"+关键词, 比如: `bookname:lucene`

(2) 范围查询: `域名+":"+[最小值 TO 最大值]`, 比如: `price:[80 TO 100]`.

> **需要注意的是: QueryParser不支持数字范围查询, 仅适用于字符串范围查询. 如果有数字范围查询需求, 请使用NumericRangeQuery. **

(3) 组合查询:

| **条件表示符**     | **符号说明**                  | **符号表示** |
| ------------------ | ----------------------------- | ------------ |
| **Occur.MUST**     | 搜索条件必须满足, 相当于AND   | +            |
| **Occur.SHOULD**   | 搜索条件可选, 相当于OR        | 空格         |
| **Occur.MUST_NOT** | 搜索条件不能满足, 相当于NOT非 | -            |

## 2.2  使用QueryParser

需求: 查询图书名称中包含java, 并且图书名称中包含"Lucene"的图书.

```java
/**
 * 测试使用QueryParser: 需求: 查询图书名称中包含Lucene, 且包含java的图书
 */
@Test
public void testQueryParser() throws Exception {
    // 1.创建查询对象
    // 1.1.创建分析器对象
    Analyzer analyzer = new IKAnalyzer();
    // 1.2.创建查询解析器对象
    QueryParser qp = new QueryParser("bookName", analyzer);
    // 1.3.使用QueryParser解析查询表达式
    Query query = qp.parse("bookName:java AND bookName:lucene");
    
    // 2.执行搜索
    this.seracher(query);
}
```

![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731233712304-564682659.jpg)

注意: 使用QueryParser, 表达式中的组合关键字`AND/OR/NOT`必须要大写. 设置了默认搜索域后, 若查询的域没有改变, 则可不写.