## Request Body Search

- patch parameters <index>。 索引名称
- request body。[request body 参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#search-search-api-request-body).
- Fast check for any matching docs。可以设置terminate_after设置每个shard返回的文档数。或设置size=0，快速返回。
- Doc value fields。See [Doc value fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#docvalue-fields).
- Field collapsing。See [*Collapse search results*](https://www.elastic.co/guide/en/elasticsearch/reference/current/collapse-search-results.html).
- Highlighting。See [*Highlighting*](https://www.elastic.co/guide/en/elasticsearch/reference/current/highlighting.html).
- Index Boost。多索引权重。
- Inner hits。See [*Retrieve inner hits*](https://www.elastic.co/guide/en/elasticsearch/reference/current/inner-hits.html).
- min_score。最低分数文档cutoff。
- Named queries。See [Named queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html#named-queries).
- Post filter。filter query查询。
- Preference。自定义shard查询。preference=_shards:2,3。
- Rescoring。精排。如果sort不用_score将报错。尽量不要同时使用from翻页。
    - Query rescorer.
    - Multiple Resources.
- Script Fields。See [Script fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#script-fields).
- Scroll。
    - See [Scroll search results](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#scroll-search-results).
    - Clear scroll API. See [Clear scroll](https://www.elastic.co/guide/en/elasticsearch/reference/current/clear-scroll-api.html).
    - Sliced scroll. See [Sliced Scroll](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#slice-scroll).
- Search After。See [Search after](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html#search-after).
- Search Type.
    - Query Then Fetch。search_type=query_then_fetch。默认。每个shard独立计算得分，然后聚合到协调节点排序。
    - Dfs, Query Then Fetch。search_type=dfs_query_then_fetch。计算全局的dfs（所有shard）。
- Sort。See [*Sort search results*](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html).
- Source filtering。See [Source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#source-filtering).
- Stored fields。See [Stored fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#stored-fields).
- Track total hits。
    - track_total_hits=false。不显示命中文档数。
    - track_total_hits=true。精确显示命中文档数。
    - track_total_hits=100。total.value返回100表示，至少有100个文档匹配。返回42表示只有42个文档。



### Query rescorer

Query rescorer只对查询和筛选后，返回的Top-K结果执行第二个查询。每个shard上检查的文档数量可以通过`window_size`大小参数来控制，默认值是10。

默认情况下，原始查询和rescore查询的分数被线性组合，从而为每个文档生成最终分数。原始查询和rescore查询的相对重要性可以分别通过`query_weight`和`rescore_query_weight`来控制。两者都默认为1。





|      |      |      |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |

