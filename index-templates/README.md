# [Index templates](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/index-templates.html)

索引模板 是一種在創建索引時告訴 Elasticsearch 如何配置索引的方法。  
對於數據流，索引模板在創建時配置 stream 的 backing indices。  
模板在**創建索引之前**配置。  
創建索引時 - 手動或通過索引文檔 - 模板設置 用作於 創建索引的基礎。

有兩種類型的模板：index templates 和 [component templates](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/indices-component-template.html)。 
Component templates 是可重複使用的構建塊，用於配置 mappings、settings 和 aliases。  
雖然您可以使用 component templates 來構建 index templates，但它們不會直接應用於一組索引。  
Index templates 可以包含 component templates 的集合，也可以直接指定 settings、mappings 和 aliases。
