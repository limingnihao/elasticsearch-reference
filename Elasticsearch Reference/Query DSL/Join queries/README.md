# Join queries

在像Elasticsearch这样的分布式系统中执行完整的sql风格的连接非常昂贵。相反，Elasticsearch提供了两种连接形式，它们被设计为水平扩展。



### nested query

文档可以包含 `nested` 嵌套类型的字段。这些字段用于为对象数组建立索引，可以将每个对象(使用nested查询)作为一个独立的文档进行查询。



### has_child and has_parent queries

join 字段关系可以存在于单个索引中的文档之间。`has_child`返回满足子文档查询条件的父文档，而`has_parent`返回满足父文档查询条件的子文档。