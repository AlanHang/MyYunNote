[toc]

##  1.入门

### Elasticsearch 与传统关系型数据库的对比
Relational DB  -> Databases  -> Tables  -> Rows  -> Columns  
Elasticsearch  -> Indices  -> Type  -> Documents ->Fields  
//新的版本中，一个INDEX下只允许存在一个TYPE;

### 添加信息
```
curl -H "Content-Type:application/json" -XPUT 'http://主机地址:9200/索引名/类型名/id ' -d '
{
  /*添加的内容 */
} '
```

**例:**

```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/megacorp/employee/1' -d '
{
"first_name" : "John",
"last_name" : "Smith",
"age" : 25,
"about" : "I love to go rock climbing",
"interests": [ "sports", "music" ]
}
```
### 检索文档
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/1'
```
#### 简单搜索
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search'
//默认情况下回搜索会返回前10个结果
```
#### 使用DSL语句查询
Elasticsearch 提供丰富且灵活的查询语言叫做DSL查询，它允许你构建更加复杂、强大的查询：
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}'
```
#### 更加复杂的搜索
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9400/megacorp/employee/_search?pretty' -d '
{    
    "query" : {
        "bool" : {
            "filter" : {
                "range" : {
                        "age" : { "gt" : 30 }
                    }
                } ,
            "must" : {
                "match" : {
                    "last_name" : "smith"
                }
            }
        }
    }
}'
```
#### 全文搜索
搜索“rock climbing”的员工:
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}'
```
搜索结果
```
{
...
"hits": {
    "total": 2,
    "max_score": 0.16273327,
    "hits": [
        {
        ...
        "_score": 0.16273327, <1>
        "_source": {
            "first_name": "John",
            "last_name": "Smith",
            "age": 25,
            "about": "I love to go rock climbing",
            "interests": [ "sports", "music" ]
            }
        },
        {
        ...
        "_score": 0.016878016, <2>
        "_source": {
            "first_name": "Jane",
            "last_name": "Smith",
            "age": 32,
            "about": "I like to collect rock albums",
            "interests": [ "music" ]
            }
        }
        ]
    }
}
/*<1><2> 结果相关性评分。默认情况下，Elasticsearch根据结果相关性评分来对结果集进行排序，所谓的「结果相关性评分」就是文档与查询条件的匹配
程度。很显然，排名第一的 John Smith 的 about 字段明确的写到“rock climbing”。但是为什么 Jane Smith 也会出现在结果里呢？原因是“rock”
在她的 abuot 字段中被提及了。因为只有“rock”被提及而“climbing”没有，所以她的 _score 要低于John。这个例子很好的解释了Elasticsearch如何
在各种文本字段中进行全文搜索，并且返回相关性最大的结果集。相关性(relevance)的概念在Elasticsearch中非常重要，而这个概念在传统关系型数据库中
是不可想象的，因为传统数据库对记录的查询只有匹配或者不匹配。
*/
```
#### 短语搜索
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}'
```
#### 高亮我们的搜索
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search' -d '
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}'
```
### 聚合
```
/*5.x后对排序，聚合这些操作用单独的数据结构(fielddata)缓存到内存里了，需要单独开启*/
//简单来说就是在聚合前执行如下操作:
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search' -d '
{
    "properties": {
        "interests": { 
            "type":     "text",
            "fielddata": true
        }
    }
}'
```
 ```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search' -d '
{
    "aggs": {
        "all_interests": {
            "terms": { "field": "interests" }
        }
    }
}'
 ```
如果我们想知道所有姓"Smith"的人最大的共同点（兴趣爱好），我们只需要增加合适的语句既可：
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search' -d '
{
    "query": {
        "match": {
            "last_name": "smith"
        }
    },
    "aggs": {
        "all_interests": {
            "terms": { "field": "interests" }
        }
    }
}'
```
聚合也允许分级汇总。例如，让我们统计每种兴趣下职员的平均年龄：
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search' -d '
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}'
```
## 2.分布式集群

### 分布式的特性

&nbsp;&nbsp;&nbsp;&nbsp; 在章节的开始我们提到Elasticsearch可以扩展到上百（甚至上千）的服务器来处理PB级的数据。然而我们的教程只是给出了一些使用Elasticsearch的例子，并未涉及相关机制。  
Elasticsearch为分布式而生，而且它的设计隐藏了分布式本身的复杂性。Elasticsearch在分布式概念上做了很大程度上的透明化，在教程中你不需要知道任何关于分布式系统、分片、集群发现或者其他大量的分布式概念。所有的教程你既可以运行在你的笔记本上，也可以运行在拥有100个节点的集群上，其工作方式是一样的  
Elasticsearch致力于隐藏分布式系统的复杂性。以下这些操作都是在底层自动完成的：将你的文档分区到不同的容器或者分片(shards)中，它们可以存在于一个或多个节点中。将分片均匀的分配到各个节点，对索引和搜索做负载均衡。冗余每一个分片，防止硬件故障造成的数据丢失。将集群中任意一个节点上的请求路由到相应数据所在的节点。无论是增加节点，还是移除节点，分片都可以做到无缝的扩展和迁移。

### 集群健康

**查看集群健康**
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/_cluster/health'
```
status 字段提供一个综合的指标来表示集群的的服务状况。三种颜色各自的含义：  

颜色  | 意义
---|---
green| 所有主要分片和复制分片都可用
yellow|  所有主要分片可用，但不是所有复制分片都可用
red| 不是所有的主要分片都可用

### 添加索引

&nbsp;&nbsp;&nbsp;&nbsp; 
索引只是一个用来指向一个或多个分片(shards)的“逻辑命名空间(logical namespace)”.一个分片(shard)是一个最小级别“工作单元(worker unit)”,它只是保存了索引中所有数据的一部分。

