# ElasticSearch
ElasticSearch 笔记

## 1.1Es简介
ES是使用java 语言并且基于lucence编写的搜索引擎框架，他提供了分布式的全文搜索功能，提供了一个统一的基于restful风格的web 接口。

lucence:一个搜索引擎底层

分布式：突出ES的横向扩展能力

全文检索：将一段词语进行分词，并将分出的词语统一的放在一个分词库中，再搜索时，根据关键字取分词库中检索，找到匹配的内容（倒排索引）。

restful风格的web 接口：只要发送一个http请求，并且根据请求方式的不同，携带参数的不同，执行相应的功能。

应用广泛：WIKI, github,Gold man
## 1.2Es的由来
> 许多年前，一个刚结婚的名叫 Shay Banon 的失业开发者，跟着他的妻子去了伦敦，他的妻子在那里学习厨师。 在寻找一个赚钱的工作的时候，为了给他的妻子做一个食谱搜索引擎，他开始使用 Lucene 的一个早期版本。

> 直接使用 Lucene 是很难的，因此 Shay 开始做一个抽象层，Java 开发者使用它可以很简单的给他们的程序添加搜索功能。 他发布了他的第一个开源项目 Compass。

> 后来 Shay 获得了一份工作，主要是高性能，分布式环境下的内存数据网格。这个对于高性能，实时，分布式搜索引擎的需求尤为突出， 他决定重写 Compass，把它变为一个独立的服务并取名 Elasticsearch。

> 第一个公开版本在2010年2月发布，从此以后，Elasticsearch 已经成为了 Github 上最活跃的项目之一，他拥有超过300名 contributors(目前736名 contributors )。 一家公司已经开始围绕 Elasticsearch 提供商业服务，并开发新的特性，但是，Elasticsearch 将永远开源并对所有人可用。

