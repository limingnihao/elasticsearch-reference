# Install ECK using the Helm chart
 [Install ECK using the Helm chart](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-install-helm.html)

> Helm chart是实验性的，应该被认为是beta级的功能。



从1.3.0版本开始，可以使用Helm chart来实验安装ECK。它可以从Elastic Helm存储库中获得，并可以通过运行以下命令将其添加到Helm存储库列表中.

```
helm repo add elastic https://helm.elastic.co
helm repo update
```



这是默认的安装方式，相当于使用 [installing ECK using the `all-in-one.yaml` file](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-install-all-in-one.html)。

```
helm install elastic-operator elastic/eck-operator -n elastic-system --create-namespace
```



这种模式避免安装任何集群范围的资源，并限制操作人员只能管理一组预定义的名称空间。

由于crd是全局资源，所以仍然需要管理员来安装它们。这可以通过:

```
helm install elastic-operator-crds elastic/eck-operator-crds

```



任何对他们希望管理的名称空间集具有完全访问权的用户都可以安装操作符。下面的示例将操作符安装到弹性系统命名空间，并将其配置为仅管理命名空间-a和命名空间-b.

```
helm install elastic-operator elastic/eck-operator -n elastic-system --create-namespace \
  --set=installCRDs=false \
  --set=managedNamespaces='{namespace-a, namespace-b}' \
  --set=createClusterScopedResources=false \
  --set=webhook.enabled=false \
  --set=config.validateStorageClass=false
```



> eck-operator chart包含几个预定义的配置文件，帮助您在不同的配置中安装操作符。这些概要文件可以在图表目录的根目录中找到，前缀为profile-。例如，上面显示的受限制配置是在受概要文件限制的配置中定义的。Yaml文件，并可以如下使用
>
> ```
> helm install elastic-operator elastic/eck-operator -n elastic-system --create-namespace \
>   --values="${CHART_DIR}/profile-restricted.yaml" \
>   --set=managedNamespaces='{namespace-a, namespace-b}'
> ```
>
> You can find the profile files in the Helm cache directory or from the [ECK source repository](https://github.com/elastic/cloud-on-k8s/tree/1.5/deploy/eck-operator).