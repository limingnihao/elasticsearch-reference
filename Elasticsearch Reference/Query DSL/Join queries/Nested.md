## Nested query

包装另一个查询来搜索`nested`的字段。

`nested`查询搜索`nested`字段对象，就像它们作为单独的文档被索引一样。如果一个对象与搜索匹配，`nested`查询将返回父根文档。



### Example request

#### Index setup

要使用`nested`查询，索引必须包含`nested`字段映射。例如:

```
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "obj1": {
        "type": "nested"
      }
    }
  }
}
```



