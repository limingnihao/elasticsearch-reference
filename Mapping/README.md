# Mapping

Mapping是定义文档及其包含的字段如何存储和索引的过程。例如，使用mapping来定义:
* 哪些字符串字段应该被视为全文字段。
* 哪些字段包含数字、日期或地理位置。
* 日期值的格式。
* 用于控制自定义mapping [dynamically added fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-mapping.html)。

mapping定义包括：

#### Meta-fields
[Meta-fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-fields.html) 用于自定义如何处理文档的关联metadata。Meta-fields的示例包括文档的_index、_id和_source字段。

#### Fields or properties
Mapping包含与文档相关的字段或属性列表。
> 在7.0.0之前，mapping定义一个type name。更多需要参考[removal of mapping types](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)

## Field datatypes
每个字段都有一个数据类型，可以是:
* 基本类型例如： text, keyword, date, long, double, boolean or ip.
* 支持json层级接口的类型 object，nested.
* 专业高级类型流入 geo_point, geo_shape, completion.

为不同的目的以不同的方式索引相同的字段通常是有用的。例如，字符串字段可以作为全文搜索的text字段建立索引，也可以作为排序或聚合的keyword字段建立索引。或者，您可以使用标准分析器、英语分析器和法语分析器来索引字符串字段。

这就是multi-fields的目的。大多数数据类型通过fields参数支持multi-fields。

### Settings to prevent mappings explosion(防止Setting爆炸的设置)

在索引中定义太多字段会导致映射爆炸，这会导致内存错误和难以恢复的情况。这个问题可能比预期的更普遍。例如，考虑这样一种情况，其中插入的每个新文档都引入了新字段。这在动态映射中很常见。每当文档包含新字段时，这些字段就会出现在索引的映射中。这对于少量数据来说并不令人担忧，但是随着映射的增长，这可能成为一个问题。以下设置允许您限制可以手动或动态创建的字段映射的数量，以防止糟糕的文档导致映射爆炸:

#### index.mapping.total_fields.limit
索引中字段的最大数目。字段和对象映射以及字段别名都属于此限制。默认值是1000。
> 这个限制是为了防止mappings和searches变得太大。较高的值可能导致性能下降和内存问题，特别是在负载高或资源少的集群中。如果您增加这个设置，我们建议您也增加indices.query.bool.max_clause_count 子句计数设置，用于限制查询中布尔子句的最大数量。

#### index.mapping.depth.limit
field的最大深度，标识objects内部的大小。例如，如果所有字段都在根绝点，则深度为1。如果有一个object，则深度为2，以此类推。默认为20.

#### index.mapping.nested_fields.limit
The maximum number of distinct nested mappings in an index, defaults to 50.

#### index.mapping.nested_objects.limit
nested JSON object 类型单个文档的最大类型数量。默认为10000。


#### index.mapping.field_name_length.limit
字段名称的最大长度。默认为Long.MAX_VALUE。这个设置实际上并没有解决映射爆炸的问题，但是如果您想限制字段长度，它仍然很有用。通常不需要设置此设置。默认值即可，除非用户开始添加大量具有非常长的名称的字段。

## Dynamic mapping(动态映射)
字段和类型并不需要先定义在使用。新的字段名称在索引文档时将被自动添加，新字段既可以添加到顶级映射类型，也可以添加到内部对象和嵌套字段。

可以配置动态映射规则来定制用于新字段的映射。


## Explicit mappings
您对数据的了解比Elasticsearch所能猜测的要多，因此，虽然开始动态映射很有用，但是在某些情况下，您可能希望指定自己的显式映射。

你可以在创建index或已存在的index创建字段映射。

### Create an index with an explicit mapping
你可以使用create index API在创建一个新的index时明确mapping。
```
PUT /my-index
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, 
      "name":   { "type": "text"  }     
    }
  }
}
```

### Add a field to an existing mapping
你可以使用 put mapping API 像存在的index添加一个或多个字段。
```
PUT /my-index/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false
    }
  }
}
```

### Update the mapping of a field
除了受支持的映射参数之外，您不能更改现有字段的映射或字段类型。更改现有字段可能会使已经索引的数据无效。

如果需要更改字段的映射，请创建具有正确映射的新索引，并将数据重新索引到该索引中。

重命名字段将使已经在旧字段名下建立索引的数据无效。但是可以，添加一个alias字段来创建一个备用字段名。

## View the mapping of an index
你可以使用get mapping API 浏览存在的index的mapping
```
GET /my-index/_mapping
```

API返回结果：
```
{
  "my-index" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "integer"
        },
        "email" : {
          "type" : "keyword"
        },
        "employee-id" : {
          "type" : "keyword",
          "index" : false
        },
        "name" : {
          "type" : "text"
        }
      }
    }
  }
}
```

## View the mapping of specific fields

如果你想浏览其中一个字段，可以使用get field mapping API。

如果您不需要索引的完整映射，或者索引包含大量字段，那么这是非常有用的。

下面的请求检索employee-id字段的映射。

```
GET /my-index/_mapping/field/employee-id
```

API返回结果:
```
{
  "my-index" : {
    "mappings" : {
      "employee-id" : {
        "full_name" : "employee-id",
        "mapping" : {
          "employee-id" : {
            "type" : "keyword",
            "index" : false
          }
        }
      }
    }
  }
}
```