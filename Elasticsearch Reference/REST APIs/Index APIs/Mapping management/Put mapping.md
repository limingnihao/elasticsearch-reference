# Put mapping API



向现有数据流或索引添加新字段。您还可以使用put映射API来更改现有字段的搜索设置。

对于数据流，默认情况下，这些更改将应用于所有支持索引。

```
PUT /my-index-000001/_mapping
{
  "properties": {
    "email": {
      "type": "keyword"
    }
  }
}
```

> 在7.0.0之前，用于包含类型名称的映射定义。虽然现在不赞成在请求中指定类型，但如果设置了包含类型名称的请求参数，仍然可以提供类型。有关详细信息，请参阅删除映射类型。



### Request

PUT /<target>/_mapping

PUT /_mapping



### Path parameters

<target>

(Optional, string)用于限制请求的数据流、索引和索引别名的逗号分隔列表。支持通配符表达式(*)。



### Query parameters

若要针对集群中的所有数据流和索引，请忽略此参数或使用all或*。

#### **`allow_no_indices`**

(Optional, Boolean) 

#### **`allow_no_indices`**

#### **`expand_wildcards`**

#### **`ignore_unavailable`**

#### **`master_timeout`**

#### **`timeout`**

#### **`write_index_only`**





### Request body

#### **`properties`**





### Examples

#### **`Example with single target`**

put mapping API需要一个现有的数据流或索引。下面的create index API请求创建不带mapping的`publications`索引。

```
PUT /publications
```

下面的put mapping API请求将title(一个新的文本字段)添加到`publications`索引中。

```
PUT /publications/_mapping
{
  "properties": {
    "title":  { "type": "text"}
  }
}
```



#### **`Multiple targets`**

通过一个请求，可以将PUT mapping API应用于多个数据流或索引。例如，您可以同时更新my-index-000001和my-index-000002索引的mapping.

```
# Create the two indices
PUT /my-index-000001
PUT /my-index-000002

# Update both mappings
PUT /my-index-000001,my-index-000002/_mapping
{
  "properties": {
    "user": {
      "properties": {
        "name": {
          "type": "keyword"
        }
      }
    }
  }
}
```



#### **`Add new properties to an existing object field`**

可以使用put mapping API向现有对象字段添加新属性。要了解其工作原理，请尝试以下示例。

使用create index API创建具有name对象字段和内部first文本字段的索引。

```
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "name": {
        "properties": {
          "first": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

使用put mapping API将一个新的内部last文本字段添加到name字段中。

```
PUT /my-index-000001/_mapping
{
  "properties": {
    "name": {
      "properties": {
        "last": {
          "type": "text"
        }
      }
    }
  }
}
```



#### **`Add multi-fields to an existing field`**

multi-fields 允许您以不同的方式索引同一个字段。您可以使用put mapping API来更新字段映射参数，并为现有字段启用multi-fields。

使用create index API创建带有city文本字段的索引。

```
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "city": {
        "type": "text"
      }
    }
  }
}
```



虽然文本字段可以很好地进行全文搜索，但keyword字段不进行analyzed，更适合进行排序或聚合。

使用put mapping API为城市字段启用多字段。此request添加city.raw keyword multi-fields，可以用于排序。

```
PUT /my-index-000001/_mapping
{
  "properties": {
    "city": {
      "type": "text",
      "fields": {
        "raw": {
          "type": "keyword"
        }
      }
    }
  }
}
```



#### **`Change supported mapping parameters for an existing field`**

 [mapping parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html) 文档详细说明了是否可以使用put mapping API为现有字段更新它。例如，您可以使用put mapping API来更新ignore_above参数。

使用create index API创建包含user_id keyword字段的索引。user_id字段的ignore_above参数值为20。

ignore_above是允许索引的最大字符串长度。

```
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "user_id": {
        "type": "keyword",
        "ignore_above": 20
      }
    }
  }
}
```



使用put mapping API将上述ignore_above参数值更改为100。

```console
PUT /my-index-000001/_mapping
{
  "properties": {
    "user_id": {
      "type": "keyword",
      "ignore_above": 100
    }
  }
}
```



#### **`Change the mapping of an existing field`**

除了支持的mapping parameters](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)参数外，您不能更改现有字段的mapping或字段类型。更改现有字段可能会使已经建立索引的数据失效。

如果你想更改索引类型，需要参考 [*Change mappings and settings for a data stream*](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-streams-change-mappings-and-settings.html)。

也可以创建一个新的索引，并修改好类型然后使用 [reindex](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) 。

例如：

使用create index API创建具有long字段类型的`user_id`字段的索引。

```
PUT /my-index-000001
{
  "mappings" : {
    "properties": {
      "user_id": {
        "type": "long"
      }
    }
  }
}
```

插入文档：

```
POST /my-index-000001/_doc?refresh=wait_for
{
  "user_id" : 12345
}

POST /my-index-000001/_doc?refresh=wait_for
{
  "user_id" : 12346
}
```

使用index API，创建一个新的索引并修改mapping。将`user_id`修改为keyword类型。

```
PUT /my-new-index-000001
{
  "mappings" : {
    "properties": {
      "user_id": {
        "type": "keyword"
      }
    }
  }
}
```

使用 [reindex](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html)  API将文档从旧索引复制到新索引。

```console
POST /_reindex
{
  "source": {
    "index": "my-index-000001"
  },
  "dest": {
    "index": "my-new-index-000001"
  }
}
```



#### **`Rename a field`**

重命名字段会使已在旧字段名称下建立索引的数据无效。但可以添加一个别名字段来创建一个备用字段名。

例如，使用create index API创建`user_identifier`字段。

```
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "user_identifier": {
        "type": "keyword"
      }
    }
  }
}
```



使用put mapping API为现有的`user_identifier`字段添加`user_id`别名。