# Fingerprint analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-fingerprint-analyzer.html

`fingerprint` 分析器 實做了 OpenRefine 專案 用來協助 clustering 的 
[指紋演算法](https://github.com/OpenRefine/OpenRefine/wiki/Clustering-In-Depth#fingerprint)。

輸入的文本 將被轉換為小寫，並經過標準化以去除 擴展字符 (extended characters)，然後被排序、去重複 並連接成一個單一的 token。
如果配置了 stopword 列表，則 stop words 也將被移除。

## Example output

```http
POST _analyze
{
  "analyzer": "fingerprint",
  "text": "Yes yes, Gödel said this sentence is consistent and."
}
```

上面的句子將產生以下單個 term：

    [ and consistent godel is said sentence this yes ]

## Configuration

`fingerprint` 分析器 接受以下參數：

| 參數 | 說明 |
| ---- | ---- |
| `separator` | 用於連接 term 的字元。 預設為空格。 |
| `max_output_size` | 要發出的最大 token 大小。 預設為 `255`。大於此大小的 token 將被丟棄。 |
| `stopwords` | 預定義的 stop words 列表，如 `_english_` 或包含 stop words 列表的 array。 預設為 `_none_`。 |
| `stopwords_path` | 包含 stop words 的文件的路徑。 |

有關 stop word 配置的更多信息，請參閱 [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html)。

## Example configuration

在此示例中，我們將 `fingerprint` 分析器 配置為使用預定義的英語 stop words 列表：

```http
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_fingerprint_analyzer": {
          "type": "fingerprint",
          "stopwords": "_english_"
        }
      }
    }
  }
}

POST my-index-000001/_analyze
{
  "analyzer": "my_fingerprint_analyzer",
  "text": "Yes yes, Gödel said this sentence is consistent and."
}
```

上面的示例產生以下 term：

    [ consistent godel said sentence yes ]

## Definition

`fingerprint` tokenizer 包含:

* __Tokenizer__
  * [Standard Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-tokenizer.html)
* __Token Filters (按順序)__
  * [Lower Case Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenfilter.html)
  * [ASCII folding](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-asciifolding-tokenfilter.html)
  * [Stop Token Filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html) (預設為停用)
  * [Fingerprint](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-fingerprint-tokenfilter.html)

如果您需要在 配置參數 之外 對 `fingerprint` 分析器 進行 客製化，
那麼您需要將其重新創建為 custom analyzer 並進行修改，
通常是通過 添加 token filters 來實現。
這將 重新創建 built-in `fingerprint` 分析器，您可以將其作為進一步 客製化 的起點：

```http
PUT /fingerprint_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_fingerprint": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding",
            "fingerprint"
          ]
        }
      }
    }
  }
}
```
