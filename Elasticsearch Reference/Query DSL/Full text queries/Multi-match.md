# Multi-match query

multi_match query建立在match query的基础上，支持多字段查询:
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }
}
```

### fields

fields可以使用通配符*_name：
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "Will Smith",
      "fields": [ "title", "*_name" ] 
    }
  }
}
```

字段可以使用^设置权重：
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query" : "this is a test",
      "fields" : [ "subject^3", "message" ]
    }
  }
}
```
>subject字段的重要性是message字段的三倍。

如果没有提供字段，multi_match查询将默认为index.query.default_field索引设置，该设置从默认查询*:*中提取映射中符合条件的所有字段，并过滤元数据字段。然后将所有提取的字段组合起来构建一个查询。

>一次可以查询的字段的数量是有限制的。它是由 [search-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-settings.html)中的indices.query.bool.max_clause_count确定，默认为1024。

## multi_match查询类型
multi_match查询的执行方式取决于type参数，可将其设置为：

|type|说明| 转换对应语法 |
|---|---|---|
|best_fields|(默认)查找与任意字典匹配的文档，_score使用最佳字段| dis_max (match)|
|most_fields| 查找任何匹配文档的字段，并组合每个字段的_score| should(match)|
|phrase| 在每个字段上运行match_phrase，_score使用最佳字段|dis_max(match_phrase)|
|phrase_prefix| 在每个字段上运行match_phrase，_score使用最佳字段|dis_max(match_phrase_prefix)|
|cross_fields|用同一个analyzer处理字段，就好像它们是一个大字段一样。在任何字段中查找每个单词。| |
|bool_prefix|在每个字段上创建一个match_bool_prefix查询，并组合每个字段的_score。| |





## best_field
best_field通常用于在查询多个单词时找到单个最佳匹配字段。例如"brown fox"在一个字段中，要比"brown"在一个字段而"fox"在另一个字段更好。

best_field将为每个字段生成一个[match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html), 并将它们包装在[dis_max](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-dis-max-query.html)查询，以找到单个最佳匹配字段。例如：
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "brown fox",
      "type":       "best_fields",
      "fields":     [ "subject", "message" ],
      "tie_breaker": 0.3
    }
  }
}
```
被转换为：
```
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "subject": "brown fox" }},
        { "match": { "message": "brown fox" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

best_fields类型通常使用单个最佳匹配字段的得分为最后得分，但如果指定了 tie_breaker，则计算得分如下:
* tie_breaker=0。 来自最佳匹配字段的分数。
* 0< tie_breaker < 1。所有匹配字段 tie_breaker * _score 之和。

并可以使用analyzer, boost, operator, minimum_should_match, fuzziness, lenient, prefix_length, max_expansions, rewrite, zero_terms_query, cutoff_frequency, auto_generate_synonyms_phrase_query and fuzzy_transpositions参数设置，类似[match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html)。

### 重要：关于operator和minimum_should_match
best_fields和most_fields类型为每个字段生成match query。这意味着operator和minimum_should_match参数将应用于所有的field，这可能不是你想要的。

例如下面例子：
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Will Smith",
      "type":       "best_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and" 
    }
  }
}
```
所有term必须在所有field匹配，类似:
`(+first_name:will +first_name:smith) | (+last_name:will  +last_name:smith)`

更好的解决方案参考[cross_fields]()

## most_fields
most_fields类型用于相同的文本以不同的分词器去查询多个字段。例如，主字段包含同义词、词干和terms without diacritics。第二个字段包含原始temrs，第三个字段包含shingles。通过结果这三个字段的得分，我们可以用主字段匹配尽可能多的文档，使用第二三字段将相似结果排名靠前。

例如：
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown fox",
      "type":       "most_fields",
      "fields":     [ "title", "title.original", "title.shingles" ]
    }
  }
}
```
转换为下面should语法执行：
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":          "quick brown fox" }},
        { "match": { "title.original": "quick brown fox" }},
        { "match": { "title.shingles": "quick brown fox" }}
      ]
    }
  }
}
```

## phrase and phrase_prefix
phrase 和 phrase_prefix类似于 best_fields，但是他用的是match_phrase 或 match_phrase_prefix query，而不是 match query。

