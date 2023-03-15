# 快速檢查匹配的文檔

資訊來源： https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-your-data.html#quickly-check-for-matching-docs

如果只想知道是否有任何文檔匹配特定查詢，可以將 `size` 設置為 `0` 以表示我們對搜索結果不感興趣。  
您還可以將 `terminate_after` 設置為 `1` 以指示只要找到第一個匹配文檔（每個分片）就可以終止查詢執行。

```HTTP
GET /_search?q=user.id:elkbee&size=0&terminate_after=1
```

註：`terminate_after` 始終在 [`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/filter-search-results.html#post-filter) 
**之後**應用，並在分片上收集到足夠的命中時停止查詢和聚合執行。 
然而 aggregations 的 doc count 可能不會反映在 response 中的 `hits.total`，因為 aggregations 是在 post filtering **之前**應用的。

Response 將不包含任何 hits，因為 `size` 設置為 `0`。
`hits.total` 將等於 `0`，表示沒有匹配的文檔，或者大於 `0`，表示當它提前終止時至少這麼多與查詢匹配的文檔。 
此外，如果查詢提前終止，`terminated_early` 標誌將在 response 中設置為 `true`。

```JSON
{
  "took": 3,
  "timed_out": false,
  "terminated_early": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}
```

Response 中的 `took` 時間包含處理此請求所花費的毫秒數，從節點收到查詢後迅速開始，直到完成所有與搜索相關的工作並且在上述 JSON 返回給客戶端之前。  
這意味著它包括在 thread pools 中等待、在整個集群中執行分佈式搜索並收集所有結果所花費的時間。
