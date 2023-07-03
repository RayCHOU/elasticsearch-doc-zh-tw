# Anatomy of an analyzer (analyzer 的剖析)

https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer-anatomy.html

Analyzer（無論是 built-in 還是 custom）只是一個 package，其中包含三個較低級別的 blocks：character filters、tokenizer 和 token filter。

Built-in analyzers 將這些 building blocks 預先打包成適合不同 語言 和 文本類型 的分析器。 
Elasticsearch 還公開了各個 building blocks，以便可以將它們組合起來定義新的 custom analyzers。

## Character filters

https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer-anatomy.html#analyzer-anatomy-character-filters

character filter 接收 原始文本 作為一個 stream of characters，並可以通過添加、刪除或更改 characters 來轉換這個 stream。 
例如，character filters 可用於將 印度-阿拉伯數字 (٠‎١٢٣٤٥٦٧٨‎٩‎) 轉換為 阿拉伯-拉丁數字 (0123456789)，或者從 stream 中刪除 <b> 等 HTML 元素。

analyzer 可能有零個或多個按順序應用的 [character filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html)。

## Tokenizer

https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer-anatomy.html#analyzer-anatomy-tokenizer

Tokenizer 接收 stream of characters，將其分解為單獨的 tokens（通常是單獨的單詞），然後輸出 stream of tokens。 
例如，每當 [whitespace](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-whitespace-tokenizer.html) tokenizer 看到任何空白時，它就會將文本分解為 tokens。 
它將文本 `"Quick Brown Fox!"` 轉換這些 terms `[Quick, brown, fox!]`。

Tokenizer 還負責記錄每個 term 的順序或位置 以及 該術語所代表的原始單詞的 起始和結束 character 的 offset。

Analyzer 必須恰好有一個 [tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html)。

## Token filters

https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer-anatomy.html#analyzer-anatomy-token-filters

Token filter 接收 token stream 並可以添加、刪除或更改 tokens。 
例如，[lowercase](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenfilter.html) 
token filter 將所有 tokens 轉換為小寫，
[stop](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html) 
token filter 從 token stream 中刪除常見單詞（stop words），例如 `the` ，
[synonym](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html) 
token filter 將同義詞引入 token stream。

Token filters 不允許更改每個 token 的位置或 character offsets。

Analyzer 可能有零個或多個按順序應用的 token filters。
