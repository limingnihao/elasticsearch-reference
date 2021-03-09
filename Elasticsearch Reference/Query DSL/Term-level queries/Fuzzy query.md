
## Fuzzy query

返回包含与搜索项类似的项的文档，这些项由[Levenshtein编辑距离](http://en.wikipedia.org/wiki/Levenshtein_distance)度量。

编辑距离是将一个术语转换成另一个术语所需的单字符更改数。这些变化包括:
* Changing a character (box → fox)
* Removing a character (black → lack)
* Inserting a character (sic → sick)
* Transposing two adjacent characters (act → cat)

为了找到相似的术语，模糊查询创建了一组搜索术语在指定编辑距离内的所有可能变化或扩展。然后，查询返回每个扩展的精确匹配。

### Example requests
```
GET /test/_search
{
    "query": {
        "fuzzy": {
            "name": {
                "value": "abc",
                "fuzziness": "AUTO",
                "max_expansions": 50,
                "prefix_length": 0,
                "transpositions": true,
                "rewrite": "constant_score"
            }
        }
    }
}
```

### Parameters of <field>

* value。您希望查询的term。
* fuzziness。最大变成距离。参考 [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#fuzziness).
* max_expansions。创建的最大变化数。默认50。
> 避免在max_expansions参数中使用高值，特别是当prefix_length参数值为0时。max_expansions参数中的高值会导致性能低下，因为要检查的变量太多。
* prefix_length。创建扩展时未改变的起始字符数。默认值为0。
* transpositions。表示编辑是否包含两个相邻字符(ab→ba)的交换。默认值为true。
* rewrite。方法用于重写查询。有关有效值和更多信息，请参见[rewrite parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)。