下面query：
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown f",
      "type":       "phrase_prefix",
      "fields":     [ "subject", "message" ]
    }
  }
}
```

转换为：
```
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_phrase_prefix": { "subject": "quick brown f" }},
        { "match_phrase_prefix": { "message": "quick brown f" }}
      ]
    }
  }
}
```

此外, 接收 analyzer, boost, lenient and zero_terms_query，以及slop。phrase_prefix还接受max_expansions。

> fuzziness 参数不能与 phrase 和 phrase_prefix 类型一起使用。

## cross_fields
cross_fields类型对于应该匹配多个字段的结构化文档特别有用。例如，在first_name和last_name字段查询“Will Smith”时，最好的匹配可能是在一个字段中有“Will”，而在另一个字段中有“Smith”。

这个看起来像是most_fields类型，但是有两个问题：
* 第一，operator and minimum_should_match是应用于每个字段，而不是每个term。
* 第二，与相关性有关:first_name和last_name字段中的不同词频可能会产生意外结果。
 
例如，假设我们有两个人:“Will Smith” and “Smith Jones”。“Smith”作为姓是很常见的(因此重要性很低)，而“Smith”作为名是很不常见的(因此非常重要)。

如果我们搜索“Will Smith”，“Smith Jones”文档可能会出现在“Will Smith”的前面，因为first_name: Smith的得分超过了first_name: Will和last_name: Smith的综合得分。

处理这些类型查询的一种方法是简单地将first_name和last_name字段索引到单个full_name字段中。当然，这只能在索引时完成。

cross_field类型尝试通过以term为中心的方法在查询时解决这些问题。它首先将查询字符串分析为单独的term，然后在任何字段中查找每个term，就好像它们是一个大字段一样。

例如这样的query：
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Will Smith",
      "type":       "cross_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and"
    }
  }
}
```

执行为:
```
+(first_name:will  last_name:will)
+(first_name:smith last_name:smith)
```

换句话说，要匹配文档，所有术语必须至少出现在一个字段中。(将其与用于best_fields和most_fields的逻辑进行比较。)

这解决了两个问题中的一个。通过混合各字段的项频来解决不同项频的问题，以消除差异。

在实践中，first_name:smith将被视为具有与last_name:smith相同的频率，加上1。这将使first_name和last_name上的匹配具有可比较的分数，last_name有一点优势，因为它是最有可能包含smith的字段。

请注意，cross_fields通常只对短字符串字段有用，所有字段boost都是1。否则boost、term freqs和length归一化将导致混合的term统计数据不再有意义。

如果你通过验证API运行上面的查询，它会返回以下解释:
```
+blended("will",  fields: [first_name, last_name])
+blended("smith", fields: [first_name, last_name])
```

接收如下参数analyzer, boost, operator, minimum_should_match, lenient, zero_terms_query and cutoff_frequency，


## cross_field 和 analysis
cross_field类型只能在具有相同analyzer的字段上以term为中心的模式下工作。具有相同analyzer的字段分组在一起，如上面的示例所示。如果有多个组，它们将组合为一个bool查询。

例如， first和last是用相同analyzer，加上first.edge和last.edge是用edge_ngram的analyzer，查询如下：
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Jon",
      "type":       "cross_fields",
      "fields":     [
        "first", "first.edge",
        "last",  "last.edge"
      ]
    }
  }
}
```

转换为：
```
 blended("jon", fields: [first, last])
| 
(
    blended("j",   fields: [first.edge, last.edge])
    blended("jo",  fields: [first.edge, last.edge])
    blended("jon", fields: [first.edge, last.edge])
)
```

存在多个组的时候operator or minimum_should_match参数将出现相同问题。
您可以轻松地将这个查询重写为两个单独的cross_fields查询和一个bool查询，并将minimum_should_match参数仅应用于其中一个:
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        {
          "multi_match" : {
            "query":      "Will Smith",
            "type":       "cross_fields",
            "fields":     [ "first", "last" ],
            "minimum_should_match": "50%" 
          }
        },
        {
          "multi_match" : {
            "query":      "Will Smith",
            "type":       "cross_fields",
            "fields":     [ "*.edge" ]
          }
        }
      ]
    }
  }
}
```


你也可以通过在查询中指定analyzer参数，可以将所有字段强制放到同一个组中。
```
GET /_search
{
  "query": {
   "multi_match" : {
      "query":      "Jon",
      "type":       "cross_fields",
      "analyzer":   "standard", 
      "fields":     [ "first", "last", "*.edge" ]
    }
  }
}
```

转换为：
```
blended("will",  fields: [first, first.edge, last.edge, last])
blended("smith", fields: [first, first.edge, last.edge, last])
```

## bool_prefix
bool_prefix类型的得分行为类似于most_fields，但使用的是match_bool_prefix查询而不是match查询。
```
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown f",
      "type":       "bool_prefix",
      "fields":     [ "subject", "message" ]
    }
  }
}
```

analyzer, boost, operator, minimum_should_match, lenient, zero_terms_query, and auto_generate_synonyms_phrase_query参数可以设置。

支持fuzziness, prefix_length, max_expansions, rewrite, and fuzzy_transpositions参数。

slop and cutoff_frequency 不支持