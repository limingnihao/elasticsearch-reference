# Painless Debugging

## Debug.Explain

Painless没有[REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)(**read–eval–print loop**)，虽然它会很好，但它不会告诉你关于debugging painless脚本嵌入在Elasticsearch中的Painless脚本的全部，因为脚本可以访问的数据或“context”是如此重要。目前，调试嵌入脚本的最佳方法是在指定位置抛出异常。虽然你可以抛出自己的异常(抛出新的异常('whatever'))，但Painless的沙箱会阻止你访问有用的信息，比如对象的类型。Painless有一个工具方法，`Debug.explain`解释哪个为你抛出异常。您可以使用[`_explain`](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/search-explain.html)来研究 [script query](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/query-dsl-script-query.html)可用的上下文。

