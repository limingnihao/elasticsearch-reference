# Query string query

使用具有严格语法的解析器，根据提供的查询字符串返回文档。

此查询使用语法根据操作符(如and或NOT)解析和拆分所提供的查询字符串。会给每个拆分的文本进行analyzer然后返回匹配的文档。

您可以使用`query_string`查询来创建包含通配符、跨多个字段的搜索等等的复杂搜索。虽然是通用的，但查询是严格的，如果查询字符串包含任何无效语法，则返回一个错误。

>WARNING

>因为对于任何无效的语法，它都会返回一个错误，所以我们不建议对搜索框使用query_string查询。

>如果不需要支持查询语法，可以考虑使用match query。如果需要查询语法的特性，可以使用simple_query_string查询，它的要求不那么严格。

## Example request
在运行下面的搜索时，query_string 将拆分(new york
city) OR (big apple) 为两部分: new york city and big apple。content 字段 analyzer将分别进行分词。因为查询语法不使用空格作为操作符，所以new york city按原样传递给analyzer。

```
GET /_search
{
    "query": {
        "query_string" : {
            "query" : "(new york city) OR (big apple)",
            "default_field" : "content"
        }
    }
}
```

## Top-level parameters for query_string
|parameter|Required|type|desc|默认值|
|---|---|---|---|---|
|query|必填|string|要解析并用于搜索的查询字符串。|
|default_field|Optional|string|如果查询字符串中没有提供字段时的默认字段。|默认使用index setting的index.query.default_field，默认值为*. 会使用所有符合的field。如果没有指定前缀则使用所有字段。|
|allow_leading_wildcard|Optional|boolean|通配符*和?允许作为查询字符串的第一个字符。|默认值为true。|
|analyze_wildcard|Optional|boolean|如果为真，则查询将尝试分析查询字符串中的通配符项。|默认值为false。|
|analyzer|Optional|string|Analyzer用于将查询字符串中的文本转换为token。默认情况使用default_field的mapping定义的索引时analyzer。如果没有定义index analyzer，则使用索引的默认analyzer。||
|auto_generate_synonyms_phrase_query|Optional|boolean|如果为true，则为多term同义词自动创建match phrase查询|默认true|
|boost|Optional|float|得分权重|默认1.0|
|default_operator|Optional|string|未指定操作符时字符串文本之间的逻辑|默认OR|
|enable_position_increments|Optional|boolean|如果为true启动位置增量|默认为true|
|fields|Optional|array|你想搜索的多个字段数组。跨越多个字段搜索。||
|fuzziness|Optional|string|模糊匹配的最大编辑距离。|
|fuzzy_max_expansions|Optional|integer|模糊匹配的最大数量。|默认50|
|fuzzy_prefix_length|Optional|integer|模糊匹配起始字符位置。|默认0|
|fuzzy_transpositions|Optional|integer|当为true，模糊匹配编辑距离报过转置(ab->ba)|默认true|
|lenient|Optional|boolean|当为true，忽略错误|默认false|
|max_determinized_states|Optional|integer|查询的 automaton states 最大值|默认10000|
|minimum_should_match|Optional|string|最小匹配数量||
|quote_analyzer|Optional|string|quotend文本的analyzer。|默认使用default_field在mapping定义的|
|phrase_slop|Optional|integer|短语匹配的最大距离。0表示精准匹配|默认为0|
|quote_field_suffix|Optional|string|||
|rewrite|Optional|string|query改写，参考[rewrite](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)||
|time_zone|Optional|string|用于将查询字符串中的日期值转换为UTC。|


## Query string syntax
query string是一个小型语法，类似
[search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)的q参数。

查询字符串被解析为一系列的term和operator。一个term可以是单个单词(quick或brown)，也可以是一个短语(由双引号括起)(“quick brown”)，它按照相同的顺序搜索短语中的所有单词。


## Field names
你可以在查询语法中指定要搜索的字段:
* where the status field contains active。
status:active

* where the title field contains quick or brown。
title:(quick OR brown)

* where the author field contains the exact phrase "john smith"。
author:"John Smith"

* where the first name field contains Alice (使用反斜杠转义空格)。

first\ name:Alice

* where any of the fields book.title, book.content or book.date contains quick or brown (note how we need to escape the * with a backslash):

