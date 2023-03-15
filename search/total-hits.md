# Track total hits

資訊來源： https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-your-data.html#track-total-hits

通常，如果不訪問所有匹配項，就無法準確計算 total hit count，因為這對於匹配大量文檔的查詢來說成本很高。  
`track_total_hits` 參數允許您控制應如何跟踪 total number of hits。  
鑑於通常有一個命中數的下限就足夠了，例如「至少有 10000 次命中」，預設為 `10,000`。  
這意味著 requests 將準確計算 total hit，最高可達 `10,000` 次。  
如果您不需要特定門檻後的準確命中數，那麼加快搜索速度是一個很好的權衡。  

當設置為 `true` 時，search response 將始終跟踪與查詢準確匹配的 number of hits
（例如，當 `track_total_hits` 設置為 true 時，`total.relation` 將始終等於 `"eq"`）。  
否則，search response 中 "total" object 中返回的 `"total.relation"` 決定了應如何解釋 `"total.value"`。  
`"gte"` 值表示 `"total.value"` 是與查詢匹配的 total hits 的下限，`"eq"` 值表示 `"total.value"` 是準確計數。

```HTTP
GET my-index-000001/_search
{
  "track_total_hits": true,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  }
}
```

回傳：

```JSON
{
  "_shards": ...
  "timed_out": false,
  "took": 100,
  "hits": {
    "max_score": 1.0,
    "total" : {
      "value": 2048,    
      "relation": "eq"  
    },
    "hits": ...
  }
}
```

上面的 `"value": 2048` 表示：與查詢匹配的 total number of hits。  
`"relation": "eq"` 表示：計數是準確的（"eq" 表示 equal 等於）。

也可以將 `track_total_hits` 設置為整數。  
例如，以下查詢將準確跟踪匹配查詢的 total hit count，最多 100 個文檔：

```HTTP
GET my-index-000001/_search
{
  "track_total_hits": 100,
  "query": {
    "match": {
      "user.id": "elkbee"
    }
  }
}
```

Response 中的 `hits.total.relation` 將指示 `hits.total.value` 中返回的值是準確的（`"eq"`）還是總數的下限（`"gte"`）。

例如以下 response：

```JSON
{
  "_shards": ...
  "timed_out": false,
  "took": 30,
  "hits": {
    "max_score": 1.0,
    "total": {
      "value": 42,         
      "relation": "eq"     
    },
    "hits": ...
  }
}
```

* `"value": 42` 表示：有 42 個 documents 符合查詢。
* `"relation": "eq"` 表示： 這個數字是準確 (`"eq"`) 的。

... 以上表示 total 中返回的 number of hits 是準確的。

如果匹配查詢的 total number of hits 大於 `track_total_hits`中設置的值，
則 response 中的 total hits 將表明返回值為下限：

```JSON
{
  "_shards": ...
  "hits": {
    "max_score": 1.0,
    "total": {
      "value": 100,         
      "relation": "gte"     
    },
    "hits": ...
  }
}
```

* `"value": 100` 表示：至少有 100 個文檔與查詢匹配
* `"relation": "gte"` 表示： 這是一個下限（"gte"）。

如果您根本不需要追踪 total number of hits，
則可以將此選項設置為 `false` 來縮短查詢時間：

```HTTP
GET my-index-000001/_search
{
  "track_total_hits": false,
  "query": {
    "match": {
      "user.id": "elkbee"
    }
  }
}
```

回傳：

```JSON
{
  "_shards": ...
  "timed_out": false,
  "took": 10,
  "hits": {             
    "max_score": 1.0,
    "hits": ...
  }
}
```

註： total number of hits 是未知的

最後，您可以通過在 request 中將 `"track_total_hits"` 設置為 `true` 來強制進行準確計數。
