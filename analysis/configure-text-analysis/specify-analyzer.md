# Specify an analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html

Elasticsearch 提供了多種方法來指定 built-in 或 custom analyzers：

* 通過 `text` field、索引 或 查詢
* 用於 [index or search time](../concepts/analysis-index-search-time.md)

__TIP: Keep it simple__

在 不同級別 和 不同時間 指定 analyzer 的靈活性非常棒……但僅在需要時才這樣做。

在大多數情況下，簡單的方法效果最好：為每個 `text` field 指定一個分析器，如 [指定 field 的分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-field-analyzer) 中所述。

這種方法非常適合 Elasticsearch 的預設行為，讓您可以使用相同的 analyzer 進行索引和搜索。 
它還可以讓您使用 [get mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-mapping.html) 
快速查看哪個 analyzer 應用到哪個 field。

如果您通常不為索引創建 mappings，則可以使用 [index templates](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html) 
來實現類似的效果。

## How Elasticsearch determines the index analyzer

Elasticsearch 通過 按順序檢查以下參數 來確定要使用哪個 index analyzer：

1. field 的 [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html) mapping 參數。
   請參閱 [指定 field 的分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-field-analyzer)。
2. `analysis.analyzer.default` 索引設定值。
   請參閱 [指定索引的預設分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-time-default-analyzer)。

如果未指定這些參數，則使用 [`standard` analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html)。

## Specify the analyzer for a field

mapping 索引時，您可以使用 [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html) 
mapping 參數 為每個 `text` field 指定分析器。

以下 [create index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
請求將 `whitespace` analyzer 設置為 `title` field 的分析器。

```http
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "whitespace"
      }
    }
  }
}
```

## Specify the default analyzer for an index

除了 field-level 分析器之外，您還可以使用 `analysis.analyzer.default` 設定值 來設定 fallback 分析器。

以下 [create index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
請求將 `simple` analyzer 設置為 `my-index-000001` 的 fallback 分析器。

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "simple"
        }
      }
    }
  }
}
```

## How Elasticsearch determines the search analyzer

__警告__

在大多數情況下，沒有必要指定不同的 search analyzer。 
這樣做可能會對相關性產生負面影響並導致意外的搜索結果。

如果您選擇 指定單獨的 search analyzer，我們建議您在 部署到 production 之前 徹底
[測試您的 analysis 設定](https://www.elastic.co/guide/en/elasticsearch/reference/current/test-analyzer.html)。

在搜索時，Elasticsearch 通過 按順序檢查以下參數 來確定 使用哪個分析器：

1. search query 中的 [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html) 參數。
   請參閱 [指定 查詢的搜索分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-search-query-analyzer)。
2. field 的 [`search_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-analyzer.html) mapping 參數。
   請參閱 [指定 field 的 搜索分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-search-field-analyzer)。
3. `analysis.analyzer.default_search` 索引設置。
   請參閱 [指定 索引 的 預設搜索分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-search-default-analyzer)。
4. field 的 [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html) mapping 參數。
   請參閱 [指定 field 的分析器](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-index-field-analyzer)。

如果未指定這些參數，則使用 [`standard` analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html)。

## Specify the search analyzer for a query

編寫 [full-text query](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html)
時，可以使用 `analyzer` 參數指定 搜索分析器。 如果有提供，將覆蓋任何其他 搜索分析器。

以下 [search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html) 
請求將 `stop` analyzer 設置為 match query 的 搜索分析器。

```http
GET my-index-000001/_search
{
  "query": {
    "match": {
      "message": {
        "query": "Quick foxes",
        "analyzer": "stop"
      }
    }
  }
}
```

## Specify the search analyzer for a field

mapping 索引時，您可以使用 [`search_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html) 
mapping 參數 為每個 text field 指定 搜索分析器。

如果提供了 搜索分析器，則還必須使用 `analyzer` 參數 指定 索引分析器。

以下 [create index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
請求將 `simple` 分析器 設置為 `title` field 的 搜索分析器。

```http
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "whitespace",
        "search_analyzer": "simple"
      }
    }
  }
}
```

## Specify the default search analyzer for an index

The following create index API request sets the whitespace analyzer as the default search analyzer for the my-index-000001 index.

[創建索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)時，
您可以使用 `analysis.analyzer.default_search` 設置預設的 搜索分析器。

如果提供了 搜索分析器，則還必須使用 `analysis.analyzer.default` 指定預設的 索引分析器。

以下 [create index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
請求將 `whitespace` 分析器 設置為 `my-index-000001` 索引的 預設 搜索分析器。

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "simple"
        },
        "default_search": {
          "type": "whitespace"
        }
      }
    }
  }
}
```
