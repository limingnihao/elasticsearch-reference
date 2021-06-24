## Variables(变量)

变量是在[operations](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-operators.html)期间加载并存储一个值。



### Declaration(声明)

在使用前声明变量，格式为type后跟identifier。在标识符后面直接使用每个维度的开始`[`标记和结束`]`标记来声明数组类型变量。在类型后面指定以逗号分隔的标识符列表，以便在单个语句中声明多个变量。将赋值操作符与声明结合使用，可立即将值赋给变量。没有立即赋值的变量将根据类型隐式地赋值一个默认值。

**Errors**

- 如果变量在声明之前使用或没有声明。

**Grammar**

```ANTLR4
declaration : type ID assignment? (',' ID assignment?)*;
type: ID ('.' ID)* ('[' ']')*;
assignment: '=' expression;
```

**Examples**

- 变量声明的不同变体。


| 示例             | 解释                                                         |
| ---------------- | ------------------------------------------------------------ |
| int x;           | declare `int x`; store default `null` to `x`                 |
| List y;          | declare `List y`; store default `null` to `y`                |
| int x, y = 5, z; | declare `int x`; store default `int 0` to `x`; declare `int y`; store `int 5` to `y`; declare `int z`; store default `int 0` to `z`; |
| def d;           | declare `def d`; store default `null` to `d`                 |
| int i = 10;      | declare `int i`; store `int 10` to `i`                       |
| float[] f;       | declare `float[] f`; store default `null` to `f`             |
| Map[][] m;       | declare `Map[][] m`; store default `null` to `m`             |



### Assignment(分配)

使用 `assignment operator '='` 将值存储在变量中以供后续操作使用。任何操作,生成一个值可以分配给任何变量,只要[types](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-types.html)是相同的或合成的类型可以[implicitly cast](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-casting.html)的变量类型。

**Errors**

- 如果值的类型不能匹配变量的类型。

**Grammar**

```ANTLR4
assignment: ID '=' expression
```



**Examples**

- Variable assignment with an integer literal.

| 示例    | 解释                                          |
| ------- | --------------------------------------------- |
| int i;  | declare `int i`; store default `int 0` to `i` |
| i = 10; | store `int 10` to `i`                         |


- Declaration combined with immediate assignment.

| 示例            | 解释                                          |
| --------------- | --------------------------------------------- |
| int i = 10;     | declare `int i`; store `int 10` to `i`        |
| double j = 2.0; | declare `double j`; store `double 2.0` to `j` |


- Assignment of one variable to another using primitive type values.

| 示例        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| int i = 10; | declare `int i`; store `int 10` to `i`                       |
| int j = i;  | declare `int j`; load from `i` → `int 10`; store `int 10` to `j` |


- Assignment with reference types using the [new instance operator](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-operators-reference.html#new-instance-operator).

| 示例                           | 解释                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| ArrayList l = new ArrayList(); | declare `ArrayList l`; allocate `ArrayList` instance → `ArrayList reference`; store `ArrayList reference` to `l` |
| Map m = new HashMap();         | declare `Map m`; allocate `HashMap` instance → `HashMap reference`; implicit cast `HashMap reference` to `Map reference` → `Map reference`; store `Map reference` to `m` |

- Assignment of one variable to another using reference type values.

| 示例                      | 解释                                                         |
| ------------------------- | ------------------------------------------------------------ |
| List l = new ArrayList(); | declare `List l`; allocate `ArrayList` instance → `ArrayList reference`; implicit cast `ArrayList reference` to `List reference` → `List reference`; store `List reference` to `l` |
| List k = l;               | declare `List k`; load from `l` → `List reference`; store `List reference` to `k`; (note `l` and `k` refer to the same instance known as a shallow-copy) |
| List m;                   | declare `List m`; store default `null` to `m`                |
| m = k;                    | load from `k` → `List reference`; store `List reference` to `m`; (note `l`, `k`, and `m` refer to the same instance) |

- Assignment with array type variables using the [new array operator](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-operators-array.html#new-array-operator).

| 示例                         | 解释                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| int[] ia1;                   | declare `int[] ia1`; store default `null` to `ia1`           |
| ia1 = new int[2];            | allocate `1-d int array` instance with `length [2]` → `1-d int array reference`; store `1-d int array reference` to `ia1` |
| ia1[0] = 1;                  | load from `ia1` → `1-d int array reference`; store `int 1` to `index [0]` of `1-d int array reference` |
| int[] ib1 = ia1;             | declare `int[] ib1`; load from `ia1` → `1-d int array reference`; store `1-d int array reference` to `ib1`; (note `ia1` and `ib1` refer to the same instance known as a shallow copy) |
| int[][] ic2 = new int[2][5]; | declare `int[][] ic2`; allocate `2-d int array` instance with `length [2, 5]` → `2-d int array reference`; store `2-d int array reference` to `ic2` |
| ic2[1][3] = 2;               | load from `ic2` → `2-d int array reference`; store `int 2` to `index [1, 3]` of `2-d int array reference` |
| ic2[0] = ia1;                | load from `ia1` → `1-d int array reference`; load from `ic2` → `2-d int array reference`; store `1-d int array reference` to `index [0]` of `2-d int array reference`; (note `ia1`, `ib1`, and `index [0]` of `ia2` refer to the same instance) |

