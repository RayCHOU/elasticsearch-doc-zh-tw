# [Multi term vectors API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-termvectors.html)

Retrieves multiple term vectors with a single request.

使用單個請求檢索多個 term vectors。

```http
POST /_mtermvectors
{
   "docs": [
      {
         "_index": "my-index-000001",
         "_id": "2",
         "term_statistics": true
      },
      {
         "_index": "my-index-000001",
         "_id": "1",
         "fields": [
            "message"
         ]
      }
   ]
}
```

## Request

`POST /_mtermvectors`

`POST /<index>/_mtermvectors`

## Prerequisites

If the Elasticsearch security features are enabled, you must have the `read` [index privilege](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-privileges.html#privileges-list-indices) for the target index or index alias.

如果啟用了 Elasticsearch 安全功能，您必須具有目標索引或索引別名的 `read` [索引權限](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-privileges.html#privileges-list-indices)。

## Description

You can specify existing documents by index and ID or provide artificial documents in the body of the request. 
You can specify the index in the request body or request URI.

您可以通過索引和 ID 指定現有文檔，或者在 request body 中提供人工文檔。  
您可以在 request body 或 request URI 中指定索引。

The response contains a docs array with all the fetched termvectors. 
Each element has the structure provided by the [termvectors](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html) API.

Response 包含一個「包含所有獲取的 termvectors 的 docs array」。  
每個元素都具有 [termvectors](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html) API 提供的結構。

See the [termvectors](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html) API for more information about the information that can be included in the response.

有關「可包含在 response 中的信息」的更多信息，請參閱 [termvectors](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html) API。

## Path parameters

`<index>`

(Optional, string) Name of the index that contains the documents.

（可選，字串）包含文檔的索引的名稱。

## Query parameters

| 參數 | 說明 |
| ---- | ---- |
| `fields` | (Optional, string) Comma-separated list or wildcard expressions of fields to include in the statistics.<br>（可選，字串）要包含在統計信息中的 fields 的 逗號分隔列表 或 wildcard expressions。<br>Used as the default list unless a specific field list is provided in the completion_fields or fielddata_fields parameters.<br>用作預設列表，除非在 `completion_fields` 或 `fielddata_fields` 參數中提供了特定的 field 列表。 |
| `field_statistics` | (Optional, Boolean) If `true`, the response includes the document count, sum of document frequencies, and sum of total term frequencies. Defaults to `true`.<br>（可選，布林值）如果為 `true`，則 response 包括 文檔計數、文檔頻率總和 以及 總詞頻總和。 預設為 `true`。 |
| `<offsets>` | (Optional, Boolean) If `true`, the response includes term offsets. Defaults to `true`.<br>（可選，布林值）如果為 `true`，則 response 包括 term offsets。 預設為 `true`。 |
| `payloads` | (Optional, Boolean) If `true`, the response includes term payloads. Defaults to `true`.<br>（可選，布林值）如果為 `true`，則 response 包括 term payloads。 預設為 `true`。 |
| `positions` | (Optional, Boolean) If `true`, the response includes term positions. Defaults to `true`.<br>（可選，布林值）如果為 `true`，則 response 包括 term positions。 預設為 `true`。 |
| `preference` | (Optional, string) Specifies the node or shard the operation should be performed on. Random by default.<br>（可選，字串）指定應在其上執行操作的 node 或 shard。 預設為隨機。 |
| `routing` | (Optional, string) Custom value used to route operations to a specific shard.<br>（可選，字串）用於將 操作 路由 到特定 shard 的自定義值。 |
| `realtime` | (Optional, Boolean) If `true`, the request is real-time as opposed to near-real-time. Defaults to `true`. See [Realtime](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#realtime).<br>（可選，布林值）如果為 `true`，則請求是 real-time，而不是 near-real-time。 預設為 `true`。 請參閱 [Realtime](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#realtime)。 |
| `term_statistics` | (Optional, Boolean) If `true`, the response includes term frequency and document frequency. Defaults to `false`.<br>（可選，布林值）如果為 `true`，則 response 包括 term frequcncy 和 document frequency。 預設為 `false`。 |
| `version` | (Optional, Boolean) If `true`, returns the document version as part of a hit.<br>（可選，布林值）如果為 `true`，則返回 文檔版本 作為 hit 的一部分。 |
| `version_type` | (Optional, enum) Specific version type: `external`, `external_gte`.<br>（可選，枚舉）特定版本類型：`external`、`external_gte`。 |

### Examplese

If you specify an index in the request URI, the index does not need to be specified for each documents in the request body:

如果在 request URI 中指定索引，則不需要為 request body 中的每個文檔指定索引：

```http
POST /my-index-000001/_mtermvectors
{
   "docs": [
      {
         "_id": "2",
         "fields": [
            "message"
         ],
         "term_statistics": true
      },
      {
         "_id": "1"
      }
   ]
}
```

If all requested documents are in same index and the parameters are the same, you can use the following simplified syntax:

如果所有請求的文檔都在同一索引中並且參數相同，則可以使用以下簡化語法：

```http
POST /my-index-000001/_mtermvectors
{
  "ids": [ "1", "2" ],
  "parameters": {
    "fields": [
      "message"
    ],
    "term_statistics": true
  }
}
```

### Artificial documents

You can also use `mtermvectors` to generate term vectors for *artificial* documents provided in the body of the request. The mapping used is determined by the specified `_index`.

您還可以使用 `mtermvectors` 為 request body 中提供的 *人工* 文檔 生成 term vectors。 使用的 mapping 由指定的 `_index` 決定。

```http
POST /_mtermvectors
{
   "docs": [
      {
         "_index": "my-index-000001",
         "doc" : {
            "message" : "test test test"
         }
      },
      {
         "_index": "my-index-000001",
         "doc" : {
           "message" : "Another test ..."
         }
      }
   ]
}
```
