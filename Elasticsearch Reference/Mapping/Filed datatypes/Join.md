## Join field type

join数据类型是一个特殊的字段，它在相同索引的文档中创建parent/child关系。在文档中定义了一组关系，每个关系分别是parent名和child名。parent/child关系可以定义如下:

```
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "my_id": {
        "type": "keyword"
      },
      "my_join_field": { ①
        "type": "join",
        "relations": {
          "question": "answer" ②
        }
      }
    }
  }
}
```

* ① The name for the field

* ② 定义一个单独的关系，其中question是answer的父关系。



要用join为文档建立索引，必须在source中提供关联的name和parent。例如，下面的示例在question上下文中创建了两个parent文档。

```
PUT my-index-000001/_doc/1?refresh
{
  "my_id": "1",
  "text": "This is a question",
  "my_join_field": {
    "name": "question" 
  }
}

PUT my-index-000001/_doc/2?refresh
{
  "my_id": "2",
  "text": "This is another question",
  "my_join_field": {
    "name": "question"
  }
}
```



当索引child时，必须在_scoure中添加关系的name以及文档的parent id。

```
PUT my-index-000001/_doc/3?routing=1&refresh ①
{
  "my_id": "3",
  "text": "This is an answer",
  "my_join_field": {
    "name": "answer", ②
    "parent": "1" ③
  }
}

PUT my-index-000001/_doc/4?routing=1&refresh
{
  "my_id": "4",
  "text": "This is another answer",
  "my_join_field": {
    "name": "answer",
    "parent": "1"
  }
}
```



* ① 路由值是强制性的，因为父文档和子文档必须在同一个分片上建立索引。

* ② `answer`是本文档的连接名称。

* ③ 这个子文档的parent id。



### Parent-join and performance(性能)

join字段不应该像在关系数据库中的join那样使用。在Elasticsearch中，获得良好性能的关键是将数据反规范化为文档。每个join字段、has_child或has_parent查询都会给查询性能增加很大的负担。它还可以触发建立 [global ordinals](https://www.elastic.co/guide/en/elasticsearch/reference/current/eager-global-ordinals.html)。



### Parent-join restrictions(限制)

* 每个索引只允许一个join字段映射。
* parent文档和child文档必须在同一个分片上建立索引。这意味着在获取、删除或更新子文档时需要提供相同的路由值。
* 一个元素可以有多个子元素，但只能有一个父元素。
* 可以向现有的join字段添加新的关系。
* 也可以向现有元素添加child元素，但前提是该元素已经是父元素。



### Searching with parent-join

parent-join创建一个字段来索引文档中关系的名称(`my_parent`, `my_child`, …)。

它还为每个parent/child关系创建一个字段。该字段的名称是join字段的名称，后跟#和关系中的父字段的名称。