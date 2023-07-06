# Stop analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-analyzer.html

`stop` 分析器 與 [`simple` 分析器](simple.md) 相同，但增加了對刪除 stop words 的支持。 
它預設使用 `_english_` stop words。

## Example output

```http
POST _analyze
{
  "analyzer": "stop",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

上面的句子會產生下列 terms:

    [ quick, brown, foxes, jumped, over, lazy, dog, s, bone ]

## Configuration

`stop` analyzer 接受下列參數:

| 參數 | 說明 |
| ---- | ---- |
| `stopwords` | 預定義的 stop words 列表，如 `_english_` 或包含 stop words 列表的 array。 預設為 `_english_`。 |
| `stopwords_path` | 包含 stop words 的文件的路徑。 該路徑相對於 Elasticsearch 配置目錄。 |

有關 stop word 配置的更多信息，請參閱 [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html)。

## Example configuration

在此示例中，我們將 `stop` 分析器 配置為使用指定的單詞列表作為 stop words：

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_stop_analyzer": {
          "type": "stop",
          "stopwords": ["the", "over"]
        }
      }
    }
  }
}

POST my-index-000001/_analyze
{
  "analyzer": "my_stop_analyzer",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

上面範例產生下列 terms:

    [ quick, brown, foxes, jumped, lazy, dog, s, bone ]

## Definition

它由下列構成：

* __Tokenizer__
  * [Lower Case Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenizer.html)
* __Token filters__
  * [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html)

如果您需要在 配置參數 之外對 `stop`分析器 進行 客製化，
那麼您需要將其重新創建為 `custom` 分析器並對其進行修改，通常是通過添加 token filters。 
這將重新創建 內建 `stop` 分析器，您可以將其用作進一步 客製化 的起點：

```http
PUT /stop_example
{
  "settings": {
    "analysis": {
      "filter": {
        "english_stop": {
          "type":       "stop",
          "stopwords":  "_english_" ❶
        }
      },
      "analyzer": {
        "rebuilt_stop": {
          "tokenizer": "lowercase",
          "filter": [
            "english_stop" ❷
          ]
        }
      }
    }
  }
}
```

❶ 可以使用 `stopwords` 或 `stopwords_path` 參數 覆蓋 預設的 stopwords。

❷ 您可以在 `english_stop` 之後添加任何 token filters。
