# Query DSL

Elasticsearch提供了一个完整的基于JSON的查询DSL(Domain Specific Language)来定义查询。可以将查询DSL看作查询的AST(Abstract Syntax Tree抽象语法树)，它由两种类型的子句组成:

**Leaf query clauses**

​	叶查询子句在特定字段中查找特定值，例如match、term或range查询。这些查询可以自己使用。



**Compound query clauses**

​	复合查询子句包装了其他叶子查询或复合查询，并用于以逻辑方式组合多个查询(比如bool或dis_max查询)，或者改变它们的行为(比如constant_score查询)。



查询子句的行为取决于它们是在query context中使用还是在filter context使用。(query and filter context)



**Allow expensive queries**

  某些类型的查询通常执行缓慢，这是因为它们的实现方式会影响集群的稳定性。这些查询可以按以下方式分类:

* 需要做线性扫描来识别匹配的查询. Queries that need to do linear scans to identify matches:
    * [script queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-query.html)

* 具有高预先成本的查询. Queries that have a high up-front cost :
    * [`fuzzy queries` 模糊查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#wildcard-field-type) fields)
    * [`regexp queries` 正则表达式查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-regexp-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#wildcard-field-type) fields)
    * [`prefix queries` 前缀查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#wildcard-field-type) fields or those without [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-prefixes.html))
    * [`wildcard queries` 通配符查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-wildcard-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#wildcard-field-type) fields)
    * [`range queries`](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html) fields

* Joining queries。嵌套查询
* Queries on [deprecated geo shapes](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-shape.html#prefix-trees)
* Queries that may have a high per-document cost:
    * [`script score queries` 脚本查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-score-query.html)
    * [`percolate queries`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-percolate-query.html)



`search.allow_expensive_queries=true`可以防止此类查询的执行。(默认为true)

