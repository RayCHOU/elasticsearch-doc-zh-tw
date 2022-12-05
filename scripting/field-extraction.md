# Field extraction

Field extraction 的目標很簡單；您的數據中有包含大量信息的 fields，但您只想提取片段和部分。

您有兩種選擇：

* [Grok](https://www.elastic.co/guide/en/elasticsearch/reference/current/grok.html) 是一種 regular expression 方言，它支持可以重複使用的別名表達式。 因為 Grok 位於 regular expressions (regex) 之上，所以任何 regular expressions 在 grok 中也有效。
* [Dissect](https://www.elastic.co/guide/en/elasticsearch/reference/current/dissect.html) 從文本中提取結構化 fields，使用 delimiters 定義匹配模式。 與 grok 不同，dissect 不使用 regular expressions。

讓我們從一個簡單的例子開始，將 `@timestamp` 和 `message` fields 作為 indexed fileds 添加到 `my-index` mapping 中。 為了保持靈活性，使用 `wildcard` 作為 `message` 的 field type：

```http
PUT /my-index/
{
  "mappings": {
    "properties": {
      "@timestamp": {
        "format": "strict_date_optional_time||epoch_second",
        "type": "date"
      },
      "message": {
        "type": "wildcard"
      }
    }
  }
}
```

mapping 完要檢索的 fields 後，將 log 數據中的幾條記錄索引到 Elasticsearch 中。 以下 request 使用 [bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) 將原始日誌數據索引到 `my-index`。 您可以使用一個小樣本來試驗 runtime fields，而不是索引所有日誌數據。

```http
POST /my-index/_bulk?refresh
{"index":{}}
{"timestamp":"2020-04-30T14:30:17-05:00","message":"40.135.0.0 - - [30/Apr/2020:14:30:17 -0500] \"GET /images/hm_bg.jpg HTTP/1.0\" 200 24736"}
{"index":{}}
{"timestamp":"2020-04-30T14:30:53-05:00","message":"232.0.0.0 - - [30/Apr/2020:14:30:53 -0500] \"GET /images/hm_bg.jpg HTTP/1.0\" 200 24736"}
{"index":{}}
{"timestamp":"2020-04-30T14:31:12-05:00","message":"26.1.0.0 - - [30/Apr/2020:14:31:12 -0500] \"GET /images/hm_bg.jpg HTTP/1.0\" 200 24736"}
{"index":{}}
{"timestamp":"2020-04-30T14:31:19-05:00","message":"247.37.0.0 - - [30/Apr/2020:14:31:19 -0500] \"GET /french/splash_inet.html HTTP/1.0\" 200 3781"}
{"index":{}}
{"timestamp":"2020-04-30T14:31:22-05:00","message":"247.37.0.0 - - [30/Apr/2020:14:31:22 -0500] \"GET /images/hm_nbg.jpg HTTP/1.0\" 304 0"}
{"index":{}}
{"timestamp":"2020-04-30T14:31:27-05:00","message":"252.0.0.0 - - [30/Apr/2020:14:31:27 -0500] \"GET /images/hm_bg.jpg HTTP/1.0\" 200 24736"}
{"index":{}}
{"timestamp":"2020-04-30T14:31:28-05:00","message":"not a valid apache log"}
```

## Extract an IP address from a log message (Grok)

如果要檢索包含 `clientip` 的結果，可以將該 field 添加為映射中的 runtime field。 以下 runtime script 定義了一個從 `message` field 中提取結構化field 的 grok 模式。

該腳本匹配 `%{COMMONAPACHELOG}` 日誌模式，該模式了解 Apache 日誌的結構。 如果模式匹配 (`clientip != null`)，腳本將發出匹配 IP 地址的值。 如果模式不匹配，腳本只會返回 field value 而不會崩潰。

```http
PUT my-index/_mappings
{
  "runtime": {
    "http.clientip": {
      "type": "ip",
      "script": """
        String clientip=grok('%{COMMONAPACHELOG}').extract(doc["message"].value)?.clientip;
        if (clientip != null) emit(clientip);  # 註1
      """
    }
  }
}
```

註1: 此條件確保即使 message 的模式不匹配，腳本也不會發出任何內容。

您可以定義一個簡單的查詢來運行對特定 IP 地址的搜索並返回所有相關 fields。 使用 search API 的 `field` 參數來檢索 `http.clientip` runtime field。

```http
GET my-index/_search
{
  "query": {
    "match": {
      "http.clientip": "40.135.0.0"
    }
  },
  "fields" : ["http.clientip"]
}
```

The response includes documents where the value for http.clientip matches 40.135.0.0.

```json
{
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my-index",
        "_id" : "Rq-ex3gBA_A0V6dYGLQ7",
        "_score" : 1.0,
        "_source" : {
          "timestamp" : "2020-04-30T14:30:17-05:00",
          "message" : "40.135.0.0 - - [30/Apr/2020:14:30:17 -0500] \"GET /images/hm_bg.jpg HTTP/1.0\" 200 24736"
        },
        "fields" : {
          "http.clientip" : [
            "40.135.0.0"
          ]
        }
      }
    ]
  }
}
```