**添加索引**
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/blogs' -d '
{
"settings" : {
"number_of_shards" : 3,
"number_of_replicas" : 1
}
}'
/*默认情况下，一个索引被分配5个主分片*/
```
## 3.数据

### 什么是文档？

&nbsp;&nbsp;&nbsp;&nbsp; 
程序中大多的实体或对象能够被序列化为包含键值对的JSON对象，键(key)是字段(field)或属性(property)的名字，值(value)可以是字符串、数字、布尔类型、另一个对象、值数组或者其他特殊类型，比如表示日期的字符串或者表示地理位置的对象。  

**文档和对象的差异**
对象(Object)是一个JSON结构体——类似于哈希、hashmap、字典或者关联数组；对象
(Object)中还可能包含其他对象(Object)。 在Elasticsearch中，文档(document)这个术语有着特殊含义。它特指最顶层结构或者根对象(root object)序列化成的JSON数据（以唯一ID标识并存储于Elasticsearch中）。
### 文档

#### 文档元数据

&nbsp;&nbsp;&nbsp;&nbsp; 一个文档不只有数据。它还包含了元数据(metadata)——关于文档的信息。三个必须的元数
据节点是：

节点| 说明
---|---
_index| 文档存储的地方
_type| 文档代表的类
_id| 文档的唯一标识

**_index**  
&nbsp;&nbsp;&nbsp;&nbsp; 事实上，我们的数据被存储和索引在分片(shards)中，索引只是一个把一个或多个分片分组在一起的逻辑空间，而这只是一些内部细节——我们的程序完全不用关心分片。对于我们的程序而言，文档存储在索引(index)中。
> _index字必须是全部小写，不能以下划线开头，不能包含逗号。

**_type**  
&nbsp;&nbsp;&nbsp;&nbsp; 在Elasticsearch中，我们使用相同类型(type)的文档表示相同的“事物”，因为他们的数据结构也是相同的。  
&nbsp;&nbsp;&nbsp;&nbsp; 每个类型(type)都有自己的映射(mapping)或者结构定义，就像传统数据库表中的列一样。所有类型下的文档被存储在同一个索引下，但是类型的映射(mapping)会告诉Elasticsearch不同的文档如何被索引。
> _type 的名字可以是大写或小写，不能包含下划线或逗号。

**_id**    
&nbsp;&nbsp;&nbsp;&nbsp;
id仅仅是一个字符串，它与 _index 和 _type 组合时，就可以在Elasticsearch中唯一标识一个文档。当创建一个文档，你可以自定义 _id ，也可以让Elasticsearch帮你自动生成。
### 索引一个文档
&nbsp;&nbsp;&nbsp;&nbsp;
文档通过 indexAPI被索引——使数据可以被存储和搜索。但是首先我们需要决定文档所在。正如我们讨论的，文档通过其 _index 、 _type 、id唯一确定。们可以自己提供一个 _id ，或者也使用 index API 为我们生成一个。  

使用自己的_id:
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/website/blog/123' -d '
{
    "title": "My first blog entry",
    "text": "Just trying this out...",
    "date": "2014/01/01"
}'
```
elasticsearch的响应：
```
{
    "_index": "website",
    "_type": "blog",
    "_id": "123",
    "_version": 1,
    "created": true
}
/*响应指出请求的索引已经被成功创建，这个索引中包含 _index、_type 和 _id元数据，以及一个新元素：_version 。
Elasticsearch中每个文档都有版本号，每当文档变化（包括删除）都会使 _version 增加。*/
```
自增id  
&nbsp;&nbsp;&nbsp;&nbsp;
请求结构发生了变化： PUT 方法——“在这个URL中存储文档” 变成了 POST 方法—— "在这个类型下存储文档"。（译者注：原来是把文档存储到某个ID对应的空间，现在是把这个文档添加到某个 _type 下）。
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/blog/' -d '
{
"title": "My second blog entry",
"text": "Still trying this out...",
"date": "2014/01/01"
}'
```
elasticsearch响应内容:
```
{
    "_index":"website",
    "_type":"blog",
    "_id":"5xQwcGwBXDGie0pdgjfb",
    "_version":1,
    "result":"created",
    "_shards":{
        "total":2,
        "successful":2,
        "failed":0
    },
    "_seq_no":0,
    "_primary_term":1
    /*自动生成的ID有22个字符长，URL-safe, Base64-encoded string universally unique
identifiers, 或者叫 UUIDs。*/
}
```
### 检索文档
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/website/blog/_search?pretty'
```
> pretty在任意的查询字符串中增加 pretty 参数，类似于上面的例子。会让Elasticsearch美化输出(pretty-print)JSON响应以便更加容易阅读。 _source 字段不会被美化，它的样子与我
们输入的一致。

> 可以在curl后加上-i参数获得响应头

> 多个查询字符串？之后用&拼起来 如：

```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/website/blog/1?_source=title,text&pretty'
```

#### 检索文档的一部分
&nbsp;&nbsp;&nbsp;&nbsp;
通常， GET 请求将返回文档的全部，存储在 _source 参数中。但是可能你感兴趣的字段只是 title 。请求个别字段可以使用 _source 参数。多个字段可以使用逗号分隔：
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/website/blog/1?_source=title,text'
```

#### 检索文档是否存在
如果你想做的只是检查文档是否存在——你对内容完全不感兴趣——使用 HEAD 方法来代替 GET 。 HEAD 请求不会返回响应体，只有HTTP头：
```
curl -i -H "Content-Type:application/json" -XHEAD 'http://172.16.44.29:9200/website/blog/123'
```
Elasticsearch将会返回 200 OK 状态如果你的文档存在：
```
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```
如果不存在返回 404 Not Found ：
```
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

### 更新整个文档
文档在Elasticsearch中是不可变的——我们不能修改他们。如果需要更新已存在的文档，我们可以使用《索引文档》章节提到的 index API 重建索引(reindex) 或者替换掉它。
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/website/blog/123?pretty' -d'
{
    "title": "My first blog entry",
    "text": "I am starting to get the hang of this...",
    "date": "2014/01/02"
}'
```
在响应中我们可以看见Elasticsearch 把_version 增加了。
```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "123",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

