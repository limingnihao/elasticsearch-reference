# How painless dispatches functions

Painless使用receiver、name和arity来进行方法分派。例如，s.foo(a, b)的解析方法是首先获取s的类，然后用两个参数查找方法foo。这与使用参数的运行时类型的Groovy和使用参数的编译时类型的Java不同。

这样做的结果是Painless不支持像Java那样的重载方法，当它允许来自Java标准库的类时，会导致一些麻烦。例如，在Java和Groovy中，Matcher有两个方法:group(int)和group(String)。Painless不能同时使用这两个方法，因为它们具有相同的名称和相同数量的参数。所以它有group(int)和namedGroup(String)。



对于这种不同的分派方法，我们有一些理由：

1. 它使得对def类型的操作更简单，也可能更快。使用receiver、name和arity意味着，当Painless看到def对象上的调用时，它可以分派适当的方法，而不必对参数的类型进行昂贵的比较。对于使用def类型参数的调用也是如此。
2. 它使事情保持一致。如果使用了任何def类型的参数，Painless的行为就像Groovy一样，而Java的行为就不一样了。如果它一直表现得像Groovy，就会变得很慢。
3. 它保持Painless可维护性。添加类似于Java或Groovy的方法分派，会让人感觉它增加了大量的复杂性，从而使维护和其他改进变得更加困难。