# [Scalability and resilience: clusters, nodes, and shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/scalability.html)

Elasticsearch 旨在始終可用並根據您的需求進行擴展。
它通過 distributed by nature 來做到這一點。
您可以將 servers（nodes）添加到 cluster 集群以增加容量，
Elasticsearch 會自動將您的數據和 query load 分佈到所有可用 nodes 上。
無需大修您的應用程序，Elasticsearch 知道如何平衡 multi-node clusters 以提供可擴展性和高可用性。
Nodes 越多越好。

這是如何運作的？
Elasticsearch 索引實際上只是一個或多個 physical shards 的邏輯分組，
其中每個 shard 實際上是一個自包含索引。
通過將索引中的 documents 分佈在多個 shards 上，並將這些 shards 分佈在多個 nodes 上，
Elasticsearch 可以確保 redundancy，這既可以防止硬件故障，又可以在將 nodes 添加到 cluster 時增加查詢容量。
隨著 cluster 的增長（或縮小），Elasticsearch 會自動遷移 shards 以重新平衡 cluster。

Shards 有兩種類型：主分片和副本分片。
索引中的每個 document 都屬於一個主分片。副本分片是主分片的副本。
副本提供數據的冗余副本以防止硬件故障並增加處理讀取請求（如搜索或檢索文檔）的容量。

索引中 主分片 的數量在創建索引時就固定了，
但 副本分片 的數量可以隨時更改，而不會中斷索引或查詢操作。

## It depends…

Shard 的大小、數量會影響效能。
剛開始的時候：

* 將 shard 平均大小保持在幾 GB 到幾十 GB 之間。
* 避免大量 shards 問題。一個 node 可以容納的分片數量與可用的 heap space 成正比。作為一般規則，每 GB 的 heap space 的 shards 數量應小於 20。

為您的 use case 確定最佳配置的最佳方法是 [使用您自己的數據和查詢進行測試](https://www.elastic.co/elasticon/conf/2016/sf/quantitative-cluster-sizing)。

(略)
