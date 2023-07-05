# Create a custom analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html

當 built-in analyzers 不能滿足您的需求時，您可以創建一個 custom analyzer，可適當使用以下的組合：

* 零個或多個 [character filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html)
* 一個 [tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html)
* 零個或多個 [token filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html)

## Configuration

custom analyzer 接受以下參數：

| parameter   | 說明 |
| ----------- | ---- |
| `type`        | analyzer 類型。 接受 [built-in analyzer 類型](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html)。 對於 custom analyzers，請使用 `custom` 或省略此參數。 |
| `tokenizer`   | 一個 built-in 或定制的 [tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html)。 （必需的） |
| `char_filter` | 可選的陣列，包含 built-in 或自定義 [character filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html)。 |
| `filter`      | 可選的陣列，包含 built-in 或自定義 [token filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html)。 |
| `position_increment_gap` | 對一個由 text values 組成的陣列 進行索引時，Elasticsearch 會在一個值的最後一個 term 與下一個值的第一個 term 項之間插入一個假的 "gap"，以確保 phrase query 不會匹配來自不同 陣列元素 的兩個 terms。 預設值為 `100`。有關更多信息，請參閱 [`position_increment_gap`](https://www.elastic.co/guide/en/elasticsearch/reference/current/position-increment-gap.html)。 |

## Example configuration

這是一個結合了以下內容的範例：

* __Character Filter__
  * [HTML Strip Character Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-htmlstrip-charfilter.html)
* __Tokenizer__
  * [Standard Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-tokenizer.html)
* __Token Filters__
  * [Lowercase Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenfilter.html)
  * [ASCII-Folding Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-asciifolding-tokenfilter.html)

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",  ❶
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  }
}

POST my-index-000001/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Is this <b>déjà vu</b>?"
}
```

❶ 對於 `custom` analyzers，`type` 請使用 `custom` 或 省略 `type` 參數。

上面的示例產生以下 terms：

    [ is, this, deja, vu ]

前面的示例使用了 tokenizer、token filters 和 character filters 及其 預設配置，
這些都可以建立經過配置的版本 (configured version)，並在 custom analyzer 中使用它們。

這是一個更複雜的示例，結合了以下內容：

* __Character Filter__
  * [Mapping Character Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-mapping-charfilter.html),
    設定成：將 `:)` 取代為 `_happy_`，並將 `:(` 取代為 `_sad_`
* __Tokenizer__
  * [Pattern Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-tokenizer.html),
    設定為: 按 標點符號字元 分割
* __Token Filters__
  * [Lowercase Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenfilter.html)
  * [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html),
    設定為 使用預定義的英語 stop words 列表

這是一個例子：

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {  ❶
          "char_filter": [
            "emoticons"
          ],
          "tokenizer": "punctuation",
          "filter": [
            "lowercase",
            "english_stop"
          ]
        }
      },
      "tokenizer": {
        "punctuation": {  ❷
          "type": "pattern",
          "pattern": "[ .,!?]"
        }
      },
      "char_filter": {
        "emoticons": {  ❸
          "type": "mapping",
          "mappings": [
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      },
      "filter": {
        "english_stop": {  ❹
          "type": "stop",
          "stopwords": "_english_"
        }
      }
    }
  }
}

POST my-index-000001/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I'm a :) person, and you?"
}
```

❶ 為索引分配一個預設的 custom analyzer `my_custom_analyzer`。 
此分析器使用稍後在 request 中定義的 custom tokenizer、character filter 和 token filter。 
該分析器還省略了 `type` 參數。

❷ 定義 custom `punctuation` tokenizer。

❸ 定義 custom `emoticons` character filter。

❹ 定義 custom `english_stop` token filter。

上面的示例產生以下 terms：

    [ i'm, _happy_, person, you ]
