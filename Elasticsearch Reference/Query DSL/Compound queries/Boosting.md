## Boosting query

返回匹配的positive查询的文档，同时降低不匹配negative查询的文档的相关性得分。

您可以使用boosting查询来降级某些文档，而不将它们从搜索结果中排除。

### Example request
```
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "text" : "apple"
                }
            },
            "negative" : {
                 "term" : {
                     "text" : "pie tart fruit crumble tree"
                }
            },
            "negative_boost" : 0.5
        }
    }
}
```

### Top-level parameters for boosting 

#### positive
(必填，查询对象)您希望运行的查询。任何返回的文档都必须匹配此查询。

#### negative 
(必填，查询对象)查询用来降低匹配文档的相关分数。
如果返回的文档匹配positive查询和此查询，boosting查询计算文档的最终相关性得分，如下所示:
1. 从positive查询拿到原始得分。
2. 将在negative命中的文档乘以 negative_boost。

#### negative_boost
(必选，浮点数)浮点数之间 0 and 和 1.0。用来减少匹配negative查询的文档的相关性得分


>译者备注：就是把negative命中的文档，通过negative_boost作为系数，进行降权。如果是1则就是不降权；0.5就是将原始分数乘以0.5；设置成0的话，就是negative命中的都是0分。