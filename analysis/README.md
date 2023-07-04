# [Text analysis](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/analysis.html)

文本分析 是將 非結構化文本（如電子郵件正文或產品說明）轉換為針對搜索優化的 結構化格式 的過程。

## [何時配置文本分析](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/analysis.html#when-to-configure-analysis)

Elasticsearch 在 索引 或 搜索 [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/text.html) fields 時 執行 文本分析。

如果您的索引不包含 text fields，則無需進一步設置； 您可以跳過本節中的頁面。

但是，如果您使用 text fields 或您的 text searches 沒有按預期返回結果，配置 文本分析 通常會有所幫助。  
如果您使用 Elasticsearch 來執行以下操作，您還應該查看 analysis configuration：

* 建立一個 搜索引擎
* 挖掘 非結構化數據
* 微調特定語言的搜索
* 進行 詞典編纂 或 語言學研究

## 本小節中包含

* [Overview](overview.md)
* [Concepts](concepts/README.md)
* [Configure text analysis](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/configure-text-analysis.html)
* [Built-in analyzer reference](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/analysis-analyzers.html)
* [Tokenizer reference](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/analysis-tokenizers.html)
* [Token filter reference](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/analysis-tokenfilters.html)
* [Character filters reference](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/analysis-charfilters.html)
* [Normalizers](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/analysis-normalizers.html)
