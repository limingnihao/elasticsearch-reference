# Match phrase query 短语查询
`match_phrase`查询将文本分词，并且根据分词的文本创建`phrase`查询。例如：
```
GET /_search
{
    "query": {
        "match_phrase" : {
            "message" : "this is a test"
        }
    }
}
```

短语查询将以任意顺序最多可以匹配slop(默认为0)的词汇。Transposed terms have a slop of 2。


可以设置哪个分词器将对文本执行分词。它默认为字段在mapping的定义，或默认的搜索分词器，例如:
```
GET /_search
{
    "query": {
        "match_phrase" : {
            "message" : {
                "query" : "this is a test",
                "analyzer" : "my_analyzer"
            }
        }
    }
}
```

该查询也接受zero_terms_query，如[match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html)中所述。