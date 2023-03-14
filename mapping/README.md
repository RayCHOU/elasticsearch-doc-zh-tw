# [Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping.html)

Mapping 是定義 document 及其包含的 fields 如何存儲和索引的過程。

每個 document 都是 fields 的集合，每個 field 都有自己的 [data type](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping-types.html)。  
Mapping 資料時，您創建一個 mapping 定義，其中包含與 document 相關的 fields 列表。  
Mapping 定義還包括 [metadata fields](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping-fields.html)，
例如 `_source` field，它自定義如何處理 document 的關聯 metadata。

使用 動態 mapping 和 顯式 mapping 來定義資料。  
每種方法根據您在數據旅程中所處的位置提供不同的好處。  
例如，顯式映射您不想使用預設值的 fields，或者更好地控制創建哪些 fields。  
然後，您可以允許 Elasticsearch 動態添加其他 fields。

## [Dynamic mapping](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping.html#mapping-dynamic)

[動態映射](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/dynamic-field-mapping.html)允許您在剛開始時試驗和探索數據。
Elasticsearch 自動添加新 fields，只需通過 indexing document 即可。
您可以將 fields 添加到 top-level mapping，以及內部 [`object`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/object.html) 和 [`nested`](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/nested.html) fields。

使用 [動態模板](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/dynamic-templates.html) 定義 custom mappings，根據 匹配條件 應用於 動態添加的 fields。

## [Explicit mapping](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping.html#mapping-explicit)

[顯式映射](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/explicit-mapping.html) 允許您精確地選擇如何定義 mapping definition，例如：

* 哪些 string fields 應被視為 full text fields。
* 哪些 fields 包含數字、日期或地理位置。
* 日期值的 [格式](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping-date-format.html)。
* 用於控制 [動態添加 fields](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/dynamic-mapping.html) 映射的自定義規則。

使用 [runtime fields](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/runtime-mapping-fields.html) 在不重新索引的情況下進行架構更改。  
您可以結合使用 runtime fields 和 indexed fields 來平衡資源使用和性能。  
您的索引會更小，但搜索性能會更慢。

## [防止 mapping 爆炸的 settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping.html#mapping-limit-settings)

在索引中定義太多 fields 會導致映射爆炸，從而導致 memory 不足錯誤和難以恢復的情況。

考慮這樣一種情況，每個插入的新文檔都會引入新字段，例如動態映射。  
每個新字段都會添加到索引映射中，隨著映射的增長，這可能會成為一個問題。

使用 [mapping limit settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/mapping-settings-limit.html) 
來限制字段映射（手動或動態創建）的數量，防止文檔引起映射爆炸。
