##Constant score query

包装filter查询，返回每个匹配的文档时，设定所有文档的其相关性得分等于boost参数值。

### Example request
```
{
    "_source": {
  	    "include": ["favoriteFruit", "name", "eyeColor", "age"]
    },	
    "query": {
        "constant_score": { 
            "filter": {
               "term": {  "eyeColor": "blue" }
            },
            "boost": 0.5
        }
    }
}
```

### Top-level parameters for constant_score
#### filter
(必选，查询对象)筛选您希望运行的查询。任何返回的文档都必须匹配此查询。


#### boost
(可选，浮点数)浮点数用作常数的相关性得分，对于每个文档filter查询匹配的文档. 默认值为 1.0.

>译者备注：给filter的查询结果赋值一个固定得分。