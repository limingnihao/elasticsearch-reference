# Span queries
> 原文地址：https://www.elastic.co/guide/en/elasticsearch/reference/current/span-queries.html

> github: 

- --
Span查询是低级的位置跨度查询，它提供了对指定术语的顺序和邻近性的专家控制。这些通常用于实现对法律文件或专利的非常具体的查询。

只允许在span query外部设置boost。符合span查询，例如span_near，only use the list of matching spans of inner span queries in order to find their own spans，然后使用这些span生成一个分数。Scores are never computed on inner span queries, which is the reason why boosts are not allowed: they only influence the way scores are computed, not spans.

不能将Span查询与非Span查询混合使用(除了span_multi查询之外)。

Span查询不会进行anaylzer。

这组查询包括：
| query              | Lucene                                                       | desc                                                         |      |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| span_containing    | SpanContainingQuery extends SpanContainQuery                 |                                                              |      |
| field_masking_span | SpanFieldMaskingQuery                                        |                                                              |      |
| span_fist          | SpanFirstQuery                                               | query在文档中的命中位置符合小于end。                         |      |
| span_multi         | Wraps a term, range, prefix, wildcard, regexp, or fuzzy query. |                                                              |      |
| span_near          | SpanNearQuery                                                | 组合多个span查询，这些查询必须在指定的距离内，可能是相同的顺序。 |      |
| span_not           | SpanNotQuery                                                 | 包装另一个query，并且排除与该查询匹配的任何文档。            |      |
| span_or            | SpanOrQuery                                                  | 组合多个span查询，返回匹配的任何查询的文档。                 |      |
| span_term          | SpanTermQuery                                                | 单term查询。不做analyzer。与[term query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html)等价，但是用于其他span查询。 |      |
| span_within        | SpanWithinQuery extends SpanContainQuery                     | 在一个结果里找符合预期的结果                                 |      |

span一些查询比较难理解给出测试数据,进行具体分析：
```
curl -X PUT "elasticsearch.localhost.com/test/_doc/1" -H 'Content-Type: application/json' -d \
'
{
	"id": 1,
	"name": "abcdef",
	"desc": "ab cd ef"
}
'

curl -X PUT "elasticsearch.localhost.com/test/_doc/2" -H 'Content-Type: application/json' -d \
'
{
	"id": 2,
	"name": "cdefgh",
	"desc": "cd ef gh"
}
'

curl -X PUT "elasticsearch.localhost.com/test/_doc/3" -H 'Content-Type: application/json' -d \
'
{
	"id": 3,
	"name": "efghij",
	"desc": "ef gh ij"
}
'

curl -X PUT "elasticsearch.localhost.com/test/_doc/4" -H 'Content-Type: application/json' -d \
'
{
	"id": 4,
	"name": "ghijkl",
	"desc": "gh ij kl"
}
'

curl -X PUT "elasticsearch.localhost.com/test/_doc/5" -H 'Content-Type: application/json' -d \
'
{
	"id": 5,
	"name": "ijklmn",
	"desc": "ij kl nm"
}
'

curl -X PUT "elasticsearch.localhost.com/test/_doc/6" -H 'Content-Type: application/json' -d \
'
{
	"id": 6,
	"name": "klmn",
	"desc": "kl mn"
}
'
curl -X PUT "elasticsearch.localhost.com/test/_doc/7" -H 'Content-Type: application/json' -d \
'
{
	"id": 7,
	"name": "abefcd",
	"desc": "ab ef cd"
}
'
```

## Span containing query
The span containing query maps to Lucene SpanContainingQuery

返回包含另一个span查询的匹配项。`big`和`little`可以是任何span类型的查询。返回从`big`到`little`包含匹配的匹配范围。

>通span within query类似，需要查清区别。


## Span field masking query
The span field masking query maps to Lucene’s SpanFieldMaskingQuery

Wrapper to allow span queries to participate in composite single-field span queries by lying about their search field. 

可以用于支持`span-near`或`span-or`跨不同字段的查询。这通常是不允许的。

Span field masking query is invaluable in conjunction with multi-fields when same content is indexed with multiple analyzers. For instance we could index a field with the standard analyzer which breaks text up into words, and again with the english analyzer which stems words into their root form.


## Span first query
The span first query maps to Lucene SpanFirstQuery. 

接近字段开头的跨度匹配。Matches spans near the beginning of a field. 

match 可以是任何span类型的query。`end`参数控制最大允许匹配的末尾位置。

例子:表示为查询的文本“n”，在文档中的`分词位置`要小于10。（使用ngram分词位置会很靠后）
```
"name" : "John Doe",
"name" : "Option Braw"
```
```
GET /test/_search
{
   "query": {
     "span_first": {
       "match": {
         "span_term": {
           "name": "n"
         }
       },
       "end": 10
     }
   }
}
```

## Span multi-term query


