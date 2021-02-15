# Use Painless scripts in runtime fields

> 该功能还处于测试阶段，可能会发生变化。设计和代码不如官方的GA特性成熟，并且是按原样提供的，没有任何保证。Beta特性不受官方GA特性的支持SLA的约束

runtime field是在查询时计算的字段。定义runtime field时，可以立即在搜索请求、聚合、过滤和排序中使用它。

在定义runtime field时，您可以包括一个在查询时计算的Painless脚本。这个脚本可以访问文档的整个上下文，包括原始_source和任何映射字段及其值。在查询时，脚本运行并为查询中包含的每个脚本字段生成值。

您可以在映射定义下映射运行时部分中的runtime field，或者定义仅作为搜索请求的一部分存在的runtime field。无论您在何处定义runtime field，脚本语法都是相同的。

> 在定义与运行时字段一起使用的painless脚本时，必须包含emit以返回计算值。



### Define a runtime field in the mapping

在映射定义下添加一个运行时部分，以便在不索引字段的情况下研究数据。

下面请求中的脚本从@timestamp字段中提取星期几，该字段被定义为日期类型。该脚本根据timestamp的值计算星期几，并使用emit返回计算的值。

```console
PUT my-index/
{
  "mappings": {
    "runtime": {
      "day_of_week": {
        "type": "keyword",
        "script": {
          "source":
          """emit(doc['@timestamp'].value.dayOfWeekEnum
          .getDisplayName(TextStyle.FULL, Locale.ROOT))"""
        }
      }
    },
    "properties": {
      "timestamp": {"type": "date"}
    }
  }
}
```



### Define a runtime field only in a search request

搜索请求中创建runtime field仅作为查询的一部分存在的字段。您还可以在查询时覆盖现有字段的字段值，而无需修改字段本身。

这种灵活性允许您尝试使用数据模式，并修复索引映射中的错误，而无需重新索引数据。

在下面的请求中，week字段的值将动态计算，并且仅在此搜索请求的上下文中计算:

```console
GET my-index/_search
{
  
  ,
  "aggs": {
    "day_of_week": {
      "terms": {
        "field": "day_of_week"
      }
    }
  }
}
```

