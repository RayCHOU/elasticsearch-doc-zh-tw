# [Field extraction](https://www.elastic.co/guide/en/elasticsearch/reference/current/scripting-field-extraction.html)

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

response 包括「`http.clientip` 的值與 `40.135.0.0` 匹配」的文檔。

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

## Parse a string to extract part of a field (Dissect)

您可以定義一個 dissect pattern 來包含要丟棄的字符串部分，而不是像前面的示例那樣在日誌模式上進行匹配。

例如，本節開頭的日誌數據包含一個 `message` field。 該 field 包含幾條數據：

```json
"message" : "247.37.0.0 - - [30/Apr/2020:14:31:22 -0500] \"GET /images/hm_nbg.jpg HTTP/1.0\" 304 0"
```

您可以在 runtime field 中定義一個 dissect pattern 來提取 [HTTP response code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)，在上一個示例中為 304。

```
PUT my-index/_mappings
{
  "runtime": {
    "http.response": {
      "type": "long",
      "script": """
        String response=dissect('%{clientip} %{ident} %{auth} [%{@timestamp}] "%{verb} %{request} HTTP/%{httpversion}" %{response} %{size}').extract(doc["message"].value)?.response;
        if (response != null) emit(Integer.parseInt(response));
      """
    }
  }
}
```

然後，您可以執行一個 query 以使用 `http.response` runtime field 檢索特定的 HTTP response：

```http
GET my-index/_search
{
  "query": {
    "match": {
      "http.response": "304"
    }
  },
  "fields" : ["http.response"]
}
```

response 包括單個文檔，其中 HTTP response 為 304：

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
        "_id" : "Sq-ex3gBA_A0V6dYGLQ7",
        "_score" : 1.0,
        "_source" : {
          "timestamp" : "2020-04-30T14:31:22-05:00",
          "message" : "247.37.0.0 - - [30/Apr/2020:14:31:22 -0500] \"GET /images/hm_nbg.jpg HTTP/1.0\" 304 0"
        },
        "fields" : {
          "http.response" : [
            304
          ]
        }
      }
    ]
  }
}
```

## Split values in a field by a separator (Dissect)

假設您想要像前面的示例一樣提取 field 的一部分，但您想要 split 特定的值。  
您可以使用 dissect pattern 僅提取所需的信息，並以特定格式返回該數據。

例如，假設您有一堆來自 Elasticsearch 的垃圾收集 (gc) 日誌數據，格式如下：

```
[2021-04-27T16:16:34.699+0000][82460][gc,heap,exit]   class space    used 266K, capacity 384K, committed 384K, reserved 1048576K
```

您只想提取 `used`、`capacity` 和 `committed` 數據，以及相關的值。  
讓我們索引一些包含日誌數據的文檔作為示例：

```http
POST /my-index/_bulk?refresh
{"index":{}}
{"gc": "[2021-04-27T16:16:34.699+0000][82460][gc,heap,exit]   class space    used 266K, capacity 384K, committed 384K, reserved 1048576K"}
{"index":{}}
{"gc": "[2021-03-24T20:27:24.184+0000][90239][gc,heap,exit]   class space    used 15255K, capacity 16726K, committed 16844K, reserved 1048576K"}
{"index":{}}
{"gc": "[2021-03-24T20:27:24.184+0000][90239][gc,heap,exit]  Metaspace       used 115409K, capacity 119541K, committed 120248K, reserved 1153024K"}
{"index":{}}
{"gc": "[2021-04-19T15:03:21.735+0000][84408][gc,heap,exit]   class space    used 14503K, capacity 15894K, committed 15948K, reserved 1048576K"}
{"index":{}}
{"gc": "[2021-04-19T15:03:21.735+0000][84408][gc,heap,exit]  Metaspace       used 107719K, capacity 111775K, committed 112724K, reserved 1146880K"}
{"index":{}}
{"gc": "[2021-04-27T16:16:34.699+0000][82460][gc,heap,exit]  class space  used 266K, capacity 367K, committed 384K, reserved 1048576K"}
```

再次檢視資料，有一個 timestamp，一些其他你不感興趣的數據，然後是 `used`，`capacity`，`committed` 數據：

```
[2021-04-27T16:16:34.699+0000][82460][gc,heap,exit]   class space    used 266K, capacity 384K, committed 384K, reserved 1048576K
```

您可以為 gc field 中數據的每個部分 assign variable，然後只返回您想要的部分。  
大括號 `{}` 中的任何內容都被視為 variable。  
例如，這些變數 `[%{@timestamp}][%{code}][%{desc}]` 將匹配前三個數據塊，它們都在方括號 `[]` 中。

```
[%{@timestamp}][%{code}][%{desc}]  %{ident} used %{usize}, capacity %{csize}, committed %{comsize}, reserved %{rsize}
```

您的 dissect pattern 可以包括 `used`、`capacity` 和 `committed` 這些 terms，而不是使用 variables，因為您想要準確地返回這些 terms。  
您還可以將 variables assign 給要返回的值，例如 `%{usize}`、`%{csize}` 和 `%{comsize}`。  
日誌數據中的分隔符是逗號，因此您的 dissect pattern 也需要使用該分隔符。

現在您有了一個 dissect pattern，您可以將它作為 runtime field 的一部分包含在 Painless 腳本中。  
該腳本使用您的 dissect pattern 來拆分 `gc` field，然後準確返回您想要的信息，如 `emit` 方法所定義的那樣。  
因為 dissect 使用簡單的語法，你只需要準確地告訴它你想要什麼。

以下模式告訴 dissect 返回 `used` 這個 term、一個空格、`gc.usize` 的值和一個逗號。  
然後對於您要檢索的其他數據重複此模式。  
雖然此模式在 prodution 中可能用處不大，但它提供了很大的靈活性來試驗和操作您的數據。  
在 prodution 設置中，您可能只想使用 `emit(gc.usize)` 然後聚合該值或在計算中使用它。

```javascript
emit("used" + ' ' + gc.usize + ', ' + "capacity" + ' ' + gc.csize + ', ' + "committed" + ' ' + gc.comsize)
```
