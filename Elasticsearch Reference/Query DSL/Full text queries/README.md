## Full Text Queries

>原文地址：https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html

- --
全文检索是让你能够搜索被 [analyzed](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) 的文本字段，例如email正文。查询的字符串必须使用和indexing期间相同的analyzed进行处理。


这组的查询有：
| query                     | description                                                  | 备注                                   |
| ------------------------- | ------------------------------------------------------------ | -------------------------------------- |
| intervals query           | 允许对匹配项的顺序和邻近性进行细粒度控制的全文查询。         |                                        |
| match query               | 执行全文查询的标准查询，包括模糊匹配和短语或邻近查询。       | 分词查询，命中个数、编辑记录都可以配置 |
| match_bool_prefix query   | 创建一个bool查询，为每个term创建term query，但最后一个词除外，最后一个作为prefix查询匹配。 |                                        |
| match_phrase query        | 与match查询类似，但用于匹配确切的短语或单词接近匹配。        | 分词，必须全命中，顺序一致，无距离     |
| match_phrase_prefix query | 类似于match_phrase查询，但是对最后一个单词进行通配符搜索。   |                                        |
| multi_match query         | match查询的多字段版本。                                      | 以上几种查询的多字段版本               |
| common terms query        | 一个更专业的查询，它对不常见的单词给予更多的偏爱。           |                                        |
| query_string query        | 支持简洁的Lucene 查询字符串语法 ，使您可以在单个查询字符串中指定AND | OR                                     |
| simple_query_string query | query_string语法的更简单、更健壮的版本，适合直接向用户公开。 |                                        |