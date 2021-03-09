## Match query
返回匹配了提供的文本，数字，日期或布尔值类型字段的文档。提供的文本必须先进行analyzed。

math query是标准的全文搜索查询，还包括模糊匹配选项。

### Example request
```
GET /kibana_sample_data_ecommerce/_search
{
    "_source": {
        "include": ["currency", "customer_full_name"]
    },
    "query": {
        "match" : {
            "customer_full_name" : {
                "query" : "Baker"
            }
        }
    }
}
```

### match 顶级参数
#### <field>
(必填，对象) 你需要搜索的字段。

### <field>参数
| 参数名称 | 选项 | 类型 |说明 |
|---|---|---|---|---|
| query | 必选 | 对象 | 您希望在提供的<field>找到的文本，数字，布尔值或日期。match语句会在执行search之前对提供的文本进行analyzes。 |
| analyzer| 可选 | 字符串 | Analyzer用于转换文本为tokens。默认同<field>写索引的analyzer。如果为设定，则使用默认的。 |
| auto_generate_synonyms_phrase_query | 可选 | 布尔 | 为true时会自动创建多term同义词的[math phrase](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html)查询。默认为true。|
| fuzziness | 可选 | 字符串 | match允许的最大编辑距离。参考[Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness)|
| max_expansions | 可选 | 整数 | 查询最大扩展的term数。默认为50. |
| prefix_length | 可选 | 整数 | 模糊匹配起始字符数。默认为0.|
| transpositions | 可选 | 布尔 | 如果为true，则用于模糊匹配将包括相邻字符的转置（ab->ba）。默认为true |
| fuzzy_rewrite | 可选 | 字符串 | 用于重写查询的方法。更多说明参考[rewrite parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)。如果fuzziness参数不为0 ，则match查询默认使用top_terms_blended_freqs_${max_expansions}的rewrite方法。 |
| lenient | 可选 | 布尔 | 如果为true，则忽略基于格式的错误，例如数字类型。| 
| operator | 可选 | 字符串 | 用于解释query中文本之间的布尔逻辑。OR(默认),和AND |
| minimum_should_match | 可选 | 字符串 | 返回文档必须匹配的最小子句。更多说明参考[minimum-should-matc parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html)| 
| zero_terms_query | 可选 | 字符串 | 表示在analyzer删除所有token后是否返回空结果，例如使用stop filter时。none（默认）：不返回文档。all：返回所有文档。类似[all_match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-all-query.html) 。有关实例参考[Zero terms query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-zero)| 

## 笔记
### Short request example

您可以通过组合<field>和query参数来简化匹配查询语法。 例如：
```
GET /kibana_sample_data_ecommerce/_search
{
    "_source": {
        "include": ["currency", "customer_full_name"]
    },
    "query": {
        "match" : {
            "customer_full_name" : "Baker"
        }
    }
}
```

### How the match query works
match查询的类型为boolean 。 这意味着将对提供的文本进行analyzed，并且analysis过程将从提供的文本中构造一个布尔查询。operator参数可以设置or或and来控制布尔子句（默认为or）。可以使用 [minimum_should_match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html) 参数设置匹配的最小数量。

这个例子查询结果为Clarice rania Baker三个词，至少匹配2个。
```
GET /kibana_sample_data_ecommerce/_search
{
    "_source": {
        "include": ["currency", "customer_full_name"]
    },
    "query": {
        "match" : {
            "customer_full_name" : {
                "query" : "Clarice rania Baker",
                "operator": "or",
                "minimum_should_match": 2
            }
        }
    }
}
```
analyzer参数设置使用什么分词器对文本进行分词。默认使用mapping邓毅的，或者默认的搜索分词器。

lenient参数设置为true，以忽略由数据类型不匹配引起的异常，例如，尝试使用文本查询字符串查询数字字段。 默认为false 。

### Fuzziness in the match query

fuzziness 设置基于被查询字段类型的模糊匹配。参考设置[Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness)。

prefix_length 和 max_expansions 用于控制模糊匹配过程。如果设置了模糊选项，查询将使用top_terms_blended_freqs_${max_expansions}作为其重写方法，[fuzzy_rewrit](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)e参数允许控制如何重写查询.

默认情况下允许使用模糊换位(ab→ba)，但是可以通过将fuzzy_transpositions设置为false来禁用。

>Note：
Fuzzy matching is not applied to terms with synonyms or in cases where the analysis process produces multiple tokens at the same position. Under the hood these terms are expanded to a special synonym query that blends term frequencies, which does not support fuzzy expansion.
```
GET /_search
{
    "query": {
        "match" : {
            "message" : {
                "query" : "this is a test",
                "operator" : "and"
            }
        }
    }
}
```

### Zero terms queryedit
如果analyzer删除查询中的所有tokens，则默认行为是根本不匹配任何文档。可以使用zero_terms_query选项改变，该选项接受none(default)和all的类似于一个match_all查询。
```
GET /_search
{
    "query": {
        "match" : {
            "message" : {
                "query" : "to be or not to be",
                "operator" : "and",
                "zero_terms_query": "all"
            }
        }
    }
}
```

### Cutoff frequency 
> Deprecated in 7.3.0


### Synonyms 同义词
match查询支持使用多术语同义词扩展 [synonym_graph](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-graph-tokenfilter.html) token filter。使用此筛选器时，解析器将为每个多术语同义词创建短语查询。例如，下面的同义词“ny, new york”会产生:
```
(ny OR ("new york"))
```


也可以用连词来匹配多个同义词:


上面的例子创建了一个布尔查询:
```
(ny OR (new AND york)) city

```
将文档与术语`ny`或连接词`new and york`匹配。默认情况下，参数auto_generate_synonyms_phrase_query设置为true。