# Term-level queries
可以使用 term-level queries 根据结构化数据中的精确值查找文档。结构化数据的示例包括日期范围、IP地址、价格或id。

与 full-text queries 不同，term-level queries不分析搜索项。相反，term-level查询与存储在字段中的准确term匹配。

> Term-level queries仍然使用normalizer属性对关键字字段的搜索项进行规范化。有关详细信息，请参见 [normalizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/normalizer.html)。

| query     | description                        |
| --------- | ---------------------------------- |
| exists    | 字段是否存在                       |
| fuzzy     | 模糊查询                           |
| ids       | id查询                             |
| prefix    | 前缀查询                           |
| range     | 区间查询                           |
| regexp    | 正则匹配查询                       |
| term      | term查询，完全匹配。包括大小写     |
| terms     | term多值查询，完全匹配。包括大小写 |
| terms_set | term多值查询，可设置匹配个数       |
| type      | 类型查询，弃用                     |
| wildcard  | 通配符查询?*                       |