# [Reading indices from older Elasticsearch versions](https://www.elastic.co/guide/en/elasticsearch/reference/current/archive-indices.html)

在以前的主要版本中創建的索引，Elasticsearch 仍具有完整的查詢和寫入支持。  
如果您在 Elasticsearch 版本 5 或 6 中創建了索引，您現在也可以使用 archive functionality 將它們導入到較新的 Elasticsearch 版本中。

出於 compliance 或 regulatory 原因、偶爾的回顧或調查，或對部分數據進行 rehydrate，
archive 功能提供了對舊 Elasticsearch 數據的較慢 read-only 訪問。 
預計對數據的訪問不頻繁，因此可能會在性能和查詢能力有限的情況下發生。

為此，Elasticsearch 能夠訪問較舊的 snapshot 存儲庫（回到版本 5）。 
snapshot 存儲庫中的舊索引可以恢復，也可以通過可搜索的快照直接訪問，這樣歸檔數據甚至不需要完全駐留在本地磁盤上，即可 access。

## [Supported field types](https://www.elastic.co/guide/en/elasticsearch/reference/current/archive-indices.html#archive-indices-supported-field-types)

略

## [Supported APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/archive-indices.html#_supported_apis)

略

## [How to upgrade older Elasticsearch 5 or 6 clusters?](https://www.elastic.co/guide/en/elasticsearch/reference/current/archive-indices.html#_how_to_upgrade_older_elasticsearch_5_or_6_clusters)

略
