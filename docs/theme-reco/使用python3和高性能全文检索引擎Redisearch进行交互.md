---
title: 使用python3和高性能全文检索引擎Redisearch进行交互
date: 2020-10-09 16:45:12
tags: [全文检索,Redisearch]
---



介绍一款[高性能全文检索引擎Redisearch](https://v3u.cn/a_id_105)，它不仅性能强劲，部署也方便，这里介绍一下如何用python客户端和它进行交互。使用redisearch-python:<https://github.com/RediSearch/redisearch-py>

​    首先，安装

​    

```
pip3 install redisearch
```

​    基本操作:

​    

```
from redisearch import Client, TextField
# Creating a client with a given index name
client = Client('myIndex',host='localhost',port='6666')


# Creating the index definition and schema
client.create_index((TextField('title'), TextField('body')))

# Indexing a document
client.add_document('doc2', title = '你好', body = '我在北京学习人工智能',language='chinese')

# Simple search
res = client.search("人工智能")

print(res.docs[0].title)
```

​    可以看到，基本上和命令行中的操作方式没有太大区别，只是在search时不需要指定语言了，程序可以自主判断。

​    其实它的官方文档很简单，只是介绍了基本用法，但是你如果阅读了它的源码，发现一些常用操作它也进行了封装，比如

​    

```
#删除索引
client.drop_index()

#获取当前索引的基本信息
client.info()

#删除文档
client.delete_document('doc2')
```

​    还是非常简单的，基本上，我们可以抛弃ES了，因为研发人员都是喜新厌旧的。