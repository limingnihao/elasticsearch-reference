# Match phrase prefix query
返回包含提供文本的单词的文档，顺序与提供的相同。提供文本的最后一个term作为前缀，匹配已此term开头的任意单词。

## Example request
下面的搜索返回在message字段包含`quick brown f`的文档。

此查询将匹配message值为`quick brown fox`、`two quick brown ferrets`, 但是不能匹配` quick and brown`。
```
GET /_search
{
    "query": {
        "match_phrase_prefix" : {
            "message" : {
                "query" : "quick brown f"
            }
        }
    }
}
```

## match_phrase_prefix 顶级参数 
###<field>
（必填，对象）将由于搜索的字段。

## <field> 参数
|参数|必填|类型|说明|
|---|---|---|---|
|query           |必填|string|搜索的文本|
|analyzer        |必填|string|用于将query的值转换成token。使用index的analyzer，否则使用默认。|
|max_expansions  |选填|integer|最后一个term模糊查询最大数量。默认为50|
|slop            |选填|integer|token之间最大间隙。默认0.Transposed terms have a slop of 2.|
|zero_terms_query|选填|string|当analyzer删除所有的token，例如使用停用词时。有效值：none(无结果), all(所有文档，类似match_all )|


## Notes
使用短语模糊匹配在搜索自动完成

虽然设置起来很容易，但是使用match_phrase_prefix查询进行搜索自动完成有时会产生令人困惑的结果。

例如，考虑查询字符串quick brown f。该查询通过用quick和brown创建短语查询来工作(例如，必须存在quick这个词，并且后面必须跟brown这个词)。然后，它查看已排序的term字典，查找前50个term中以f开头的，并将这些term添加到短语查询中。

问题是，前50个term可能不包括fox一词，所以quick brown fox一词就找不到了。这通常不是问题，因为用户将继续输入更多的字母，直到他们要查找的单词出现。

有关“按类型搜索”的更好解决方案，请参阅 [completion suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester) 和 [search-as-you-type](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html)。