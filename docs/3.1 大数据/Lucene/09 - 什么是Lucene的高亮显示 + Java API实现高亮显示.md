[09 - 什么是Lucene的高亮显示 + Java API实现高亮显示](https://www.cnblogs.com/shoufeng/p/9439392.html)



# 1  什么是高亮显示

高亮显示是全文检索的一个特点, 指的在搜索结果中对关键词突出显示(加粗和增加颜色).

![图片](https://images2018.cnblogs.com/blog/1438655/201808/1438655-20180807205109194-2135068815.jpg)

# 2  高亮显示实现

Lucene提供了高亮显示组件, 支持高亮显示.

## 2.1  配置pom.xml文件, 加入高亮显示支持

```xml
<project>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!-- mysql版本 -->
        <mysql.version>5.1.30</mysql.version>
        <!-- lucene版本 -->
        <lucene.version>4.10.3</lucene.version>
        <!-- ik分词器版本 -->
        <ik.version>2012_u6</ik.version>
    </properties>
    
    <dependencies>
        // ......
        <!--lucene高亮显示 -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-highlighter</artifactId>
            <version>${lucene.version}</version>
        </dependency>
    </dependencies>
</project>
```

## 2.2  代码实现

步骤为:

> (1) 创建分值对象(QueryScorer), 用于计算高亮显示内容的评分;
>
> (2) 创建输出片段对象(Fragmenter), 用于将高亮显示内容切片;
>
> (3) 创建高亮组件对象(Highlighter), 实现高亮显示;
>
> (4) 创建分析器对象(Analyzer), 用于分词;
>
> (5) 使用TokenSources类, 获取高亮显示内容的流对象(TokenStream);
>
> (6) 使用Highlighter对象, 完成高亮显示.

```java
/**
 * 封装搜索方法(高亮显示的方法)
 */
private void searcherHighlighter(Query query) throws Exception {
    // 打印Query对象生成的查询语法
    System.out.println("查询语法: " + query);
 
    // 1.创建索引库目录位置对象(Directory), 指定索引库的位置
    Directory directory = FSDirectory.open(new File("/Users/healchow/Documents/index"));   

    // 2.创建索引读取对象(IndexReader), 用于将索引数据读取到内存中
    IndexReader reader = DirectoryReader.open(directory);
   
    // 3.创建索引搜索对象(IndexSearcher), 用于执行搜索
    IndexSearcher searcher = new IndexSearcher(reader);   

    // 4. 使用IndexSearcher对象执行搜索, 返回搜索结果集TopDocs
    // 参数一:使用的查询对象, 参数二:指定要返回的搜索结果排序后的前n个
    TopDocs topDocs = searcher.search(query, 10);
   
    // 增加高亮显示处理 ============================== start
    // 1.建立分值对象(QueryScorer), 用于对高亮显示内容打分
    QueryScorer qs = new QueryScorer(query); 
    // 2.建立输出片段对象(Fragmenter), 用于把高亮显示内容切片
    Fragmenter fragmenter = new SimpleSpanFragmenter(qs);   
    // 3.建立高亮组件对象(Highlighter), 实现高亮显示
    Highlighter lighter = new Highlighter(qs);
    // 设置切片对象
    lighter.setTextFragmenter(fragmenter);
    // 4.建立分析器对象(Analyzer), 用于分词
    Analyzer analyzer = new IKAnalyzer();   
    // 增加高亮显示处理 ============================== end

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

        // 实现图书名称的高亮显示
        String bookName = doc.get("bookName");
        if(bookName != null) {
            // 5.使用TokenSources类, 获取高亮显示内容流对象(TokenStream)
            // getTokenStream方法: 获取当前文档的流对象
            // 参数一: 当前的文档对象
            // 参数二: 要高亮显示的域名称
            // 参数三: 分析器对象
            TokenStream tokenStream = TokenSources.getTokenStream(doc, "bookName", analyzer);
          
            // 6.使用高亮组件对象，完成高亮显示
            // getBestFragment方法:获取高亮显示结果内容
            // 参数一: 当前文档对象的流对象
            // 参数二: 要高亮显示的目标内容
            bookName = lighter.getBestFragment(tokenStream, bookName);
        }
        
        System.out.println("图书名称: " + bookName);
        System.out.println("图书价格: " + doc.get("bookPrice"));
        System.out.println("图书图片: " + doc.get("bookPic"));
        System.out.println("图书描述: " + doc.get("bookDesc"));
    }
    
    // 8. 关闭资源
    reader.close();
}
/**
 * 测试高亮显示 需求：把搜索结果中，图书名称进行高亮显示(关键词值java)
 * @throws Exception 
 */
@Test
public void testHighlighter() throws Exception {
    //1.创建查询对象
    TermQuery tq = new TermQuery(new Term("bookName","java"));
   
    // 2.执行高亮搜索
    this.searcherHighlighter(tq);
}
```

![图片](https://images2018.cnblogs.com/blog/1438655/201808/1438655-20180807205127053-31682461.jpg)

## 2.3  自定义html标签高亮显示

问题: 实际项目中，如何实现用自定义的HTML标签，进行搜索结果的高亮显示?

> ① 创建HTML标签格式化对象(SimpleHTMLFormatter);
>
> ② 创建高亮显示组件对象(Highlighter), 指定使用SimpleHTMLFormatter对象.

```java
// 增加高亮显示处理 ============================== start
// 1.建立分值对象(QueryScorer), 用于对高亮显示内容打分
QueryScorer qs = new QueryScorer(query);

// 2.建立输出片段对象(Fragmenter), 用于把高亮显示内容切片
Fragmenter fragmenter = new SimpleSpanFragmenter(qs);

// 3.建立高亮组件对象(Highlighter), 实现高亮显示
// 3.1.实现自定义的HTML标签进行高亮显示搜索结果
// 1) 建立高亮显示HTML格式化标签对象(SimpleHTMLFormatter), 参数说明: 
// preTag: 指定HTML标签的开始部分(<font color='red'>)
// postTag: 指定HTML标签的结束部分(</font>)
SimpleHTMLFormatter formatter = new SimpleHTMLFormatter("<font color='red'>", "</font>");
// 2) 指定高亮显示组件对象（Highter）, 使用SimpleHTMLFormatter对象
Highlighter lighter = new Highlighter(formatter, qs);

// 设置切片对象
lighter.setTextFragmenter(fragmenter);

// 4.建立分析器对象(Analyzer), 用于分词
Analyzer analyzer = new IKAnalyzer();
// 增加高亮显示处理 ============================== end
```

![图片](https://images2018.cnblogs.com/blog/1438655/201808/1438655-20180807205218919-1154480536.jpg)

> # 