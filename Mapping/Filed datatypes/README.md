# Field datatypes

Elasticsearch文档字段支持的数据类型：

## Core datatypes
|大类|description|说明|
|---|---|---|
|String|text keyword|text会进行分词、keyword不分词|
|Numeric|long, integer, short, byte, double, float, half_float, scaled_float||
|Date|date||
|Date nanoseconds|date_nanos||
|Boolean|boolean||
|Binary|binary|类似base64编号的字符串。不存储不搜索。|
|Range|integer_range, float_range, long_range, double_range, date_range, ip_range||


## Complex datatypes
|type|description|
|---|---|
|Object|object for single JSON objects|
|Nested|nested for arrays of JSON objects|

## Geo datatypes
|type|description|
|---|---|
|Geo-point|geo_point for lat/lon points|
|Geo-shap|geo_shape for complex shapes like polygons|

## Specialised datatypes

|type|description|
|---|---|
|IP|ip for IPv4 and IPv6 addresses|
|Completion datatype|completion to provide auto-complete suggestions|
|Token count|token_count to count the number of tokens in a string|
|mapper-murmur3|murmur3 to compute hashes of values at index-time and store them in the index|
|mapper-annotated-text|annotated-text to index text containing special markup (typically used for identifying named entities)|
|Percolator|Accepts queries from the query-dsl|
|Join|Defines parent/child relation for documents within the same index|
|Rank feature|Record numeric feature to boost hits at query time.|
|Rank features|Record numeric features to boost hits at query time.|
|Dense vector|Record dense vectors of float values.|
|Sparse vector|Record sparse vectors of float values.|
|Search-as-you-type|A text-like field optimized for queries to implement as-you-type completion|
|Alias|Defines an alias to an existing field.|
|Flattened|Allows an entire JSON object to be indexed as a single field.|
|Shape|shape for arbitrary cartesian geometries.|
|Histogram|histogram for pre-aggregated numerical values for percentiles aggregations.|
|Constant keyword|Specialization of keyword for the case when all documents have the same value.|

## Arrays

在Elasticsearch中，数组不需要专门的字段数据类型。默认情况下，任何字段都可以包含零个或多个值，但是数组中的所有值必须具有相同的数据类型。

## Multi-fields
为不同的目的以不同的方式索引相同的字段通常是有用的。例如，可以将字符串字段映射为用于全文本搜索的文本字段，以及用于排序或聚合的关键字字段。或者，您可以使用标准分析器、英语分析器和法语分析器索引文本字段。

这就是multi-fields的目的。大多数数据类型通过fields参数支持多字段。