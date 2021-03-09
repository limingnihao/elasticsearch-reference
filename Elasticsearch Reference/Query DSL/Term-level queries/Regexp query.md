
## Regexp query
返回包含与 [正则表达式](https://en.wikipedia.org/wiki/Regular_expression) 匹配的项的文档。

正则表达式是一种使用占位符(称为操作符)匹配数据模式的方法。有关regexp查询支持的操作符列表，请参阅正则表达式语法。

### Example request
```
GET /_search
{
    "query": {
        "regexp": {
            "user": {
                "value": "k.*y",
                "flags" : "ALL",
                "max_determinized_states": 10000,
                "rewrite": "constant_score"
            }
        }
    }

```

### Parameter for <field>
* value。查询的正则表达式。默认情况下，正则表达式被限制为1000个字符。您可以使用索引更改此限制。max_regex_length设置。
* flags。 为正则表达式启用可选操作符。有关有效值和更多信息，请参见[正则表达式语法](https://www.elastic.co/guide/en/elasticsearch/reference/current/regexp-syntax.html#regexp-optional-operators)。
* max_determinized_states. 查询所需的最大自动机状态数。默认是10000。

Elasticsearch在内部使用Apache Lucene解析正则表达式。Lucene将每个正则表达式转换为一个包含若干确定状态的有限自动机。

您可以使用此参数来防止该转换无意中消耗太多资源。您可能需要增加这个限制来运行复杂的正则表达式。
* rewrite。 方法用于重写查询。有关有效值和更多信息，请参见[rewrite parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)。
 
