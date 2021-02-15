# Painless scripting language

Painless是一种简单、安全的脚本语言专门为Elasticsearch设计的。它是Elasticsearch的默认脚本语言，可以安全地用于内联和存储脚本。开始使用Painless请看 [Painless Guide](https://www.elastic.co/guide/en/elasticsearch/painless/7.11/painless-guide.html)。有关Painless语法和语言特性的详细描述，请参见[Painless Language Specification](https://www.elastic.co/guide/en/elasticsearch/painless/7.11/painless-lang-spec.html)。

您可以在Elasticsearch中使用Painless。Painless提供：

* 快速性能:Painless scripts的运行速度比替代方案快好几倍。[run several times faster](https://benchmarks.elastic.co/index.html#search_qps_scripts) 
* 安全性:具有方法调用/字段粒度的细粒度allowlists。请参阅 [Painless API Reference](https://www.elastic.co/guide/en/elasticsearch/painless/7.11/painless-api-reference.html)以获得可用类和方法的完整列表。
* 可选类型:变量和参数可以使用显式类型或动态def类型。
* 语法:扩展Java语法的一个子集，以提供额外的脚本语言特性。
* 优化:专门为Elasticsearch脚本设计。

