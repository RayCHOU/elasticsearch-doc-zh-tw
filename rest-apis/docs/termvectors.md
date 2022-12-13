# [Term vectors API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html)

Retrieves information and statistics for terms in the fields of a particular document.

檢索特定文檔 fields 中 terms 的 資訊 和 統計信息。

```http
GET /my-index-000001/_termvectors/1
```

## Request

`GET /<index>/_termvectors/<_id>`

## Prerequisites

If the Elasticsearch security features are enabled, you must have the `read` [index privilege](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-privileges.html#privileges-list-indices) for the target index or index alias.

如果啟用了 Elasticsearch 安全功能，您必須具有 目標索引 或 索引別名 的 讀取索引 權限。

## Description

You can retrieve term vectors for documents stored in the index or for artificial documents passed in the body of the request.

您可以檢索「存儲在索引中的文檔」或「request body 中傳遞的人工文檔」的 term vectors。

You can specify the fields you are interested in through the `fields` parameter, or by adding the fields to the request body.

您可以通過 `fields` 參數指定您感興趣的 fields，也可以將 fields 添加到 request body 中。

```http
GET /my-index-000001/_termvectors/1?fields=message
```

Fields can be specified using wildcards, similar to the [multi match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html).

可以使用 wildcards 指定 fields，類似於 [multi match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)。

Term vectors are [real-time](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#realtime) by default, not near real-time. This can be changed by setting `realtime` parameter to false.

預設情況下，term vectors 是 [real-time](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#realtime) 的，而不是 near real-time 的。  
這可以通過將 `realtime` 參數設置為 `false` 來更改。

You can request three types of values: *term information*, *term statistics* and *field statistics*. By default, all term information and field statistics are returned for all fields but term statistics are excluded.

您可以請求三種類型的值：*term 資訊*、*term 統計* 和 *field 統計*。  
預設情況下，返回所有 fields 的所有「term 資訊」和「field 統計信息」，但不包括「term 統計信息」。

## Term information

* term frequency in the field (always returned)
* term positions (`positions` : true)
* start and end offsets (`offsets` : true)
* term payloads (`payloads` : true), as base64 encoded bytes

* field 中的 term frequency（總是返回）
* term positions (`positions` : true)
* 開始和結束 offsets (`offsets` : true)
* term payloads (`payloads` : true)，作為 base64 編碼 bytes

If the requested information wasn’t stored in the index, it will be computed on the fly if possible. 
Additionally, term vectors could be computed for documents not even existing in the index, but instead provided by the user.

如果請求的信息未存儲在索引中，則將盡可能即時計算。  
此外，可以為甚至不存在於索引中但由用戶提供的文檔計算 term vectors。


**WARNING**: Start and end offsets assume UTF-16 encoding is being used. 
If you want to use these offsets in order to get the original text that produced this token, you should make sure that the string you are taking a sub-string of is also encoded using UTF-16.

**警告**：開始和結束 offsets 假定正在使用 UTF-16 編碼。  
如果您想使用這些偏移量來獲取生成此 token 的原始文本，則應確保您要獲取其子字串的字串也使用 UTF-16 編碼。

## Term statistics

Setting term_statistics to true (default is false) will return

* total term frequency (how often a term occurs in all documents)
* document frequency (the number of documents containing the current term)

將 `term_statistics` 設置為 `true`（預設為 `false`）將返回

* 總詞頻（一個詞在所有文檔中出現的頻率）
* document frequency（包含當前術語的文檔數量）

By default these values are not returned since term statistics can have a serious performance impact.

預設情況下不會返回這些值，因為 term statistics 可能會對性能產生嚴重影響。

## Field statistics

Setting `field_statistics` to `false` (default is `true`) will omit :

* document count (how many documents contain this field)
* sum of document frequencies (the sum of document frequencies for all terms in this field)
* sum of total term frequencies (the sum of total term frequencies of each term in this field)

將 `field_statistics` 設置為 `false`（預設為 `true`）將忽略：

* document count（有多少文檔包含該 field）
* sum of document frequency（該 field 所有 term 的 document frequencies 之和）
* sum of total term frequency（該 field 中每個 term 的 總詞頻 之和）

## Terms filtering

With the parameter `filter`, the terms returned could also be filtered based on their tf-idf scores. 
This could be useful in order find out a good characteristic vector of a document. 
This feature works in a similar manner to the [second phase](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html#mlt-query-term-selection) of the [More Like This Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html). 
See [example 5](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html#docs-termvectors-terms-filtering) for usage.

使用 `filter` 參數，返回的 terms 也可以根據它們的 tf-idf 分數進行過濾。  
這對於找出文檔的良好特徵向量可能很有用。  
此功能的工作方式與 [More Like This Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html) 的 [第二階段](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html#mlt-query-term-selection)類似。  
用法見 [示例 5](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html#docs-termvectors-terms-filtering)。

The following sub-parameters are supported:

支援以下 子參數：

| 參數 | 說明 |
| ---- | ---- |
| `max_num_terms` | Maximum number of terms that must be returned per field. Defaults to 25.<br>每個 field 必須返回的最大 terms 數量。 預設為 25。 |
| `min_term_freq` | Ignore words with less than this frequency in the source doc. Defaults to 1.<br>忽略 source doc 中頻率低於此頻率的詞。 預設為 1。 |
| `max_term_freq` | Ignore words with more than this frequency in the source doc. Defaults to unbounded.<br>忽略 source doc 中頻率高於此頻率的詞。 預設為無限制。 |
| `min_doc_freq` | Ignore terms which do not occur in at least this many docs. Defaults to 1.<br>忽略至少在這麼多文檔中沒有出現的術語。 預設為 1。 |
| `max_doc_freq` | Ignore words which occur in more than this many docs. Defaults to unbounded.<br>忽略出現在超過這麼多文檔中的單詞。 預設為無限制。 |
| `min_word_length` | The minimum word length below which words will be ignored. Defaults to 0.<br>最小 word 長度，低於該長度的字將被忽略。 預設為 0。 |
| `max_word_length` | The maximum word length above which words will be ignored. Defaults to unbounded (0).<br>最大單詞長度，超過該單詞將被忽略。 預設為無限制 (0)。 |

## Behaviour

The term and field statistics are not accurate. 
Deleted documents are not taken into account. 
The information is only retrieved for the shard the requested document resides in. 
The term and field statistics are therefore only useful as relative measures whereas the absolute numbers have no meaning in this context. 
By default, when requesting term vectors of artificial documents, a shard to get the statistics from is randomly selected. 
Use routing only to hit a particular shard.

term 和 field 統計數據不准確。 不考慮已刪除的文檔。 僅為所請求文檔所在的 shard 檢索信息。  
因此，term 和 field 統計信息僅用作相對度量，而絕對數字在此上下文中沒有意義。  
預設情況下，在請求 人工文檔 的 詞向量 時，會隨機選擇一個 分片(shard) 來獲取統計信息。 僅使用路由來命中特定的 分片。

## Path parameters

<dl>
  <dt>&lt;index><dt>
  <dd>
    (Required, string) Name of the index that contains the document.<br>
    （必需，字串）包含文檔的索引的名稱。
  </dd>
  <dt>&lt;_id></dt>
  <dd>
    (Optional, string) Unique identifier of the document.<br>
    （可選，字串）文檔的唯一標識符。
  </dd>
</dl>

## Query parameters

| 參數 | 說明 |
| ---- | ---- |
| `fields` | (Optional, string) Comma-separated list or wildcard expressions of fields to include in the statistics.<br>（可選，字串）要包含在統計信息中的 fields 的 逗號分隔列表 或 wildcards 表達式。<br>Used as the default list unless a specific field list is provided in the `completion_fields` or `fielddata_fields` parameters.<br>用作預設列表，除非在 `completion_fields` 或 `fielddata_fields` 參數中提供了特定的 field 列表。 |
| `field_statistics` | (Optional, Boolean) If `true`, the response includes the document count, sum of document frequencies, and sum of total term frequencies. Defaults to `true`.<br>（可選，布林值）如果為 `true`，則 response 包括 文檔計數、文檔頻率總和 以及 總詞頻總和。 預設為 `true`。 |
| `<offsets>` | (Optional, Boolean) If `true`, the response includes term offsets. Defaults to `true`.<br>（可選，布林值）如果為 `true`，則 response 包括 術語偏移量。 預設為 `true`。 |
| `payloads` | (Optional, Boolean) If `true`, the response includes term payloads. Defaults to `true`.<br>（可選，布林值）如果為 `true`，則 response 包括 term payloads。 預設 為 `true`。 |
| `positions` | (Optional, Boolean) If `true`, the response includes term positions. Defaults to `true`.<br>（可選，布林值）如果為 `true`，則 response 包括 術語位置。 預設為 `true`。 |
| `preference` | (Optional, string) Specifies the node or shard the operation should be performed on. Random by default.<br>（可選，字串）指定應在其上執行操作的 node 或 shard。  預設為隨機。 |
| `routing` | (Optional, string) Custom value used to route operations to a specific shard.<br>（可選，字串）用於將 操作 路由 到特定 shard 的自定義值。 |
| `realtime` | (Optional, Boolean) If `true`, the request is real-time as opposed to near-real-time. Defaults to `true`. See [Realtime](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#realtime).<br>（可選，布林值）如果為 `true`，則 request 是 real-time 的，而不是 near-real-time 的。 預設為 `true`。 請參閱 [Realtime](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#realtime)。 |
| `term_statistics` | (Optional, Boolean) If `true`, the response includes term frequency and document frequency. Defaults to `false`.<br>（可選，布林值）如果為 `true`，則 response 包括 term frequency 和 document frequency。 預設為 `false`。 |
| `version` | (Optional, Boolean) If `true`, returns the document version as part of a hit.<br>（可選，布林值）如果為 `true`，則返回文檔版本作為 hit 的一部分。 |
| `version_type` | (Optional, enum) Specific version type: `external`, `external_gte`.<br>（可選，枚舉）具體版本類型：`external`，`external_gte`。 |

## Examples

### Returning stored term vectors

First, we create an index that stores term vectors, payloads etc. :

首先，我們創建一個索引來存儲 term vectors、payloads 等：

```http
PUT /my-index-000001
{ "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "term_vector": "with_positions_offsets_payloads",
        "store" : true,
        "analyzer" : "fulltext_analyzer"
       },
       "fullname": {
        "type": "text",
        "term_vector": "with_positions_offsets_payloads",
        "analyzer" : "fulltext_analyzer"
      }
    }
  },
  "settings" : {
    "index" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 0
    },
    "analysis": {
      "analyzer": {
        "fulltext_analyzer": {
          "type": "custom",
          "tokenizer": "whitespace",
          "filter": [
            "lowercase",
            "type_as_payload"
          ]
        }
      }
    }
  }
}
```

Second, we add some documents:

其次，我們添加一些文件：

```http
PUT /my-index-000001/_doc/1
{
  "fullname" : "John Doe",
  "text" : "test test test "
}

PUT /my-index-000001/_doc/2?refresh=wait_for
{
  "fullname" : "Jane Doe",
  "text" : "Another test ..."
}
```

The following request returns all information and statistics for field text in document 1 (John Doe):

以下 request 返回文檔 `1` (John Doe) 中 `text` field 的所有信息和統計信息：

```http
GET /my-index-000001/_termvectors/1
{
  "fields" : ["text"],
  "offsets" : true,
  "payloads" : true,
  "positions" : true,
  "term_statistics" : true,
  "field_statistics" : true
}
```

Response:

```json
{
  "_index": "my-index-000001",
  "_id": "1",
  "_version": 1,
  "found": true,
  "took": 6,
  "term_vectors": {
    "text": {
      "field_statistics": {
        "sum_doc_freq": 4,
        "doc_count": 2,
        "sum_ttf": 6
      },
      "terms": {
        "test": {
          "doc_freq": 2,
          "ttf": 4,
          "term_freq": 3,
          "tokens": [
            {
              "position": 0,
              "start_offset": 0,
              "end_offset": 4,
              "payload": "d29yZA=="
            },
            {
              "position": 1,
              "start_offset": 5,
              "end_offset": 9,
              "payload": "d29yZA=="
            },
            {
              "position": 2,
              "start_offset": 10,
              "end_offset": 14,
              "payload": "d29yZA=="
            }
          ]
        }
      }
    }
  }
}
```

### Generating term vectors on the fly

Term vectors which are not explicitly stored in the index are automatically computed on the fly. 
The following request returns all information and statistics for the fields in document `1`, even though the terms haven’t been explicitly stored in the index. 
Note that for the field `text`, the terms are not re-generated.

未明確存儲在索引中的 term vectors 會在運行時自動計算。  
以下請求返回文檔 `1` 中 fields 的所有信息和統計信息，即使這些 terms 尚未明確存儲在索引中。  
請注意，對於 `text` field，不會重新生成 terms。

```http
GET /my-index-000001/_termvectors/1
{
  "fields" : ["text", "some_field_without_term_vectors"],
  "offsets" : true,
  "positions" : true,
  "term_statistics" : true,
  "field_statistics" : true
}
```

### Artificial documents

Term vectors can also be generated for artificial documents, that is for documents not present in the index. 
For example, the following request would return the same results as in example 1. 
The mapping used is determined by the `index`.

也可以為人工文檔生成 term vectors，即索引中不存在的文檔。  
例如，以下請求將返回與 示例 1 中相同的結果。所使用的映射由 `index` 確定。

**If dynamic mapping is turned on (default), the document fields not in the original mapping will be dynamically created.**

**如果啟用 dynamic mapping（預設），將動態創建不在 original mapping 中的 document fields。**

```http
GET /my-index-000001/_termvectors
{
  "doc" : {
    "fullname" : "John Doe",
    "text" : "test test test"
  }
}
```

#### Per-field analyzer

Additionally, a different analyzer than the one at the field may be provided by using the `per_field_analyzer` parameter. 
This is useful in order to generate term vectors in any fashion, especially when using artificial documents. 
When providing an analyzer for a field that already stores term vectors, the term vectors will be re-generated.

此外，可以使用 `per_field_analyzer` 參數提供與 filed 不同的 analyzer。  
這對於以任何方式生成 term vectors 很有用，尤其是在使用人工文檔時。  
當為已經存儲 詞向量 的 field 提供 analyzer 時，將重新生成 詞向量。

```http
GET /my-index-000001/_termvectors
{
  "doc" : {
    "fullname" : "John Doe",
    "text" : "test test test"
  },
  "fields": ["fullname"],
  "per_field_analyzer" : {
    "fullname": "keyword"
  }
}
```

Response:

```json
{
  "_index": "my-index-000001",
  "_version": 0,
  "found": true,
  "took": 6,
  "term_vectors": {
    "fullname": {
       "field_statistics": {
          "sum_doc_freq": 2,
          "doc_count": 4,
          "sum_ttf": 4
       },
       "terms": {
          "John Doe": {
             "term_freq": 1,
             "tokens": [
                {
                   "position": 0,
                   "start_offset": 0,
                   "end_offset": 8
                }
             ]
          }
       }
    }
  }
}
```

### Terms filtering

Finally, the terms returned could be filtered based on their tf-idf scores. 
In the example below we obtain the three most "interesting" keywords from the artificial document having the given "plot" field value. 
Notice that the keyword "Tony" or any stop words are not part of the response, as their tf-idf must be too low.

最後，返回的 term 可以根據它們的 tf-idf 分數進行過濾。  
在下面的示例中，我們從具有給定 "plot" field 值的人工文檔中獲取三個最 "有趣" 的關鍵字。  
請注意，關鍵字 "Tony" 或任何 stop words 都不是 response 的一部分，因為它們的 tf-idf 一定太低了。

```http
GET /imdb/_termvectors
{
  "doc": {
    "plot": "When wealthy industrialist Tony Stark is forced to build an armored suit after a life-threatening incident, he ultimately decides to use its technology to fight against evil."
  },
  "term_statistics": true,
  "field_statistics": true,
  "positions": false,
  "offsets": false,
  "filter": {
    "max_num_terms": 3,
    "min_term_freq": 1,
    "min_doc_freq": 1
  }
}
```

Response:

```json
{
   "_index": "imdb",
   "_version": 0,
   "found": true,
   "term_vectors": {
      "plot": {
         "field_statistics": {
            "sum_doc_freq": 3384269,
            "doc_count": 176214,
            "sum_ttf": 3753460
         },
         "terms": {
            "armored": {
               "doc_freq": 27,
               "ttf": 27,
               "term_freq": 1,
               "score": 9.74725
            },
            "industrialist": {
               "doc_freq": 88,
               "ttf": 88,
               "term_freq": 1,
               "score": 8.590818
            },
            "stark": {
               "doc_freq": 44,
               "ttf": 47,
               "term_freq": 1,
               "score": 9.272792
            }
         }
      }
   }
}
```
