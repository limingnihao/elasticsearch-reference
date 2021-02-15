# Plugins

## payload query

当需要将term的权重存储到索引中时，需要保存成payload的格式：

类似于：

```ruby
the|0 brown|3 fox|4 is|0 quick|10
```

查询的时候，如果需要用到保存好的value，则需要lucene 的PayloadScoreQuery或者PayloadCheckQuery。

自定义插件使用PayloadScoreQuery。



查询方式:

```
{
    "query": {
        "payload_score": {
            "func": "sum",
            "includeSpanScore": "false",
            "query": {
                "span_or": {
                    "clauses": [
                        { "span_term": { "color": "red"  } }
                    ]
                }
            }
        }
    }
}

```

