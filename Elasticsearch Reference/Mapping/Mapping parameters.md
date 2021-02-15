# Mapping parameters


接受以下参数:

| 参数名称                   | 应用类型      | 说明                                                         | 举例 |
| :------------------------- | ------------- | ------------------------------------------------------------ | ---- |
| analyzer                   | text          | 在索引或搜索时用于文本字段(除非被search_analyzer覆盖)。默认为默认索引分析器或standard analyzer |      |
| boost                      | text、keyword、numeric、date、boolean | 查询时进行boosting。接收float类型，默认值为1.0               |      |
| coerce                     | numeric |                                                              |      |
| copy_to                    |               |                                                              |      |
| doc_values                 | keyword、numeric、date、boolean、binary | 字段是否应该以column-stride格式存储在磁盘上，以便以后用于排序、聚合或脚本编写?接受true(默认)或false。 |      |
| dynamic                    |               |                                                              |      |
| eager_global_ordinals      | text、keyword | 是否在刷新的时候加载全局序数。默认false。经常使用聚合查询的字段推荐开启。 |      |
| enabled |  |  | |
| fielddata                  | text          | 字段可以使用内存的fielddata进行排序、聚合、脚本执行。接受true或false(默认)。 |      |
| fielddata_frequency_filter | text          | 专家设置，允许决定在启用fielddata时在内存中加载哪些值。默认情况下，加载所有值。 |      |
|fields|text、keyword|multi-fields允许为不同的目的以多种方式对相同的字符串值进行索引，例如一个字段用于搜索，一个multi-field用于排序和聚合，或者由不同的分析程序分析相同的字符串值。||
|format|date|解析日期的格式。默认为strict_date_optional_time||
|ignore_above|keyword|不要索引任何超过这个值的字符串。默认值为2147483647，以便接受所有值。但是请注意，默认动态映射规则创建了一个子关键字字段，通过设置ignore_above: 256覆盖这个默认值。||
|ignore_malformed|numeric、date|如果为true，则忽略格式不正确的数字。如果为false(默认值)，格式不正确的数字将抛出异常并拒绝整个文档。||
|index|text、keyword、numeric、date、boolean|字段是否可以被搜索？默认true||
|index_options|text、keyword|索引中应该存储哪些信息，以便进行搜索和高亮显示。默认为positions。（docs、freqs、offsets）||
|index_prefixes|text|如果启用，在2到5个字符之间的term前缀将被索引到一个单独的字段中。这允许前缀搜索以更大的索引为代价更有效地运行。||
|index_phrases|text|如果启用，两个term单词组合(shingles)将被索引到一个单独的字段中。这允许精确的短语查询(没有slop)以更大的索引为代价更有效地运行。注意，当stopwords没有删除时，这是最有效的，因为包含stopwords的短语不会使用附属字段，而是返回到标准短语查询。接受true或false(默认)。||
|normalizer|keyword|如何预处理关键字在索引之前。默认值为null，意味着关键字保持原样。||
|norms|text、keyword|在对查询进行评分时是否应该考虑字段长度。接受true或false(默认)。||
|null_value|keyword、numeric、date、boolean|接受替换为任何显式空值的字符串值。默认值为null，这意味着该字段被视为缺失。||
|position_increment_gap|text|插入到字符串数组的每个元素之间的伪term位置。默认值为100。之所以算则100，是因为防止具有较大slop（小于100）的跨字段term匹配。||
|properties||||
|store|text、keyword、numeric、date、boolean、binary|字段值是否应该与_source字段分开存储和检索。接受true或false(默认)。||
|search_analyzer|text|在文本字段上搜索时应该使用的分析器。默认使用analyzer设置。||
|search_quote_analyzer|text|当遇到短语时，应该在搜索时使用的分析器。默认设置为search_analyzer。||
|similarity|text、keyword|应该使用哪种评分算法或相似性。默认为BM25。||
|term_vector|text|是否存储字段的项向量。默认为没有。||
|split_queries_on_whitespace|keyword|在全文搜索时是否使用空格切分输入。接受true或false(默认)。||
|meta|text、keyword、numeric、date、boolean|关于字段的元数据。||
|locale|date|The locale to use when parsing dates since months do not have the same names and/or abbreviations in all languages. The default is the [`ROOT` locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#ROOT),||

