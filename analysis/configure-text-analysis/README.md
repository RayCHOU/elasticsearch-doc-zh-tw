# Configure text analysis

https://www.elastic.co/guide/en/elasticsearch/reference/current/configure-text-analysis.html

預設情況下，Elasticsearch 使用 [`standard` analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html) 
進行所有的文本分析。 
`standard` analyzer 為您提供對大多數 自然語言 和用例的 開箱即用支持。 
如果您選擇按原樣使用 `standard` analyzer，則無需進一步配置。

如果 standard analyzer 不能滿足您的需求，請查看並測試 Elasticsearch 其他內建的 
[built-in analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html)。 
Built-in analyzers 不需要配置，但需要一些可用於調整其行為的支持選項。 
例如，您可以使用「要刪除的 自訂 stop words 清單」來配置 `standard` analyzer。

如果沒有 built-in analyzer 滿足您的需求，您可以測試並創建自定義分析器。 
Custom analyzers 涉及選擇和組合不同的 analyzer 組件，使您能夠更好地控製過程。

* [測試 analyzer](test-analyzer.md)
* [配置 built-in analyzers](configuring-analyzers.md)
* 創建 custom analyzer
* 指定 analyzer
