## Nosql 数据库介绍

是一种非关系型数据库服务，它能解决常规数据库的并发能力，比如传统的数据库的IO与性能的瓶颈，同样它是关系型数据库的一个补充，有着比较好的高效率与高性能。专注于key-value查询的redis、memcached、ttserver。
解决以下问题：

-   对数据库的高并发读写需求
-   大数据的高效存储和访问需求
-   高可扩展性和高可用性的需求

## MongoDB 基础概念
MongoDB 是面向文档的 NoSQL 数据库，用于大量数据存储。
#### MongoDB功能

每个数据库都包含集合，而集合又包含文档。每个文档可以具有不同数量的字段。每个文档的大小和内容可以互不相同。文档结构更符合开发人员如何使用各自的编程语言构造其类和对象。开发人员经常会说他们的类不是行和列，而是具有键值对的清晰结构。从 NoSQL 数据库的简介中可以看出，行（或在MongoDB 中调用的文档）不需要预先定义架构。相反，可以动态创建字段。MongoDB 中可用的数据模型使我们可以更轻松地表示层次结构关系，存储数组和其他更复杂的结构。可伸缩性– MongoDB 环境具有很高的可伸缩性。全球各地的公司已经定义了自己的集群，其中一些集群运行着100多个节点，数据库中包含大约数百万个文档。

## mongodb优点
- 灵活。数据存储在文档中，不以关系类型格式存储。
- 支持索引。可以创建索引以提高MongoDB中的搜索性能。MongoDB文档中的任何字段都可以建立索引。
-  MongoDB可以提供副本集的高可用性。副本集由两个或多个mongo数据库实例组成。每个副本集成员可以随时充当主副本或辅助副本的角色。主副本是与客户端交互并执行所有读/写操作的主服务器。辅助副本使用内置复制维护主数据的副本。当主副本发生故障时，副本集将自动切换到辅助副本，然后它将成为主服务器。
-  负载平衡-MongoDB使用分片的概念，通过在多个MongoDB实例之间拆分数据来水平扩展。MongoDB可以在多台服务器上运行，以平衡负载或复制数据，以便在硬件出现故障时保持系统正常运行。