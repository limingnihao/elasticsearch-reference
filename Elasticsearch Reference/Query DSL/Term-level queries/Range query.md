
## Range query
返回包含在提供的范围内文档。

### Example request

```
GET /test/_search
{
  "query": {
      "range": {
        "production_date": {
          "gte": "2020-03-01",
          "lte": "2020-03-01"
        }
      }
  }
}
```

### Paramsters for <field>
* gt. Greater than
* gte. Greater than or equal to.
* lt. Less than.
* lte. Less than or equal to.
* format. 用于在查询中转换日期值的日期格式。参考 [format](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html)
* relation. 指示范围查询如何匹配范围字段的值。有效值:INTERSECTS、CONTAINS、WITHIN。
* time_zone. 例如："+01:00"
* boost. 浮点数，用于减少或增加查询的相关分数。默认为1.0。

