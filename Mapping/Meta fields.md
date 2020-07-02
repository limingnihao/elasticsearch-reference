# Meta-Fields



### Identity meta-fields

|        |        |                      |
| ------ | ------ | -------------------- |
| _index | String | 索引名称             |
| _type  | String | 文档的映射类型。_doc |
| _id    | String | 文档ID               |



### Document source meta-fields

|         |      |                                                |
| ------- | ---- | ---------------------------------------------- |
| _source | JSON | 表示文档主体的原始JSON。                       |
| _size   |      | _source字段的字节大小，由mapper-size插件提供。 |



### Indexing meta-fields

|              |      |                                                          |
| ------------ | ---- | -------------------------------------------------------- |
| _field_names |      | 文档中包含非空值的所有字段的名称。                       |
| _ignored     |      | 在索引时由于ignore_malformed而被忽略的文档中的所有字段。 |



### Routing meta-field

|          |        |                                                   |
| -------- | ------ | ------------------------------------------------- |
| _routing | String | 自定义路由值，用于将文档路由到特定的分片(shard)。 |





Other meta-field

|       |      |                          |
| ----- | ---- | ------------------------ |
| _meta |      | 特定于应用程序的元数据。 |

