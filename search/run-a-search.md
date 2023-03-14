# Run a search

資訊來源： https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-your-data.html#run-an-es-search

您可以使用 [search API](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-search.html) 
來搜索和 [aggregate](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-aggregations.html) 
存儲在 Elasticsearch data streams 或索引中的數據。  
API 的 `query` request body parameter 接受用 [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl.html) 編寫的 queries。

以下 request 使用 [match](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/query-dsl-match-query.html) query 搜索 `my-index-000001`。  
此 query 匹配 `user.id` 值為 `kimchy` 的文檔。

```HTTP
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

API response 在 hits.hits 屬性中返回與 query 匹配的前 10 個文檔。

```JSON
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.3862942,
    "hits": [
      {
        "_index": "my-index-000001",
        "_id": "kxWFcnMByiguvud1Z8vC",
        "_score": 1.3862942,
        "_source": {
          "@timestamp": "2099-11-15T14:12:12",
          "http": {
            "request": {
              "method": "get"
            },
            "response": {
              "bytes": 1070000,
              "status_code": 200
            },
            "version": "1.1"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "source": {
            "ip": "127.0.0.1"
          },
          "user": {
            "id": "kimchy"
          }
        }
      }
    ]
  }
}
```
