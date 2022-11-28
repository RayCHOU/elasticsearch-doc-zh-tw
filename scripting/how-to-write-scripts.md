# [How to write scriptsedit](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html)

無論 Elasticsearch API 在哪裡支持腳本，語法都遵循相同的模式； 您指定腳本的語言，提供腳本邏輯（或 source），並添加傳遞給腳本的參數：

```
"script": {
  "lang": "...",
  "source" | "id": "...",
  "params": { ... }
}
```

|     |     |
| --- | --- |
| **`lang`** | 指定編寫腳本的語言。預設為 `painless`。 |
| **`source`**, **`id`** | 腳本本身，您將其指定為 inline script 的 source 或 stored script 的 id。 使用 [stored script APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/script-apis.html#stored-script-apis) 創建和管理存儲腳本。 |
| **`params`** | 指定作為變數傳遞到腳本中的任何命名參數。 使用參數而不是 hard-coded values 來減少編譯時間。 |

## 撰寫你的第一個 script

Painless 是 Elasticsearch 的預設腳本語言。 它是安全的、高性能的，並且為任何有一點編碼經驗的人提供了一種自然的語法。

Painless 腳本由一個或多個 statements 構成，並且在開頭可選地具有一個或多個用戶定義的函數。 腳本必須包含至少一個 statement。

[Painless execute API](https://www.elastic.co/guide/en/elasticsearch/painless/8.5/painless-execute-api.html) 提供了使用簡單的用戶定義參數測試腳本並接收結果的能力。 讓我們從一個完整的腳本開始，並回顧它的組成部分。

首先，使用單個 field 索引文檔，以便我們可以處理一些數據：

```http
PUT my-index-000001/_doc/1
{
  "my_field": 5
}
```

然後，我們可以構建一個對該 field 進行操作的腳本，並將該腳本作為 query 的一部分運行。  
以下查詢使用 search API 的 [`script_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#script-fields) 參數來取得 script valuation。  
這裡發生了很多事情，但我們會將其分解為組件以單獨理解它們。  
現在，您只需要了解此腳本接受 `my_field` 並對其進行操作。

```http
GET my-index-000001/_search
{
  "script_fields": {
    "my_doubled_field": {
      "script": {  # 註1
        "source": "doc['my_field'].value * params['multiplier']",  # 註2
        "params": {
          "multiplier": 2
        }
      }
    }
  }
}
```

* 註1: `script` object
* 註2: `script` source

`script` 是一個標準的 JSON object，它定義了 Elasticsearch 中大多數 API 下的腳本。  
這個 object 需要 `source` 來定義腳本本身。  
該腳本未指定語言，因此預設為 Painless。

## 在你的 script 裡使用參數

Elasticsearch 第一次看到新腳本時，它會編譯腳本並將編譯後的版本存儲在 cache 裡。  
編譯可能是一個繁重的過程。  
與其將數值 hard-coding 在 script 裡，不如將它們作為 命名參數 傳遞。

例如，在前面的腳本中，我們可以將數值 hard coded 在 script 裡，這樣的 script 看起來比較不那麼複雜。  
我們可以只取得 `my_field` 的第一個值，然後將其乘以 2：

    "source": "return doc['my_field'].value * 2"

雖然它有效，但這個解決方案非常不靈活。  
我們必須修改腳本源來改變乘數，而每次乘數發生變化時，Elasticsearch 都必須重新編譯腳本。

不將數值 hard-coding 在 script 裡，改用 命名參數 使腳本靈活，並減少腳本運行時的編譯時間。  
您現在可以更改乘數參數，而無需讓 Elasticsearch 重新編譯腳本。

```
"source": "doc['my_field'].value * params['multiplier']",
"params": {
  "multiplier": 2
}
```

預設情況下，您每 5 分鐘最多可以編譯 150 個腳本。  
對於 ingest contexts，預設腳本編譯速率是無限的。

    script.context.field.max_compilations_rate=100/10m


**重要**：如果您在短時間內編譯了太多 unique scripts，Elasticsearch 會拒絕新的 dynamic scripts 並返回 `circuit_break_exception` 錯誤。

## 縮短你的 script

使用 Painless 自帶的句法功能，您可以減少腳本中的冗長並縮短它們。 這是一個我們可以縮短的簡單腳本：

```http
GET my-index-000001/_search
{
  "script_fields": {
    "my_doubled_field": {
      "script": {
        "lang":   "painless",
        "source": "doc['my_field'].value * params.get('multiplier');",
        "params": {
          "multiplier": 2
        }
      }
    }
  }
}
```

讓我們看一下腳本的縮短版本，看看它有哪些改進：

```http
GET my-index-000001/_search
{
  "script_fields": {
    "my_doubled_field": {
      "script": {
        "source": "field('my_field').get(null) * params['multiplier']",
        "params": {
          "multiplier": 2
        }
      }
    }
  }
}
```

此版本的腳本刪除了幾個組件並顯著簡化了語法：

* `lang` 宣告。 因為 Painless 是預設語言，所以如果您正在編寫 Painless 腳本，則無需指定語言。
* `return` 關鍵字。 Painless 自動使用腳本中的最後一個 statement（如果可能）在需要的腳本 context 中產生 return value。
* `get` 方法，用方括號 `[]` 代替。 Painless 為 `Map` 類型使用了一個快捷方式，它允許我們使用括號而不是更長的 `get` 方法。
* `source` statement 結尾的分號。 Painless 區塊的最後一個 statement 不需要分號。 但是在其他情況下，確實需要他們來消除歧義。

在 Elasticsearch 支持腳本的任何地方使用這種縮寫語法，例如在您創建 [runtime fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/runtime-mapping-fields.html) 時。

## 儲存、取得 scripts

您可以使用 [stored script APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/script-apis.html#stored-script-apis) 從 cluster state 存儲和取得腳本。  
Stored scripts 可減少編譯時間並使搜索速度更快。

註：與常規腳本不同，stored scripts 要求您使用 `lang` 參數指定腳本語言。

要創建腳本，請使用 [create stored script API](https://www.elastic.co/guide/en/elasticsearch/reference/current/create-stored-script-api.html)。  
例如，以下 request 創建一個名為 `calculate-score` 的存儲腳本。

```http
POST _scripts/calculate-score
{
  "script": {
    "lang": "painless",
    "source": "Math.log(_score * 2) + params['my_modifier']"
  }
}
```

您可以使用 [get stored script API](https://www.elastic.co/guide/en/elasticsearch/reference/current/get-stored-script-api.html) 來取得該腳本。

```http
GET _scripts/calculate-score
```

要在 query 中使用 stored script，請在腳本宣告中包含腳本 ID：

```http
GET my-index-000001/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
            "message": "some message"
        }
      },
      "script": {
        "id": "calculate-score", # 註1
        "params": {
          "my_modifier": 2
        }
      }
    }
  }
}
```

註1: stored script 的 `id`

要刪除存儲的腳本，請提交 [delete stored script API](https://www.elastic.co/guide/en/elasticsearch/reference/current/delete-stored-script-api.html) 請求。

```http
DELETE _scripts/calculate-score
```

## 使用 scripts 更新 documents

您可以使用 [update API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html) 來更新具有指定腳本的文檔。  
該腳本可以更新、刪除或跳過修改文檔。  
Update API 還支持傳遞部分文檔，該部分文檔會合併到現有文檔中。

首先，讓我們為一個簡單的文檔建立索引：

```http
PUT my-index-000001/_doc/1
{
  "counter" : 1,
  "tags" : ["red"]
}
```

要增加計數器，您可以使用以下腳本提交更新請求：

```http
POST my-index-000001/_update/1
{
  "script" : {
    "source": "ctx._source.counter += params.count",
    "lang": "painless",
    "params" : {
      "count" : 4
    }
  }
}
```

同樣，您可以使用更新腳本將標籤添加到標籤列表中。  
因為這只是一個列表，所以即使存在標籤也會添加：

```http
POST my-index-000001/_update/1
{
  "script": {
    "source": "ctx._source.tags.add(params['tag'])",
    "lang": "painless",
    "params": {
      "tag": "blue"
    }
  }
}
```

您還可以從標籤列表中刪除標籤。  
Java `List` 的 `remove` 方法在 Painless 中可用。  
它需要您要刪除的元素的 index。  
為避免可能的 runtime error，您首先需要確保標籤存在。  
如果列表包含重複的標記，則此腳本只會刪除一次。

```http
POST my-index-000001/_update/1
{
  "script": {
    "source": "if (ctx._source.tags.contains(params['tag'])) { ctx._source.tags.remove(ctx._source.tags.indexOf(params['tag'])) }",
    "lang": "painless",
    "params": {
      "tag": "blue"
    }
  }
}
```

您還可以在文檔中添加和刪除 fields。  
例如，此腳本添加 `new_field` 這個 field：

```http
POST my-index-000001/_update/1
{
  "script" : "ctx._source.new_field = 'value_of_new_field'"
}
```

相反，此腳本刪除 new_field 這個 field：

```http
POST my-index-000001/_update/1
{
  "script" : "ctx._source.remove('new_field')"
}
```

除了更新文檔之外，您還可以更改從腳本中執行的操作。  
例如，如果 `tags` field 包含 `green`，則此請求將刪除文檔。 否則它什麼也不做（`noop`）：

```http
POST my-index-000001/_update/1
{
  "script": {
    "source": "if (ctx._source.tags.contains(params['tag'])) { ctx.op = 'delete' } else { ctx.op = 'none' }",
    "lang": "painless",
    "params": {
      "tag": "green"
    }
  }
}
```
