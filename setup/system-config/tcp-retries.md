# [TCP retransmission timeout](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config-tcpretries.html)

每對 Elasticsearch 節點通過多個 TCP 連接進行通信，
這些連接 [保持打開狀態](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#long-lived-connections)，
直到其中一個節點關閉 或 節點之間的通信因底層基礎設施故障而中斷。

(略)