```
### 创建一个新的文档
&nbsp;&nbsp;&nbsp;&nbsp;
如果想使用自定义的 _id ，我们必须告诉Elasticsearch应该
在 _index 、 _type 、 _id 三者都不同时才接受请求。为了做到这点有两种方法，它们其实做的是同一件事情。你可以选择适合自己的方式：  
第一种方法使用 op_type 查询参数：
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/website/blog/123?op_type=create' -d'
{
    "title": "My first blog entry",
    "text": "I am starting to get the hang of this...",
    "date": "2014/01/02"
}'
```
或者第二种方法是在URL后加 /_create 做为端点：
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/website/blog/123/_create?pretty' -d'
{
    "title": "My first blog entry",
    "text": "I am starting to get the hang of this...",
    "date": "2014/01/02"
}'
```
如果请求成功的创建了一个新文档，Elasticsearch将返回正常的元数据且响应状态码是Created 。
```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "124",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```
另一方面，如果包含相同的 _index 、 _type 和 _id 的文档已经存在，Elasticsearch将返回 409 Conflict 响应状态码，错误信息类似如下：
```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "version_conflict_engine_exception",
        "reason" : "[blog][123]: version conflict, document already exists (current version [2])",
        "index_uuid" : "0CF8ZVEzSKqCvSofbgdQWA",
        "shard" : "0",
        "index" : "website"
      }
    ],
    "type" : "version_conflict_engine_exception",
    "reason" : "[blog][123]: version conflict, document already exists (current version [2])",
    "index_uuid" : "0CF8ZVEzSKqCvSofbgdQWA",
    "shard" : "0",
    "index" : "website"
  },
  "status" : 409
}
```
### 删除文档
删除文档的语法模式与之前基本一致，只不过要使用 DELETE 方法：
```
curl -H "Content-Type:application/json" -XDELETE 'http://172.16.44.29:9200/website/blog/123/?pretty'
```
如果文档被找到，Elasticsearch将返回 200 OK 状态码和以下响应体。注意 _version 数字已经增加了。
```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "123",
  "_version" : 3,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```
如果文档未找到，我们将得到一个 404 Not Found 状态码，响应体是这样的：
```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "123",
  "_version" : 4,
  "result" : "not_found",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 6,
  "_primary_term" : 1
}
```
尽管文档不存在—— "found" 的值是 false —— _version 依旧增加了。这是内部记录的一部分，它确保在多节点间不同操作可以有正确的顺序。
> 正如在《更新文档》一章中提到的，删除一个文档也不会立即从磁盘上移除，它只是被标记成已删除。Elasticsearch将会在你之后添加更多索引的时候才会在后台进行删除内容的清理。

### 处理冲突
当使用 index API更新文档的时候，我们读取原始文档，做修改，然后将整个文档(wholedocument)一次性重新索引。最近的索引请求会生效——Elasticsearch中只存储最后被索引的任何文档。如果其他人同时也修改了这个文档，他们的修改将会丢失。在数据库中，有两种通用的方法确保在并发更新时修改不丢失：  

**悲观并发控制（Pessimistic concurrency control）**
这在关系型数据库中被广泛的使用，假设冲突的更改经常发生，为了解决冲突我们把访问区块化。典型的例子是在读一行数据前锁定这行，然后确保只有加锁的那个线程可以修改这行数据。

**乐观并发控制（Optimistic concurrency control）：**  
被Elasticsearch使用，假设冲突不经常发生，也不区块化访问，然而，如果在读写过程中数据发生了变化，更新操作将失败。这时候由程序决定在失败后如何解决冲突。实际情况中，可以重新尝试更新，刷新数据（重新读取）或者直接反馈给用户。

#### 乐观并发控制
Elasticsearch是分布式的。当文档被创建、更新或删除，文档的新版本会被复制到集群的其它节点。Elasticsearch即是同步的又是异步的，意思是这些复制请求都是平行发送的，并无序(out of sequence)的到达目的地。这就需要一种方法确保老版本的文档永远不会覆盖新的版本。  

上文我们提到 index 、 get 、 delete请求时，我们指出每个文档都有一个 _version 号码，这个号码在文档被改变时加一。Elasticsearch使用这个 _version 保证所有修改都被正确排序。当一个旧版本出现在新版本之后，它会被简单的忽略。  

我们利用 _version 的这一优点确保数据不会因为修改冲突而丢失。我们可以指定文档的 version 来做想要的更改。如果那个版本号不是现在的，我们的请求就失败了。
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/website/blog/1?version=3' -d'
{
    "title": "My first blog entry",
    "text": "Starting to get the hang of this..."
}'
```
我们只希望文档的 _version 是 3 时更新才生效。
```
{
    "_index":"website",
    "_type":"blog",
    "_id":"1",
    "_version":4,
    "result":"updated",
    "_shards":{
        "total":2,
        "successful":2,
        "failed":0
    },
    "_seq_no":5,
    "_primary_term":1
}
```
然而，如果我们重新运行相同的索引请求，依旧指定 version=1 ，Elasticsearch将返回 409 Conflict 状态的HTTP响应。响应体类似这样：
```
{
    "error":{
        "root_cause":[{
            "type":"version_conflict_engine_exception",
            "reason":"[blog][1]: version conflict, current version [4] is different than the one provided [3]",
            "index_uuid":"0CF8ZVEzSKqCvSofbgdQWA",
            "shard":"3",
            "index":"website"
        }],
        "type":"version_conflict_engine_exception",
        "reason":"[blog][1]: version conflict, current version [4] is different than the one provided [3]",
        "index_uuid":"0CF8ZVEzSKqCvSofbgdQWA",
        "shard":"3",
        "index":"website"
    },
    "status":409
}
```
我们需要做什么取决于程序的需求。我们可以告知用户其他人修改了文档，你应该在保存前再看一下。
#### 使用外部版本控制系统
一种常见的结构是使用一些其他的数据库做为主数据库，然后使用Elasticsearch搜索数据，这意味着所有主数据库发生变化，就要将其拷贝到Elasticsearch中。如果有多个进程负责这些数据的同步，就会遇到上面提到的并发问题。  

如果主数据库有版本字段——或一些类似于 timestamp 等可以用于版本控制的字段——是你就可以在Elasticsearch的查询字符串后面添加 version_type=external 来使用这些版本号。版本号必须是整数，大于零小于 9.2e+18 ——Java中的正的 long 。 

外部版本号与之前说的内部版本号在处理的时候有些不同。它不再检查 _version 是否与请求
中指定的一致，而是检查是否小于指定的版本。如果请求成功，外部版本号就会被存储到 _version 中。  