> 据说，Shay 的妻子还在等着她的食谱搜索引擎…
## 1.3Es和solr对比
> 1.solr 查询死数据，速度比es快。但是数据如果是改变的，solr查询速度会降低很多，ES的查询速度没有明显的改变
> 2. solr搭建集群 依赖ZK，ES本身就支持集群搭建
> 3. 最开始solr的社区很火爆，针对国内文档很少，ES出现后，国内社区火爆程度迅速上升，ES的文档非常健全
> 4. ES对云计算和大数据支持很好
## 1.4倒排索引
![es倒排索引图解](https://github.com/djh-A/ElasticSearch/blob/main/image-20200727144457339.png)
##### 查询步骤
> 1. 将存放的数据以一定的方式进行分词，并将分词的内容存放到一个单独的分词库中
> 2. 当用户取查询数据时，会将用户的查询关键字进行分词，然后去分词库中匹配内容，最终得到数据的id标识
> 3. 根据id标识去存放数据的位置拉去指定数据
## 1.5Es&Kibana安装
> 1. 新建一个存放es的目录：mkdir /usr/local/docker-es,然后新建docker-compose.yml文件，内容如下：
```docker
version: "1"
services: 
    elasticsearch:
        image: daocloud.io/library/elasticsearch:6.5.4
        ports:
            - 9200:9200
        container_name: elasticsearch
        #如果是虚拟机需要加上environment下面三个配置，内存足够则不需要
        environment:
            - "ES_JAVA_OPTS=-Xms64m -Xmx128m"
            - "discovery.type=single-node"
            - "COMPOSE_PROJECT_NAME=elasticsearch-server"
        restart: always
     kibana:
        image: daocloud.io/library/kibana:6.5.4
        ports: 
            - 5601:5601
        container_name: kabana
        restart: always
        environment: 
     #换成自己的ip
            - ELASTICSEARCH_HOSTS=http://127.0.0.1:9200
     
```
> 2. 如果没有安装docker-compose命令，先进行安装 __pip install docker-compose__,然后直接执行**docker-compose up -d** 进行安装
> 3. 浏览器访问自己的ip:9200,能够看到json格式的数据，说明安装成功
> 4. 安装ik分词器
>> 4.1. 地址：<https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.5.4/elasticsearch-analysis-ik-6.5.4.zip>
>> 4.2. 官方的安装方法：进行安装es的容器，执行./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.5.4/elasticsearch-analysis-ik-6.5.4.zip 进行安装  
>> 4.3 分词测试：
``` js
    POST _analyze {
        "analyzer":"ik_max_word",//启用ik分词器
        "text":"我是中国人"
    } 
```

## 2Es的基本操作
> #### 2.1 es的结构：
>> 2.1.1 索引index、切片、备份
>>> ES服务中会创建多个索引  
>>> 每个索引默认被分成5个分片
>>> 每个分片存在至少一个备份分片
>>> 备份分片 不会帮助检索数据(当ES检索压力特别大的时候，备份分片才会帮助检索数据)
>>> 备份的分片必须放在不同的服务器中
>>> ![索引](https://github.com/djh-A/ElasticSearch/blob/main/2139373-20200904160541996-20526323.png)  
>>> 
>> 2.1.2 索引type
>>> ps：一个索引下可以创建多个类型，版本不同类型的创建也不同
>>> ![索引type](https://github.com/djh-A/ElasticSearch/blob/main/2139373-20200904160732060-933479344.png)
>>> 
>> 2.1.3 文档document】
>>> 一个类型下可以有多个文档，类似于mysql表中的多行数据
>>>  ![文档](https://github.com/djh-A/ElasticSearch/blob/main/2139373-20200904160854840-609508977.png)
>> 2.1.3 属性field
>>> 一个文档中可以包含多个字段 类似于mysql行中的多个字段(列)
>>> ![字段](https://github.com/djh-A/ElasticSearch/blob/main/2139373-20200904161054901-739105557.png)
> #### 2.2 es restful语法
>> GET请求：
>>> **http://ip:port/index** : 查询索引信息 
>>> **http://ip:port/index/type/doc_id** : 查询指定文档的信息
>>> 
>> POST请求：
>>> **http://ip:port/type/\_search** : 查询文档，可以在请求体中添加json字符串来代表查询条件
>>>**http://ip:port/type/doc_id/\_update** : 修改文档，可以在请求体中添加json字符串来代表修改条件
>>>
>> PUT请求：
>>> **http://ip:port/index** : 创建一个索引，需要在请求体中指定索引的信息
>>> **http://ip:port/index/type/\_mappings** : 代表创建索引时，指定索引文档存储属性的信息
>>> 
>> DELETE请求：    
>>> **http://ip:port/index** : 删库跑路
>>> **http://ip:port/index/type/doc_id** : 删除指定的文档 

> #### 2.3 es索引的操作
>> 2.3.1 创建一个索引
``` php
    #创建索引
    #number_of_shards 分片数
    #number_of_replicas 备份数
    PUT /person
    {
      "settings": {
        "number_of_replicas": 1,
        "number_of_shards": 5
      }
    }
```
>> 2.3.2 查看索引
>>> 1.Kibana->Management->Index Management·
>>> 2.
``` php
    GET /person
```
>> 2.3.3 删除索引
>>> 1.Kibana->Index Management->Manage idnex->Delete index
>>> 2.
```php
    DELETE /person
```
>> #### 2.4 ES中的field可以指定的类型
>>> 官方文档：<https://www.elastic.co/guide/en/elasticsearch/reference/6.8/mapping-types.html>
>>> 字符串类型：
>>>> text: 一般用于全文检索，将当前field进行分词
>>>> keyword: 当前field不会进行分词
>>>> 
>>> 数值类型：
>>>> long:
>>>> Intger:
>>>> short:
>>>> byte:
>>>> double:
>>>> float:
>>>> half_float:精度比float小一半
>>>> scaled_float：根据一个long和scaled来表达一个浮点型 long-345 scaled-100 -> 3.45
>>>> 
>>> 时间类型：
>>>> date类型，根据时间类型指定具体的格式
``` php
    PUT /my_index
{
  "mappings": {
    "_doc":{
      "properties":{
        "date":{
          "type":"date",
          "format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}
```
>>> 布尔类型：
>>>> boolean 类型，true 、false
>>>> 
>>> 二进制类型：
>>>> binary类型暂时支持Base64编码的字符串
>>>> 
>>> 范围类型：
>>>> integer_range:
>>>> float_range:
>>>> long_range:赋值时，无需指定具体的内容，只需存储一个范围即可，gte、lte、gt、lt
>>>> double_range:
>>>> date_range:
>>>> ip_range
``` php
    //创建索引
    PUT /range_index
    {
      "settings": {
        "number_of_shards": 2
      },
      "mappings": {
        "_doc":{
          "properties":{
            "excpetd_change":{
              "type":"integer_range"
            },
            "time_frame":{
              "type":"date_range",
              "format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
            }
          }
        }
      }
    }
    //添加数据
    PUT /range_index/_doc/1?refresh
    {
      "excpetd_change":{
        "gt":10,
        "lt": 20
      },
      "time_frame":{
        "gte":"2020-10-31 12:00:00",
        "lte":"2020-11-01"
      }
    }
```
>>> 经纬度类型：
>>>> geo_point:用来存储经纬度
>>> IP类型:
>>>> ip:可以存储ipv4和ipv6
>>> 其他数据类型参考官网
>> #### 2.5 创建索引并指定数据结构
``` php
    
#创建索引，指定数据类型
PUT /book
{
  "settings": {
    #分片数
    "number_of_shards": 5,
    #备份数
    "number_of_replicas": 1
  },
    #指定数据类型
 "mappings": {
    #类型 Type
   "novel":{
    #文档存储的field
     "properties":{
       #field属性名
       "name":{
         #类型
         "type":"text",
         #指定分词器
         "analyzer":"ik_max_word",
         #指定当前的field可以被作为查询的条件
         "index":true,
         #是否需要额外存储
         "store":false
       },
       "author":{
         "type":"keyword"
       },
       "count":{
         "type":"long"
       },
       "on-sale":{
         "type":"date",
           #指定时间类型的格式化方式
         "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
       },
        "descr":{
          "type":"text",
          "analyzer":"ik_max_word"
       }
     }
   }
 }
}
```
>> #### 2.6 文档操作
>>> 文档在ES服务中的唯一标识，_index, \_type, \_id 
>>> 2.6.1 新建文档
``` php

自动生成id
#添加文档，自动生成id
POST /book/novel
{
  "name":"盘龙",
  "author":"我吃西红柿",
  "count":100000,
  "on-sale":"2001-01-01",
  "descr":"大小的血睛鬃毛狮，力大无穷的紫睛金毛猿，毁天灭地的九头蛇皇，携带着毁灭雷电的恐怖雷龙……这里无奇不有，这是一个广博的魔幻世界。强者可以站在黑色巨龙的头顶遨游天际，恐怖的魔法可以焚烧江河，可以毁灭城池，可以夷平山岳……"
}

#添加文档,手动指定id
PUT /book/novel/1
{
  "name":"红楼梦",
  "author":"曹雪芹",
  "count":10000000,
  "on-sale":"2501-01-01",
  "descr":"中国古代章回体长篇小说，中国古典四大名著之一，一般认为是清代作家曹雪芹所著。小说以贾、史、王、薛四大家族的兴衰为背景，以富贵公子贾宝玉为视角，以贾宝玉与林黛玉、薛宝钗的爱情婚姻悲剧为主线，描绘了一批举止见识出于须眉之上的闺阁佳人的人生百态，展现了真正的人性美和悲剧美"
}
```
>>> 2.6.2 修改文档
>>>> 1.覆盖式修改 
``` php
    
PUT /book/novel/1
{
  "name":"红楼梦",
  "author":"曹雪芹",
  "count":1000444,
  "on-sale":"2501-01-01",
  "descr":"中国古代章回体长篇小说，中国古典四大名著之一，一般认为是清代作家曹雪芹所著。小说以贾、史、王、薛四大家族的兴衰为背景，以富贵公子贾宝玉为视角，以贾宝玉与林黛玉、薛宝钗的爱情婚姻悲剧为主线，描绘了一批举止见识出于须眉之上的闺阁佳人的人生百态，展现了真正的人性美和悲剧美"
}
```
>>>> 2.使用doc方式修改
``` php
    
#修改文档，使用doc 方式
POST /book/novel/1/_update
{
  "doc":{
      #指定需要修改的field和对应的值
    "count":566666
  }
}
```
>>> 2.6.3 删除文档
>>> 根据id删除：DELETE /book/novel/1




