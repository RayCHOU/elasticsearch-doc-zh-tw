# [Cluster-level shard allocation and routing settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html)

Shard allocation 是將 shards 分配給 nodes 的過程。  
這可能發生在 initial recovery、replica allocation、rebalancing 或添加或刪除節點時。

master 的主要角色之一是決定將哪些 shards 分配給哪些 nodes，  
以及何時在 nodes 之間移動 shards 以重新平衡 cluster。

(略)
