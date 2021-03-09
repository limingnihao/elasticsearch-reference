## Prefix query
返回在提供的字段中包含特定前缀的文档。

### Example request
下面的搜索返回文档，其中user字段包含一个以`ab`开头的术语。
```
GET /test/_search
{
  "query": {
      "prefix": {
        "name": {
          "value": "ab"
        }
      }
  }
}
```

### Short request example
你可以简化prefix query。使用<field>和value参数，例如：
```
GET /test/_search
{
  "query": {
      "prefix": {
        "name": "ab"
      }
  }
}
```

### Parameters for <field>
* value。查询的term
* rewrite。方法用于重写查询。有关有效值和更多信息，请参见[rewrite parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)。
 

### Speed up prefix queries
您可以使用index_prefixes映射参数来加速前缀查询。如果启用，Elasticsearch索引将在单独的字段中添加2到5个字符之间的前缀。这使得Elasticsearch能够以更大的索引为代价更有效地运行前缀查询。

