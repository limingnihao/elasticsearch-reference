# More like this query
More like This Query查找一些和给定的文档类似的文档。为此，MLT在输入文档中选择一些有代表性的term，用于查询。用户可以控制输入文档、如何选择term和如何执行query。

例如：我们要在“title”和“description”字段中寻找类似"Once upon a time"的文本，限制term数量为12.
```
GET /_search
{
    "query": {
        "more_like_this" : {
            "fields" : ["title", "description"],
            "like" : "Once upon a time",
            "min_term_freq" : 1,
            "max_query_terms" : 12
        }
    }
}
```