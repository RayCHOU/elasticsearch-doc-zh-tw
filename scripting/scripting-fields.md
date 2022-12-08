# [Accessing document fields and special variables](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-fields.html)

根據使用腳本的位置，它可以訪問某些 special variables 和 document fields。

## Update scripts

[update](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)、[update-by-query](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html) 或 [reindex](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) API 中使用的腳本將有權訪問公開的 `ctx` variable：

| variable | 說明 |
| -------- | ---- |
| `ctx._source` | 訪問 document [`_source` field](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html)。 |
| `ctx.op` | 應該應用於 document 的操作：索引或刪除。|
| `ctx._index` etc | 訪問 document metadata field，其中一些可能是 read-only。|

## Search and aggregation scripts

除了每次 search hit 中執行一次的 script fields 外，search 和 aggregations 中使用的腳本將為每個可能匹配查詢或聚合的文檔執行一次。  
根據您擁有的 documents 數量，這可能意味著數百萬或數十億次執行：這些腳本需要足夠快！

可以使用 [doc-values](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-fields.html#modules-scripting-doc-vals)、`_source` field 或 [stored fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-fields.html#modules-scripting-stored) 從腳本訪問 field values，下面將對每一個進行解釋。

## Accessing the score of a document within a script

[`function_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html)、[script-based sorting](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html) 或 [aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) 中使用的腳本可以訪問代表文檔當前相關性分數的 `_score` variable。

以下是在 [`function_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html) 中使用腳本來更改每個 document 的相關性 `_score` 的示例：

```http
PUT my-index-000001/_doc/1?refresh
{
  "text": "quick brown fox",
  "popularity": 1
}

PUT my-index-000001/_doc/2?refresh
{
  "text": "quick fox",
  "popularity": 5
}

GET my-index-000001/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "text": "quick brown fox"
        }
      },
      "script_score": {
        "script": {
          "lang": "expression",
          "source": "_score * doc['popularity']"
        }
      }
    }
  }
}
```

## Doc values

到目前為止，從腳本訪問 field value 最快最有效的方法是使用 `doc['field_name']` 語法，它從 [doc values](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html) 中檢索 field value。  
Doc values 是一個 columnar field value store，默認情況下在除 [analyzed `text` fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) 之外的所有 fields 上啟用。

```http
PUT my-index-000001/_doc/1?refresh
{
  "cost_price": 100
}

GET my-index-000001/_search
{
  "script_fields": {
    "sales_price": {
      "script": {
        "lang":   "expression",
        "source": "doc['cost_price'] * markup",
        "params": {
          "markup": 0.2
        }
      }
    }
  }
}
```

Doc-values 只能返回「簡單」的 field values，如數字、日期、地理位置、術語等，或者如果 field 是多值的，則返回這些值的 arrays。  
它不能返回 JSON objects。

### NOTE: Missing fields

如果 mappings 中缺少 field，`doc['field']` 將拋出錯誤。  
`Painless` 可以使用 `doc.containsKey('field')` 進行檢查以保護對 doc map 的訪問。  
遺憾的是，無法檢查 expression script 中 mappings 是否存在該 field。


### NOTE: Doc values and text fields

如果啟用了 [`fielddata`](https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html)，`doc['field']` 語法也可用於 [analyzed `text` fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)，但**請注意**：在 `text` field 上啟用 `fielddata` 需要將所有 terms 加載到 JVM heap 中，這在 memory 和 CPU 方面都很耗資源。  
從 scripts 訪問 text field 幾乎沒有意義。

## The document `_source`

可以使用 `_source.field_name` 語法訪問 document [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html)。  
`_source` 作為 map-of-maps 加載，因此可以訪問 object fields 中的屬性，例如，`_source.name.first`。

IMPORTANT: Prefer doc-values to _source

* 訪問 `_source` field 比使用 doc-values 慢得多。 `_source` field 針對每個結果返回多個 fields 進行了優化，而 doc values 針對訪問許多文檔中特定 field 的值進行了優化。
* 在為搜索結果的前十名 hits 生成 [script field](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#script-fields) 時使用 `_source` 是有意義的，但對於其他搜索和 aggregation 用例，總是更喜歡使用 doc values。

例如：

```http
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "first_name": {
        "type": "text"
      },
      "last_name": {
        "type": "text"
      }
    }
  }
}

PUT my-index-000001/_doc/1?refresh
{
  "first_name": "Barry",
  "last_name": "White"
}

GET my-index-000001/_search
{
  "script_fields": {
    "full_name": {
      "script": {
        "lang": "painless",
        "source": "params._source.first_name + ' ' + params._source.last_name"
      }
    }
  }
}
```

## Stored fields

Stored fields — 在 mapping 中明確標記為 [`"store": true`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html) 的 fields — 可以使用`_fields['field_name'].value` 或 `_fields['field_name']` 語法訪問：

```http
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "full_name": {
        "type": "text",
        "store": true
      },
      "title": {
        "type": "text",
        "store": true
      }
    }
  }
}

PUT my-index-000001/_doc/1?refresh
{
  "full_name": "Alice Ball",
  "title": "Professor"
}

GET my-index-000001/_search
{
  "script_fields": {
    "name_with_title": {
      "script": {
        "lang": "painless",
        "source": "params._fields['title'].value + ' ' + params._fields['full_name'].value"
      }
    }
  }
}
```


TIP: Stored vs `_source`

* `_source` field 只是一個特殊的 stored field，所以性能和其他 stored fields 差不多。 `_source` 提供對被索引的原始文檔主體的訪問（包括區分 `null` values 和 empty field、single-value array 和 plain scalars 等的能力）。
* 使用 stored field 而不是 `_source` field 真正有意義的唯一時間是當 `_source` 非常大並且訪問一些小的 stored fields 而不是整個 `_source` 的成本更低時。
