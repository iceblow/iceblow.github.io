---
layout: post
title: Elasticsearch
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
---

Elasticsearch是高度可伸缩的开源全文搜索和分析引擎。它允许我们快速实时地存储、搜索、分析大数据。

Elasticsearch使用Lucene作为内部引擎，但是在你使用它做全文搜索时，只需要使用统一开发好的API即可，而不需要了解其背后复杂的Lucene的运行原理。它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

不过，Elasticsearch不仅仅是Lucene和全文搜索，我们还能这样去描述它：

- **分布式**的**实时文件存储**，每个字段都被索引并可被搜索
- 分布式的**实时分析搜索引擎**
- 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

