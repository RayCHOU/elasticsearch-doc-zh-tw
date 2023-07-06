# Language analyzers

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html

一組旨在分析特定語言文本的分析器。 支持以下類型：
`arabic`, `armenian`, `basque`, `bengali`, `brazilian`, `bulgarian`, `catalan`, [`cjk`](cjk-analyzer.md), `czech`, `danish`, `dutch`, 
`english`, `estonian`, `finnish`, `french`, `galician`, `german`, `greek`, `hindi`, `hungarian`, `indonesian`, `irish`, 
`italian`, `latvian`, `lithuanian`, `norwegian`, `persian`, `portuguese`, `romanian`, `russian`, `sorani`, `spanish`, `swedish`, `turkish`, `thai`.

## Configuring language analyzers

### Stopwords

所有分析器都支持在配置中內部設置自定義 `stopwords`，或者通過設置 `stopwords_path` 使用外部 stopwords 文件。 
檢視 [Stop Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-analyzer.html) 了解更多詳細信息。

### Excluding words from stemming

`stem_exclusion` 參數允許您指定 不應被 詞幹化 的 小寫單詞 array。 

在內部，此功能是通過添加 [`keywords_marker` token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-marker-tokenfilter.html)
來實現的，其中 `keywords` 設置為 `stem_exclusion` 參數的值。

以下分析器支持設置自定義 `stem_exclusion` 列表：
`arabic`, `armenian`, `basque`, `bengali`, `bulgarian`, `catalan`, `czech`, `dutch`, `english`, `finnish`, `french`, 
`galician`, `german`, `hindi`, `hungarian`, `indonesian`, `irish`, `italian`, `latvian`, `lithuanian`, `norwegian`, 
`portuguese`, `romanian`, `russian`, `sorani`, `spanish`, `swedish`, `turkish`.

## Reimplementing language analyzers

內建語言分析器 可以重新實做為 `custom` 分析器（如下所述），以便 客製化 其行為。

__NOTE__

* 如果您不打算從 詞幹化 中排除單詞（相當於上面的 `stem_exclusion` 參數），
  那麼您應該從 custom 分析器 配置中刪除 `keyword_marker` token filter。

### cjk analyzer

__NOTE__

* 您可能會發現 ICU analysis plugin 中的 `icu_analyzer` 比 `cjk` analyzer 更適合 CJK 文本。 
  以您的文本和查詢進行實驗。

`cjk` 分析器可以重新實做為 `custom` 分析器，如下所示：

```http
PUT /cjk_example
{
  "settings": {
    "analysis": {
      "filter": {
        "english_stop": {
          "type":       "stop",
          "stopwords":  [  ❶
            "a", "and", "are", "as", "at", "be", "but", "by", "for",
            "if", "in", "into", "is", "it", "no", "not", "of", "on",
            "or", "s", "such", "t", "that", "the", "their", "then",
            "there", "these", "they", "this", "to", "was", "will",
            "with", "www"
          ]
        }
      },
      "analyzer": {
        "rebuilt_cjk": {
          "tokenizer":  "standard",
          "filter": [
            "cjk_width",
            "lowercase",
            "cjk_bigram",
            "english_stop"
          ]
        }
      }
    }
  }
}
```

❶ 可以使用 `stopwords` 或 `stopwords_path` 參數覆蓋 預設 stopwords。 
預設 stop words 與 `_english_` set 幾乎相同，但不完全相同。
