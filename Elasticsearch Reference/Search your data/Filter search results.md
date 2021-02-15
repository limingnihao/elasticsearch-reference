# Filter search results

您可以使用两种方法来filter搜索结果:

* 使用带filter子句的boolean查询。搜索请求同时对search hits和aggregations应用boolean filter。
* 使用搜索API的post_filter参数。搜索请求只对search hits应用post filters，而不是aggregations。您可以使用post filter根据更广泛的结果集计算aggregations，从而进一步缩小结果范围。

您还可以在post filter之后使用rescore，以提高相关性并重新排序结果。



## Post filter

当使用post_filter参数过滤搜索结果时，aggregations计算完成后将过滤搜search hits。post filter对聚合结果没有影响。

例如，你出售的衬衫有以下属性:

```
PUT /shirts
{
  "mappings": {
    "properties": {
      "brand": { "type": "keyword"},
      "color": { "type": "keyword"},
      "model": { "type": "keyword"}
    }
  }
}

PUT /shirts/_doc/1?refresh
{
  "brand": "gucci",
  "color": "red",
  "model": "slim"
}
```

假设用户指定了两个过滤器:

`color:red` and `brand:gucci`。你只想在搜索结果中给他们展示Gucci制造的red shirts 。通常你会用bool查询来做这个:

```
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  }
}
```

但是，您还希望使用*faceted*导航来显示用户可以单击的其他选项列表。也许您有一个model字段，将用户搜索结果限制为红色的Gucci的值为 `t-shirts` or `dress-shirts`.。



可以这么写： [`terms` aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/search-aggregations-bucket-terms-aggregation.html):

```console
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  },
  "aggs": {
    "models": {
      "terms": { "field": "model" } 
    }
  }
}
```



但也许你还想告诉用户有多少其他颜色的Gucci shirts可供选择。如果您只是在颜色字段中添加一个terms aggregation，那么您将只返回红色，因为您的查询只返回Gucci的红色shirts。

相反，您希望在aggregation过程中包含所有颜色的shirts，然后只对搜索结果应用颜色过滤器。这就是post_filter的目的:

```
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "brand": "gucci" } ①
      }
    }
  },
  "aggs": {
    "colors": {
      "terms": { "field": "color" } ②
    },
    "color_red": {
      "filter": {
        "term": { "color": "red" } ③
      },
      "aggs": {
        "models": {
          "terms": { "field": "model" } ③
        }
      }
    }
  },
  "post_filter": { ④
    "term": { "color": "red" }
  }
}
```

1. 主要的查询对象是所有的gucci的shirts，无论颜色。

2. colors agg回归了gucci的shirts的所有颜色。

3. color_red agg限制了models的子集合，只穿read的Gucci shirts。

4. 最后，post_filter从搜索结果中删除红色以外的颜色。

> 其实就是在query的结果上在做一次filter，然后才是最后的hits。因为一般agg需要全量数据，然而返回结果并不需要全量数据，所以在做一次filter。这样agg聚合了所有数据所计算，result hits也只返回了需要的数据。



## Rescore filtered search results

rescoring可以帮助提高精度，只需重新排序query和post_filter阶段的top文档(例如100 - 500)，使用二级算法(通常代价更高)，而不是对索引中的所有文档应用昂贵的算法。

在每个分片返回其结果并由处理整个搜索请求的节点进行排序之前，在每个分片上执行rescore请求。

目前rescore API只有一个实现:query rescorer，它使用一个查询来调整评分。在将来，可以提供其他的rescorer程序，例如，pair-wise rescorer。

> 如果使用rescore query提供显式排序(不是降序排序的_score)，则会抛出错误。

> 当向用户展示页面时，您不应该在逐步浏览每个页面时更改window_size(通过传递不同的from值)，因为这可能会改变top hits，导致用户逐步浏览页面时的结果发生令人困惑的变化。



### Query rescorer

query rescorer 只对query和post_filter阶段返回的Top-K结果执行第二次查询。每个分片上要检查的文档数量可以由window_size参数控制，该参数默认为10。

默认情况下，原始查询的分数和rescore查询的分数线性组合，为每个文档生成最终的_score。原始查询和rescore查询的相对重要性可以分别用query_weight和rescore_query_weight来控制。都默认为1。

例如：

```console
POST /_search
{
   "query" : {
      "match" : {
         "message" : {
            "operator" : "or",
            "query" : "the quick brown"
         }
      }
   },
   "rescore" : {
      "window_size" : 50,
      "query" : {
         "rescore_query" : {
            "match_phrase" : {
               "message" : {
                  "query" : "the quick brown",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }
}
```

分数的组合方式可以通过score_mode来控制:

| Score Mode | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `total`    | Add the original score and the rescore query score. The default. |
| `multiply` | Multiply the original score by the rescore query score. Useful for [`function query`](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/query-dsl-function-score-query.html) rescores. |
| `avg`      | Average the original score and the rescore query score.      |
| `max`      | Take the max of original score and the rescore query score.  |
| `min`      | Take the min of the original score and the rescore query score. |



### Multiple rescores

也可以按顺序执行多个rescore:

```console
POST /_search
{
   "query" : {
      "match" : {
         "message" : {
            "operator" : "or",
            "query" : "the quick brown"
         }
      }
   },
   "rescore" : [ {
      "window_size" : 100,
      "query" : {
         "rescore_query" : {
            "match_phrase" : {
               "message" : {
                  "query" : "the quick brown",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }, {
      "window_size" : 10,
      "query" : {
         "score_mode": "multiply",
         "rescore_query" : {
            "function_score" : {
               "script_score": {
                  "script": {
                    "source": "Math.log10(doc.count.value + 2)"
                  }
               }
            }
         }
      }
   }]
}
```

第一个获取查询的结果，第二个获取第一个的结果，以此类推。第二次重新评分将“看到”第一次重新评分完成的排序，因此可以在第一次重新评分时使用一个大窗口将文档拉到一个较小的窗口中进行第二次重新评分。