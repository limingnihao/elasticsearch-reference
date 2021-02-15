# Painless Guide

Painless是一种简单、安全的，专门为Elasticsearch设计的脚本语言。它是Elasticsearch的默认脚本语言，可以安全地用于内联和存储脚本。要想快速进入Painless，请参见 [A Brief Painless Walkthrough](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-walkthrough.html)。有关Painless语法和语言特性的详细描述，请参阅 [Painless Language Specification](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-lang-spec.html)。

您可以在Elasticsearch中使用脚本的任何地方使用Painless脚本。Painless提供的能力:

* 快速性能:Painless scripts的运行速度比替代方案快好几倍。[run several times faster](https://benchmarks.elastic.co/index.html#search_qps_scripts) 
* 安全性:具有方法调用/字段粒度的细粒度allowlists。请参阅 [Painless API Reference](https://www.elastic.co/guide/en/elasticsearch/painless/7.11/painless-api-reference.html)以获得可用类和方法的完整列表。
* 可选类型:变量和参数可以使用显式类型或动态def类型。
* 语法:扩展Java语法的一个子集，以提供额外的脚本语言特性。
* 优化:专门为Elasticsearch脚本设计。