book.\*:(quick OR brown)

* where the field title has any non-null value:
_exists_:title



## Wildcards（通配符）
通配符搜索可以在单独的术语上运行，使用?替换单个字符，*替换零个或多个字符:

> qu?ck bro*

请注意，通配符查询可能会使用大量的内存，并且执行得非常糟糕，需考虑需要查询多少项才能匹配查询字符串“a* b* c*”。
```
GET /kibana_sample_data_ecommerce/_search
{
    "_source": "customer_first_name", 
    "query": {
        "query_string": {
            "query" : "customer_first_name:qu?ck bro*"
        }
    }
}
```

> 开头使用通配符(例如“*ing”)尤其困难，因为需要检查索引中的所有项，以防它们匹配。通过将allow_leading_wildcard设置为false，可以禁用前导通配符。


## Regular expressions（正则表达式）
正则表达式模式可以被嵌入到查询字符串中，方法是用前斜杠("/")将它们括起来:

`name:/joh?n(ath[oa]n)/`

支持的正则表达式语法参考[Regular expression syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/regexp-syntax.html)

> allow_leading_wildcard 参数不对正则表达式语法起作用。所以`/.*n/`查询可能将进行扫全库。

## Fuzziness (模糊查询)
我们可以使用“fuzzy”操作符搜索与我们的搜索词相似但又不完全相同的词:

`quikc~ brwn~ foks~`

