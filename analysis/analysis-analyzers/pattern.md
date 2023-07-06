# Pattern analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-analyzer.html

`pattern` 分析器 使用 正則表達式 將文本拆分為 terms。 正則表達式 應該匹配 token 分隔符 而不是 tokens 本身。 
正則表達式 預設為 \W+（或所有 non-word 字符）。

__WARNING: 謹防 病態 的 正則表達式__

* pattern 分析器使用 [Java 正則表達式](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)。
* 寫得不好的 正則表達式 可能會運行得很慢，甚至拋出 StackOverflowError 並導致其運行的 node 突然退出。
* 閱讀有關 [病態正則表達式 以及 如何避免它們](https://www.regular-expressions.info/catastrophic.html)的更多信息。

## Example output

```http
POST _analyze
{
  "analyzer": "pattern",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

上面的句子將產生以下 terms：

    [ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]

## Configuration

`pattern` 分析器 接受以下參數：

| 參數 | 說明 |
| ---- | ---- |
| `pattern` | Java 正則表達式，預設為 \W+。 |
| `flags`   | Java 正則表達式 [flags](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html#field.summary)。 Flags 應該用 豎線(\|) 分隔，例如 `"CASE_INSENSITIVE\|COMMENTS"`。 |
| `lowercase` | terms 是否應該小寫。 預設為 true。 |
| `stopwords` | 預定義的 stop words 列表，如 `_english_` 或包含停用詞列表的 array。 預設為 `_none_`。 |
| `stopwords_path` | 包含 stop words 的文件的路徑。 |

有關 stop word 配置的更多信息，請參閱 [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html)。

## Example configuration

在此示例中，我們將 pattern 析器配置為 按 non-word 字符 或 下劃線 (`\W|_`) 拆分 電子郵件地址，並將結果小寫：

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_email_analyzer": {
          "type":      "pattern",
          "pattern":   "\\W|_",  ❶
          "lowercase": true
        }
      }
    }
  }
}

POST my-index-000001/_analyze
{
  "analyzer": "my_email_analyzer",
  "text": "John_Smith@foo-bar.com"
}
```

❶ 將 pattern 指定為 JSON string 時，需要對 pattern 中的 反斜杠 進行 escaped。

上面的示例產生以下 terms：

    [ john, smith, foo, bar, com ]

### CamelCase tokenizer

以下更複雜的示例將 CamelCase 文本拆分為 tokens：

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "camel": {
          "type": "pattern",
          "pattern": "([^\\p{L}\\d]+)|(?<=\\D)(?=\\d)|(?<=\\d)(?=\\D)|(?<=[\\p{L}&&[^\\p{Lu}]])(?=\\p{Lu})|(?<=\\p{Lu})(?=\\p{Lu}[\\p{L}&&[^\\p{Lu}]])"
        }
      }
    }
  }
}

GET my-index-000001/_analyze
{
  "analyzer": "camel",
  "text": "MooseX::FTPClass2_beta"
}
```

上面的示例產生以下 terms：

    [ moose, x, ftp, class, 2, beta ]

上面的 正則表達式 更容易理解為：

```
  ([^\p{L}\d]+)                 # 吞下 非字母 和 數字,
| (?<=\D)(?=\d)                 # 或 非數字 跟著 數字,
| (?<=\d)(?=\D)                 # 或 數字 跟著 非數字,
| (?<=[ \p{L} && [^\p{Lu}]])    # 或  小寫
  (?=\p{Lu})                    #   跟著 大寫
| (?<=\p{Lu})                   # 或 大寫
  (?=\p{Lu}                     #   跟著 大寫
    [\p{L}&&[^\p{Lu}]]          #   然後 小寫
  )
```

## Definition

`pattern` analyzer 由以下構成：

* __Tokenizer__
  * [Pattern Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-tokenizer.html)
* __Token Filters__
  * [Lower Case Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenfilter.html)
  * [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html) (預設停用)

如果您需要在 配置參數 之外的對 `pattern` 分析器 進行 客製化，
那麼您需要將其重新創建為 `custom` analyzer 並對其進行修改，
通常是通過添加 token filters。 
這將重新創建 built-in `pattern` 分析器，您可以將其用作進一步 客製化 的起點：

```http
PUT /pattern_example
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "split_on_non_word": {
          "type":       "pattern",
          "pattern":    "\\W+"  ❶
        }
      },
      "analyzer": {
        "rebuilt_pattern": {
          "tokenizer": "split_on_non_word",
          "filter": [
            "lowercase"  ❷
          ]
        }
      }
    }
  }
}
```

❶ 預設 pattern 是 `\W+`，它會分割 non-word 字符，您可以在此處更改它。
❷ 您可以在 `lowercase` 后添加其他 token filters。
