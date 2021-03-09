# Term query

返回在提供的字段中包含确切term的文档。

您可以使用term query来查找基于诸如价格、产品ID或用户名等精确值的文档。

> WARING。 尽量匹配在text类型的字段上使用term query。因为默认情况下，es会对text字段进行analysis。这会使得在text字段上进行精确匹配变得困难。在text字段上搜索，使用match query。

## Example request
```
GET /_search
{
    "query": {
        "term": {
            "user": {
                "value": "Kimchy",
                "boost": 1.0
            }
        }
    }
}
```

## Top-level parameters for term
* <field> (Required, object) 你需要搜索的字段。


### Parameters for <field>
#### value
（required, string）您希望在提供的<字段>中找到的term。

返回的文档，term必须与字段值完全匹配，包括空格和大小写。


#### boost
（Optional, float）浮点数，用于减少或增加查询的相关分数。默认为1.0。

您可以使用boost参数来调整包含两个或多个查询的搜索的相关分数。

Boost值相对于默认值1.0。在0和1.0之间的boost值会降低相关性得分。大于1.0的值将增加相关性得分。