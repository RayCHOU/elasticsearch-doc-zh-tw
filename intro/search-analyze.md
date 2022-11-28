# [Information out: search and analyze](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-analyze.html)

Elasticsearch 提供了一個簡單、連貫的 REST API，
用於管理您的 cluster 以及索引和搜索您的數據。
在您的 applications 中，您可以將 [Elasticsearch client](https://www.elastic.co/guide/en/elasticsearch/client/index.html) 用於您選擇的語言：Java、JavaScript、Go、.NET、PHP、Perl、Python 或 Ruby。

## Searching your data

Elasticsearch REST API 支援 structured quries、full text queries 和將兩者結合的 complex queries。
Structured quries 類似於您可以在 SQL 中構造的查詢類型。
例如，您可以在 `employee` index 中搜索 `gender` 和 `age` fields，並按 `hire_date` field 對匹配進行排序。
Full-text queries 查找與 query string 匹配的所有文檔，並按相關性（它們與您的 search term 的匹配程度）排序返回它們。

除了搜索單個術語之外，
您還可以執行 phrase searches、similarity searches 和 prefix searches，
並獲得 autocomplete suggestions。

您可以使用 Elasticsearch 全面的 JSON 樣式查詢語言 ([Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)) 存取所有這些搜索功能。
您還可以構建 SQL 樣式的查詢。

## Analyzing your data

Elasticsearch aggregations 使您能夠構建複雜的數據摘要，並深入了解關鍵指標、模式和趨勢。
Aggregations 不僅可以找到眾所周知的「大海撈針」，還可以讓您回答以下問題：

* 大海裡有多少針？
* 針的平均長度是多少？
* 針的中位長度是多少，按製造商細分？
* 在過去的六個月中，每年有多少針被添加到大海中？

您還可以使用 aggreations 來回答更微妙的問題，例如：

* 最受歡迎的針頭製造商？
* 是否有任何不尋常或異常的針團？
 
## But wait, there’s more

想要自動分析您的時間序列數據？
您可以使用 [機器學習](https://www.elastic.co/guide/en/machine-learning/8.5/ml-ad-overview.html) 功能在數據中創建準確的正常行為基線並識別異常模式。
通過機器學習，您可以檢測：

* 與值、計數或頻率的時間偏差相關的異常
* 統計稀有性
* 群體成員的異常行為

最棒的是什麼？
您無需指定 演算法、模型或其他與 data science 相關的 configurations 即可執行此操作。