外部版本号不仅在索引和删除请求中指定，也可以在创建(create)新文档中指定。例如，创建一个包含外部版本号 5 的新博客，我们可以这样做：
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/website/blog/2?version=10&version_type=external&pretty' -d '
{
"title": "My first external blog entry",
"text": "Starting to get the hang of this..."
}'
```
在响应中，我们能看到当前的 _version 号码是 10 ：
```
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "2",
  "_version" : 10,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```
如果你重新运行这个请求，就会返回一个像之前一样的冲突错误，因为指定的外部版本号不大于当前在Elasticsearch中的版本。
```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "version_conflict_engine_exception",
        "reason" : "[blog][2]: version conflict, current version [10] is higher or equal to the one provided [10]",
        "index_uuid" : "0CF8ZVEzSKqCvSofbgdQWA",
        "shard" : "2",
        "index" : "website"
      }
    ],
    "type" : "version_conflict_engine_exception",
    "reason" : "[blog][2]: version conflict, current version [10] is higher or equal to the one provided [10]",
    "index_uuid" : "0CF8ZVEzSKqCvSofbgdQWA",
    "shard" : "2",
    "index" : "website"
  },
  "status" : 409
}
```
### 文档局部更新
最简单的 update 请求表单接受一个局部文档参数 doc ，它会合并到现有文档中——对象合并在一起，存在的标量字段被覆盖，新字段被添加。举个例子，我们可以使用以下请求为博客添加一个 tags 字段和一个 views 字段：
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/blog/1/_update?&pretty' -d '
{
    "doc" : {
        "tags" : [ "testing" ],
        "views": 0
    }
}'
```
#### 使用脚本局部更新

脚本能够使用 update API改变 _source字段的内容，它在脚本内 部以 ctx._source 表示。例如，我们可以使用脚本增加博客的 views 数量：
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/blog/1/_update?pretty' -d '
{
    "script" : "ctx._source.views+=1"
}'
```
我们还可以使用脚本增加一个新标签到 tags数组中。 在这个例子中，我们定义了一个新标签做为参数而不是硬编码在脚本里。这允许Elasticsearch未来可以重复利用脚本，而不是在想要增加新标签时必须每次编译新脚本：
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/blog/1/_update?pretty' -d '
{
    "script":{
        "source":"ctx._source.tags.add(params.new_tag)",
        "params" : {
        "new_tag" : "search"
        }
    }
}'
```
通过设置 ctx.op 为 delete 我们可以根据内容删除文档：
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/blog/1/_update?pretty' -d '
{
    "script": {
        "source" : "ctx.op = ctx._source.views == params.count ? \"delete\" : \"none\"",
        "params" : {
            "count": 1
        }
    }
}'
```
当我们试图更新一个不存在的文档，更新将失败。在这种情况下，我们可以使用 upsert 参数定义文档来使其不存在时被创建。
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/pageviews/4/_update?pretty' -d '
{
    "script" : "ctx._source.views+=1",
    "upsert": {
        "views": 1
    }
}'
/*该结果会报错，在6.0及以上版本一个索引(index)下不允许存在多个类型(type)*/
//以下是报错信息
{
  "error" : {
    "root_cause" : [
      {
        "type" : "remote_transport_exception",
        "reason" : "[es-node-3][172.16.44.31:9300][indices:data/write/update[s]]"
      }
    ],
    "type" : "illegal_argument_exception",
    "reason" : "Rejecting mapping update to [website] as the final mapping would have more than 1 type: [pageviews, blog]"
  },
  "status" : 400
}
```

```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/blog/1/_update?pretty' -d '
{
    "script" : "ctx._source.views+=1",
    "upsert": {
        "views": 1
    }
}'
```
### 检索多个文档
像Elasticsearch一样，检索多个文档依旧非常快。合并多个请求可以避免每个请求单独的网络开销。如果你需要从Elasticsearch中检索多个文档，相对于一个一个的检索，更快的方式是在一个请求中使用multi-get或者 mget API
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/_mget?pretty' -d '
{
    "docs" : [
        {
        "_index" : "website",
        "_type" : "blog",
        "_id" : 2
        },
        {
        "_index" : "website",
        "_type" : "blog",
        "_id" : 1,
        "_source": "views"
        }
    ]
}'
```
如果所有文档具有相同 _index 和 _type ，你可以通过简单的 ids 数组来代替完整
的 docs 数组：
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/blog/_mget?pretty' -d '
{
    "ids" : [ "2", "1" ]
}'

/*或者*/
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/website/blog/_mget?pretty' -d '
{
    "docs" : [
        { "_id" : 2 },
        { "_id" : 1 }
    ]
}'
```
事实上第二个文档不存在并不影响第一个文档的检索。每个文档的检索和报告都是独立的。
> 注意：  
> 
> 尽管前面提到有一个文档没有被找到，但HTTP请求状态码还是 200 。事实上，就算所有文档都找不到，请求也还是返回200 ，原因是 mget 请求本身成功了。如果想知道每个文档是否都成功了，你需要检查 found 标志。

### 更新时的批量操作
bulk API允许我们使用单一请求来实现多个文档的 create 、 index 、 update 或delete。这对索引类似于日志活动这样的数据 流非常有用，它们可以以成百上千的数据为一个批次按序进行索引。  

bulk 请求体如下，它有一点不同寻常：
```
{ action: { metadata }}\n
{ request body }\n
{ action: { metadata }}\n
{ request body }\n
...
```
这种格式类似于用 "\n"符号连接起来的一行一行的JSON文档流 (stream)。 两个重要的点需要注意：
- 每行必须以 "\n"符号结尾，包括最后一行。这些都是作为每行有效的分离而做的标记。
- 每一行的数据不能包含未被转义的换行符，它们会干扰分析——这意味着JSON不能被美化打印

action/metadata这一行定义了文档行为(whataction)发生在哪个文档(which document)之上。  

行为(action)必须是以下几种:
行为| 解释
-|-
create | 当文档不存在时创建之。
index| 创建新文档或替换已有文档。
update| 局部更新文档。
delete| 删除一个文档。
在索引、创建、更新或删除时必须指定文档的 _index 、 _type 、 _id 这些元数据(metadata)。  
例如删除请求看起来像这样：
```
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
```
请求体(request body)由文档的 _source组成——文档所包含的一 些字段以及其值。它被 index 和 create操作所必须，这是有道 理的：你必须提供文档用来索引。  

