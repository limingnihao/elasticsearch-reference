# Painless execute API

Painless excecute API允许执行任意脚本并返回结果。



| Name            | Required | Default         | Description                                   |
| --------------- | -------- | --------------- | --------------------------------------------- |
| `script`        | yes      | -               | The script to execute.                        |
| `context`       | no       | `painless_test` | The context the script should be executed in. |
| `context_setup` | no       | -               | Additional parameters to the context.         |



## Contexts

上下文控制脚本如何执行、运行时哪些变量可用以及返回类型是什么。



### Painless test context

painless_test上下文按原样执行脚本，并且不添加任何特殊参数。唯一可用的变量是params，它可用于访问用户定义的值。脚本的结果总是被转换为字符串。如果没有指定上下文，则默认使用此上下文。

**Example**

Request:

```console
POST /_scripts/painless/_execute
{
  "script": {
    "source": "params.count / params.total",
    "params": {
      "count": 100.0,
      "total": 1000.0
    }
  }
}
```

Response:

```console-result
{
  "result": "0.1"
}
```



### Filter context

`filter`上下文执行脚本，就像在脚本查询中执行脚本一样。出于测试目的，必须提供一个文档，以便在内存中临时建立索引，并可以从脚本访问该文档。更准确地说，此类文档的_source、存储字段和doc值对测试脚本可用。

以下参数可以在筛选器上下文的context_setup中指定:

**document**

包含将在内存中临时建立索引并可从脚本访问的文档。



**index**

包含与被索引的文档兼容的映射的索引的名称。



**Example**

```console
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "field": {
        "type": "keyword"
      }
    }
  }
}

POST /_scripts/painless/_execute
{
  "script": {
    "source": "doc['field'].value.length() <= params.max_length",
    "params": {
      "max_length": 4
    }
  },
  "context": "filter",
  "context_setup": {
    "index": "my-index-000001",
    "document": {
      "field": "four"
    }
  }
}
```



Response:

```console-result
{
  "result": true
}
```



### Score context

`score`上下文执行脚本，就像在function_score查询中的script_score函数中执行脚本一样。

以下参数可以在分数上下文context_setup中指定:

**document**

包含将在内存中临时建立索引并可从脚本访问的文档。



**index**

包含与被索引的文档兼容的映射的索引的名称。



**query**

如果在脚本中使用`_score`，则查询可以指定将使用它来计算分数。



```console
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "field": {
        "type": "keyword"
      },
      "rank": {
        "type": "long"
      }
    }
  }
}


POST /_scripts/painless/_execute
{
  "script": {
    "source": "doc['rank'].value / params.max_rank",
    "params": {
      "max_rank": 5.0
    }
  },
  "context": "score",
  "context_setup": {
    "index": "my-index-000001",
    "document": {
      "rank": 4
    }
  }
}
```



Response:

```console-result
{
  "result": 0.8
}
```