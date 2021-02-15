# How to use scripts

Elasticsearch API支持脚本的地方，语法遵循相同的模式:

```
  "script": {
    "lang":   "...",  1
    "source" | "id": "...", 2
    "params": { ... } 3
  }
```

1. 编写脚本所用的语言，默认情况下是`painless`。

2. 
   脚本本身可以指定为内联脚本的`source`或存储脚本的`id`。

3. 
   应该传递到脚本中的任何命名参数。



例如，下面的脚本用于搜索请求返回脚本字段:

```
PUT my_index/_doc/1
{
  "my_field": 5
}

GET my_index/_search
{
  "script_fields": {
    "my_doubled_field": {
      "script": {
        "lang":   "expression",
        "source": "doc['my_field'] * multiplier",
        "params": {
          "multiplier": 2
        }
      }
    }
  }
}
```



## Script parameters

###### lang

指定编写脚本所用的语言。默认为`painless`

###### source,id

指定脚本的源。内联脚本的指定源如上面的示例所示。存储脚本被指定为id，并从集群状态中检索(请参阅 [Stored Scripts](#stored-scripts))。

###### params

指定作为变量传递到脚本中的任何命名参数。



> IMPORTANT Prefer parameters

当Elasticsearch第一次看到一个新的脚本时，它将编译它并将编译后的版本存储在缓存中。编译可能是一个繁重的过程。

如果您需要将变量传递到脚本中，您应该将它们作为指定的参数传递，而不是将硬编码值编码到脚本本身中。例如，如果您希望能够将字段值乘以不同的乘数，那么不要将乘数硬编码到脚本中:

```
"source": "doc['my_field'] * 2"
```

相反，将其作为命名参数传递进来

```
"source": "doc['my_field'] * multiplier",
"params": {
    "multiplier": 2
}
```

第一个版本每次multiplier变化时，都必须重新编译。第二个版本只编译一次。

如果您在很短的时间内编译了太多的唯一脚本，Elasticsearch将拒绝新的动态脚本，并会出现`circuit_breaking_exception`错误。默认情况下，大多数contexts每5分钟编译75个脚本，ingest contexts每5分钟编译375个脚本。你可以修改次此设置，`script.context.$CONTEXT.max_compilations_rate` 例如： `script.context.field.max_compilations_rate=100/10m`.



## Short script from

为了简洁，可以使用简短的脚本形式。在简短的形式中，脚本是由字符串而不是对象表示的。这个字符串包含脚本的源。

```
"script": "ctx._source.likes++"
```



正常形式的相同脚本：

```
"script": {
    "source": "ctx._source.likes++"
  }
```



## Stored scripts

脚本可以保存到服务器。

如果启用了Elasticsearch安全特性，则必须具有以下权限来创建、检索和删除存储的脚本

* cluster: `all` or `manage`

For more information, see [Security privileges](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-privileges.html).



### Request examples

下面是使用存储脚本的示例，使用`/_scripts/{id}`。

首先，在集群状态中创建名为calculate-score的脚本：

```
POST _scripts/calculate-score
{
  "script": {
    "lang": "painless",
    "source": "Math.log(_score * 2) + params.my_modifier"
  }
}
```

您还可以将在url指定编译存储的脚本：

```
POST _scripts/calculate-score/score
{
  "script": {
    "lang": "painless",
    "source": "Math.log(_score * 2) + params.my_modifier"
  }
}
```



可以使用：

```
GET _scripts/calculate-score
```



可以通过如下方式指定id参数来使用存储脚本：

```
GET twitter/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
            "message": "some message"
        }
      },
      "script": {
        "id": "calculate-score",
        "params": {
          "my_modifier": 2
        }
      }
    }
  }
}
```



删除：

```
DELETE _scripts/calculate-score
```





## Search templates

您还可以使用`_scripts` API来存储搜索模板。搜索模板使用占位符值保存特定的搜索请求，称为模板参数。

您可以使用存储的搜索模板来运行搜索，而无需写出整个查询。只需提供存储的模板ID和模板参数。当您希望快速且无错误地运行一个常用查询时，这是非常有用的。

搜索模板使用 [mustache templating language](http://mustache.github.io/mustache.5.html)。 更多信息 [Search Template](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-template.html) 。



## Script caching

默认情况下，所有脚本都被缓存，因此它们只需要在更新发生时重新编译。默认情况下，脚本没有基于时间的过期，但是您可以通过使用script.cache来更改此行为。设置到期。您可以使用脚本.cache来配置这个缓存的大小。最大大小设置。默认情况下，缓存大小为100。



> 脚本的大小被限制为65,535字节。这可以通过设置`cript.max_size_in_bytes`改变。但如果脚本真的很大，那么应该考虑使用 [native script engine](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-engine.html)。



## Query and script

query和script结合使用

```
GET kibana_sample_data_ecommerce/_search
{
    "_source": {
  	  "include": ["customer_first_name", "taxful_total_price"]
    },	
    "query": {
        "bool": {
            "should": [
                { "term": { "customer_first_name.keyword": "Clarice"  } },
                { "term": { "customer_first_name.keyword": "Pia"  } }
            ],
            "minimum_should_match": 1
        }
    },
    "script_fields": {
      "my_doubled_field": {
        "script": {
          "lang":   "expression",
          "source": "doc['taxful_total_price'] * multiplier",
          "params": {
            "multiplier": 2
          }
        }
      }
    }
}
```



输出的结果为：

```
{
    "_index": "kibana_sample_data_ecommerce",
    "_type": "_doc",
    "_id": "ECvQlHMBmG29mOFQ3esV",
    "_score": 3.5636153,
    "_source": {
        "customer_first_name": "Clarice",
        "taxful_total_price": 34.98
    },
    "fields": {
        "my_doubled_field": [
            69.9375
        ]
    }
}
```



## Scripts and search speed

脚本不能使用Elasticsearch的索引结构或相关优化。这有时会导致搜索速度变慢。

如果您经常使用脚本转换索引数据，那么可以通过在摄取过程中进行这些更改来加快搜索速度。然而，这通常意味着索引速度变慢。



### Example

一个索引, `my_test_scores`, 包含两个 `long` 字段:

- `math_score`
- `verbal_score`

在运行搜索时，用户通常使用脚本根据这两个字段值的和对结果进行排序。

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_script": {
        "type": "number",
        "script":{
          "source": "doc['taxful_total_price'].value + doc['taxless_total_price'].value"
        },
        "order": "desc"
      }
    }
  ]
}
```



为了加快搜索速度，您可以在ingest过程中执行这个计算，并将求和索引到一个字段。