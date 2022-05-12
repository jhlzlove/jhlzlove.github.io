---
title: ElasticSearch + Kibana总结
categories:
  - Java
  - ELK
music:
  server: netease
  type: song
  id: 430793674
abbrlink: b279c45e
---

ES的内部使用了Restful API，并且使用的 JSON 语法，对程序员友好；Kibana提供了友好的客户端操作页面，具有相关的语法提示、图表分析等强大的功能。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=true} -->

<!-- code_chunk_output -->

- [ElasticSearch分布式搜索引擎的认识](#elasticsearch分布式搜索引擎的认识)
- [安装配置](#安装配置)
  - [1. ES](#1-es)
    - [插曲：ES启动可能出现的问题](#插曲es启动可能出现的问题)
  - [2. Kibana](#2-kibana)
- [ES基本使用](#es基本使用)
  - [1. 核心概念](#1-核心概念)
  - [2. 索引操作](#2-索引操作)
  - [3. 映射操作](#3-映射操作)
  - [4. 文档操作](#4-文档操作)
- [高级查询](#高级查询)
  - [1. 关键字查询\<term>](#1-关键字查询term)
  - [2. 范围查询\<range>](#2-范围查询range)
  - [3. 前缀查询\<prefix>](#3-前缀查询prefix)
  - [4. 通配符查询\<wildcard>](#4-通配符查询wildcard)
  - [5. 多id查询\<ids>](#5-多id查询ids)
  - [6. 模糊查询\<fuzzy>](#6-模糊查询fuzzy)
  - [7. 布尔查询\<bool>](#7-布尔查询bool)
  - [8. 多字段查询<multi_match>](#8-多字段查询multi_match)
  - [9. 默认字段分词查询<query_string>](#9-默认字段分词查询query_string)
  - [10. 高亮查询\<highlight>](#10-高亮查询highlight)
  - [11. 返回指定条数\<size>](#11-返回指定条数size)
  - [12. 分页查询\<form>](#12-分页查询form)
  - [13. 排序\<sort>](#13-排序sort)
  - [14. 返回指定字段<_source>](#14-返回指定字段_source)
- [分词器](#分词器)
  - [1. ES 内置分词器](#1-es-内置分词器)
  - [2. 中文分词器](#2-中文分词器)
  - [3. 扩展词、停用词配置](#3-扩展词停用词配置)
  - [4. 过滤查询](#4-过滤查询)
- [SpringBoot整合ES开发](#springboot整合es开发)
  - [1. ElasticsearchOperations](#1-elasticsearchoperations)
    - [相关注解](#相关注解)
  - [2. RestHighLevelClient](#2-resthighlevelclient)
- [ES 集群搭建](#es-集群搭建)
  - [Head插件查看ES状态](#head插件查看es状态)

<!-- /code_chunk_output -->

## ElasticSearch分布式搜索引擎的认识

ES中的默认使用的是标准分词器(StandardAnalyzer)：中文使用单字分词；英文使用单词分词。
ES中只有 text 类型是分词的，剩下的 keyword、integer、date 等类型都是不分词的。
版本异同：
> es5 一个索引可以创建多个类型，在es6中仍可以使用,但是已经不推荐。
> es6 之后一个索引只对应一个类型。
> es7 的默认分片(即备份)从之前的5调到了1,如果想用原来的可以自己设置；
> es7 使用jdk11+，已经内置。不过可能会出现使用本地的jdk的情况，如果本地jdk版本低于11，需要配置。
> 编辑环境变量：`vim /etc/profile`，添加变量 `export ES_JAVA_HOME=指定ES安装目录中的jdk`，然后 `source /etc/profile`
> 类型已经在新版本中删除，7之后的版本不在有类型。

## 安装配置

注意，kibana 必须和 ES 同版本。这里以安装的 7.14 版本为例，安装方式都是解压 `.tar.gz` 包的方式，docker 方式安装比较简单。

ES 默认 web 端口：9200；tcp端口：9300(集群通信)；kibana默认端口：5601

启动 Kibana 之前确保 ES 服务正常启动。

### 1. ES

ES 不能以 root 用户启动，否则会报运行时异常，请使用普通用户启动 ES 服务；首先关闭防火墙。之后修改安装目录下的 config 文件夹内的 `elasticsearch.yml` 文件,修改以下内容：

```yml{.line-numbers}
# 开启远程连接
network.host: 0.0.0.0
# 使用一个节点初始化集群
cluster.initial_master_nodes: ["node-1"]
```

设置完成以后可以启动服务；进入安装的 bin 目录，两个命令二选一，如果是直接启动的，操作时可以再打开一个会话窗口。

```bash{.line-numbers}
# 直接启动
./elasticsearch
# 后台启动 以守护进程启动
./elasticsearch -d
```

#### 插曲：ES启动可能出现的问题

另外，一般在启动会出现默认配置较小的错误，我们需要更改系统文件。启动失败时 es 服务会在终端给出建议的大小，我们去修改即可：

1. 以 root 用户修改文件 `/etc/security/limits.conf` ，在最后加入以下内容（CentOS系统需要。其它的系统可跳过）

    ```{.line-numbers}
    *       soft	nofile      65536
    *       hard	nofile      65536
    *       soft	nproc       4096
    *       hard	nproc       4096
    ```

2. 编辑文件 `sudo vim /etc/sysctl.conf`，如果没有该文件可以手动创建，加入内容：`vm.max_map_count=262144`。然后使用 `sysctl -p` 使其生效并查看输出内容是否和自己设置的一样。

### 2. Kibana

同样是修改 kibana 安装目录下的 `config/kibana.yml` 文件，修改以下内容：

```yml{.line-numbers}
# 开启远程连接
server.host: 0.0.0.0
# 监控 ES 的地址信息
elasticsearch.hosts: [es的ip地址:端口号]
```

## ES基本使用

### 1. 核心概念

1. 索引（Index）：一个索引就是一个拥有几分相似特征的文档的集合（类似MySQL的库的概念），**ES索引的名称必须是小写**。
2. 映射（Mapping）：映射是定义一个文档和它所包含的字段如何被存储和索引的过程。可以简单认为映射就是常规数据库的字段信息。在默认配置下，ES可以根据插入的数据自动地创建mapping，也可以手动创建mapping。mapping中主要包括字段名、字段类型等。
3. 文档（Document）：文档是索引中存储的一条条数据。一条文档是一个可被索引的最小单元。ES的文档采用了轻量级的JSON格式数据来表示。

### 2. 索引操作

```json{.line-numbers}
// 查看es中的索引
GET /_cat/indices?v
// 创建索引
PUT /index_name
// 删除索引
DELETE /index_name
```

> 谨慎使用 `delete /*` 的操作，删除掉会导致服务无法正常访问，需要重启服务。
> 查看索引有 health 标题，包含三种属性：黄色代表可用，但是有危险；绿色代表健康，可用；红色代表索引不可用。
> 索引没有修改操作。

### 3. 映射操作

一般都是在创建索引的时候创建映射，脱离了索引，映射也就没有意义了。创建映射时不需要指定类型的长度。

```json{.line-numbers}
// 创建映射，一般在创建索引时手动创建映射
PUT /index_name {
    // 指定分片信息，可以不写，使用默认
    "settings":{
        // 分片为1
        "number_of_shards": 1,
        // 副本为0
        "number_of_replicas": 0
    },
    "mappings":{
        // 默认字段，映射属性信息必须写在这里面
        "properties":{
            // 属性名称及其类型
            "id": {
                "type": "integer"
            },
            "title": {
                "type": "keyword"
            },
            "price": {
                "type": "double"
            },
            "created_at": {
                "type": "date"
            },
            "desc": {
                "type": "text"
            },
        }
    },
}

// 查看映射信息
GET /index_name/_mapping
```

> 映射信息不能删除和修改。

### 4. 文档操作

文档操作，插入一条文档 put /索引/类型/id（指定Id用put，让系统自动创建用post）

```json{.line-numbers}
// 添加文档 手动指定 id
POST /index_name/_doc/1{
    "id": 1,
    "title": "风中捉刀",
    "price": 0.5,
    "created_at": "2021-12-12",
    "description": "天元轮魁"
}
// 添加文档 自动创建文档 id：取UUID的一部分
POST /index_name/_doc/{
    "title": "无情葬月",
    "price": 0.5,
    "created_at": "2021-10-12",
    "description": "血不染"
}

// 文档查询 基于 id 查询
GET /index_name/_doc/document_id

// 删除文档 基于 id 删除
DELETE /index_name/_doc/document_id

// 更新文档 先删除再添加，不保留其它数据
PUT /index_name/_doc/document_id
{
    "filed_name": "修改值"
}
// 更新文档 保留原始内容
PUT /index_name/_doc/document_id/_update
{
    // doc为默认字段，必填
    "doc": {
        "filed_name": "target_value"
    }
}
```

> es7 在添加文档时必须在索引名称后面加上 `_doc`，

批量操作:_bulk(批量操作)添加、修改、删除（弱化事务甚至没有事务）

```json{.line-numbers}
POST /index_name/_doc/_bulk
{
    // 手动添加id
    "index": {"_id": 2}
}
{
    "id": 2,
    "title": "风中捉刀",
    "price": 0.5,
    "created_at": "2021-12-12",
    "description": "天元轮魁"
}
{"index": {"_id": 3}}
{
    "id": 3,
    "title": "玲珑雪霏",
    "price": 0.5,
    "created_at": "2021-12-12",
    "description": "天元轮魁"
}

// 文档批量操作 添加 更新 删除
POST /index_name/_doc/_bulk
{"index": {"_id": 4}}
 {"id": 4,"title": "风中捉刀","price": 0.5,"created_at":"2021-12-12","description": "天元轮魁"}
{"update": {"_id": 3}}
 {"doc": {"title": "荻花题叶"}}
{"delete": {"_id": 2}}
```

> 如果添加报错，那么信息需要放到同一行。

## 高级查询

ES官方提供两种检索方式，queryString(查询参数),queryDSl(特定领域查询);官方更推荐使用第二种，简洁强大。
Query DSL是利于Restful API传递JSON格式的请求体数据与ES进行交互。

### 1. 关键字查询\<term>

```json{.line-numbers}
// 查询所有 match_all,_doc可以不写，这样在Kibana中会有语法提示
GET /index_name/_doc/_search
{
    "query": {
        "match_all": {}
    }
}

// 关键词查询 term
// keyword 类型需要输入全部内容搜索
// text 类型需要输入单个词/字搜索
GET /index_name/_search
{
    "query":{
        "term":{
            // 搜索 description 关键词
            "description":{
                // 关键词包含的值
                "value":"天元"
            }
        }
    }
}

// 查询 keyword 类型，全部输入
GET /index_name/_search
{
    "query":{
        "term":{
            "title":{
                "value":"风中捉刀"
            }
        }
    }
}
```

> 由于ES默认使用的是标准分词器：英文单词分词，中文单字分词。所以description 只能输入单字搜索。
> keyword 类型不分词，所以必须输入全部内容才能精准查询。
> ES 中，除了 text 类型之外的其它类型都不分词。
> 对于中文分词器，我们一般会使用其它的分词器。

### 2. 范围查询\<range>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "range":{
            // 根据price查询范围在 0=<price<=5 的数据
            "price": {
                "gte":0,
                "lte":5
            }
        }
    }
}
```

### 3. 前缀查询\<prefix>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "prefix":{
            "title": {
                "value":""
            }
        }
    }
}
```

### 4. 通配符查询\<wildcard>

?：匹配一个
*：匹配多个

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "wildcard":{
            "description": {
                "value":"go*"
            }
        }
    }
}
```

### 5. 多id查询\<ids>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "ids":{
            // 多个文档id
            "values": [1, 2, 4]
        }
    }
}
```

### 6. 模糊查询\<fuzzy>

最多允许 0-2 次模糊(至多模糊匹配 0 ~ 2 个字符)

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "fuzzy":{
            // 也可以搜索到 风中捉刀
            "description": "风中捉枪"
            // 搜索不到 风中捉刀
            // "description": "风中捉西瓜"
        }
    }
}
```

> 搜索关键词小于等于2，不允许模糊。
> 搜索关键词长度为3-5，只允许一次模糊。
> 搜索关键词长度大于5，最多允许2次模糊。

### 7. 布尔查询\<bool>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "bool":{
            // 以下条件必须都满足才能查询到数据
            "must":{
                "ids": {
                    "values":[1]
                },
                {"term":{
                    "title":{
                        "value": "风中捉刀"
                        }
                    }
                }
            }
        }
    }
}
```

> must 必须满足才能查询，must_not 都不满足才能查询。

### 8. 多字段查询<multi_match>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "multi_match":{
            "query": "风中捉刀",
            "fields": ["title", "description"]
        }
    }
}
```

> 字段类型分词，将查询条件分词之后进行查询，如果不分词，就讲查询条件作为整体进行查询。

### 9. 默认字段分词查询<query_string>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "query_string":{
            "default_field": "description",
            "query": "可叹，落叶飘零"
        }
    }
}
```

### 10. 高亮查询\<highlight>

让符合条件的文档高亮显示。只有能分词的才支持高亮。
注意，查询出来的文档的关键词，ES会对其加入\<em>标签标记，具体高亮样式由我们自己定义。高亮并没有直接修改原始文档，而是放到了另一个标签。

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "query_string":{
            "default_field": "description",
            "query": "可叹，落叶飘零"
        }
    },
    "highlight": {
        "fields": {"*":{}},
        // 自定义样式
        "pre_tags":["<span style='color:red;'>"],
        "post_tags":["</span>"],
        "require_field_match": "false"
    }
}
```

### 11. 返回指定条数\<size>

ES查询数据，默认只显示前10条内容。

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "query_string":{
            "default_field": "description",
            "query": "可叹，落叶飘零"
        }
    },
    "highlight": {
        "fields": {"*":{}},
        // 自定义样式
        "pre_tags":["<span style='color:red;'>"],
        "post_tags":["</span>"],
        "require_field_match": "false"
    },
    // 指定返回条数
    "size":3
}
```

### 12. 分页查询\<form>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "query_string":{
            "default_field": "description",
            "query": "可叹，落叶飘零"
        }
    },
    "highlight": {
        "fields": {"*":{}},
        // 自定义样式
        "pre_tags":["<span style='color:red;'>"],
        "post_tags":["</span>"],
        "require_field_match": "false"
    },
    "from": 2,
    "size": 3
}
```

### 13. 排序\<sort>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "query_string":{
            "default_field": "description",
            "query": "可叹，落叶飘零"
        }
    },
    "highlight": {
        "fields": {"*":{}},
        // 自定义样式
        "pre_tags":["<span style='color:red;'>"],
        "post_tags":["</span>"],
        "require_field_match": "false"
    },
    "from":0,
    "size":3,
    "sort":[
        {
            "price":{
                "order":"desc"
            }
        }
    ]
}
```

> 排序会干预ES的内部的规则。

### 14. 返回指定字段<_source>

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "query_string":{
            "default_field": "description",
            "query": "可叹，落叶飘零"
        }
    },
    "_source":["id", "title", "description"]
}
```

## 分词器

### 1. ES 内置分词器

Standard Analyzer：默认分词器，英文按单词拆分，统一小写处理。
Simple Analyzer：按照单词切分，过滤符号，中文按照空格分词，统一小写处理。
Stop Analyzer：统一小写处理，停用词过滤。
Whitespace Analyzer：按照空格切分，不转小写。
Keyword Analyzer：不分词，直接将输入当作输出。

可以在创建索引的时候为映射字段指定分词器，默认的就是标准分词器（standard）：

```json{.line-numbers}
PUT /index_name
{
    "mapping":{
        "properties":{
            "title":"text",
            "analyzer": "standard|simple|stop|whitespace|keyword"
        }
    }
}
```

### 2. 中文分词器

ES支持的中文分词器有IK、smartCN等，推荐的是 [IK分词器](https://github.com/medcl/elasticsearch-analysis-ik)。

> **注意：** IK分词器的版本要和你安装的ES版本一致。
> Docker插件所在的目录 `/usr/share/elasticsearch/plugins`

IK分词器类型：ik_smart_word(组粒度拆分)、ik_max_word(细粒度拆分)

### 3. 扩展词、停用词配置

定义扩展词典和停用词典可以修改IK分词器中 `config` 目录中 `IKAnalyzer.cfg.xml` 文件。

```xml{.line-numbers}
<properties>
    <comment>扩展词配置</comment>
    <!-- 扩展词典 -->
    <entry key="ext_dict">ext_dict.dic</entry>
    <!-- 停用词典 -->
    <entry key="ext_stopwords">ext_stopwords.dic</entry>
</properties>
```

在ik分词器目录下config目录中创建 `ext_dict.dic`、`ext_stopwords.dic` 文件，文件编码必须为 UTF-8，添加扩展词即可。

### 4. 过滤查询

在ES中，可以使用过滤查询获得更快的查询速度。使用过滤查询，就需要前面使用的布尔查询。

```json{.line-numbers}
GET /index_name/_search
{
    "query":{
        "bool":{
            "must":[
                "term":{
                    "description":{
                        "value":"xxx"
                    }
                }
            ],
            "filter":[
                {
                    // 支持 term terms range exists ids 过滤
                    "term":{
                        "description":""
                    }
                }
            ]
        }
    }
}
```

> 会先执行过滤查询，再执行目标查询。一般用于大数据量的查询。
> 过滤查询是ES中一种重要的优化手段。

## SpringBoot整合ES开发

1. 导入es依赖
2. 配置客户端

    ```java{.line-numbers}
    @Configuration
    public class RestClientConfig extends AbstractElasticsearchConfiguration {
        @Bean
        @Override
        public RestHighLevelClient elasticsearchClient() {
            final ClientConfiguration client = ClientConfiguration.builder()
            .connectedTo("esIP地址:端口")
            .build();
            return RestHighLevelClient.create(client).rest();
        }
    }
    ```

3. 进行客户端操作

    客户端对象有两个，如果进行了上面的配置，那么都会在Spring工厂中创建：
    - ElasticsearchOperations
    - RestHighLevelClient（推荐使用）

### 1. ElasticsearchOperations

特点：始终使用面向对象方式操作ES

- 索引：用来存放相似文档集合
- 映射：决定文档的每个字段以什么方式录入到ES中 字段类型 分词器
- 文档：可以被索引的最小单元 json 数据格式

#### 相关注解

|注解名称|说明|
|--|--|
|@Document(indexName = "index_name", createIndex = true)|用于类，指定索引名,是否创建索引|
|@Id|将放入对象 id 值作为文档 _id 进行映射|
|@Field(type = FieldType.Keyword)|指定字段类型

### 2. RestHighLevelClient

使用 RestHighLevelClient 非常简单，会使用Kibana就可以，它支持使用原生的编写 ES 的语句，只要会。可能这也是大家喜欢使用的原因之一吧。

```java{.line-numbers}
SearchRequest search = new SearchRequest(document);
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(条件);
search.source(sourceBuilder);

SearchResponse searchResponse = restHignLevelClient.search(search, RequestOptions.DEFAULT);
```

## ES 集群搭建

准备三个节点

cluster-|node-1-|config/elasticsearch.yml

```yml{.line-numbers}
# 集群名称，3个节点必须相同
cluster.name: es-cluster
# 指定节点名称
node.name: 
# 开放远程连接
network.host: 0.0.0.0
# 指定使用发布地址进行集群间通信
network.publish_host: 192.168.124.3
# 指定 web 端口
http.port: 9201
# 指定 tcp 端口
transport.tcp.port: 9301
# 指定所有节点的 TCP 通信
discovery.seed_hosts: ["192.168.10.1:9301", "192.168.10.1:9302", "192.168.10.1:9303"]
# 指定可以初始化集群的节点名称
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
# 集群最少几个节点可用
gateway.recover_after_nodes: 2
# 解决跨域问题
http.cors.enabled: true
http.cors.allow-origin: "*"
```

其它节点配置类似，只要修改对应的节点名称、IP以及对应的端口即可。

查看集群状态：`http://任意集群节点ip:端口/_cat/health?v`

### Head插件查看ES状态

我们可以使用 Github 上的大佬开发的查看 ES 状态信息的插件监控服务，该插件基于 js 编写，需要 Node 环境。

```bash{.line-numbers}
git clone https://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head

# 安装依赖
npm install
# 启动
npm run start
open http://localhost:9100/
```
