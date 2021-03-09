
## Exists query
返回字段索引值存在的文档。

由于各种原因，文档字段可能不存在索引值:
* The field in the source JSON is null or []
* The field has "index" : false set in the mapping
* The length of the field value exceeded an ignore_above setting in the mapping
* The field value was malformed and ignore_malformed was defined in the mapping

但以下情况，表示字段存在：
* Empty strings, such as "" or "-"
* Arrays containing null and another value, such as [null, "foo"]
* A custom null-value, defined in field mapping

### Example  request
```
GET /_search
{
    "query": {
        "exists": {
            "field": "user"
        }
    }
}
```

### Find documents missing indexed values
要查找缺少字段索引值的文档，请对exists查询使用must_not布尔查询。

下面的搜索返回用户字段中缺少索引值的文档。
```
GET /_search
{
    "query": {
        "bool": {
            "must_not": {
                "exists": {  "field": "user" }
            }
        }
    }
}
```