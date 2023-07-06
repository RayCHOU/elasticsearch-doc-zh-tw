# Standard analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html

`standard` 分析器是 預設分析器，如果未指定，則使用它。 
它提供基於語法的 tokenization（基於 Unicode Text Segmentation algorithm，如 
[Unicode Standard Annex #29](https://unicode.org/reports/tr29/) 中指定），並且適用於大多數語言。

## Example output

```http
POST _analyze
{
  "analyzer": "standard",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

上面的句子會產以下 terms:

    [ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog's, bone ]

## Configuration

`standard` analyzer 接受下列參數：

| 參數 | 說明 |
| ---- | ---- |
| `max_token_length` | 最大 token 長度。 如果發現 token 超過此長度，則按 `max_token_length` 間隔進行拆分。 預設為 `255`。 |
| `stopwords` | 預定義的 stop words 列表，如 `_english_` 或包含 stop words 列表的 array。 預設為 `_none`。 |
| `stopwords_path` | 包含 stop words 的文件的路徑。 |

有關 stop word 配置的更多信息，請參閱 [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html)。

## Example configuration

在此示例中，我們將 `standard` 分析器 配置為 `max_token_length` 為 5（用於演示目的），並使用預定義的英語 stop words 列表：

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english_analyzer": {
          "type": "standard",
          "max_token_length": 5,
          "stopwords": "_english_"
        }
      }
    }
  }
}

POST my-index-000001/_analyze
{
  "analyzer": "my_english_analyzer",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

上面的範例會產生下列 terms:

    [ 2, quick, brown, foxes, jumpe, d, over, lazy, dog's, bone ]

## Definition

`standard` analyzer 由以下構成：

* __Tokenizer__
  * [Standard Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-tokenizer.html)
* __Token Filters__
  * [Lower Case Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenfilter.html)
  * [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html) (預設停用)

如果您需要在 配置參數之外 對 `standard` 分析器 進行 客製化，
那麼您需要將其重新創建為 `custom` analyzer 並對其進行修改，通常是通過添加 token filters。 
這將重新創建 內建 `standard` 分析器，您可以將其用作起點：

```http
PUT /standard_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_standard": {
          "tokenizer": "standard",
          "filter": [
            "lowercase" ❶ 可以在 `lowercase` 之後新增 token filters
          ]
        }
      }
    }
  }
}
```