这些还被 update 操作所必需，而且请求体的组成应该与 update API（ doc , upsert ,script 等等）一致。删除操作不需要请 求体(request body)。
```
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title": "My first blog post" }
/*如果未定义_id，ID将会被自动创建*/
```
为了将这些放在一起， bulk 请求表单是这样的：
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/_bulk?pretty' -d '
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}\n
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}\n
{ "title": "My first blog post" }\n
{ "index": { "_index": "website", "_type": "blog" }}\n
{ "title": "My second blog post" }\n
{ "update": { "_index": "website", "_type": "blog", "_id": "123"}}\n
{ "doc" : {"title" : "My updated blog post"} }\n
'
```
这些说明 bulk 请求不是原子操作——它们不能实现事务。每个请求操作时分开的，所以每个请求的成功与否不干扰其它操作。

## 4.分布式文档存储
### 路由文档到分片
```
shard = hash(routing)% nuber_of_primary_shars
```
routing值是一个任意的字符串，它的默认值是_id但是也可以自定义。routing字符串通过hash函数生成一个数字，然后除以主切片的数量的到一个余数，余数范围永远是0——number_of_primary_shards-1,这个数字就是特定文档所在的分片。  

这也解释了为什么主分片的数量只能在创建索引时定义且不能修改：如果主分片的数量在未来改变了，所有先前的路由值就失效了，文档也就永远找不到了。

## 5.搜索

**搜索可以：**  
- 在类似于 gender 或者 age 这样的字段上使用结构化查询， join_date 这样的字段上使用排序，就像SQL的结构化查询一样。 
- 全文检索，可以使用所有字段来匹配关键字，然后按照关联性(relevance)排序返回结果。
- 或者结合以上两条。
很多搜索都是开箱即用的，为了充分挖掘Elasticsearch的潜力，需要理解三个概念：

概念| 解释
---| ---
映射(MAapping)|数据在每个字段中的解释说明
分析(Analysis)|全文是如何处理得可以被搜索的
领域特定语言查询(Query DSL)| Elasticsearch使用的灵活的请打的查询语言
### 空搜索

#### hits

响应中最重要的部分，它包含了total字段来表示的文档总数，hits数组还包含了匹配到的前10条数据。
#### took

告诉我们花费的整个搜索请求花费的毫秒数

#### shards

告诉我们参与查询分片数(total字段)，有多少时成功的(successful字段)，有多少时失败的(failed字段)。通常我们都不希望分片失败，不过这个是有可能发生的。如果我们遭受一些重大的故障导致主分片和复制分片都故障，那这个分派你的数据都将无法响应给搜索请求。这种情况下,Elasticsearch将报告分片failed，但仍将继续返回剩余分片的结果。
#### timeout

告诉我们查询是否超时。一般的，搜索请求不会超时。
> **警告**
> 
> 需要注意的是 timeout不会停止执行查询，它仅仅告诉你目前顺利返回结果的节点然后关闭连接。在后台，其他分片可能依旧执行查询，尽管结果已经被发送。 
> 
> 使用超时是因为对于你的业务需求（译者注：SLA，Service-Level Agreement服务等级协议，在此我翻 译为业务需求）来说非常 重要，而不是因为你想中断执行长时间运行的查询。

### 多索引和多类别

```
通常，当然，你可能想搜索一个或几个自定的索引或类型，我们能通过定义URL中的索引或
类型达到这个目的，像这样：
/_search
在所有索引的所有类型中搜索
/gb/_search
在索引 gb 的所有类型中搜索
/gb,us/_search
在索引 gb 和 us 的所有类型中搜索
/g*,u*/_search
在以 g 或 u 开头的索引的所有类型中搜索
/gb/user/_search
在索引 gb 的类型 user 中搜索
/gb,us/user,tweet/_search
在索引 gb 和 us 的类型为 user 和 tweet 中搜索
/_all/user,tweet/_search
```

### 分页
和SQL使用 LIMIT关键字返回只有一页的结果一样，Elasticsearch接受 from 和 size 参数：  
- size : 结果数，默认 10
- from : 跳过开始的结果数，默认 0  如果你想每页显示5个结果，页码从1到3，那请求如下：
```
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```
## 6.映射

### 映射详解
Elasticsearch 在对megacrop索引中对employee类型进行mapping后如何解读我们的文档结构：
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/_mapping/employee'
```
analyze 语法在5.0后进行更新，被分析的文本放到text中，使用：
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/_analyze?pretty' -d '
{                       
"text" : "text to analyze"
}'
```
**更新映射**  
你可以在第一次创建索引的时候指定映射的类型。此外，你也可以晚些时候为新类型添加映射（或者为已有的类型更新映射）。
> <font size=4>重要</font>
>  
> 你可以向已有映射中增加字段，但你不能修改它。如果一个字段在映射中已经存在，这可能意味着那个字段的数据已经被索引。如果你改变了字段映射，那已经被索引的数据将错误并且不能被正确的搜索到。

然后创建一个新索引，指定 tweet 字段的分析器为 english ：
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/gb' -d '
{
    "mappings": {
        "tweet" : {
            "properties" : {
                "tweet" : {
                    "type" : "text",
                    "analyzer": "english"
                },
                "date" : {
                    "type" : "date"
                },
                "name" : {
                    "type" : "text"
                },
                "user_id" : {
                    "type" : "long"
                }
            }
        }
    }
}'
```
再后来，我们决定在 tweet 的映射中增加一个新的 not_analyzed 类型的文本字段，叫做 tag ，使用 _mapping 后缀:
```
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/gb/_mapping/tweet' -d '
{
    "properties" : {
        "tag" : {
            "type" : "keyword",
            "index": true
        }
    }
}'
```
**测试映射**  
你可以通过名字使用 analyzeAPI测试字符串字段的映 射。对比这两个请求的输出：
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/gb/_analyze?pretty' -d '
{
    "field":"tweet",
    "text":"black-cats"
}'

curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/gb/_analyze?pretty' -d '
{
    "field":"tag",
    "text":"black-cats"
}'
/*tweet 字段产生两个词， "black" 和 "cat" , tag 字段产生单独的一个词 "Black-cats" 。换
言之，我们的映射工作正常*/
```
### 复合类型
**多值字段**  
数组中所有值必须为同一类型。你不能把日期和字符窜混合。如果你创建一个新字段，这个字段索引了一个数组，Elasticsearch将使用第一个值的类型来确定这个新字段的类型。
> 当你从Elasticsearch中取回一个文档，任何一个数组的顺序和你索引它们的顺序一致。你取回的 _source 字段的顺序同样与索引它们的顺序相同。
>
> 然而，数组是做为多值字段被索引的，它们没有顺序。在搜索阶段你不能指定“第一个值”或者“最后一个值”。倒不如把数组当作一个值集合(bag of values)

**空字段**  
当然数组可以是空的。这等价于有零个值。事实上，Lucene没法存放 null 值，所以一个 null 值的字段被认为是空字段。

这四个字段将被识别为空字段而不被索引：
```
"empty_string": "",
"null_value": null,
"empty_array": [],
"array_with_null_value": [ null ]
```
## 7.结构化查询
### 最重要的查询过滤语句
**term 过滤**  
term 主要用于精确匹配哪些值，比如数字，日期，布尔值或 not_analyzed 的字符串(未经分析的文本数据类型)：
```
{ "term": { "age": 26 }}
{ "term": { "date": "2014-09-01" }}
{ "term": { "public": true }}
{ "term": { "tag": "full_text" }}
```

**terms 过滤**  
terms 跟 term 有点类似，但 terms允许指定多个匹配条件。 如果某个字段指定了多个值，那么文档需要一起去做匹配：
```
{
    "terms": {
        "tag": [ "search", "full_text", "nosql" ]
    }
}
```

**range 过滤**  
range 过滤允许我们按照指定范围查找一批数据：
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search?pretty' -d '
{
    "query":{
        "bool":{
            "filter":{
                "range": {
                    "age": {
                    "gte": 20,
                    "lt": 30
                    }
                }
            }
        }
    }
}'
/*范围操作符包含：
gt :: 大于
gte :: 大于等于
lt :: 小于
lte :: 小于等于*/
```

**exists 和 missing 过滤**  
exists 和 missing 过滤可以用于查找文档中是否包含指定字段或没有某个字段，类似于SQL语句中的 IS_NULL 条件
```
{
    "exists": {
        "field": "title"
    }
}
/*这两个过滤只是针对已经查出一批数据来，但是想区分出某个字段是否存在的时候使用*/
```

**bool 过滤**  
bool 过滤可以用来合并多个过滤条件查询结果的布尔逻辑，它包含一下操作符：  

must :: 多个查询条件的完全匹配,相当于 and 。  
must_not :: 多个查询条件的相反匹配，相当于 not 。  
should :: 至少有一个查询条件匹配, 相当于 or 。  

这些参数可以分别继承一个过滤条件或者一个过滤条件的数组：
```
{
    "bool": {
        "must": { "term": { "folder": "inbox" }},
        "must_not": { "term": { "tag": "spam" }},
        "should": [
            { "term": { "starred": true }},
            { "term": { "unread": true }}
        ]
    }
}
```

**matchall 查询**  
使用 match_all可以查询到所有文档，是没有查询条件下的默认语句。
```
{
    "match_all": {}
}
```
此查询常用于合并过滤条件。比如说你需要检索所有的邮箱,所有的文档相关性都是相同的，所以得到的 _score 为1

**match 查询**  
match 查询是一个标准查询，不管你需要全文本查询还是精确查询基本上都要用到它。  
如果你使用 match查询一个全文本字段，它会在真正查询之前用分析器先分析 match 一下查询字符：
```
{
    "match": {
        "tweet": "About Search"
    }
}
```
如果用 match 下指定了一个确切值，在遇到数字，日期，布尔值或者 not_analyzed 的字符串时，它将为你搜索你给定的值：  
```
{ "match": { "age": 26 }}
{ "match": { "date": "2014-09-01" }}
{ "match": { "public": true }}
{ "match": { "tag": "full_text" }}
```
> 提示： 做精确匹配搜索时，你最好用过滤语句，因为过滤语句可以缓存数据。

**multi_match 查询**  
multi_match 查询允许你做 match 查询的基础上同时搜索多个字段：
```
{
    "multi_match": {
        "query": "full text search",
        "fields": [ "title", "body" ]
    }
}
```

**bool 查询**  
bool 查询与 bool 过滤相似，用于合并多个查询子句。不同的是， bool 过滤可以直接给出是否匹配成功， 而 bool 查询要计算每一个查询子句的 _score （相关性分值）。 

must :: 查询指定文档一定要被包含。  
must_not :: 查询指定文档一定不要被包含。  
should :: 查询指定文档，有则可以为文档相关性加分。

以下查询将会找到 title 字段中包含 "how to make millions"，并且 "tag" 字段没有被标为spam 。 如果有标识为 "starred" 或者发布日期为2014年之前，那么这些匹配的文档将比同类
网站等级高：
```
{
    "bool": {
        "must": { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag": "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```
> 提示： 如果 bool 查询下没有 must 子句，那至少应该有一个 should 子句。但是如果有 must 子句，那么没有 should 子句也可以进行查询。

```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search?pretty' -d '
{
    "query":{
        "bool":{
            "must":{"match":{"about":"love"}},
            "must_not":{"match":{"first_name":"Jane"}}
    }   
    }
}'
```

**查询与过滤条件的合并**  
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search?pretty' -d '
{
    "query":{
        "bool": {
            "must": {
                "match": {
                    "about": "love"
                }
            },
            "filter": { 
                "term": { 
                "age": 25 
                }
            }
        }
    }
}'
```

**验证查询**  
查询语句可以变得非常复杂，特别是与不同的分析器和字段映射相结合后，就会有些难度。validate API 可以验证一条查询语句是否合法。
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_validate/query?pretty' -d '
{
    "query":{
        "bool": {
            "must": {
                "match": {
                    "about": "love"
                }
            },
            "filter": { 
                "term": { 
                "age": 25 
                }
            }
        }
    }
}'
```

## 8.排序

### 相关性排序
**字段值排序**  
字段值默认以顺序排列，而 score 默认以倒序排列。
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search?pretty' -d '
{
    "sort" : {"age":{"order":"desc"}}
}'
/*默认排序写法：{"sort":"age"}*/
```
**多级排序**  
如果我们想要合并一个查询语句，并且展示所有匹配的结果集使用第一排序是 age ，第二排序是 _score ：
```
curl -H "Content-Type:application/json" -XGET 'http://172.16.44.29:9200/megacorp/employee/_search?pretty' -d '
{
    "sort" :[
    {"age":{"order":"desc"}},
    {"_score":{ "order": "desc" }}
    ]
}'
```
### 字符串排序
**为多值字段字符串排序**   
在 _source 下相同的字符串上排序两次会造成不必要的资源浪费。 而我们想要的是同一个字段中同时包含这两种索引方式，我们只需要改变索引(index)的mapping即可。 方法是在所有核心字段类型上，使用通用参数 fields 对mapping进行修改。 比如，我们原有mapping如下：
```
"tweet": {
"type": "text",
"analyzer": "english"
}
```
改变后的多值字段mapping如下：
```
    "tweet": { <1>
        "type": "text",
        "analyzer": "english",
        "fields": {
            "raw": { <2>
            "type": "keyword",
            "index": true
            }
        }
    }
    /*修改已经确定的映射*/
