# Index APIs



### Index management(索引管理)

| API            | Request                                | Description              |
| -------------- | -------------------------------------- | ------------------------ |
| Create index   | PUT /<index>                           | 创建index                |
| Delete index   | DELETE /<index>                        | 删除index                |
| Get index      | GET /<index>                           | 获取index                |
| Index exists   | HEAD /<index>                          | 判断index是否存在        |
| Close index    | POST /<index>/_close                   | 关闭index。不能读写      |
| Open index     | POST /<index>/_open                    | 打开index。将clode状态的 |
| Shrink index   | POST /<index>/_shrink/<target-index>   | 收缩index。              |
| Split index    | POST /<index>/_split/<target-index>    | 拆分index。              |
| Clone index    | POST /<index>/_clone/<target-index>    | 克隆index。              |
| Rollover index | POST /<alias>/_rollover/<target-index> |                          |
| Freeze index   | POST /<alias>/_freeze                  | 冻结index。只读。        |
| Unfreeze index | POST /<alias>/_unfreeze                | 解冻index。              |
| Resolve index  | GET /_resolve/index/<name>             |                          |



### Mapping management

| API               | Request | Description |
| ----------------- | ------- | ----------- |
| Put mapping       |         |             |
| Get mapping       |         |             |
| Get field mapping |         |             |



### Alias management

| API                | Request | Description |
| ------------------ | ------- | ----------- |
| Add index alias    |         |             |
| Delete index alias |         |             |
| Get index alias    |         |             |
| Index alias exists |         |             |
| Update index alias |         |             |



### Index settings

| API                   | Request | Description |
| --------------------- | ------- | ----------- |
| Update index settings |         |             |
| Get index settings    |         |             |
| Analyze               |         |             |





### Index templates

| API                   | Request | Description |
| --------------------- | ------- | ----------- |
| Put index template    |         |             |
| Get index template    |         |             |
| Delete index template |         |             |
| Put component template |         |             |
| Get component template |         |             |
| Delete component template |         |             |
| Index template exists |         |             |
| Simulate index |         |             |






### Monitoring
| API                   | Request | Description |
| --------------------- | ------- | ----------- |
| Index stats |         |             |
| Index segments |         |             |
| Index recovery |         |             |
| Index shard stores |         |             |



### Status management

| API                   | Request | Description |
| --------------------- | ------- | ----------- |
|  Clear cache  |         |             |
|  Refresh  |         |             |
|  Flush  |         |             |
|  Synced flush  |         |             |
|  Force merge  |         |             |



### Dangling indices:
| API                   | Request | Description |
| --------------------- | ------- | ----------- |
|  List dangling indices  |         |             |
|  Import dangling index  |         |             |
|  Delete dangling index  |         |             |
