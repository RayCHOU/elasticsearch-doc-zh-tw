# 定義只在 query 中存在的 fields

資訊來源： https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-your-data.html#run-search-runtime-fields

您可以定義僅作為 search query 的一部分存在的 [runtime fields](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/runtime-search-request.html)，
而不是為您的數據編制索引然後進行搜索。
您在 search request 中指定一個 `runtime_mappings` 部分來定義 runtime field，它可以選擇包含一個 Painless script。

例如，以下 query 定義了一個名為 `day_of_week` 的 runtime field。
包含的 script 根據 `@timestamp` field 的值計算星期幾，並使用 `emit` 返回計算值。

該查詢還包括一個在 `day_of_week` 上運行的 [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/search-aggregations-bucket-terms-aggregation.html)。

```HTTP
GET /my-index-000001/_search
{
  "runtime_mappings": {
    "day_of_week": {
      "type": "keyword",
      "script": {
        "source":
        """emit(doc['@timestamp'].value.dayOfWeekEnum
        .getDisplayName(TextStyle.FULL, Locale.ROOT))"""
      }
    }
  },
  "aggs": {
    "day_of_week": {
      "terms": {
        "field": "day_of_week"
      }
    }
  }
}
```

Response 包括基於 `day_of_week` runtime field 的 aggregation。  
在 `buckets` 下是一個值為 `Sunday` 的 `key`。  
Query 根據 `day_of_week` runtime field 中定義的 script 動態計算此值，而無需對該字段建立索引。

```JSON
{
  ...
  ***
  "aggregations" : {
    "day_of_week" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Sunday",
          "doc_count" : 5
        }
      ]
    }
  }
}
```