使用 [Damerau-Levenshtein_distance](https://en.wikipedia.org/wiki/Damerau-Levenshtein_distance) 编辑距离寻找所有最大有2个改动的term（默认2），其中一个更改是插入、删除或替换单个字符、或交换两个相邻字符。

默认的编辑距离是2，但是编辑距离1应该足以捕获80%的人类拼写错误。它可以被指定为:`quikc~1`

> 不支持模糊搜索和通配符同时使用。

## Proximity searches
短语查询(如“john smith”)期望所有的term都以完全相同的顺序出现，而proximity查询则允许指定的单词之间有进一步的距离或以不同的顺序出现。就像模糊查询可以为单词中的字符指定最大编辑距离一样，proximity搜索允许我们为短语中单词指定最大编辑距离:`"fox quick"~5`

字段中的文本越接近查询字符串中指定的原始顺序，则认为该文档越相关。与上面的示例查询相比，短语“quick fox”会被认为比“quick brown fox”更相关。

## Ranges 
可以为日期、数字或字符串字段指定范围。包含范围用方括号[min TO max]指定，排他范围用大括号{min TO max}指定。

* All days in 2012:

`date:[2012-01-01 TO 2012-12-31]`

* Numbers 1..5

`count:[1 TO 5]`

* Tags between alpha and omega, excluding alpha and omega:

`tag:{alpha TO omega}`

* Numbers from 10 upwards

`count:[10 TO *]`

* Dates before 2012

`date:{* TO 2012-01-01}`

Curly and square brackets can be combined:

* Numbers from 1 up to but not including 5

`count:[1 TO 5}`

Ranges with one side unbounded can use the following syntax:
```
age:>10
age:>=10
age:<10
age:<=10
```

要用简化的语法组合上界和下界，您需要用and操作符连接两个子句:
```
age:(>=10 AND <20)
age:(+>=10 +<20
```

The parsing of ranges in query strings can be complex and error prone. It is much more reliable to use an explicit [range query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html).

## Boosting
使用boost操作符^使一个term比另一个更相关。例如，如果我们想找到关于`fox`的所有文件，但是我们对`quick foxes`特别感兴趣:
```
quick^2 fox
```
默认boost值为1，但可以是任何正的浮点数。在0和1之间进行提升会降低相关性。

boost也可以应用于短语或组:
```
"john smith"^2   (foo bar)^4
```


## Boolean operators
含一个或多个foo bar或baz的文档。我们已经讨论了上面的default_operator，它允许您强制要求所有的条件，但是也有布尔运算符，可以在查询字符串本身中使用它们来提供更多的控制。

首选的操作符是+(这一项必须存在)和-(这一项必须不存在)。所有其他条款都是可选的。例如，这个查询:
```
quick brown +fox -news

```
表示：
* fox 必须存在
* news 必须不存在
* quick and brown 是可选的 — 存在则增加相关性

我们熟悉的布尔运算符AND、OR和NOT(也写作&&、||和!)也受到支持，但是要注意它们不遵循通常的优先规则，所以当多个运算符一起使用时，应该使用括号。例如，前面的查询可以重写为:
```
((quick AND fox) OR (brown AND fox) OR fox) AND NOT news
```

该表单现在可以正确地复制原始查询的逻辑，但是相关性评分与原始查询几乎没有相似之处。

对比来说，使用match查询重写的查询将如下所示:
```
{
    "bool": {
        "must":     { "match": "fox"         },
        "should":   { "match": "quick brown" },
        "must_not": { "match": "news"        }
    }
}
```

## Grouping
多个term或子句可以用括号组合在一起，形成子查询:
```
(quick OR brown) AND fox
```

组可用于针对特定字段，或增强子查询的结果:
```
status:(active OR pending) title:(full text search)^2
```


## Reserved characters（保留字符）
如果您需要在查询本身中使用作为操作符的字符(而不是作为操作符)，那么您应该使用一个前导反斜杠来转义它们。例如，要搜索(1+1)=2，您需要将查询写成\(1\+1\)\=2。当使用JSON作为请求体时，需要前面两个反斜杠(\\);反斜杠是JSON字符串中的保留转义字符。


The reserved characters are:` + - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ / `

>未能正确转义这些特殊字符可能导致语法错误，从而阻止查询运行。
> > 和 < 不能被转义，最好在创建query是将他们删除。

## Whiespaces and empty queries
空格不被视为操作符。

如果查询字符串为空或只包含空格，则查询将产生空结果集。

## Avoid using the query_string query for nested documents
query_string搜索不返回嵌套文档。要搜索嵌套文档，请使用[嵌套查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-nested-query.html)。

## Search multiples fields
可以使用fields参数跨多个字段执行query_string搜索。

对多个字段运行query_string查询的想法是将每个查询项扩展为一个OR子句，如下所示:
```
field1:query_term OR field2:query_term | ...
```

## Additional parameters for multiple field searches
当对多个字段运行query_string查询时，支持以下附加参数。支持的type如同multi-match

## Synonyms and the query_string query
query_string查询通过同义词图标记过滤器支持多术语同义词扩展。使用此筛选器时，解析器将为每个多术语同义词创建短语查询。例如，下面的同义词:ny, new york would produce:

```(ny OR ("new york"))```

创建的boolean query:
```
(ny OR (new AND york)) city

```
## How minimum_should_match works
query_string围绕每个操作符分割查询，为整个输入创建一个布尔查询。您可以使用minimum_should_match来控制结果查询中应该匹配多少个“should”子句。

## How minimum_should_match for multiple fields

```
GET /_search
{
    "query": {
        "query_string": {
            "fields": [
                "title",
                "content"
            ],
            "query": "this that thus",
            "minimum_should_match": 2
        }
    }
}
```
创建的boolean query为：
`((content:this content:that content:thus) | (title:this title:that title:thus))`

title和content的最大分离值匹配。这里不能应用minimum_should_match参数。

```
GET /_search
{
    "query": {
        "query_string": {
            "fields": [
                "title",
                "content"
            ],
            "query": "this OR that OR thus",
            "minimum_should_match": 2
        }
    }
}
```
添加显式操作符将强制将每个术语视为单独的子句。

创建的boolean查询为：

`((content:this | title:this) (content:that | title:that) (content:thus | title:thus))~2`

它将文档与三个“should”子句中的至少两个相匹配，每个子句由每个术语的字段上的析取最大值组成。


## HOw minimum_should_match work for cross-field searches
type字段中的cross_fields值表示在分析输入时将具有相同分析器的字段分组在一起。
```
GET /_search
{
    "query": {
        "query_string": {
            "fields": [
                "title",
                "content"
            ],
            "query": "this OR that OR thus",
            "type": "cross_fields",
            "minimum_should_match": 2
        }
    }
}
```

 boolean query:
```
(blended(terms:[field2:this, field1:this]) blended(terms:[field2:that, field1:that]) blended(terms:[field2:thus, field1:thus]))~2
```
它将文档与每个词的三个混合查询中的至少两个匹配。

> 查询子句上线定义`indices.query.bool.max_clause_count`，默认为1024.