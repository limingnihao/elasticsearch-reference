# Wildcard query

返回包含匹配通配符模式的term的文档。

通配符是匹配一个或多个字符的占位符。例如，*通配符操作符匹配零个或多个字符。可以将通配符操作符与其他字符组合起来创建通配符模式。

## Example request

下面的搜索返回文档，其中用户字段包含以ki开头、以y结尾的术语。这些匹配的术语可能包括kiy、kity或kimchy。
```
GET /_search
{
    "query": {
        "wildcard": {
            "user": {
                "value": "ki*y",
                "boost": 1.0,
                "rewrite": "constant_score"
            }
        }
    }
}
```

## Top-level parameters for term
* <field> (Required, object) 你需要搜索的字段。


### Parameters for <field>

#### value
（required, string）您希望在提供的<field>中找到的通配符模式的term。

此参数支持两个通配符操作符:

* ?，它匹配任何单个字符
* *，它可以匹配零个或多个字符，包括一个空字符
> 避免使用*或?开头的模式。这可能会增加查找匹配项所需的迭代，并降低搜索性能。

#### boost
（Optional, float）浮点数，用于减少或增加查询的相关分数。默认为1.0。

您可以使用boost参数来调整包含两个或多个查询的搜索的相关分数。

Boost值相对于默认值1.0。在0和1.0之间的boost值会降低相关性得分。大于1.0的值将增加相关性得分。

#### rewrite
(Optional, string) 方法用于重写查询。有关有效值和更多信息，请参见[重写参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)。