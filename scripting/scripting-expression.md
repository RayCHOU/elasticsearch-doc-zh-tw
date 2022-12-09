# [Lucene expressions language](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-expression.html)

Lucene’s expressions compile a `javascript` expression to bytecode. They are designed for high-performance custom ranking and sorting functions and are enabled for inline and stored scripting by default.

Lucene 的 expressions 將 `javascript` expression 編譯為 bytecode。  
它們專為高性能自定義排名和排序功能而設計，默認情況下啟用 inline 和 stored scripting。

## Performance

Expressions were designed to have competitive performance with custom Lucene code. This performance is due to having low per-document overhead as opposed to other scripting engines: expressions do more "up-front".

設計 expressions 的目的是為了與 custom Lucene code 在性能表現上能有競爭性。  
這種性能是由於與其他腳本引擎相比，每個文檔的開銷較低：expressions 更 “預先(up-front)”。

This allows for very fast execution, even faster than if you had written a native script.

這允許非常快速的執行，甚至比您編寫 native script 還要快。

## Syntax

Expressions support a subset of javascript syntax: a single expression.

Expressions 支援 javascript 語法的子集： a single expression。

See the expressions module documentation for details on what operators and functions are available.

有關可用的 operators 和 funtions 的詳細信息，請參閱 [expressions module documentation](https://lucene.apache.org/core/9_4_2/expressions/index.html?org/apache/lucene/expressions/js/package-summary.html)。

Variables in expression scripts are available to access:

* document fields, e.g. `doc['myfield'].value`
* variables and methods that the field supports, e.g. `doc['myfield'].empty`
* Parameters passed into the script, e.g. `mymodifier`
* The current document’s score, `_score` (only available when used in a `script_score`)

expression scripts 中的 variables 可用於訪問：

* documets 的 fields，例如 `doc['myfield'].value`
* 該 field 支援的 variables 和 methods，例如 `doc['myfield'].empty`
* 傳遞給 script 的參數，例如 `mymodifier`
* 當前 document 的 score，`_score`（僅在 `script_score` 中使用時可用）

You can use Expressions scripts for `script_score`, `script_fields`, sort scripts, and numeric aggregation scripts, simply set the `lang` parameter to `expression`.

您可以將 Expressions scripts 用於 `script_score`、`script_fields`、排序腳本和 numeric aggregations scripts，只需將 `lang` 參數設置為 `expression`。

## Numeric field API

| Expression | Description |
| ---------- | ----------- |
| `doc['field_name'].value` | 該 field 的值，作為 double |
| `doc['field_name'].empty` | 一個 boolean 值，指示該 field 在 doc 中是否沒有值。 |
| `doc['field_name'].length` | The number of values in this document. <br>本文檔中 values 的數量。 |
| `doc['field_name'].min()` | 本文檔中該 field 的最小值。 |
| `doc['field_name'].max()` | 本文檔中該 field 的最大值。 |
| `doc['field_name'].median()` | 本文檔中該 field 的 中值(median value)。 |
| `doc['field_name'].avg()` | 本文檔中該 field 的平均值。 |
| `doc['field_name'].sum()` | 本文檔中該 field 值的總和。 |

When a document is missing the field completely, by default the value will be treated as `0`. You can treat it as another value instead, e.g. `doc['myfield'].empty ? 100 : doc['myfield'].value`

當文檔完全缺少該 field 時，默認情況下該值將被視為 0。  
您可以將其視為另一個值，例如 `doc['myfield'].empty ？ 100 : doc['myfield'].value`

When a document has multiple values for the field, by default the minimum value is returned. You can choose a different value instead, e.g. `doc['myfield'].sum()`.

當文檔的 field 有多個值時，默認情況下返回最小值。  
您可以選擇不同的值，例如 `doc['myfield'].sum()`。

When a document is missing the field completely, by default the value will be treated as `0`.

當文檔完全缺少該 field 時，默認情況下該值將被視為 `0`。

Boolean fields are exposed as numerics, with `true` mapped to `1` and `false` mapped to `0`. For example: `doc['on_sale'].value ? doc['price'].value * 0.5 : doc['price'].value`

Boolean fields 顯示為數字，`true` 映射到 `1`，`false` 映射到 `0`。  
例如：`doc['on_sale'].value ? doc['price'].value * 0.5 : doc['price'].value`

## Date field API

Date fields are treated as the number of milliseconds since January 1, 1970 and support the Numeric Fields API above, plus access to some date-specific fields:

日期 fields 被視為自 1970 年 1 月 1 日以來的毫秒數，並支持上面的 Numeric Fields API，以及對一些特定於日期的 fields 的訪問：

| Expression | Description |
| ---------- | ----------- |
| `doc['field_name'].date.centuryOfEra` | 世紀 (1-2920000) |
| `doc['field_name'].date.dayOfMonth` | 第 (1-31) 日，例如 `1` 為本月的第一天。 |
| `doc['field_name'].date.dayOfWeek` | 星期幾 (1-7)，例如 `1` 為星期一。 |
| `doc['field_name'].date.dayOfYear` | 一年中的某一天，例如 `1` 為 1 月 1 日。 |
| `doc['field_name'].date.era` | 紀元：BC 為 `0`，AD 為 `1`。 |
| `doc['field_name'].date.hourOfDay` | 小時 (0-23). |
| `doc['field_name'].date.millisOfDay` | 一天中的第幾毫秒 (0-86399999) |
| `doc['field_name'].date.millisOfSecond` | 一秒之中的第幾毫秒 (0-999) |
| `doc['field_name'].date.minuteOfDay` | 一天之中的第幾分鐘 (0-1439) |
| `doc['field_name'].date.minuteOfHour` | 一小時之中的第幾分鐘 (0-59) |
| `doc['field_name'].date.monthOfYear` | 一年之中的第幾個月 (1-12), 例如 `1` 為一月。 |
| `doc['field_name'].date.secondOfDay` | 一天之中的第幾秒 (0-86399) |
| `doc['field_name'].date.secondOfMinute` | 一分鐘之中的第幾秒 (0-59) |
| `doc['field_name'].date.year` | 年 (-292000000 - 292000000) |
| `doc['field_name'].date.yearOfCentury` | 世紀之中的第幾年 (1-100) |
| `doc['field_name'].date.yearOfEra` | 紀元(era) 之中的第幾年 (1-292000000) |

The following example shows the difference in years between the `date` fields date0 and date1:

以下示例顯示 `date` field date0 和 date1 之間的年份差異：

    doc['date1'].date.year - doc['date0'].date.year

## `geo_point` field API

| Expression | Description |
| ---------- | ----------- |
| `doc['field_name'].empty` | 一個 boolean 值，指示該 field 在 doc 中是否沒有值。 |
| `doc['field_name'].lat` | 地理點的緯度。 | 
| `doc['field_name'].lon` | 地理點的經度。 | 

以下示例計算距華盛頓特區的距離（以公里為單位）：

    haversin(38.9072, 77.0369, doc['field_name'].lat, doc['field_name'].lon)

In this example the coordinates could have been passed as parameters to the script, e.g. based on geolocation of the user.

在這個例子中，坐標可以作為參數傳遞給腳本，例如 基於用戶的地理位置。

## Limitations

There are a few limitations relative to other script languages:

* Only numeric, boolean, date, and geo_point fields may be accessed
* Stored fields are not available

相對於其他腳本語言有一些限制：

* 只能訪問 數字、布爾值、日期和 geo_point field
* stored fields 不可用
