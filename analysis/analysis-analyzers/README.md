# Built-in analyzer reference

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html

Elasticsearch 附帶了各種 built-in 分析器，無需進一步配置即可在任何索引中使用：

[__標準分析器__](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html)

`standard` 分析器 會根據 Unicode文本分段演算法 的定義，將文本在 word boundaries 的地方 劃分為多個 terms。
它刪除了大多數 標點符號、小寫 terms，並支持刪除 stop words。

[__簡單分析器__](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-simple-analyzer.html)

`simple` 分析器 只要遇到 非字母的字符，就會將文本分成 terms。 它將所有 terms 都轉小寫。

[__空白分析器__](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-whitespace-analyzer.html)

`whitespace` 析器 遇到任何 空白字符 時，都會將文本分成 terms。 它不會將 terms 轉為小寫。

[__停止分析器__](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-analyzer.html)

`stop` 分析器 與 `simple` 分析器 類似，但也支持刪除 stop words。

[__關鍵詞分析器__](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-analyzer.html)

`keyword` 分析器 是一個 “noop” 分析器，它接受給定的任何文本，並輸出完全相同的文本作為單個 term。

[__模式分析器__](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-analyzer.html)

`pattern` 分析器 使用 正則表達式 將文本拆分為 terms。 它支持 小寫 和 stop words。

[__語言分析器__](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html)

Elasticsearch 提供了許多特定語言的分析器，例如 `english` 或 `french`。

[__指紋分析器__](fingerprint.md)

`fingerprint` 分析器 是一種專門的分析器，它能創建一個可以用於 重複檢測 的指紋。

## Custom analyzers

如果您找不到適合您需求的分析器，您可以創建 custom analyzer，結合適當的 character filters、tokenizer 和 token filters。
