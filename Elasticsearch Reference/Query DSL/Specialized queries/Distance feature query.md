# Distance feature query
将文档的相关分数提高到更接近提供的原始的date或point。例如，您可以使用此查询为更接近某个date或location的文档赋予更大的权重。

您可以使用distance_feature查询来查找某个位置的最近邻居。您还可以在bool搜索的should筛选器中使用查询来将提升的相关性分数添加到bool查询的分数中。

## Top-level parameters for distance_featureedit

|type|Required|type|desc|
|---|---|---|---|
|field|Required|stirng|用于计算距离的字段的名称。|
|origin|Required|string|计算距离的日期或起点。|
|pivot|Required|time or distance|范围.[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#time-units)，[距离单位](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#distance-units)|
|boost|Optional|float|权重|

### field
用于计算距离的字段的名称。必须满足：
* date，date_nanos或geo_point字段。
* mapping的index为ture，默认值。
* mapping的doc_values为true，默认值。

### origin
计算距离的日期或起点。
* 如果字段值是date或date_nanos字段，则原始值必须是date。支持诸如now-1h之类的[Date Match](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#date-math)。
* 如果字段值是geo_point字段，则原点值必须是一个geopoint。

### pivot
距离原点的距离，在该距离处，相关性得分获得一半的提升值。

* 如果字段值是date或date_nanos字段，那么主值必须是一个时间单元，比如1h或10d。常用单位（d、h、m、s、ms）
* 如果字段值是geo_point字段，则轴心值必须是距离单位，如1km或12m。常用单位(km、m、cm、mm)

### boost
浮点数，用来乘以匹配文档的相关分数。这个值不能为负。默认为1.0。



## Example request

### Index setup

要使用distance_feature查询，索引必须包含date、date_nanos或geo_point字段。
要查看如何为distance_feature查询设置索引，请尝试以下示例。

1.创建一个项目索引与以下字段映射:
* name, keyword类型的字段
* production_date, date类型的字段
* location, geo_point类型的字段
* 
```
PUT /items
{
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword"
      },
      "production_date": {
        "type": "date"
      },
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```


## Example queries

### Boost documents based date

下面的bool搜索返回所有文档。使用distance_feature查询来增加文档的相关分数，其中的production_date值更接近当前值。

```
GET /test/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_all": {}
        }
      ],
      "should": [
        {
          "distance_feature": {
            "field": "production_date",
            "origin": "now",
            "pivot": "1d"
          }
        }
      ]
    }
  } 
}
```

### Boost documents based on location
下面的bool搜索返回所有的文档。还使用distance_feature查询来增加位置值接近[-71.3,41.15]的文档的相关性得分。
```
GET /test/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_all": {}
        }
      ],
      "should": {
        "distance_feature": {
          "field": "location",
          "pivot": "1000m",
          "origin": [-71.3, 41.15]
        }
      }
    }
  } 
}
```

## Notes
### distance_feature如何计算得分
distance_feature查询动态计算原始值与文档字段值之间的距离。然后，它使用这个距离作为一个特征来提高较近文档的相关性得分。

distance_feature查询计算文档的相关分数如下:

`relevance score = boost * pivot / (pivot + distance)`

距离是原始值和文档字段值之间的绝对差值。

### Skip non-competitive hits
与function_score查询或其他更改相关分数的方法不同，distance_feature查询在track_total_hits参数不为真时有效地跳过非竞争的命中。