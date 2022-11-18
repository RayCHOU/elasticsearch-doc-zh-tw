# [Upgrade Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html)

Elasticsearch clusters 通常可以一次升級一個 node，因此升級不會中斷服務。 
有關升級說明，請參閱 [升級到 Elastic 8.5.1](https://www.elastic.co/guide/en/elastic-stack/8.5/upgrading-elastic-stack.html)。

## 從 7.x 升級

要從 7.16 或更早版本升級到 8.5.1，您必須先升級到 7.17，  
即使您選擇進行 full-cluster 重啟而不是滾動升級也是如此。  
這使您可以使用 升級助手 來識別和解決問題，重新索引在 7.0 之前創建的索引，然後執行滾動升級。  
在繼續升級之前，您必須解決所有關鍵問題。  
有關說明，請參閱 [準備從 7.x 升級](https://www.elastic.co/guide/en/elastic-stack/8.5/upgrading-elastic-stack.html#prepare-to-upgrade)。

## Index compatibility

Elasticsearch 對在以前的主要版本中創建的索引具有完整的查詢和寫入支持。  
如果您在 6.x 或更早版本中創建了索引，則可以使用 [archive functionality](https://www.elastic.co/guide/en/elasticsearch/reference/current/archive-indices.html) 將它們導入到較新的 Elasticsearch 版本中，
或者您必須在升級到 8.5.1 之前重新索引或刪除它們。  
如果存在不兼容的索引，Elasticsearch nodes 將無法啟動。  
6.x 或更早版本索引的 snapshots 只能使用 [archive functionality](https://www.elastic.co/guide/en/elasticsearch/reference/current/archive-indices.html) 恢復到 8.x cluster，
即使它們是由 7.x cluster 創建的。  
7.17 中的升級助手 (Upgrade Assistant) 會識別任何需要重新索引或刪除的索引。

## REST API compatibility

[REST API compatibility](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-api-compatibility.html) 是一項按請求選擇加入的功能，可以幫助 REST 客戶端減輕對 REST API 的不兼容（破壞性）更改。

## FIPS Compliance and Java 17

Elasticsearch 8.5.1 需要 Java 17 或更高版本。  
當您在 FIPS 140-2 模式下運行 Elasticsearch 8.5.1 時，還沒有可用於 Java 17 的 FIPS 認證安全模塊。  
如果您在 FIPS 140-2 模式下運行，則需要向安全組織請求例外以升級到 Elasticsearch 8.5.1，或者在 Java 17 獲得認證之前繼續使用 Elasticsearch 7.x。  
或者，考慮在 [FedRAMP-certified GovCloud region](https://www.elastic.co/industries/public-sector/fedramp) 中使用 Elasticsearch Service。
