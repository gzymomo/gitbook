[07 - 对Lucene的索引库进行增删改查](https://www.cnblogs.com/shoufeng/p/9399728.html)



> 数据保存在关系型数据库中, 需要实现增、删、改、查操作; 索引保存在索引库中, 也需要实现增、删、改、查操作.

# 1  添加索引

参考 [Lucene 02 - Lucene的入门程序(Java API的简单使用)](https://www.cnblogs.com/shoufeng/p/9367789.html) 中的内容:

![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731233725702-1570320147.jpg)

# 2  删除索引

## 2.1  根据Term删除索引

根据Term删除索引的步骤为:

> (1) 创建分析器对象(Analyzer), 用于分词;
>
> (2) 创建索引配置对象(IndexWriterConfig), 用于配置Lucene;
>
> (3) 创建索引库目录对象(Directory), 用于指定索引库的位置;
>
> (4) 创建索引写入对象(IndexWriter), 用于操作索引;
>
> (5) 创建删除条件对象(Term);
>
> (6) 使用IndexWriter对象, 执行删除;
>
> (7) 释放资源.

```java
/**
 * 根据Term删除索引
 */
@Test
public void deleteIndexByTerm() throws IOException {
    // 1.创建分析器对象(Analyzer), 用于分词
    Analyzer analyzer = new IKAnalyzer(); 

    // 2.创建索引配置对象(IndexWriterConfig), 用于配置Lucene
    IndexWriterConfig iwc = new IndexWriterConfig(Version.LUCENE_4_10_4, analyzer);

    // 3.创建索引库目录对象(Directory), 用于指定索引库的位置
    Directory directory = FSDirectory.open(new File("/Users/healchow/Documents/index"));
    
    // 4.创建索引写入对象(IndexWriter), 用于操作索引
    IndexWriter writer = new IndexWriter(directory, iwc);
    
    // 5.创建删除条件对象(Term)
    // 删除图书名称域中, 包含"java"的索引
    // delete from table where name="java"
    // 参数一: 删除的域的名称, 参数二: 删除的条件值
    Term term = new Term("bookName", "java"); 

    // 6.使用IndexWriter对象, 执行删除
    // 可变参数, 能传多个term
    writer.deleteDocuments(term);
  
   // 7.释放资源
    writer.close();
}
```

根据Term执行删除操作(`indexWriter.deleteDocuments(term)`)时, **要求对应的Field不能分词且只能是一个词, 且这个Field必须索引过**, Lucene将先去搜索, 然后将所有满足条件的记录删除(伪删除, 做了".del"标记) -- 最好定义一个唯一标识来做删除操作.
 ![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731233905132-480067076.jpg)

是否删除索引, 需要分情况讨论: 我们知道, Lucene是以段(segment)来组织索引内容的,  通过Term执行删除操作(indexWriter.deleteDocuments(term))时,  若索引段中仍包含符合条件的文档对象的其他分词的索引, 就会保留整个索引数据(若采取更新操作, 则会降低性能), 如果没有, 则也将删除索引数据:
 ![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731233857746-963949999.jpg)

查看删除了的文档的编号:
 ![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731234057152-902287296.jpg)

## 2.2  删除全部索引(慎用)

删除全部索引的步骤为:

> (1) 创建分析器对象(Analyzer), 用于分词;
>
> (2) 创建索引配置对象(IndexWriterConfig), 用于配置Lucene;
>
> (3) 创建索引库目录对象(Directory), 指定索引库位置;
>
> (4) 创建索引写入对象(IndexWriter), 用于操作索引库;
>
> (5) 使用IndexWriter对象, 执行删除;
>
> (6) 释放资源.

```java
/**
* 删除全部索引
*/
@Test
public void deleteAllIndex() throws IOException {
   // 1.创建分析器对象(Analyzer), 用于分词
   Analyzer analyzer = new IKAnalyzer();

   // 2.创建索引配置对象(IndexWriterConfig), 用于配置Lucene
   IndexWriterConfig iwc = new IndexWriterConfig(Version.LUCENE_4_10_4, analyzer);
   
   // 3.创建索引库目录对象(Directory), 用于指定索引库的位置
   Directory directory = FSDirectory.open(new File("/Users/healchow/Documents/index"));
   
   // 4.创建索引写入对象(IndexWriter), 用于操作索引
   IndexWriter writer = new IndexWriter(directory, iwc);
   
   // 5.使用IndexWriter对象, 执行删除
   writer.deleteAll();
   
   // 6.释放资源
  writer.close();
}
```

删除后的索引结果:
 ![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731234113496-232488901.jpg)

> 删除全部索引, 将文档域的数据, 索引域的数据都删除.
>
> 类似于关系型数据库的Truncate删除: 完全删除数据, 包括存储结构, 因而更快速.

# 3  更新索引

> Lucene是根据Term对象更新索引: 先根据Term执行查询, 查询到则执行更新, 查询不到则执行添加索引.

更新索引的步骤为:

> (1) 创建分析器对象(Analyzer), 用于分词;
>
> (2) 创建索引配置对象(IndexWriterConfig), 用于配置Lucene;
>
> (3) 创建索引库目录对象(Directory), 用于指定索引库的位置;
>
> (4) 创建索引写入对象(IndexWriter), 用于操作索引库;
>
> (5) 创建文档对象(Document);
>
> (6) 创建Term对象;
>
> (7) 使用IndexWriter对象, 执行更新;
>
> (8) 释放资源.

```java
/**
 * 更新索引
 */
@Test
public void updateIndexByTerm() throws IOException{
    // 1.创建分析器对象(Analyzer), 用于分词
    Analyzer analyzer = new IKAnalyzer();
   
    // 2.创建索引配置对象(IndexWriterConfig), 用于配置Lucene
    IndexWriterConfig iwc = new IndexWriterConfig(Version.LUCENE_4_10_4, analyzer);   

    // 3.创建索引库目录对象(Directory), 用于指定索引库的位置
    Directory directory = FSDirectory.open(new File("/Users/healchow/Documents/index"));   

    // 4.创建索引写入对象(IndexWriter), 用于操作索引
    IndexWriter writer = new IndexWriter(directory, iwc);
 
    // 5.创建文档对象(Document)
    Document doc = new Document();
    doc.add(new TextField("id", "1234", Store.YES));
    // doc.add(new TextField("name", "MyBatis and SpringMVC", Store.YES));
    doc.add(new TextField("name", "MyBatis and Struts2", Store.YES)); 

    // 6.创建Term对象
    Term term = new Term("name", "SpringMVC"); 

    // 7.使用IndexWriter对象, 执行更新
    writer.updateDocument(term, doc);
   
    // 8.释放资源
    writer.close();
}
```

第一次执行, 由于没有查找到对应的索引, 故执行添加功能, 结果如图(不区分大小写):
 ![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731234129306-1488602472.jpg)

第二次执行时, 由于索引库中已有`"name"="SpringMVC"`的内容, 故执行更新操作: 将整个TextField的内容:`MyBatis and Struts2`添加到索引库中, 并与上次的结果合并, 结果如下图示:

![图片](https://images2018.cnblogs.com/blog/1438655/201807/1438655-20180731234143699-1293111662.jpg)

若第二次执行时更改了Term中name域的条件值(索引中没有对应的), 将继续执行添加功能: 将整个TextField中的内容添加到索引中.