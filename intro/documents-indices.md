# [Data in: documents and indices](https://www.elastic.co/guide/en/elasticsearch/reference/current/documents-indices.html)

可以儲存複雜資料結構的 JSON document。

Elasticsearch 使用 inverted index:
inverted index 列出出現在任何 document 中的每個 unique word，並標識了每個單詞出現的所有 documents。

index 可以被想成是 documents 的優化集合，每個 document 文檔都是 fields 的集合，fields 是包含資料的 key-value pairs。

text fields 存儲在 inverted indices 中，而 numeric 數字和 地理 geo fields 存儲在 BKD trees 中。

如果啟動 dynamic mapping，Elasticsearch 會自動偵測 field 的資料類型。

您也可以明確定義 mappings 以完全控制 fields 的存儲和索引方式。

自定 mappings 使您能夠：

* 區分 full-text string fields 以及 exact value string fields
* 執行特定語言的文本分析
* 優化 fields 以進行部分匹配
* 使用自定義日期格式
* 使用無法自動檢測的 `geo_point`、`geo_shape` 等數據類型

出於不同目的以不同方式索引同一 field 通常很有用。
例如，您可能希望將 string field 索引為用於全文搜索的 text field 和用於排序或聚合(aggregating)數據的 keyword field。
或者，您可以選擇使用多個語言分析器來處理包含用戶輸入的 string field 內容。

在索引期間應用於 full-text field 的 分析鏈 (analysis chain) 也在搜索時使用。
當您查詢 full-text field 時，query text 會在索引中查找術語之前進行相同的分析。
