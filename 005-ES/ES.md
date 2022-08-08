# 全文检索引擎

### 概述

Lucene 只是一个工具包，它不是一个完整的全文检索引擎。Lucene 的目的是为软件开发人员提供一个简单易用的工具包，以方便的在目标系统中实现全文检索的功能，或者是以此为基础建立起完整的全文检索引擎。

目前以 Lucene 为基础建立的开源可用全文搜索引擎主要是 Solr 和 Elasticsearch。

Solr 和 Elasticsearch 都是比较成熟的全文搜索引擎，能完成的功能和性能也基本一样。

 Lucene 能实现全文搜索主要是因为它实现了倒排索引的查询结构 

为了创建倒排索引，我们通过分词器将每个文档的内容域拆分成单独的词（我们称它为词条或 Term），创建一个包含所有不重复词条的排序列表，然后列出每个词条出现在哪个文档。 

ES 是使用 Java 编写的一种开源搜索引擎，它在内部使用 Lucene 做索引与搜索，通过对 Lucene 的封装，隐藏了 Lucene 的复杂性，取而代之的提供一套简单一致的 RESTful API 



### ES有那些数据类型？

- 数字类型：
  - byte、short、integer、long
  - float、double
  - unsigned_long
- **字符串类型**：
  - text ： 会进行分词
  - keyword ： 不会进行分词，适用于email、主机地址、邮编等
- 日期和时间类型：
  - date





### 中文一般使用什么ES分词器？

- ik分词器，其中包含两种分词策略：ik_max_word 、ik_smart 
- 如果不使用ik这类第三方分词器，可以使用ES原生的ngram或edge-ngram分词器，默认ngram分词器最小字符数为1，步长为2，"Quick Fox"将被分为Q, Qu, u, ui, i, ic, c, ck, k, "k ", " ", " F", F, Fo, o, ox, x —— 即空格也会被考虑在内，若不想考虑空格，可设置分词器中的token_chars属性（包括letter、digit、whitespace、punctuation、symbol等）

```json
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 3,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
  }
}

POST my-index-000001/_analyze
{
  "analyzer": "my_analyzer",
  "text": "2 Quick Foxes."
}

// 分词结果为：Qui, uic, ick, Fox, oxe, xes 
```



### 节点角色

每个节点既可以是候选主节点也可以是数据节点，通过在配置文件 ../config/elasticsearch.yml 中设置即可，默认都为 true。 

> node.master: true  // 是否候选主节点  
>
> node.data: true    // 是否数据节点  



- 主节点 负责 创建索引、删除索引、跟踪哪些节点是群集的一部分，并决定哪些分片分配给相关的节点、追踪集群中节点的状态等，稳定的主节点；
- 数据节点 负责 数据的存储和相关的操作，例如对数据进行增、删、改、查、聚合等操作，所以数据节点（Data 节点）对机器配置要求比较高，对 CPU、内存和 I/O 的消耗很大；
- 通常随着集群的扩大，需要增加更多的数据节点来提高性能和可用性；
- 候选主节点可以被选举为主节点（Master 节点），集群中只有候选主节点才有选举权和被选举权，其他节点不参与选举的工作



### 索引index VS 分片shard VS 副本replicas

索引用来存储我们要搜索的数据，以倒排索引结构进行存储。例如，要搜索商品数据，可以创建一个商品数据的索引，其中存储着所有商品的数据；  

当索引中存储了大量数据时，大量的磁盘io操作会降低整体搜索新能，这时需要对数据进行分片存储。在一个索引中存储大量数据会造成性能下降，这时可以对数据进行分片存储。每个节点上都创建一个索引分片，把数据分散存放到多个节点的索引分片上，减少每个分片的数据量来提高io性能：每个分片都是一个**独立的索引**，数据分散存放在多个分片中，也就是说，每个分片中存储的都是不同的数据。搜索时会**同时搜索多个分片**，并将搜索结果进行汇总。 

如果一个节点宕机分片不可用，则会造成**部分数据无法搜索**： 对分片创建多个副本，那么即使一个节点宕机，其他节点中的副本分片还可以继续工作，不会造成数据不可用





### 如何对搜索结果进行高亮显示？

- 原生ES有特殊的语法，比如highlight属性可以给搜索到的内容添加JavaScript标签；
- Springboot则有专门的注解；



### 了解solr吗？

1. 以 Lucene 为基础建立的开源可用全文搜索引擎主要是 Solr 和 Elasticsearch。
2. Solr 和 Elasticsearch 都是比较成熟的全文搜索引擎，能完成的功能和性能也基本一样。



### 如何进行数据库和ES的同步？

