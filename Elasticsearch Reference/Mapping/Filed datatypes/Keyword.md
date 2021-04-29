## Keyword type family

keyword family包括以下字段类型:

* keyword, 用于结构化内容，如id、电子邮件地址、主机名、状态码、邮政编码或标签。
* constant_keyword, 用于始终包含相同值的关键字字段
* wildcard, 为grep通配符查询优化日志行和类似的关键字值。



keyword field通常用于 [sorting](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html), [aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html), 和[term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html)，比如[`term`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html)。



> 避免使用keyword字段进行全文搜索。使用text字段类型代替。

