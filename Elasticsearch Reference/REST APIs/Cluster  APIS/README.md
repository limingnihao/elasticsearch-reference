# Cluster APIs



| API      | Request                          | Description                    |
| -------- | -------------------------------- | ------------------------------ |
|          | GET /_cluster/allocation/explain | 提供关于集群中的分片分配的解释 |
|          | GET /_cluster/settings           |                                |
| 集群状态 | GET _cluster/health/<target>     | 返回群集的运行状况状态。       |
|          | POST /_cluster/reroute           |                                |
|          |                                  |                                |
|          |                                  |                                |
|          |                                  |                                |
|          |                                  |                                |
|          |                                  |                                |





```
GET /_cluster/allocation/explain
{
  "index": "my-index-000001",
  "shard": 0,
  "primary": true
}
```







一些集群级api可以对节点的子集进行操作，可以用节点筛选器指定这些子集。例如，任务管理、节点状态和节点信息api都可以报告一组经过筛选的节点的结果，而不是所有节点的结果。



节点筛选器被编写为逗号分隔的单个筛选器列表，每个筛选器从选定的子集中添加或删除节点。每个筛选器可以是以下选项之一：

_all，所有节点的子集

_local，本地节点的子集。

_master，当前主节点的子集。

根据节点id

IP地址或主机名

使用*通配符的模式，名称、地址或主机名
`master:true`, `data:true`, `ingest:true`, `voting_only:true`, `ml:true`, or `coordinating_only:true`

