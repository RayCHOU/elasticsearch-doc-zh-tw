# Keyword analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-analyzer.html

`keyword` 分析器 是一個 “noop” 分析器，它將整個 輸入字串 作為單個 token 回傳。

## Example output

```http
POST _analyze
{
  "analyzer": "keyword",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

上面的句子將產生以下單個 term：

    [ The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. ]

## Configuration

`keyword` 分析器 是無法 configure 的。

## Definition

`keyword` 分析器包括：

* __Tokenizer__
  * [Keyword Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-tokenizer.html)

如果您需要 客製化 `keyword` 分析器，那麼您需要將其重新創建為 custom 分析器 並對其進行修改，
通常是通過添加 token filters。 

通常，當您希望 string 不要被分割成 tokens 時，您應該優先選擇 
[Keyword type](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)，
但是如果您需要的話，以下方式可以重新創建 built-in `keyword` 分析器，您可以將其作為進一步 客製化 的起點：

```http
PUT /keyword_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_keyword": {
          "tokenizer": "keyword",
          "filter": [  ❶ 您可以在此處添加任何 token filters。
          ]
        }
      }
    }
  }
}
```