curl -H "Content-Type:application/json" -XPUT 'http://172.16.44.29:9200/gb/_mapping/tweet?pretty' -d '
{
    "properties" : {
        "tweet" : {
            "type" : "text",
            "analyzer": "english",
            "fields":{
                "raw": { 
                "type": "keyword",
                "index": true
                }
            }
        }
    }
}'
```
<1> tweet 字段用于全文本的 analyzed 索引方式不变。  
<2> 新增的 tweet.raw 子字段索引方式是 not_analyzed 。  

现在，在给数据重建索引后，我们既可以使用 tweet 字段进行全文本搜索，也可以用 tweet.raw 字段进行排序：
```
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    },
    "sort": "tweet.raw"
}
```

**数据字段**  
当你对一个字段进行排序时，ElasticSearch 需要进入每个匹配到的文档得到相关的值。 倒排索引在用于搜索时是非常卓越的，但却不是理想的排序结构。
- 当搜索的时候，我们需要用检索词去遍历所有的文档。
- 当排序的时候，我们需要遍历文档中所有的值，我们需要做反倒序排列操作。  

为了提高排序效率，ElasticSearch会将所有字段的值加载到内存中，这就叫做"数据字段"。
> 重要： ElasticSearch将所有字段数据加载到内存中并不是匹配到的那部分数据。 而是索引下所有文档中的值，包括所有类型。

ElasticSearch中的字段数据常被应用到以下场景：
- 对一个字段进行排序
- 对一个字段进行聚合
- 某些过滤，比如地理位置过滤
- 某些与字段相关的脚本计算  

毫无疑问，这会消耗掉很多内存，尤其是大量的字符串数据 -- string字段可能包含很多不同的值。值得庆幸的是，内存不足是可以通过横向扩展解决的，我们可以增加更多的节点到集群。
## 9.分布式搜索

**搜索选项**  
preference（偏爱）  
preference 参数允许你控制使用哪个分片或节点来处理搜索请求。她接受如下一些参数_primary ， _primary_first ， _local ， _only_node:xyz ，_prefer_node:xyz 和 _shards:2,3 。这些参数在文档搜索偏好（search preference）里有详细描述。  

然而通常最有用的值是一些随机字符串，它们可以避免结果震荡问题（the bouncing resultsproblem）。

*结果震荡（Bouncing Results）*  
- 想像一下，你正在按照 timestamp 字段来对你的结果排序，并且有两个document有相同的timestamp。由于搜索请求是在所有有效的分片副本间轮询的，这两个document可能在原始分片里是一种顺序，在副本分片里是另一种顺序。
- 这就是被称为结果震荡（bouncing results）的问题：用户每次刷新页面，结果顺序会发生变化。避免这个问题方法是对于同一个用户总是使用同一个分片。方法就是使用一个随机字符串例如用户的会话ID（session ID）来设置 preference 参数。  


**search_type（搜索类型）**  

*count（计数）*  
搜索类型只有一个 query（查询） 的阶段。当不需要搜索结果只需要知道满足查询的document的数量时，可以使用这个查询类型。  
search_type = count被size = 0 代替：
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/_search?pretty' -d '{
"query":{"match_all":{}},
"size":0  
}'
```

*query_and_fetch（查询并且取回）*  
query_and_fetch（查询并且取回） 搜索类型将查询和取回阶段合并成一个步骤。这是一个内部优化选项，当搜索请求的目标只是一个分片时可以使用。  

*dfs_query_then_fetch 和 dfs_query_and_fetch*  
dfs 搜索类型有一个预查询的阶段，它会从全部相关的分片里取回项目频数来计算全局的项目频数。

**scroll(滚屏)**  
滚屏可以用来解决深分页带来的内存大量消耗的问题
```
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/_search?scroll=4m&pretty' -d '{
"query":{"match_all":{}},
"size":3                                            }'
/*返回结果中存在一个_scroll_id,利用这个可以查询下一滚屏的信息*/
curl -H "Content-Type:application/json" -XPOST 'http://172.16.44.29:9200/_search/scroll?pretty' -d '{
"scroll":"4m",
"scroll_id":"DnF1ZXJ5VGhlbkZldGNoEgAAAAAAAAA9FlVDaE5UVkg3Uk9td2xBaDQzbUdsSGcAAAAAAAAAPBZmdExhUlVFNlRfS1VCNktCUXVKcVFnAAAAAAAAADsWZnRMYVJVRTZUX0tVQjZLQlF1SnFRZwAAAAAAAAA-FlVDaE5UVkg3Uk9td2xBaDQzbUdsSGcAAAAAAAAAPxZMTFAxanVSRlEzcUJUNlVjb2FVbE93AAAAAAAAAEIWTExQMWp1UkZRM3FCVDZVY29hVWxPdwAAAAAAAABBFkxMUDFqdVJGUTNxQlQ2VWNvYVVsT3cAAAAAAAAAPxZVQ2hOVFZIN1JPbXdsQWg0M21HbEhnAAAAAAAAAEAWTExQMWp1UkZRM3FCVDZVY29hVWxPdwAAAAAAAAA9FmZ0TGFSVUU2VF9LVUI2S0JRdUpxUWcAAAAAAAAAQhZVQ2hOVFZIN1JPbXdsQWg0M21HbEhnAAAAAAAAAEEWVUNoTlRWSDdST213bEFoNDNtR2xIZwAAAAAAAABAFlVDaE5UVkg3Uk9td2xBaDQzbUdsSGcAAAAAAAAAQxZMTFAxanVSRlEzcUJUNlVjb2FVbE93AAAAAAAAAD4WZnRMYVJVRTZUX0tVQjZLQlF1SnFRZwAAAAAAAABAFmZ0TGFSVUU2VF9LVUI2S0JRdUpxUWcAAAAAAAAAPxZmdExhUlVFNlRfS1VCNktCUXVKcVFnAAAAAAAAAEEWZnRMYVJVRTZUX0tVQjZLQlF1SnFRZw=="
}'
```
## 10.索引管理

