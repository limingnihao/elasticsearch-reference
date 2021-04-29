## Text datatype

索引全文值的字段，如电子邮件正文或产品说明。对这些字段进行analyzed，即在索引之前，通过分析器将字符串转换为单个terms列表。分析过程允许Elasticsearch在每个全文域中搜索单个单词。文本字段不用于排序，也很少用于聚合(尽管重要的文本聚合是一个明显的例外)。

如果您需要索引结构化内容，如电子邮件地址、主机名、状态代码或标记，则很可能应该使用keyword field。

下面是一个text字段的mapping示例：
```
PUT my_index
{
  "mappings": {
    "properties": {
      "full_name": {
        "type":  "text"
      }
    }
  }
}
```

### Use a field as both text and keyword
有时，同时拥有同一个字段的全文(text)和关键字(keyword)版本是很有用的:一个用于全文搜索，另一个用于聚合和排序。这可以通过multi-fields来实现。

### Parameters for text fields
文本字段接受以下参数:
|参数|说明|
|---|---|
|analyzer|在索引或搜索时用于文本字段(除非被search_analyzer覆盖)。默认为默认索引分析器或standard analyzer|
|boost|查询时进行boosting。接收float类型，默认值为1.0|
|eager_global_ordinals|是否在刷新的时候加载全局序数。默认false。经常使用聚合查询的字段推荐开启。|
|fielddata|字段可以使用内存的fielddata进行排序、聚合、脚本执行。接受true或false(默认)。|
|fielddata_frequency_filter|专家设置，允许决定在启用fielddata时在内存中加载哪些值。默认情况下，加载所有值。|
|fields|Multi-fields允许为不同的目的以多种方式对相同的字符串值进行索引，例如一个字段用于搜索，一个multi-field用于排序和聚合，或者由不同的分析程序分析相同的字符串值。|
|index|字段是否可以被搜索？默认true|
|index_options|索引中应该存储哪些信息，以便进行搜索和高亮显示。默认为positions。（docs、freqs、offsets）|
|index_prefixes|如果启用，在2到5个字符之间的term前缀将被索引到一个单独的字段中。这允许前缀搜索以更大的索引为代价更有效地运行。|
|index_phrases|如果启用，两个term单词组合(shingles)将被索引到一个单独的字段中。这允许精确的短语查询(没有slop)以更大的索引为代价更有效地运行。注意，当stopwords没有删除时，这是最有效的，因为包含stopwords的短语不会使用附属字段，而是返回到标准短语查询。接受true或false(默认)。|
|norms|在对查询进行评分时是否应该考虑字段长度。接受true或false(默认)。|
|position_increment_gap|插入到字符串数组的每个元素之间的伪term位置。默认值为100。之所以算则100，是因为防止具有较大slop（小于100）的跨字段term匹配。|
|store|字段值是否应该与_source字段分开存储和检索。接受真或假(默认)。|
|search_analyzer|在文本字段上搜索时应该使用的分析器。默认使用analyzer设置。|
|search_quote_analyzer|当遇到短语时，应该在搜索时使用的分析器。默认设置为search_analyzer。|
|similarity|应该使用哪种评分算法或相似性。默认为BM25。|
|term_vector|是否存储字段的项向量。默认为没有。|
|meta|关于字段的元数据。|