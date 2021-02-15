# Scripting

通过编写脚本，可以在Elasticsearch中计算自定义表达式。例如，您可以使用脚本返回“script fields”作为搜索请求的一部分，或者计算查询的自定义分数。

默认的脚本语言是Painless。额外的lang插件可以让你运行用其他语言编写的脚本。凡是可以使用脚本的地方，都可以包含一个lang参数来指定脚本的语言。



## General-purpose languages

这些语言可以使用scripting APIS，并提供了最大的灵活性。

| Language                                                     | Sandboxed | Required plugin |
| ------------------------------------------------------------ | --------- | --------------- |
| [`painless`](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/modules-scripting-painless.html) | yes       | built-in        |



## Special-purpose languages

这些语言没有那么灵活，但对于某些任务通常有更高的性能。

| Language                                                     | Sandboxed | Required plugin | Purpose                         |
| ------------------------------------------------------------ | --------- | --------------- | ------------------------------- |
| [`expression`](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/modules-scripting-expression.html) | yes       | built-in        | fast custom ranking and sorting |
| [`mustache`](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/search-template.html) | yes       | built-in        | templates                       |
| [`java`](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/modules-scripting-engine.html) | n/a       | you write it!   | expert API                      |



> #### Scripts and security
> sandboxed语言的设计考虑到了安全性。然而，非沙箱语言可能是一个安全问题，请阅读[Scripting and security](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/modules-scripting-security.html)了解更多细节。

