## Function score query

function_score允许您修改由查询检索的文档的分数。此功能在下面情况下很有用，例如，score函数在计算上很耗资源，并且可以在过滤后的文档集上计算分数。

 要使用function_score，用户必须定义一个查询和一个或多个函数，这些函数为查询返回的每个文档计算新分数。

 function_score只有一个函数时这样使用:
 ```
 {
    "query": {
        "function_score": {
            "query": { "match_all": {} },
            "boost": "5",
            "random_score": {}, 
            "boost_mode":"multiply"
        }
    }
}
 ```

此外，可以组合多个功能。在这种情况下，只有当文档匹配给定的过滤查询时，才会应用该函数:
```
{
    "query": {
        "function_score": {
          "query": { "match_all": {} },
          "boost": "5", 
          "functions": [
              {
                  "filter": { "match": { "test": "bar" } },
                  "random_score": {}, 
                  "weight": 23
              },
              {
                  "filter": { "match": { "test": "cat" } },
                  "weight": 42
              }
          ],
          "max_boost": 42,
          "score_mode": "max",
          "boost_mode": "multiply",
          "min_score" : 42
        }
    }
}
```


如果一个函数没有提供任何过滤器，这相当于指定“match_all”:{}。

### score_mode
首先，根据定义的函数对每个文档进行评分。参数score_mode指定计算的分数是如何组合的:
| 类型     | 说明                                                     |
| -------- | -------------------------------------------------------- |
| multiply | scores are multiplied (default)                          |
| sum      | scores are summed                                        |
| avg      | scores are averaged                                      |
| first    | the first function that has a matching filter is applied |
| max      | maximum score is used                                    |
| min      | minimum score is used                                    |

因为分数可以在不同的尺度上(例如，衰减函数的分数在0到1之间，但是field_value_factor的分数是任意的)，而且有时函数对分数的影响不同是需要的，所以每个函数的分数可以根据用户定义的权重进行调整。权重可以在functions数组(上面的例子)中的每个函数中定义，并与各自函数计算的分数相乘。如果在没有任何其他函数声明的情况下给出了权值，那么权值将作为一个简单地返回权值的函数。

如果将score_mode设置为avg，则各个分数将通过加权平均值进行组合。例如，如果两个函数的返回值分别为1和2，并且它们各自的权重分别为3和4，那么它们的值将被合并为(1*3+2*4)/(3+4)，而不是(1*3+2*4)/2。

#### max_boost
通过设置 max_boost 参数，可以将新分数限制为不超过某个限制。max_boost的默认值是FLT_MAX。

#### boost_mode
新计算的分数与查询的分数相结合。参数boost_mode定义了:
| 类型     | 说明                                                    |
| -------- | ------------------------------------------------------- |
| multiply | query score and function score is multiplied (default)  |
| replace  | only function score is used, the query score is ignored |
| sum      | query score and function score are added                |
| avg      | average                                                 |
| max      | max of query score and function score                   |
| min      | min of query score and function score                   |

默认情况下，修改分数不会改变哪些文档匹配。

#### min_score
要排除不满足某个分数阈值的文档，可以将min_score参数设置为所需的分数阈值。

>NOTE 要使min_score正常工作，需要对查询返回的所有文档进行评分，然后逐一过滤。


### function_score查询提供的函数
| function           | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| script_score       | 用自定义脚本可以完全控制评分计算，实现所需逻辑。             |
| weight             | 为每个文档应用一个简单而不被规范化的权重提升值：当 weight 为 2 时，最终结果为 2 * _score 。 |
| random_score       | 为每个用户都使用一个不同的随机评分对结果排序，但对某一具体用户来说，看到的顺序始终是一致的。 |
| field_value_factor | 使用这个值来修改 _score ，如将 popularity 或 votes （受欢迎或赞）作为考虑因素。 |
| decay              | 将浮动值结合到评分 _score 中，例如结合 publish_date 获得最近发布的文档，结合 geo_location 获得更接近某个具体经纬度（lat/lon）地点的文档，结合 price 获得更接近某个特定价格的文档 |


#### script_score
script_score函数允许您包装另一个查询并使用脚本表达式从文档中的其他数值字段值派生的计算可选地自定义该查询的得分。以下是一个简单的例子:
```
{
    "query": {
        "function_score": {
            "query": {
                "match": { "message": "elasticsearch" }
            },
            "script_score" : {
                "script" : {
                  "source": "Math.log(2 + doc['likes'].value)"
                }
            }
        }
    }
}
```
>所有文档得分都是正32位浮点数。

>如果script_score函数生成精度更高的分数，则将其转换为最近的32位浮点数。

>同样，分数必须是非负的。否则，Elasticsearch返回一个错误。

在不同的脚本字段值和表达式上，_score脚本参数可用于根据查询检索的分数进行包装。

缓存脚本编译以加快执行速度。如果脚本有需要考虑的参数，最好重用相同的脚本，并为其提供参数:
```
{
    "query": {
        "function_score": {
            "query": {
                "match": { "message": "elasticsearch" }
            },
            "script_score" : {
                "script" : {
                    "params": {
                        "a": 5,
                        "b": 1.2
                    },
                    "source": "params.a / Math.pow(params.b, doc['likes'].value)"
                }
            }
        }
    }
}
```
注意，与custom_score查询不同，查询的得分与脚本得分的结果是相乘的。如果你想要禁止它，设置"boost_mode": "replace"

#### weight
weight是将分数乘以提供的weight的值。这有时是需要的，因为针对特定查询设置的boost值会被规范化，而对于这个score函数则不会。number值的类型是float。

#### random_score
random_score生成从0到但不包括1的均匀分布的分数。默认情况下，它使用内部Lucene doc id作为随机性的一个来源，这是非常有效的，但不幸的是不能复制，因为文档可能被合并重新编号。

#### field_value_factor

#### decay functions: gauss, linear, exp