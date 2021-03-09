# Match boolean prefix query(布尔前缀匹配查询)

`match_bool_prefix`把输入进行分词并将分词后的term构建为[bool query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)。除最后一个term外，每个term都会使用一个`term`query。最后一个词将使用`prefix`查询。例如一个`match_bool_prefix`查询如下：
```
GET /_search
{
    "query": {
        "match_bool_prefix" : {
            "message" : "quick brown f"
        }
    }
}
```
将会分成quick，brown和f的term，类似于下面的`bool`查询么?
```
GET /_search
{
    "query": {
        "bool" : {
            "should": [
                { "term": { "message": "quick" }},
                { "term": { "message": "brown" }},
                { "prefix": { "message": "f"}}
            ]
        }
    }
}
```

`match_bool_prefix`和[match_phrase_prefix](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase-prefix.html)一个重要的区别是`match_phrase_prefix`查询只匹配短语terms，但是`match_bool_prefix`查询可以在任意位置匹配terms。

这个例子`match_bool_prefix`查询可以匹配一个包含“quick brown fox”的字段，但是不能匹配“brown fox quick”。所以它可以匹配任何位置包含“quick”，“brown”和“f”开头的term。

## Parameters
默认情况下，`match_bool_prefix`查询，输入文本将使用field的mapping设置的analyzer进行分词。可以配置`analyzer`参数设置不同的分词器。
```
GET /_search
{
    "query": {
        "match_bool_prefix" : {
            "message": {
                "query": "quick brown f",
                "analyzer": "keyword"
            }
        }
    }
}
```

`match_bool_prefix`查询支持[minimum_should_match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html)和`operator`参数，详情可以参考[match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-boolean)，将应用于构建的`bool`查询。在大构造的`bool`查询子句，将是查询文本分词后的term数量。


fuzziness, prefix_length, max_expansions, fuzzy_transpositions, and fuzzy_rewrite参数，可用于除最后一个term之外的所有构造的term子查询。他们对构造的最后一个前缀查询没有任何影响。
