# Terms set query
返回在提供的字段中包含最少精确匹配term的文档。

terms_set查询与terms查询相同，只是您可以定义返回文档所需的匹配term的数量。例如:

一个字段，programming_languages，包含一系列已知的编程语言，如c++、java或php，供求职者使用。您可以使用terms_set查询来返回至少匹配这两种语言的文档。

一个字段，permissions，包含应用程序的可能用户权限列表。您可以使用terms_set查询来返回匹配这些权限子集的文档。

## Example request
```
GET /job-candidates/_search
{
    "query": {
        "terms_set": {
            "programming_languages": {
                "terms": ["c++", "java", "php"],
                "minimum_should_match_field": "2"
            }
        }
    }
}
```


## Top-level parameters for term
* <field> (Required, object) 你需要搜索的字段。


### Parameters for <field>
#### terms
（required, array of strings）您希望在提供的<field>中找到的term数组。要返回文档，所需的词汇数必须与字段值完全匹配，包括空格和大小写。

所需的匹配项数量是在minimum_should_match_field或minimum_should_match_script参数中定义的。

#### minimum_should_match_field 
（Optional, string）包含返回文档所需的匹配项数的数字字段。

#### minimum_should_match_script
(Optional, string) 包含返回文档所需的匹配项数的自定义脚本。

请查看 [scripting](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)