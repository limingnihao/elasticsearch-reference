## IDs query
根据文档的id返回文档。此查询使用存储在_id字段中的文档id。
```
GET /test/_search
{
    "query": {
        "ids" : {
            "values" : ["1", "4", "100"]
        }
    }
}
```