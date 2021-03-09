# Terms query

返回在提供的字段中包含一个或多个精确term的文档。

terms查询与term查询相同，不同之处是可以搜索多个值。

## Example request
下面的搜索返回用户字段包含kimchy或elasticsearch的文档。
```
GET /_search
{
    "query" : {
        "terms" : {
            "user" : ["kimchy", "elasticsearch"],
            "boost" : 1.0
        }
    }
}
```


## Top-level parameters for term
* <field> (Required, object) 你需要搜索的字段。


### Parameters for <field>
#### value
（required, string）您希望在提供的<字段>中找到的term。

此参数的值是希望在提供的字段中找到的term数组。要返回文档，一个或多个term必须与字段值完全匹配，包括空格和大小写。

默认情况下，Elasticsearch将条件查询限制为最多65,536个条件。您可以更改此限制 [index.max_terms_count](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-max-terms-count)。

> 要使用现有文档的字段值作为搜索项，请使用terms lookup参数。

#### boost
（Optional, float）浮点数，用于减少或增加查询的相关分数。默认为1.0。

您可以使用boost参数来调整包含两个或多个查询的搜索的相关分数。

Boost值相对于默认值1.0。在0和1.0之间的boost值会降低相关性得分。大于1.0的值将增加相关性得分。

## Notes

### Highlighting terms queries

[高亮](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html#request-body-search-highlighting)。Elasticsearch可能不会返回highlight的搜索结果，具体取决于:
* Highlighter type
* 查询中的term数量

## Terms lookup
Terms lookup获取现有文档的字段值。然后Elasticsearch使用这些值作为搜索项。这在搜索大量术语时很有帮助。

因为terms lookup从文档获取值，所以必须启用_source映射字段来使用terms lookup。默认情况下启用了_source字段。

> 默认情况下，Elasticsearch将条件查询限制为最多65,536个。这包括使用term查找获取的术语。您可以使用更改此限制设置在[index.max_terms_count](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-max-terms-count)。

### Terms lookup parameters

#### index
(Optional, string) 要从中获取字段值的索引的名称。

#### id
(Optional, string) 要从中获取字段值的文档的ID。

#### path
(Optional, string) 要从中获取字段值的字段的名称。Elasticsearch使用这些值作为查询的搜索项。

如果字段值包含嵌套的内部对象数组，则可以使用点符号语法访问这些对象。

#### routing
(Optional, string) 要从中获取术语值的文档的自定义[routing value](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-routing-field.html)。如果在文档建立索引时提供了自定义路由值，则需要此参数。

### Terms lookup example
```
GET my_index/_search?pretty
{
  "query": {
    "terms": {
        "color" : {
            "index" : "my_index",
            "id" : "2",
            "path" : "color"
        }
    }
  }
}
```

