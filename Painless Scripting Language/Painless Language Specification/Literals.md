## Literals(字面意思)

使用字面意思在[operation](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-operators.html)中直接指定值。

### Integers

使用整型字面值指定基本类型int、long、float或double的十进制、八进制或十六进制表示法中的整型值。使用以下单字母指定基本类型:l或l表示long, f或f表示float, d或d表示double。如果没有指定，类型默认为int。使用0作为前缀来指定整数字面值为八进制，使用0x或0x作为前缀来指定整数字面值为十六进制。

**Grammar**

```ANTLR4
INTEGER: '-'? ( '0' | [1-9] [0-9]* ) [lLfFdD]?;
OCTAL:   '-'? '0' [0-7]+ [lL]?;
HEX:     '-'? '0' [xX] [0-9a-fA-F]+ [lL]?;
```

**Examples**

- Integer literals.

| 示例  | 解释               |
| ----- | ------------------ |
| 0     | `int 0`            |
| 0D    | `double 0.0`       |
| 1234L | `long 1234`        |
| -90f  | `float -90.0`      |
| -022  | `int -18` in octal |
| 0xF2A | `int 3882` in hex  |



### Floats

使用浮点字面值指定[原始类型]的浮点类型值(https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-types.html#primitive-types)“double”或“float”。使用以下单字母指定基本类型:' f '或' f '表示' float '， ' d '或' d '表示' double '。如果未指定，类型默认为' double '。

**Grammar**

```ANTLR4
DECIMAL: '-'? ( '0' | [1-9] [0-9]* ) (DOT [0-9]+)? EXPONENT? [fFdD]?;
EXPONENT: ( [eE] [+\-]? [0-9]+ );
```

**Examples**

- Floating point literals.

| 示例     | 解释                                    |
| -------- | --------------------------------------- |
| 0.0      | `double 0.0`                            |
| 1E6      | `double 1000000.0` in exponent notation |
| 0.977777 | `double 0.977777`                       |
| -126.34  | `double -126.34`                        |
| 89.9F    | `float 89.9`                            |



### Strings

使用带有单引号或双引号的字符串指定的string类型。使用\"标记包含双引号作为双引号字符串的一部分。使用`\''`标记将单引号作为单引号字符串的一部分。使用`\\`标记将反斜杠作为任何字符串的一部分。

**Grammar**

```ANTLR4
STRING: ( '"'  ( '\\"'  | '\\\\' | ~[\\"] )*? '"'  )
      | ( '\'' ( '\\\'' | '\\\\' | ~[\\'] )*? '\'' );
```

**Examples**

- 使用单引号的字符串字面值。String literals using single-quotes.

  ```Painless
  'single-quoted string literal'
  '\'single-quoted with escaped single-quotes\' and backslash \\'
  'single-quoted with non-escaped "double-quotes"'
  ```

- 使用双引号的字符串字面值。String literals using double-quotes.

  ```Painless
  "double-quoted string literal"
  "\"double-quoted with escaped double-quotes\" and backslash: \\"
  "double-quoted with non-escaped 'single-quotes'"
  ```

### Characters

字符不是直接指定的。相反，使用[强制转换操作符](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-casting.html#string-character-casting)将' String '类型值转换为' char '类型值。

