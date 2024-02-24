---
title: "Lucene 一个轻量化的搜索引擎"
description: 
date: 2023-03-21T20:19:40+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
categories:
  - 杂谈
---

## Lucene简介
[官方文档](https://lucene.apache.org/core/8_11_2/)

---
Lucene是一个开源的全文检索引擎工具包，最初由Doug Cutting开发。早在1997年，资深全文检索专家Doug Cutting用一个周末的时间，使用Java语言创作了一个文本搜索的开源函数库，目的是为各种中小型应用软件加入全文检索功能。不久之后，Lucene诞生了，2000年Lucene成为Apache开源社区的一个子项目。随着Lucene被人们熟知，越来越多的用户和研发人员加入其中，完善并壮大项目的发展，Lucene已成为最受欢迎的具有完整的查询引擎和索引引擎的全文检索库。

Lucene的优点：稳定、高效、跨平台。支持搜索排名、按照字段搜索、多个索引合并搜索结果等。



Lucene主要流程：根据各种原始数据（Gather Data）加上元数据后通过Index Documents生成索引文件。然后提供接口，完成用户的查询，并返回结果。
### 查询概述
1. 首先需要分析查询，类似SQL那样，解析查询请求
2. 通过分词技术（注意这里分词器要和创建索引时一样），对查询语句分词
3. 进行关键词检索
4. 对搜索结果进行排序，返回用户检索结果。

### 索引概述
Lucene索引文档要依靠一个IndexWriter对象，创建IndexWriter需要提供两个参数，一个是IndexWriterConfig对象，该对象可以设置创建索引使用哪种分词器，另一个是索引的保存路径。IndexWriter对象的addDocument()方法用于添加文档，该方法的参数为Document对象。IndexWriter对象一次可以添加多个文档，最后调用commit()方法生成索引。
## Lucene执行索引和查询操作
官网给出的简单示例如下
```java
    Analyzer analyzer = new StandardAnalyzer();

    Path indexPath = Files.createTempDirectory("tempIndex");
    Directory directory = FSDirectory.open(indexPath);
    IndexWriterConfig config = new IndexWriterConfig(analyzer);
    IndexWriter iwriter = new IndexWriter(directory, config);
    Document doc = new Document();
    String text = "This is the text to be indexed.";
    doc.add(new Field("fieldname", text, TextField.TYPE_STORED));
    iwriter.addDocument(doc);
    iwriter.close();
    
    // Now search the index:
    DirectoryReader ireader = DirectoryReader.open(directory);
    IndexSearcher isearcher = new IndexSearcher(ireader);
    // Parse a simple query that searches for "text":
    QueryParser parser = new QueryParser("fieldname", analyzer);
    Query query = parser.parse("text");
    ScoreDoc[] hits = isearcher.search(query, 10).scoreDocs;
    assertEquals(1, hits.length);
    // Iterate through the results:
    StoredFields storedFields = isearcher.storedFields();
    for (int i = 0; i < hits.length; i++) {
      Document hitDoc = storedFields.document(hits[i].doc);
      assertEquals("This is the text to be indexed.", hitDoc.get("fieldname"));
    }
    ireader.close();
    directory.close();
    IOUtils.rm(indexPath);
```
接下来将分别简单介绍索引和搜索过程。
### 索引流程
索引需要先创建IndexWriterConfig，然后根据路径打开一个Directory（可以从磁盘或者内存打开，取决于用哪个子类），之后创建一个IndexWriter,然后通过IndexWriter添加Document,最后提交，关闭目录。大概代码如下。
```java
@SpringBootTest
public class CreateIndexTest {
    @Autowired
    Analyzer analyzer;

    public void testCreate() throws IOException {
        IndexWriterConfig iwc = new IndexWriterConfig(analyzer);
        Directory directory = FSDirectory.open(Path.of("index"));
        IndexWriter indexWriter = new IndexWriter(directory,iwc);

        /**
         * 创建document
         */

        indexWriter.addDocument();
        indexWriter.commit();
        directory.close();
    }
}
```
### Field
Field类似于一个键值对，由FieldName和FieldValue组成。一个Document中有多个Field。具体参考这里。[Field官方文档](https://lucene.apache.org/core/9_5_0/core/org/apache/lucene/document/Field.html)
Field可以通过Filed类或者它的子类构造，比如
```java
        Field field = new Field("fieldName","lorem", TextField.TYPE_NOT_STORED);
        Field pageField = new IntField("id",1);
        Field paraField = new IntPoint("para",2);
```
在上面的例子中，IntField使用字符串存储数据，IntPoint使用二进制存储。IntField支持排序，IntPoint支持范围查询，根据需求不同来区分。可以使用`IntPoint.numericValue()`来获取值。

参考[FieldType](https://lucene.apache.org/core/7_1_0/core/org/apache/lucene/document/FieldType.html)

Field有以下属性：
- tokenized：是否被分词
- indexd：是否被索引，如果需要被用作查询条件，indexd为true,比如图片路径这种不会被直接搜索的就填false。
- stored：是否被存储在文档域当中。
- storeTermVectors：是否存储词向量

### 搜索流程
搜索首先需要通过DirectoryReader（这里没有进行深入研究，猜测是类似读写锁的机制）打开一个IndexSearcher
```java
        Directory directory = FSDirectory.open(Path.of("index"));
        DirectoryReader directoryReader = DirectoryReader.open(directory);
        IndexSearcher indexSearcher = new IndexSearcher(directoryReader);
```
在Lucene中，处理用户输入的查询关键词其实就是构建Query对象的过程。Lucene搜索文档需要实例化一个IndexSearcher对象，IndexSearcher对象的search()方法完成搜索过程，Query对象作为search()方法的对象。搜索结果会保存在一个TopDocs类型的文档集合中，遍历TopDocs集合输出文档信息。

然后创建一个QueryParser,QueryParser可以通过解析字符串，创建Query,然后通过Searcher返回结果。
```java
QueryParser parser = new QueryParser (field, analyzer);
Query query = parser.parse("关键词");
```
或者通过`MultiFieldQueryParser(fields,analyzer)`生成多域搜索解析器。

#### 分组查询
[分组查询文档](https://lucene.apache.org/core/8_11_2/grouping/index.html?org/apache/lucene/search/grouping/package-summary.html)

Group分组查询将有某个相同域的整合，比如指定作者名，那么作者名这个域相同的Document会被分为一组。

Group分组需要以下几个输入：
- groupSelector: 决定了如何被分组，可以按照term分，或者按照int或double的某个范围分组。
- groupSort：组与组之间是如何排序的。
- topNGroups：保留多少组，例如10只取前十个分组
- groupOffset：指定组偏移量，比如当topNGroups的值是10的时候，groupOffset为3，则意思是返回7个分组，跳过前面3个，在分页时候很有用
- withinGroupSort：组内排序方式，默认值是Sort.RELEVANCE(关联程度)，注意和groupSort的区别，不要求和groupSort使用一样的排序方式
- maxDocsPerGroup：表示一个组内最多保留多少个文档
- withinGroupOffset：每组显示的文档的偏移量(从第多少个文档开始显示)

分组查询要经过两个Pass
> FirstPassGroupingCollector is the first of two passes necessary to collect grouped hits. This pass gathers the top N sorted groups. Groups are defined by a GroupSelector

第一次收集前N组
> SecondPassGroupingCollector runs over an already collected set of groups, further applying a GroupReducer to each group

第二次收集分组内的文档。

使用GroupSearch整合两次搜索
```java
   GroupingSearch groupingSearch = new GroupingSearch("author");
   groupingSearch.setGroupSort(groupSort);
   groupingSearch.setFillSortFields(fillFields);
 
   if (useCache) {
     // Sets cache in MB
     groupingSearch.setCachingInMB(4.0, true);
   }
 
   if (requiredTotalGroupCount) {
     groupingSearch.setAllGroups(true);
   }
 
   TermQuery query = new TermQuery(new Term("content", searchTerm));
   TopGroups<BytesRef> result = groupingSearch.search(indexSearcher, query, groupOffset, groupLimit);
 
   // Render groupsResult...
   if (requiredTotalGroupCount) {
     int totalGroupCount = result.totalGroupCount;
   }
```

