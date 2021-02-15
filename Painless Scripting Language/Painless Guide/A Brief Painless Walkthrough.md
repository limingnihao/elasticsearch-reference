# A Brief Painless Walkthrough

为了说明Painless是如何工作的，让我们将一些曲棍球统计数据加载到Elasticsearch索引中：

```console
PUT hockey/_bulk?refresh
{"index":{"_id":1}}
{"first":"johnny","last":"gaudreau","goals":[9,27,1],"assists":[17,46,0],"gp":[26,82,1],"born":"1993/08/13"}
{"index":{"_id":2}}
{"first":"sean","last":"monohan","goals":[7,54,26],"assists":[11,26,13],"gp":[26,82,82],"born":"1994/10/12"}
{"index":{"_id":3}}
{"first":"jiri","last":"hudler","goals":[5,34,36],"assists":[11,62,42],"gp":[24,80,79],"born":"1984/01/04"}
{"index":{"_id":4}}
{"first":"micheal","last":"frolik","goals":[4,6,15],"assists":[8,23,15],"gp":[26,82,82],"born":"1988/02/17"}
{"index":{"_id":5}}
{"first":"sam","last":"bennett","goals":[5,0,0],"assists":[8,1,0],"gp":[26,1,0],"born":"1996/06/20"}
{"index":{"_id":6}}
{"first":"dennis","last":"wideman","goals":[0,26,15],"assists":[11,30,24],"gp":[26,81,82],"born":"1983/03/20"}
{"index":{"_id":7}}
{"first":"david","last":"jones","goals":[7,19,5],"assists":[3,17,4],"gp":[26,45,34],"born":"1984/08/10"}
{"index":{"_id":8}}
{"first":"tj","last":"brodie","goals":[2,14,7],"assists":[8,42,30],"gp":[26,82,82],"born":"1990/06/07"}
{"index":{"_id":39}}
{"first":"mark","last":"giordano","goals":[6,30,15],"assists":[3,30,24],"gp":[26,60,63],"born":"1983/10/03"}
{"index":{"_id":10}}
{"first":"mikael","last":"backlund","goals":[3,15,13],"assists":[6,24,18],"gp":[26,82,82],"born":"1989/03/17"}
{"index":{"_id":11}}
{"first":"joe","last":"colborne","goals":[3,18,13],"assists":[6,20,24],"gp":[26,67,82],"born":"1990/01/30"}
```



###  Accessing Doc Values from Painless

可以从名为doc的映射中访问文档值。

例如，下面的脚本计算玩家的总目标。这个例子使用了强类型的int和for循环。

```console
GET hockey/_search
{
  "query": {
    "function_score": {
      "script_score": {
        "script": {
          "lang": "painless",
          "source": """
            int total = 0;
            for (int i = 0; i < doc['goals'].length; ++i) {
              total += doc['goals'][i];
            }
            return total;
          """
        }
      }
    }
  }
}
```



或者，您也可以使用script field代替function score来做同样的事情

```console
GET hockey/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "total_goals": {
      "script": {
        "lang": "painless",
        "source": """
          int total = 0;
          for (int i = 0; i < doc['goals'].length; ++i) {
            total += doc['goals'][i];
          }
          return total;
        """
      }
    }
  }
}
```



下面的示例使用了一个简单的脚本，根据玩家的姓和名的组合对他们进行排序。使用`doc['first'].value`和`doc['last'].value`。

```console
GET hockey/_search
{
  "query": {
    "match_all": {}
  },
  "sort": {
    "_script": {
      "type": "string",
      "order": "asc",
      "script": {
        "lang": "painless",
        "source": "doc['first.keyword'].value + ' ' + doc['last.keyword'].value"
      }
    }
  }
}
```



### Missing values

`doc['field'].value`如果文档中缺少字段，则抛出异常。

要检查文档是否缺少一个值，可以调用doc['field'].size() == 0。



#### Updating Fields with Painless

您还可以轻松地更新字段。您可以访问字段的原始源为`ctx._source.<field-name>`。

首先，让我们通过提交以下请求来查看播放器的源数据

```console
GET hockey/_search
{
  "query": {
    "term": {
      "_id": 1
    }
  }
}
```



要将球员的姓氏改为hockey，只需将ctx._source.last设置为新值

```console
POST hockey/_update/1
{
  "script": {
    "lang": "painless",
    "source": "ctx._source.last = params.last",
    "params": {
      "last": "hockey"
    }
  }
}
```



您还可以向文档添加字段。例如，这个脚本添加了一个包含球员昵称hockey的新字段。



### Dates

日期字段被公开为ZonedDateTime，因此它们支持像getYear, getDayOfWeek或例如，使用getMillis获取自epoch以来的毫秒数。要在脚本中使用这些方法，请省略get前缀，并继续将方法名的其余部分小写。例如，下面的语句返回每个冰球运动员的出生年份:

```console
GET hockey/_search
{
  "script_fields": {
    "birth_year": {
      "script": {
        "source": "doc.born.value.year"
      }
    }
  }
}
```



### Regular expressions

默认情况下，regex是开启的，在Setting中有一个新配置：`script.painless.regex.enabled`，limited，默认。这默认使用正则表达式，但限制了正则表达式的复杂性。看似无害的正则表达式可能具有惊人的性能和堆栈深度行为。但它们仍然是一种强大的工具。此外，对于limited，可以将该设置设置为true，就像前面一样，这允许使用正则表达式，而不限制它们。可以在`elasticsearch.yml`中开启`script.painless.regex.enabled: true`

对正则表达式的原生支持具有语法结构:

* `/pattern/`：See [Pattern flags](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-regexes.html#pattern-flags) for more.
* `=~`：find操作符返回一个布尔值，如果文本的子序列匹配，则返回true，否则返回false。
* ==~：match操作符返回一个布尔值，匹配则返回true，不匹配则返回false。

