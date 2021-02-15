## Install Elasticsearch with Docker

Elasticsearch也可以作为Docker映像使用。 这些图像使用centos：7作为基础镜像。

所有已发布的Docker映像和标签的列表可在www.docker.elastic.co中。源文件位于Github中(https://github.com/elastic/elasticsearch/blob/7.5/distribution/docker)。

这些镜像在Elastic许可下可免费使用。 它们包含开放源代码和免费的商业功能以及对付费商业功能的访问。 开始30天试用，以试用所有付费商业功能。 有关Elastic许可级别的信息，请参阅“订阅”页面(https://www.elastic.co/cn/subscriptions)。

### Pulling the image
获取Elasticsearch的Docker使用docker pull命令。
```
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.5.2
```

或者，您可以下载其他仅包含Apache 2.0许可下可用功能的Docker映像。 要下载镜像，请访问www.docker.elastic.co 。

### Starting a single node cluster with Docker
要启动用于开发或测试的单节点Elasticsearch集群，请指定单节点发现以绕过引导检查 ：

```
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.5.2
```

### Starting a multi-node cluster with Docker Compose
要在Docker中启动并运行一个三节点的Elasticsearch集群，可以使用Docker Compose：
创建一个docker-compose.yml文件：

该示例Docker Compose文件创建了一个三节点的Elasticsearch集群。 节点es01在localhost:9200上侦听，并且es02和es01通过Docker网络与es03对话。

请注意，此配置在所有网络接口上公开端口9200，并且鉴于Docker如何在Linux上操作iptables ，这意味着您的Elasticsearch集群是可公开访问的，可能会忽略任何防火墙设置。 如果您不想公开端口9200，而是使用反向代理，请在docker-compose.yml文件中将9200:9200替换为127.0.0.1:9200:9200 。 然后只能从主机本身访问Elasticsearch。

Docker named volumes data01，data02和data03，在重启后会持续存在。如果它们尚不存在，则在启动集群时， docker-compose会创建它们。

1. 确保为Docker Engine分配了至少4GiB的内存。 在Docker桌面中，您可以在首选项（macOS）或设置（Windows）的高级选项卡上配置资源使用情况。
2. 运行docker-compose来启动集群：
```
docker-compose up
```
3. 提交_cat/nodes请求以查看节点是否已启动并正在运行：
```
curl -X GET "localhost:9200/_cat/nodes?v&pretty"
```

Docker日志记录驱动程序处理控制台的日志消息。默认情况下，您可以使用docker logs访问日志。

要停止集群，请运行docker-compose down 。 使用docker-compose up重新启动集群时，将保留并加载Docker卷中的数据。 要在关闭群集时删除数据卷 ，请指定-v选项： docker-compose down -v 。



### Using the Docker images in production


#### Configuration files must be readable by the user

#### Increase ulimits for nofile and nproc

#### Disable swapping

#### Randomize published ports

#### Set the heap size

#### Pin deployments to a specific image version

#### Always bind data volumes

#### Avoid using loop-lvm mode

#### Centralize your logs

#### Configuring Elasticsearch with Docker

#### Mounting Elasticsearch configuration files

#### Using custom Docker images

