# Configuring built-in analyzers

https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-analyzers.html

Built-in analyzers 無需任何配置即可直接使用。 
然而，其中一些 analyzers 可以使用選項 來 改變其行為。 
例如，[`standard` analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html) 
可以配置為支持 stop words 列表：

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_english": {  ❶
          "type":      "standard",
          "stopwords": "_english_"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_text": {
        "type":     "text",
        "analyzer": "standard",  ❷
        "fields": {
          "english": {
            "type":     "text",
            "analyzer": "std_english"  ❸
          }
        }
      }
    }
  }
}

POST my-index-000001/_analyze
{
  "field": "my_text",  ❷
  "text": "The old brown cow"
}

POST my-index-000001/_analyze
{
  "field": "my_text.english",  ❸
  "text": "The old brown cow"
}
```

❶  我們將 `std_english` analyzer 定義為基於 `standard` analyzer，但配置為 刪除預定義的英語 stopwords 列表。

❷ `my_text` field 直接使用 `standard` analyzer，無需任何配置。 不會從此字段中刪除任何 stop words。 
結果 terms 是：`[ the, old, brown, cow ]`

❸ `my_text.english` field 使用 `std_english` analyzer，因此英語 stop words 將被刪除。 結果 terms 是：[ old, brown, cow ]
