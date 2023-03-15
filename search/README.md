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

## 本小節中包含

* [Run a search](run-a-search.md)
* [定義只在 query 中存在的 fields](define-runtime-fields.md)
* [常用搜索選項](common-options.md)
