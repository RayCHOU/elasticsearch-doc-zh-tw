# 常用搜索選項

資訊來源： https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-your-data.html#common-search-options

您可以使用以下選項來自訂您的搜索。

## Query DSL

[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl.html) 支持多種查詢類型，您可以混合搭配以獲得所需的結果。 

查詢類型包括：

* [Boolean](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-bool-query.html)
  和其他 [compound queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/compound-queries.html)，
  可讓您根據多個條件組合查詢和匹配結果
* [Term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/term-level-queries.html)：用於過濾和查找精確匹配
* [Full text queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/full-text-queries.html)：常用於搜索引擎
* [Geo](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/geo-queries.html) (地理) 和
  [spatial queries](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/shape-queries.html) (空間查詢)

## Aggregations

您可以使用 [search aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-aggregations.html) 
來獲取搜索結果的統計信息和其他分析。 
Aggregations 可幫助您回答以下問題：

* 我的服務器的平均 response time 是多少？
* 我的網絡上用戶訪問次數最多的 IP 地址是什麼？
* 客戶的總交易收入是多少？

## 搜索多個 data streams 和索引

您可以使用 逗號分隔值 和 grep-like 的 index patterns 在同一請求中搜索多個 data streams 和索引。  
您甚至可以提高(boost)特定索引的搜索結果。  
請參閱 [搜索多個數據流和索引](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-multiple-indices.html)。

## 分頁 搜索結果

預設情況下，搜索僅返回前 10 個匹配結果。  
要檢索更多或更少的文檔，請參閱 [對搜索結果進行分頁](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/paginate-search-results.html)。

# 擷取選定的 fields

Search response 的 hit.hits 屬性包括每個 hit 的完整文檔 [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping-source-field.html)。  
要僅擷取 `_source` 的子集或其他 fields，請參閱 [擷取選定字段](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-fields.html)。

# 排序搜索結果

預設情況下，search hits 按 `_score` 排序，`_score` 是衡量每個文檔與查詢匹配程度的
[relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-filter-context.html#relevance-scores)。  
要自訂這些分數的計算，請使用 [`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-script-score-query.html) 查詢。 
要按其他 field values 對 search hits 進行排序，請參閱 [對搜索結果進行排序](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/sort-search-results.html)。

## 執行 async 搜索

Elasticsearch 搜索旨在快速運行大量數據，通常在幾毫秒內返回結果。 
因此，預設情況下搜索是同步的。 Search request 在返迴 response 之前等待完整的結果。

但是，對於跨大型數據集或
[多個集群](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/modules-cross-cluster-search.html) 的搜索，
完整的結果可能需要更長的時間。

為避免長時間等待，您可以改為執行 非同步 或 async search。 
[Async search](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/async-search-intro.html) 
可讓您立即取得「長時間運行的搜索」的部分結果，稍後再獲取完整結果。
