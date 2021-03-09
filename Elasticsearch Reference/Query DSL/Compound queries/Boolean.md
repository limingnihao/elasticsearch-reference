## Boolean query

>原文地址：https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-bool-query.html

一个匹配查是包含其他查询的布尔查询。bool查询会映射为Lucene BooleanQuery。他是一个或多个boolean子句组成，每个子句都偶有一个类型。这些类型可能是：

| Occur    | score | 含义    | Description                                                  |
| -------- | ----- | ------- | ------------------------------------------------------------ |
| must     | 是    | and     | 子句(查询)必须出现在匹配的文档中，并对分数有贡献。           |
| filter   | 否    | and     | 子句(查询)必须出现在匹配的文档中。但是与must不同的是，查询的分数将被忽略。筛选器子句在筛选器上下文中执行，这意味着忽略评分，并考虑子句用于缓存。 |
| should   | 是    | or      | 子句(查询)应该出现在匹配的文档中。                           |
| must_not | 否    | and not | 子句(查询)不能出现在匹配的文档中。子句在筛选器上下文中执行，这意味着忽略评分，而子句用于缓存。因为忽略了评分，所以返回所有文档的0分。 |

bool查询采用的是“匹配越多越好”的方法，因此来自每个匹配的must或should子句的score将被添加到一起，以提供每个文档的最终_score。

### Example request
```
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```


### Using minimum_should_match
可以使用minimum_should_match参数指定返回的文档必须匹配的should子句的数量或百分比。

如果bool查询包含至少一个should子句和没有must或filter子句，则默认值为1。否则，默认值为0。（意思是说当至少有一个must filter时，默认值为0，也就是可以都不生效。没有must和filter子句，默认值为1，至少要符合一个should的子句）

should语句：如果一个query语句的bool下面，除了should语句，还包含了filter或者must语句，那么should context下的查询语句可以一个都不满足，只是_score=0。

关于参数设置：https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-minimum-should-match.html

| Type                  | Example     | Description                                                  |
| --------------------- | ----------- | ------------------------------------------------------------ |
| Integer               | 3           | 无论可选子句的数量如何，均指示一个固定值。                   |
| Negative integer      | -2          | 表示可选子句的总数，减去后此子句应该是必需匹配的数量。       |
| Percentage            | 75%         | 表示必需的可选子句总数的百分比。根据百分比计算得出的数字将四舍五入。 |
| Negative percentage   | -25%        | 从百分比中计算出的数字会四舍五入，然后从总数中减去以确定最小值。 |
| Combination           | 3<90%       | 一个正整数，跟随一个小于号，跟随前面提到的任何公式。表示子句小于等于第一个整数在都必须匹配，当大于第一个整数则使用后面的规则。例子表示：小于等于3则必须匹配，大于等于4在必须90%匹配。 |
| Multiple combinations | 2<-25% 9<-3 | 多个条件空格分隔，每条规则后面大于前面。例子表示：1或2个自己必须匹配，3-9个字符需要25%，9个以上需要总数-3个。 |

__提示：__
>当处理百分比时，负值可用于在极端情况下获得不同的行为。 在处理4个子句时，75％和-25％表示相同的内容，但是在处理5个子句时，75％表示需要3，而-25％表示需要4。

>如果基于规范的计算确定不需要任何可选子句，则有关BooleanQueries的常规规则仍将在搜索时适用（不包含必需子句的BooleanQuery必须仍与至少一个可选子句匹配）。

>无论计算达到多少，都将永远不会使用大于可选子句数量的值或小于1的值。 （即：无论计算结果的结果有多低或多高，所需匹配项的最小数量都不会低于1或大于子句的数量。


### Scoring with bool.filter

在filter元素下指定的查询对评分没有影响 - 返回的分数为0分数只收query的影响。例如，一下三个query都返回status字段包含active。

第一个查询，给所有文档赋值0分，因为没有指定打分查询:
```
{
  "query": { 
    "bool": { 
      "filter": [ 
        { "term":  { "eyeColor": "brown" }}
      ]
    }
  }
}
```

使用bool查询有一个match_all查询，它为所有文档赋值1.0。
```
{
    "_source": {
  	    "include": ["favoriteFruit", "name", "eyeColor", "age"]
    },	
    "query": {
        "bool": { 
            "must": {
                "match_all": {}
            },
          "filter": [ 
            { "term":  { "eyeColor": "brown" }}
          ]
        }
    }
}
```

constant_score查询的行为与上面的第二个示例完全相同。constant_score查询为筛选器匹配的所有文档分配1.0分。
```
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

### Using named queries to see which clauses matched
如果需要知道bool查询中的哪些子句与查询返回的文档匹配，可以使用命名查询为每个子句分配一个名称。
https://www.elastic.co/guide/en/elasticsearch/reference/7.5/search-request-body.html#request-body-search-queries-and-filters