### 根对象
映射的最高一层被称为 根对象，它可能包含下面几项：
- 一个 properties 节点，列出了文档中可能包含的每个字段的映射多个元数据字段，每一个都以下划线开头，例如 _type , _id 和 _source
- 设置项，控制如何动态处理新的字段，例如 analyzer , dynamic_date_formats 和dynamic_templates 。
- 其他设置，可以同时应用在根对象和其他 object 类型的字段上，例如 enabled ,dynamic 和 include_in_all  

**元数据：_source 字段**  
默认情况下，Elasticsearch 用 JSON 字符串来表示文档主 体保存在 _source 字段中。像其他保存的字段一样， _source 字 段也会在写入硬盘前压缩。  

这几乎始终是需要的功能，因为：
- 搜索结果中能得到完整的文档 —— 不需要额外去别的数据 源中查询文档
- 如果缺少_source字段，部分更新请求不会起作用
- 当你的映射有变化，而且你需要重新索引数据时，你可以直接在Elasticsearch中操作而不需要重新从别的数据源中取回数据。
- 你可以从_source中通过get或search请求取回部分字段，而不是整个文档。
- 这样更容易排查错误，因为你可以准确的看到每个文档中包含的内容，而不是只能从一堆 ID 中猜测他们的内容。  

即便如此，存储_source字段还是要占用硬盘空间的。假如上面的理由对你来说不重要，你可以用下面的映射禁用 _source 字段：
```
PUT /my_index
{
    "mappings": {
        "my_type": {
            "_source": {
                "enabled": false
            }
        }
    }
}
```
### 动态映射
幸运的是，你可以通过 dynamic 设置来控制这些行为，它接受下面几个选项：
- true ：自动添加字段（默认）
- false ：忽略字段
- strict ：当遇到未知字段时抛出异常


dynamic 设置可以用在根对象或任何object对象上。你可以将dynamic默认设置为strict，而在特定内部对象上启用它：
```
PUT /my_index
{
    "mappings": {
        "my_type": {
            "dynamic": "strict",            //设置了strict，添加字段报错
            "properties": {
                "title": {
                    "type": "text"
                },
                "address": {
                    "type": "object",
                    "dynamic": "true"           //设置为true可以添加字段
                }
            }
        }
    }
}
 
PUT /my_index/my_type/1
{
    "title": "my article",
    "content": "this is my article",        //添加新的字段
    "address": {
        "province": "guangdong",
        "city": "guangzhou"
    }
}
 
//报错
{
    "error": {
        "root_cause": [
            {
                "type": "strict_dynamic_mapping_exception",
                "reason": "mapping set to strict, dynamic introduction of [content] within [my_type] is not allowed"
            }
        ],
         "type": "strict_dynamic_mapping_exception",
        "reason": "mapping set to strict, dynamic introduction of [content] within [my_type] is not allowed"
    },
    "status": 400
}
 
 
PUT /my_index/my_type/1
{
    "title": "my article",
    "address": {
         "province": "guangdong",
         "city": "guangzhou"        //添加新的字段
    }
}

//不报错
GET /my_index/_mapping/my_type
 
{
  "my_index": {
    "mappings": {
      "my_type": {
        "dynamic": "strict",
        "properties": {
          "address": {
            "dynamic": "true",
            "properties": {
              "city": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              },
              "province": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              }
            }
          },
          "title": {
            "type": "text"
          }
        }
      }
    }
  }
}
```
### 索引别名
索引别名就像一个快捷方式或软连接，可以指向一个或多个索引，也可以给任何需要索引名的API使用。别名带给我们极大的灵活性，允许我们做到：
- 在一个运行的集群上无缝的从一个索引切换到另一个
- 给多个索引分类（例如， last_three_months ）
- 给索引的一个子集创建视图  

这里有两种管理别名的途径：_alias用于单个操作，_aliases用于原子化多个操作。  
开始，我们创建一个索引 my_index_v1 ，然后将别名 my_index 指向它：
```
PUT /my_index_v1 <1>
PUT /my_index_v1/_alias/my_index <2>
```
<1> 创建索引 my_index_v1 。  
<2> 将别名 my_index 指向 my_index_v1 。
你可以检测这个别名指向哪个索引：
```
GET /*/_alias/my_index
```
或哪些别名指向这个索引：
```
GET /my_index_v1/_alias/*
```
别名可以指向多个索引，所以我们需要在新索引中添加别名的同时从旧索引中删除它。这个操作需要原子化，所以我们需要用_aliases操作:
```
POST /_aliases
{
    "actions": [
        { "remove": { "index": "website", "alias": "my_index" }},
        { "add": { "index": "megacorp", "alias": "my_index" }}
    ]
}
```
这样，你的应用就从旧索引迁移到了新的，而没有停机时间。

> 提示：
即使你认为现在的索引设计已经是完美的了，当你的应用在生产环境使用时，还是有可能在今后有一些改变的。所以请做好准备：在应用中使用别名而不是索引。然后你就可以在任何时候重建索引。别名的开销很小，应当广泛使用

## 11.深入分片

**不可变性**  
写入磁盘的倒排索引是不可变的，它有如下好处：
- 不需要锁。如果从来不需要更新一个索引，就不必担心多个程序同时尝试修改。
- 一旦索引被读入文件系统的缓存(译者:在内存)，它就一直在那儿，因为不会改变。只要文件系统缓存有足够的空间，大部分的读会直接访问内存而不是磁盘。这有助于性能提升。
- 在索引的声明周期内，所有的其他缓存都可用。它们不需要在每次数据变化了都重建，因为数据不会变。
- 写入单个大的倒排索引，可以压缩数据，较少磁盘IO和需要缓存索引的内存大小。  

## 结构化查询
### 组合过滤
**bool过滤器**  

```
curl -H "Content-type:application/json" -XGET 'http://172.16.44.30:9400/_search?pretty' -d '{
    "query":{
        "bool":{
            "should":[
            {"term":{"age":20}},
            {"term":{"age":30}}
            ]
        }
    }
}'
```