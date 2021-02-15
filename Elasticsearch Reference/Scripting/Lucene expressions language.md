# Lucene expressions language

Lucene expressions将一个javascript表达式编译成字节码。它们是为高性能定制排序和排序功能而设计的，默认情况下支持`inline`和`scored`脚本。



## Performance

表达式被设计成可以赶上自定义Lucene code的性能。与其他脚本引擎相比，高性能是由于每个文档的开销较低:表达式做了更多的“预先”工作。

这允许非常快的执行，甚至比您编写本机脚本还要快。



## Syntax

表达式支持javascript语法的一个子集:单个表达式。

有关可用的操作符和函数的详细信息，请参阅表达式[expressions module documentation](http://lucene.apache.org/core/8_6_0/expressions/index.html?org/apache/lucene/expressions/js/package-summary.html)。

表达式脚本中的变量可以访问:

- 文档的字段。例如： `doc['myfield'].value`
- 字段支持的变量或方法, 例如：`doc['myfield'].empty`
- 传递到脚本的参数, 例如：`mymodifier`
- 当前文档的 score, `_score` (仅在 `script_score`时可用)

你可以使用Expressions scripts在`script_score`, `script_fields`, sort scripts, and numeric aggregation scripts, 只需将lang参数设置为expression即可。



## Numeric field API

| **Expression**             | **Description**                          |
| -------------------------- | ---------------------------------------- |
| doc['field_name'].value    | 字段的取值，double类型                   |
| doc['field_name'].empty    | 一个布尔值，指示字段在文档中是否没有值。 |
| doc['field_name'].length   | 此文档中值的数量。                       |
| doc['field_name'].min()    | 此文档中字段的最小值。                   |
| doc['field_name'].max()    | 此文档中字段的最大值。                   |
| doc['field_name'].median() | 此文档中字段的中值。                     |
| doc['field_name'].avg()    | 此文档中字段的平均值。                   |
| doc['field_name'].sum()    | 此文档中值的总和。                       |



当文档完全丢失该字段时，默认情况下该值将被视为0。你可以设置为其他值，例如：`doc['myfield'].empty ? 100 : doc['myfield'].value`

当文档的字段有多个值时，默认情况下返回最小值。你可以选择一个不同的值，例如：`doc['myfield'].sum()`

当文档完全丢失该字段时，默认情况下该值将被视为0。

布尔字段被转换为数值时，true映射为1,false映射为0，例如：doc['on_sale'].value ? doc['price'].value * 0.5 : doc['price'].value



## Date field API

日期字段被视为自1970年1月1日以来的毫秒数，支持上面数值字段API，再加上访问一些特定日期的字段:

| Expression                              | Description                                         |
| :-------------------------------------- | --------------------------------------------------- |
| `doc['field_name'].date.centuryOfEra`   | Century (1-2920000)                                 |
| `doc['field_name'].date.dayOfMonth`     | Day (1-31), e.g. `1` for the first of the month.    |
| `doc['field_name'].date.dayOfWeek`      | Day of the week (1-7), e.g. `1` for Monday.         |
| `doc['field_name'].date.dayOfYear`      | Day of the year, e.g. `1` for January 1.            |
| `doc['field_name'].date.era`            | Era: `0` for BC, `1` for AD.                        |
| `doc['field_name'].date.hourOfDay`      | Hour (0-23).                                        |
| `doc['field_name'].date.millisOfDay`    | Milliseconds within the day (0-86399999).           |
| `doc['field_name'].date.millisOfSecond` | Milliseconds within the second (0-999).             |
| `doc['field_name'].date.minuteOfDay`    | Minute within the day (0-1439).                     |
| `doc['field_name'].date.minuteOfHour`   | Minute within the hour (0-59).                      |
| `doc['field_name'].date.monthOfYear`    | Month within the year (1-12), e.g. `1` for January. |
| `doc['field_name'].date.secondOfDay`    | Second within the day (0-86399).                    |
| `doc['field_name'].date.secondOfMinute` | Second within the minute (0-59).                    |
| `doc['field_name'].date.year`           | Year (-292000000 - 292000000).                      |
| `doc['field_name'].date.yearOfCentury`  | Year within the century (1-100).                    |
| `doc['field_name'].date.yearOfEra`      | Year within the era (1-292000000).                  |



下面的示例显示日期字段date0和date1在年份上的差异：`doc['date1'].date.year - doc['date0'].date.year`



## Geo_point field API

| Expression                | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `doc['field_name'].empty` | A boolean indicating if the field has no values within the doc. |
| `doc['field_name'].lat`   | The latitude of the geo point.                               |
| `doc['field_name'].lon`   | The longitude of the geo point.                              |

下面的例子计算距离，以公里为单位：`haversin(38.9072, 77.0369, doc['field_name'].lat, doc['field_name'].lon)`

在这个例子中，坐标可以作为参数传递给脚本，例如基于用户的地理位置。



## Limitations

有一些限制相对于其他脚本语言:

- 只能访问数字、布尔值、日期和地理点字段
- 存储字段不可用