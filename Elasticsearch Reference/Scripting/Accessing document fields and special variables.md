# Accessing document fields and special variables

根据脚本使用的位置，它可以访问特定的变量和文档字段。



## Update scripts

script 可以用在 [update](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html), [update-by-query](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html), 或 [reindex](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) API 可以使用 `ctx` 变量访问:
|variable|desc|
| ---------------- | ------------------------------------------------------------ |
| `ctx._source`    | Access to the document [`_source` field](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html). |
| `ctx.op`         | The operation that should be applied to the document: `index` or `delete`. |
| `ctx._index` etc | Access to [document meta-fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-fields.html), some of which may be read-only. |



## Search and aggregation scripts

除了每次搜索命中执行一次的脚本字段外，搜索和聚合中使用的脚本将对每个可能匹配查询或聚合的文档执行一次。根据您有多少文档，这可能意味着数百万或数十亿次执行:这些脚本需要更快

可以使用脚本访问的字段 [doc-values](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-fields.html#modules-scripting-doc-vals), [the `_source` field](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-fields.html#modules-scripting-source), 或 [stored fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-fields.html#modules-scripting-stored)



### Accessing the score of a documeng within a script

脚本可以使用在 [`function_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html)，[script-based sorting](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html),， [aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) 可以访问 `_score` 变量表示文档的当前相关性得分。

下面是使用`function_score`查询改变每个文档的相关性得分：

```
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "function_score": {
      "query": {
          "match": {
          "customer_full_name": "Shaw"
        }
      },
      "script_score": {
        "script": {
          "lang": "expression",
          "source": "_score * 1"
        }
      }
    }
  }
}
```



### Doc values

到目前为止，从脚本中访问字段值的最快最有效的方法是使用doc['field name']语法，它从doc-value中检索字段值。doc-value是一个柱状字段值存储，默认情况下，除了analyzed text字段外，所有字段都启用。

doc-value只能返回“简单”的字段值，如数字、日期、地理点、术语等，如果字段是多值的，也只能返回这些值的数组。它不能返回JSON对象。

> Missing fields
>
> 如果映射中缺少字段，doc['field']将抛出一个错误。使用painless时候，首先可以使用doc. containskey ('field')进行检查，以防止访问doc map。不幸的是，在expression脚本中无法检查映射中是否存在该字段。

> Doc values and `text` fields
>
> 如果启用了[`fielddata`](https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html)，那么Doc ['field']语法也可以用于[analyzed `text` fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) ，但是要注意:在text字段上启用fielddata需要将所有的术语加载到JVM堆中，这在内存和CPU方面都非常昂贵。从脚本中访问文本字段很少有意义。



### The document _source

可以访问文档的[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html)，类似_source.field_name的语法。_source是作为map-of-maps加载的，因此可以访问对象字段中的属性，例如，`_source.name.first`。

> Prefer doc-values to _source
>
> 访问_source字段要比使用doc-values慢得多。__source字段被优化为每个结果返回多个字段，而doc values被优化为访问许多文档中特定字段的值。
>
> 在为搜索结果中的前10个搜索结果生成脚本字段时，使用_source是有意义的，但是对于其他搜索和聚合用例，总是更喜欢使用doc值。



### Stored field

显式标记为"store:true的字段，可以使用_fields['field name'].value 或 _fields['field name']。



> Stored vs _source
>
> `_source`字段只是一个特殊的stored字段，因此性能与其他stored字段类似。` _source`提供了对已建立索引的原始文档主体的访问(包括区分空值与空字段、单值数组与普通标量等的能力)。
>
> 只有在`_source`非常大且访问几个较小的存储字段(而不是整个`_source`)成本更低的情况下，才真正有意义地使用存储字段而不是`_source`字段。

