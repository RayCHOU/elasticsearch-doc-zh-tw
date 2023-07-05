# Test an analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/test-analyzer.html

[`analyze` API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html) 
是查看 analyzer 生成的 terms 的寶貴工具。 

## Test built-in analyzer

可以在 request 中 inline 指定 built-in analyzer：

Ruby:

```ruby
response = client.indices.analyze(
  body: {
    analyzer: 'whitespace',
    text: 'The quick brown fox.'
  }
)
puts response
```

API 回傳以下 response：

```json
{
  "tokens": [
    {
      "token": "The",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 0
    },
    {
      "token": "quick",
      "start_offset": 4,
      "end_offset": 9,
      "type": "word",
      "position": 1
    },
    {
      "token": "brown",
      "start_offset": 10,
      "end_offset": 15,
      "type": "word",
      "position": 2
    },
    {
      "token": "fox.",
      "start_offset": 16,
      "end_offset": 20,
      "type": "word",
      "position": 3
    }
  ]
}
```

## Test combinations

您還可以測試以下組合：

* 一個 tokenizer
* 零個或多個 token filters
* 零個或多個 character filters

```ruby
response = client.indices.analyze(
  body: {
    tokenizer: 'standard',
    filter: [
      'lowercase',
      'asciifolding'
    ],
    text: 'Is this déja vu?'
  }
)
puts response
```

API 回傳以下 response：

```json
{
  "tokens": [
    {
      "token": "is",
      "start_offset": 0,
      "end_offset": 2,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "this",
      "start_offset": 3,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "deja",
      "start_offset": 8,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "vu",
      "start_offset": 13,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 3
    }
  ]
}
```

位置 和 character offsets

從 `analyze` API 的輸出可以看出，analyzers 不僅將 words 轉換為 terms，
還記錄 每個 term 的順序或相對位置（用於 phrase queries 或 word proximity queries），
以及 每個術語位於原始文本中的起始和結束 character offsets（用於 highlighting search snippets）。 

## Test custom analyzer

或者，在特定索引上運行 analyze API 時可以引用 [`custom` analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html)：

```HTTP
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_folded": {  <!-- ❶ 定義一個名為 std_folded 的 custom analyzer。 -->
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_text": {
        "type": "text",
        "analyzer": "std_folded" # ❷ field my_text 使用 std_folded analyzer。
      }
    }
  }
}

GET my-index-000001/_analyze  # ❸ 要引用這個 analyzer，analyzer API 必須指定索引名稱。
{
  "analyzer": "std_folded", # ❹ 按名稱引用 analyzer。
  "text":     "Is this déjà vu?"
}

GET my-index-000001/_analyze # ❸ 要引用這個 analyzer，analyzer API 必須指定索引名稱。
{
  "field": "my_text",  # ❺ 引用 field my_text 使用的分析器。
  "text":  "Is this déjà vu?"
}
```

API 回傳以下 response：

```json
{
  "tokens": [
    {
      "token": "is",
      "start_offset": 0,
      "end_offset": 2,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "this",
      "start_offset": 3,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "deja",
      "start_offset": 8,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "vu",
      "start_offset": 13,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 3
    }
  ]
}
```
