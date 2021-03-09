# Common Terms Query

7.3将弃用

将高频词转为should查询

```
GET /_search
{
    "query": {
        "common": {
            "body": {
                "query": "nelly the elephant as a cartoon",
                "cutoff_frequency": 0.001,
                "low_freq_operator": "and"
            }
        }
    }
}
```

转为：
```
GET /_search
{
    "query": {
        "bool": {
            "must": [
            { "term": { "body": "nelly"}},
            { "term": { "body": "elephant"}},
            { "term": { "body": "cartoon"}}
            ],
            "should": [
            { "term": { "body": "the"}},
            { "term": { "body": "as"}},
            { "term": { "body": "a"}}
            ]
        }
    }
}
```