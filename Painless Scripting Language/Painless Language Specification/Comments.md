## Comments(注释)

使用注释来注释或解释脚本中的代码。在一行的任何地方使用`//`令牌来指定单行注释。从`//`标记到行尾的所有字符将被忽略。使用开始`/*`标记和结束`*/`标记来指定多行注释。多行注释可以在一行的任何地方开始，并且`/*`标记和`*/`标记之间的所有字符都会被忽略。注释包含在脚本的任何地方。

**Grammar**

```ANTLR4
SINGLE_LINE_COMMENT: '//' .*? [\n\r];
MULTI_LINE_COMMENT: '/*' .*? '*/';
```



**Examples**

- Single-line comments.

  ```Painless
  // single-line comment
  
  int value; // single-line comment
  ```

- Multi-line comments.

  ```Painless
  /* multi-
     line
     comment */
  
  int value; /* multi-
                line
                comment */ value = 0;
  
  int value; /* multi-line
                comment */
  
  /* multi-line
     comment */ int value;
  
  int value; /* multi-line
                comment */ value = 0;
  
  int value; /* multi-line comment */ value = 0;
  ```

