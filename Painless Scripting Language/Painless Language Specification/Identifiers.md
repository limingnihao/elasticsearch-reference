## Identifiers(标识符)

使用一个名字标识符来定义 [variable](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-variables.html), [type](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-types.html), [field](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-operators-reference.html#field-access-operator), [method](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-operators-reference.html#method-call-operator), or [function](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-functions.html).

**Errors**

如果使用[keyword](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-keywords.html)作为标识符。

**Grammar**

```ANTLR4
ID: [_a-zA-Z] [_a-zA-Z-0-9]*;
```

**Examples**

- Variations of identifiers.

```Painless
a
Z
id
list
list0
MAP25
_map25
Map_25
```

