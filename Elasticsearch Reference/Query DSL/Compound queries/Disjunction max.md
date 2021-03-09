## Disjunction max query


返回与一个或多个包装查询匹配的文档。

如果返回的文档匹配多个查询子句，dis_max查询将从任何匹配子句中为文档分配最高的相关性得分，并为任何其他匹配子查询分配一个打破平局的增量。

您可以使用dis_max在映射了不同boost因子的字段中搜索一个term。

### Example request
```
{
    "query": {
        "dis_max" : {
            "queries" : [
                { "term" : { "title" : "Quick pets" }},
                { "term" : { "body" : "Quick pets" }}
            ],
            "tie_breaker" : 0.7
        }
    }
}
```

### Top-level parameters for dis_max
#### queries
(必选，查询对象数组)包含一个或多个查询子句。返回文档必须匹配一个或多个这些query。如果一个文档匹配多个查询，则Elasticsearch使用最高查询的相关性得分。

#### tie_breaker
（可选，浮点类型）0到1.0之间的浮点数，用于增加匹配多个查询子句的文档的相关性得分。默认为0.0。

您可以使用tie_breaker值为在多个字段中包含相同term的文档分配较高的相关性分数，而不是只在多个字段中包含该term的文档中取最好的一个，这样不会将其与多个字段中两个不同term的较好情况相混淆。


如果一个文档匹配多个子句，dis_max查询将计算该文档的相关分数，如下所示:
1. 从具有最高分数的匹配子句中获取相关性分数。
2. 将任何其他匹配子句的得分乘以tie_breaker。
3. 把最高的分数加到相乘的分数上。

如果tie_breaker值大于0.0，所有匹配的子句都算数，但是分数最高的子句算数最多。

>译者备注：lucene的DisjunctionMaxQuery，solr的dismax。