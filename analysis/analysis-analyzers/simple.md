# Simple analyzer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-simple-analyzer.html

`simple` 分析器 會在任何 非字母字符處 將文本分解成 tokens，例如 數字、空格、連字符和撇號，並丟棄 非字母字符，且將大寫字母轉換為小寫字母。

## Example

```http
POST _analyze
{
  "analyzer": "simple",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

`simple` 分析器 解析該句子並產生成以下 tokens：

    [ the, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]

## Definition

`simple` 分析器由一個 tokenizer 定義：

* __Tokenizer__
  * [Lowercase Tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenizer.html)

## Customize

要 客製化 `simple` 分析器，請複制它以創建 custom 分析器的基礎。 
可以根據需要修改此 custom 分析器，通常通過添加 token filters 來修改。

```http
PUT /my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_simple_analyzer": {
          "tokenizer": "lowercase",
          "filter": [  ❶ 在這裡新增 token filters
          ]
        }
      }
    }
  }
}
```
