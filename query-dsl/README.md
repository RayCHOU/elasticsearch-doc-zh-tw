# Query DSL

資訊來源： https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl.html

Elasticsearch 提供了一個完整的基於 JSON 的 Query DSL（Domain Specific Language, 領域特定語言）來定義 queries。  
將 Query DSL 視為 queries 的 AST（Abstract Syntax Tree, 抽象語法樹），由兩種類型的子句組成：

## 葉查詢子句 (Leaf query clauses)

葉查詢子句在特定 field 中查找特定值，例如 [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-match-query.html)、
[`term`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-term-query.html)
或 [`range`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-range-query.html) 查詢。 這些查詢可以自己使用。

## 複合查詢子句

複合查詢子句包裝其他 葉查詢 或 複合查詢，用於以邏輯方式組合多個查詢（例如
[`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-bool-query.html) 
或 [`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-dis-max-query.html) 
查詢），或改變它們的行為（例如 [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-constant-score-query.html) 
查詢）。
查詢子句的行為有所不同，具體取決於它們是在
[查詢上下文中 還是在 篩選上下文中](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-filter-context.html) 使用。

## 允許昂貴的查詢

某些類型的查詢通常會由於其實現方式而執行緩慢，這會影響集群的穩定性。  
這些查詢可以分類如下：

* 需要進行線性掃描以識別匹配項的查詢：
  * [`script` queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-script-query.html)
  * 查詢 [數字](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/number.html)、
    [日期](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/date.html)、
    [布林值](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/boolean.html)、
    [ip](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/ip.html)、
    [geo_point](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/geo-point.html) 
    或未編入索引但啟用了 [doc values](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/doc-values.html)
    的 [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/keyword.html) field
* 前期成本高的查詢：
  * [`fuzzy` 查詢](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-fuzzy-query.html)
    （[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/keyword.html#wildcard-field-type) fields 除外）
  * [`regexp` 查詢](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-regexp-query.html)
    （[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/keyword.html#wildcard-field-type) fields 除外）
  * [`prefix` 查詢](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-prefix-query.html)
    （[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/keyword.html#wildcard-field-type) 或沒有
    [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/index-prefixes.html) 的 fields 除外）
  * [`wildcard` 查詢](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-wildcard-query.html)
    （[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/keyword.html#wildcard-field-type) fields 除外）
  * [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/text.html) 和 
    [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/keyword.html) fields 的 
    [`range` 查詢](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-range-query.html)
* [Joining queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/joining-queries.html)
* 每個文檔成本可能很高的查詢：
  * [`script_score` queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-script-score-query.html)
  * [`percolate` 滲透查詢](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-percolate-query.html)
  
可以通過將 `search.allow_expensive_queries` setting 的值設置為 `false`（預設為 `true`）來阻止執行此類查詢。

## 本主題下的子題

* [Query and filter context](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-filter-context.html)
* [複合查詢](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/compound-queries.html): 將其他複合查詢或葉查詢包在一起
* [Full text queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/full-text-queries.html) (待譯)
* [Geo queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/geo-queries.html): 地理查詢
* [Shape queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/shape-queries.html): 二維形狀查詢
* [Joining queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/joining-queries.html)
* [Match all query](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-match-all-query.html): 全部文件都符合
* [Span queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/span-queries.html)
* [Specialized queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/specialized-queries.html)
* [Term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/term-level-queries.html)
* [`minimum_should_match` 參數](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-minimum-should-match.html)
* [`rewrite` 參數](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-multi-term-rewrite.html)
* [Regular expression syntax](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/regexp-syntax.html)
