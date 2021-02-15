# Search API

返回与请求中定义的查询匹配的搜索结果。

```
GET /my-index-000001/_search
```



###  Request

```
GET /<target>/_search
GET /_search
POST /<target>/_search
POST /_search
```



### Description

允许您执行搜索查询并返回与查询匹配的搜索结果。您可以使用q查询字符串参数或request body提供搜索查询。



### Path parameters

#### <target>

(Optional, string) 要搜索的数据流、索引和索引别名的逗号分隔列表。支持通配符(*)表达式。

要搜索集群中的所有数据流和索引，请忽略此参数或使用all或*。



### Query parameters

> IMPORTANT
>
> url参数。
>
> 此API的几个选项，可以使用query参数或request body参数。如果两个都指定了参数，则只使用query参数(url参数)。

| parameter        | Optional | Type | Default | Description | example |
| :----------- | ------------------------- | ------- | ------------ | ------------ | ------------ |
| **allow_no_indices** | Optional | boolean | true | 如果为true，则如果通配符表达式或`_all`只检索丢失或关闭的索引，则请求不返回错误。 |  |
| **allow_partial_search_results** | Optional | boolean | true | 如果为true，则在存在请求超时或分片失败时返回部分结果。如果为false，则返回不包含部分结果的错误。默认值为true。 |  |
| **batched_reduce_size** | Optional | integer | 512 | 每个shard最多返回的数量。 |  |
| **ccs_minimize_roundtrips** | Optional | boolean |  |  |  |
| **docvalue_fields** | Optional | string |  |  |  |
| **expand_wildcards** | Optional | string |  |  |  |
| **explain** | Optional | boolean |  | 返回关于分数计算的详细信息 |  |
| **ignore_throttled** | Optional | boolean | true | true：concrete, expanded or aliased indices将被忽略 |  |
| **ignore_unavailable** | Optional | boolean | false | true：响应中不包括缺失或关闭的索引。 |  |
| **max_concurrent_shard_requests** | Optional | integer | 5 | 定义此搜索可并发执行的每个节点并发切分请求的数量。这个值应该用于限制搜索对集群的影响，从而限制并发切分请求的数量。 |  |
| **pre_filter_shard_size** | Optional | integer |  |  |  |
| **preference** | Optional | String |  | 指定应该在其上执行操作的节点或shard。默认随机。 | preference=_shards:2,3 |
| **q** | Optional | String |  | 使用Lucene query语法 |  |
| **request_cache** | Optional | Boolean |  | 对大小为0的请求启用了搜索结果缓存。 |  |
| **rest_total_hits_as_int** | Optional | Boolean | false |  |  |
| **routing** | Optional | String |  | 指定主shard |  |
| **scroll** | Optional | time value |  | 期间保留搜索上下文scrolling。这个值不能超过1d(24小时)。您可以使用search.max_keep_alive更改此限制。集群级别。 |  |
| **search_type** | Optional | String |  | query_then_fetch、dfs_query_then_fetch |  |
| **seq_no_primary_term** | Optional | |  |  |  |
| **sort** | Optional | String |  | <field>:<direction> 排序 |  |
| **_source** | Optional | String | true | 指示为匹配的文档返回哪些源字段。这些字段在hits中返回。默认值为true。 |  |
| **_source_excludes** | Optional | |  |  |  |
| **_source_includes** | Optional | |  |  |  |
| **stats** | Optional | |  |  |  |
| **stored_fields** | Optional | |  |  |  |
| **suggest_field** | Optional | |  | 指定用于建议的字段。 |  |
| **suggest_text** | Optional | |  | 应该返回建议的源文本。 |  |
| **terminate_after** | Optional | integer | 0 | 为每个shard收集的最大文档数，达到此数目后查询执行将提前终止 |  |
| **timeout** | Optional | time units |  | 指定等待响应的时间。如果在超时到期前没有收到响应，则请求失败并返回错误。默认没有超时。 |  |
| **track_scores** | Optional | Boolean | false | 计算并返回文档分数，即使这些分数不用于排序。 |  |
| **track_total_hits** | Optional | Boolean/integer |  |  |  |
| **typed_keys** | Optional | |  |  |  |
| **version** | Optional | Boolean | False | 返回一个_version字段。                                       |  |
| **from** | Optional | integer | 0 | 文档开始的偏移量。需要是非负的，默认值为0 |  |
| **size** | Optional | integer | 10 | 默认情况下，不能使用from和size参数翻页超过10,000个文档。[`index.max_result_window`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-result-window)索引设置 |  |



### Request body

请求体参数

| parameter        | Optional | Type | Default | Description | example |
| ---- | ---- | ---- | ---- | ---- | ---- |
| **docvalue_fields** | Optional | array of strings and objects |      |      |      |
| **explain** | Optional | Bollean | false | 返回关于分数计算的详细信息。 |      |
| **from** | Optional | integer | 0 |      |      |
| **size** | Optional | integer | 10 |      |      |
| **query** | Optional | query object |      | [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html)语法 | "query": {<br/>    "match_all": {}<br/>  } |
| **seq_no_primary_term** | Optional | Bollean |      |      |  |
| **_source** | Optional |      |      | 指示为匹配文档返回哪些源字段。字段在hits._source中返回。 | "_source": ["traceID", "spanID"], |
| **terminate_after** | Optional | integer | 0 | 为每个切分收集的最大文档数，达到此数目后查询执行将提前终止。 | "terminate_after": "20" |
| **timeout** | Optional | time units | | 超时时间 | "timeout": "1s" |
| **version** | Optional | Boolean | false | 返回一个_version字段。 | "version": true |



### Response body

| field      | Type    | Description |
| ---------- | ------- | ----------- |
| _scroll_id | string  |             |
| took       | integer | 毫秒耗时。 |
| timed_out  | boolean | true表示：请求在完成前超时;返回的结果可能是部分的或空的。 |
|terminated_early|Boolean||
| _shards    | object | shard请求的计数器 |
| _shards.total | integer | 需要查询的shard总数，包括未分配的shard。 |
| _shards.successful | integer | 成功执行请求的shard数 |
| _shards.skipped | integer | 跳过请求的shard数，因为轻量级检查时可以认识到这个shard上可能没有需要的文档。这通常发生在搜索请求包含范围过滤器，而shard只有在该范围之外的文档时。 |
| _shards.failed | integer | 未能执行请求的shard数。注意，未分配的shard将被认为既不成功也不失败。因此，failed+successful小于total，表明一些碎片没有被分配。 |
| hits | objects | 返回的文档对象的数组。 |
| hits.total.value | integer | 返回的文档总数 |
| hits.total.relation | string | eq：准确的。gte：下限，包括返回的文档 |
| hits.max_score | float |  |
| hits.hits |  |  |

