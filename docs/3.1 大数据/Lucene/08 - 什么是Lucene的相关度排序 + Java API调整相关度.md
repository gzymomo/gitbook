[08 - 什么是Lucene的相关度排序 + Java API调整相关度](https://www.cnblogs.com/shoufeng/p/9410785.html)



# 1  什么是相关度

概念: 相关度指两个事物之间的关联关系(相关性). Lucene中指的是搜索关键词(Term)与搜索结果之间的相关性. 比如搜索bookname域中包含java的图书, 则根据java在bookname中出现的次数和位置来判断结果的相关性.

# 2  相关度评分

Lucene对查询关键字和索引文档的相关度进行打分, 得分越高排序越靠前.

(1) Lucene的打分方法: Lucene在用户进行检索时根据**实时搜索**的关键字计算分值, 分两步:

> ① 计算出词(Term)的权重;
>
> ② 根据词的权重值, 计算文档相关度得分.

(2) 什么是词的权重?

通过索引部分的说明, 易知索引的最小单位是Term(索引词典中的一个词). 搜索也是从索引域中查询Term, 再根据Term找到文档. **Term对文档的重要性称为Term的权重. **

(3) 影响Term权重的因素有两个:

> ① Term Frequency(tf): **指这个Term在当前的文档中出现了多少次. tf 越大说明越重要. **
>
> 词(Term)在文档中出现的次数越多, 说明此词(Term)对该文档越重要, 如"Lucene"这个词, 在文档中出现的次数很多, 说明该文档可能就是讲Lucene技术的.
>
> ② Document Frequency(df): **指有多少个文档包含这个Term. df 越大说明越不重要. **
>
> 比如: 在某篇英文文档中, this出现的次数很多, 能说明this重要吗? 不是的, 有越多的文档包含此词(Term), 说明此词(Term)越普通, 不足以区分这些文档, 因而重要性越低.

# 3  相关度设置

Lucene通过设置关键词Term的权重(boost)值, 影响相关度评分, 从而影响搜索结果的排序.

## 3.1  更改相关度的需求

出版社做了广告推广: 收到广告费之后, 将《Lucene Java精华版》排到第一.

![图片](https://images2018.cnblogs.com/blog/1438655/201808/1438655-20180802230100108-62706918.jpg)

## 3.2  实现需求-设置广告

```java
/**
 * 相关度排序, 通过修改索引库的方式, 修改需要更改的图书的权重
 */
@Test
public void updateIndexBoost() throws IOException {
    // 1.建立分析器对象(Analyzer), 用于分词
    Analyzer analyzer = new IKAnalyzer();

    // 2.建立索引库配置对象(IndexWriterConfig), 配置索引库
    IndexWriterConfig iwc = new IndexWriterConfig(Version.LUCENE_4_10_4, analyzer);
   
    // 3.建立索引库目录对象（Directory），指定索引库位置
    Directory directory  = FSDirectory.open(new File("/Users/healchow/Documents/index")); 

    // 4.建立索引库操作对象(IndexWriter), 操作索引库
    IndexWriter writer = new IndexWriter(directory,iwc);
   
    // 5.建立文档对象(Document)
    Document doc = new Document(); 
    // 5 Lucene Java精华版 80 5.jpg 
    doc.add(new StringField("bookId", "5", Store.YES));  
    TextField nameField = new TextField("bookName", "Lucene Java精华版", Store.YES);
    // 设置权重值为100. 默认是1
    nameField.setBoost(100f);
    doc.add(nameField);
    doc.add(new FloatField("bookPrice", 80f, Store.YES));
    doc.add(new StoredField("bookPic","5.jpg"));

    // 6.建立更新条件对象（Term）
    Term term = new Term("bookId", "5");
    
    // 7.使用IndexWriter对象，执行更新
    writer.updateDocument(term, doc);
   
    // 8.释放资源
    writer.close();
}
// 或在创建索引时即修改权重: 
// 打个广告: 收到钱之后, 将《Lucene Java精华版》排到第一 
// 5 Lucene Java精华版 80 5.jpg 
TestField bookNameField = new TextField("bookName", book.getBookname(), Store.YES); 
if (book.getId() == 5) {
    // 设置权重值为100. 默认是1
    bookNameField.setBoost(100f);
}
document.add(bookNameField);
```

![图片](https://images2018.cnblogs.com/blog/1438655/201808/1438655-20180802230122559-425006923.jpg)

> # 