## Span near query
The span near query maps to Lucene SpanNearQuery. 

一个Span查询与另一个的距离。可以指定`slop`，设置之间的距离，和设置`in-order`是否要保持顺序。

示例，查询的a、c和d之间的距离需要小于15，in_order=false不用保证顺序：
```
GET /test/_search
{
   "query": {
    "span_near": {
      "clauses": [
        {
          "span_term": { "name": "a" }
        },
        {
          "span_term": { "name": "c" }
        },
        {
          "span_term": { "name": "d" }
        }
      ],
      "slop": 15,
      "in_order": true
    }
   }
}
```


## Span not query
The span not query maps to Lucene SpanNotQuery.

`include`和`exclude`子句可以是任何span类型的查询。include子句是筛选了匹配项的span查询，而exclude子句是匹配项不能与返回的匹配项重叠的span查询。

将include查询的结果排除掉exclude的结果。因为如果span_near查询时两个term达到的路径有多种情况(索引时term是重复的)。用来限制两个term之间不会出现exclude中定义的term。

其他选项：
* pre

If set the amount of tokens before the include span can’t have overlap with the exclude span. Defaults to 0.

* post

If set the amount of tokens after the include span can’t have overlap with the exclude span. Defaults to 0.

* dist

If set the amount of tokens from within the include span can’t have overlap with the exclude span. Equivalent of setting both pre and post.



示例1：查询在`a`和`c`之间跨度小于12的文本，但是他们之间不能包含`f`。
```
GET /test/_search
{
   "query": {
    "span_not": {
      "include": {
        "span_near": {
          "clauses": [
            { "span_term": { "name": "a" } },
            { "span_term": { "name": "c" } }
          ],
          "slop": 12,
          "in_order": false
        }
      },
      "exclude": {
        "span_term": {
          "name": "f"
        }
      }
    }
   }
}
```

示例2：查询包含`h`的文本，排除掉两遍有`J`和`e`的文本，最大距离为10。
```
GET /test/_search
{
   "query": {
    "span_not": {
      "include": {
         "span_term": { "name": "h" }
      },
      "exclude": {
        "span_near": {
          "clauses": [
            { "span_term": { "name": "J" } },
            { "span_term": { "name": "e" } }
          ],
          "slop": 10,
          "in_order": false
        }
      }
    }
   }
}
```

示例3：
```
GET /test/_search
{
   "query": {
    "span_not": {
      "include": {
        "span_near": {
          "clauses": [
            { "span_term": { "name": "J" } },
            { "span_term": { "name": "e" } }
          ],
          "slop": 10,
          "in_order": false
        }
      },
      "exclude": {
        "span_near": {
          "clauses": [
            { "span_term": { "name": "J" } },
            { "span_term": { "name": "W" } }
          ],
          "slop": 10,
          "in_order": false
        }
      },
      "pre": 1
    }
   }
}
```

## Span or query
The span or query maps to Lucene SpanOrQuery

所有span子句匹配结果的并集。

例如：返回term包含`a`或`f`的文档。
```
GET /test/_search
{
   "query": {
    "span_or": {
      "clauses": [
        { "span_term": { "name": "a" } },
        { "span_term": { "name": "f" } }
      ]
    }
   }
}
```


## Span term query
The span term query maps to Lucene SpanTermQuery。

下面是一个例子:
查询到的是完全匹配了Term：`a`的文本，不会对query进行analyzer。
```
GET /test/_search
{
   "query": {
       "span_term": { "name": "a"  }
   }
}
```
## Span within query
The span within query maps to Lucene SpanWithinQuery. 

返回包含于另一个span query的匹配结果。

`big` 和 `little` 子句可以是任何 span 类型的查询.little查询的结果包含在big中。 Matching spans from little that are enclosed within big are returned.

示例1：使用little检索包含字母f的文档，使用big检索包含little结果存字母e和g距离小于2o的文档。
```
GET /test/_search
{
   "query": {
    "span_within": {
      "little": {
        "span_term": { "name": "f"  }
      },
      "big": {
        "span_near": {
          "clauses": [
            { "span_term": { "name": "e" } },
            { "span_term": { "name": "g" } }
          ],
          "slop": 20,
          "in_order": false
        }
      }
    }
   }
}
```

示例2：使用little检索出包含d和e字母并且距离小于20的文档。big检索包含little结果并存在c和f字母距离小于20的文档。
```
GET /test/_search
{
   "query": {
    "span_within": {
      "little": {
        "span_near": {
          "clauses": [
            { "span_term": { "name": "d" } },
            { "span_term": { "name": "e" } }
          ],
          "slop": 20,
          "in_order": false
        }
        
      },
      "big": {
        "span_near": {
          "clauses": [
            { "span_term": { "name": "c" } },
            { "span_term": { "name": "f" } }
          ],
          "slop": 20,
          "in_order": false
        }
      }
    }
   }
}
```