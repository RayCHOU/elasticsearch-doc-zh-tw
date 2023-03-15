# Search timeout

預設情況下，搜索請求不會超時。  
該請求在返迴 response 之前等待每個分片(shard)的完整結果。

雖然 [async search](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/async-search-intro.html) 
是為「長時間運行的搜索」而設計的，
但您也可以使用 `timeout` 參數來指定您希望在每個分片上等待完成的持續時間。  
每個分片在指定時間內收集 hits。  
如果在時間結束時收集尚未完成，Elasticsearch 僅使用截至該時間點累積的 hits。  
Search request 的整體延遲取決於「搜索所需的分片數量」和「concurrent shard requests」的數量。

```HTTP
GET /my-index-000001/_search
{
  "timeout": "2s",
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

要為所有 search requests 設置 cluster-wide 的 預設 timeout，
請使用 [cluster settings API](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/cluster-update-settings.html) 
配置 `search.default_search_timeout`。  
如果 request 中未傳遞 `timeout` 參數，則使用此 global timeout。  
如果 global search timeout 在 search request 完成之前到期，則使用 [task cancellation](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/tasks.html#task-cancellation)
來取消請求。  
`search.default_search_timeout` 設置 預設為 -1（無超時）。

