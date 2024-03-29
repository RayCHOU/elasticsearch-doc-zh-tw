# Search your data

資訊來源： https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-your-data.html

Search query 或 query 是對有關 Elasticsearch data streams 或 索引中數據的信息的 request。

您可以將 query 視為一個問題，以 Elasticsearch 理解的方式編寫。  
根據您的數據，您可以使用 query 來獲取以下問題的答案：

* 我服務器上的哪些 processes 響應時間超過 500 毫秒？
* 我網絡上的哪些用戶在上週執行了 `regsvr32.exe`？
* 我網站上的哪些頁面包含特定的詞或 phrase？

Search 由一個或多個 queries 組成，這些 queries 組合在一起並發送到 Elasticsearch。  
與 search's queries 匹配的 documents 將在 response 的 hits 或 search results 中返回。

Search 還可能包含用於更好地處理它的 queries 的附加信息。  
例如，search 可能僅限於特定索引或僅返回特定數量的結果。

## 本頁中包含

* [Run a search](run-a-search.md)
* [定義只在 query 中存在的 fields](define-runtime-fields.md) (runtime field)
* [常用搜索選項](common-options.md) 
* [搜索超時](timeout.md) (timeout)
* [Search cancellation](cancel.md)
* [追踪 total hits](total-hits.md) (符合查詢的文件數量)
* [快速檢查文檔匹配](quick-check.md) (只想知道有沒有任何文件符合查詢)

## 本主題中包含

* [折疊(collapse)搜索結果](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/collapse-search-results.html) (欄位值相同的只取一筆)
* [過濾(filter)搜索結果](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/filter-search-results.html)
* [Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/highlighting.html)
* [花費長時間的搜索](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/async-search-intro.html) (Long-running searches)
* [接近即時回應的搜索](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/near-real-time.html) (Near real-time search)
* [分頁搜索結果](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/paginate-search-results.html) (Paginate)
* [Retrieve inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/inner-hits.html)
* [從搜索中擷取選定的欄位](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-fields.html) (Retrieve selected fields from a search)
* [跨集群搜索](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/modules-cross-cluster-search.html)
* [搜索多個 data streams 和索引](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-multiple-indices.html)
* [Search shard routing](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-shard-routing.html)
* [搜索模板](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-template.html) (預先儲存的搜尋，可使用不同變數來執行)
* [排序搜索結果](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/sort-search-results.html) (Sort)
* [kNN search](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/knn-search.html) (k-nearest neighbor)
