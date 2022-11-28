# [DNS cache settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/networkaddress-cache-ttl.html)

Elasticsearch 與 security manager 一起運行。 

使用安全管理器後，JVM 默認無限期 緩存 (caching) 正主機名解析 (positive hostname resulutions)，默認緩存 負主機名解析 (negative hostname resolutions) 十秒。 

Elasticsearch 使用默認值覆蓋此行為，將正查找緩存 60 秒，將負查找緩存 10 秒。 

這些值應該適用於大多數環境，包括 DNS 解析隨時間變化的環境。 

如果沒有，您可以在 [JVM 選項](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-options) 中編輯 `es.networkaddress.cache.ttl` 和 `es.networkaddress.cache.negative.ttl`。 

請注意，[Java 安全策略](https://docs.oracle.com/javase/8/docs/technotes/guides/security/PolicyFiles.html) 中的 [`networkaddress.cache.ttl=<timeout>`](https://docs.oracle.com/javase/8/docs/technotes/guides/net/properties.html) 和 [`networkaddress.cache.negative.ttl=<timeout>`](https://docs.oracle.com/javase/8/docs/technotes/guides/net/properties.html) 會被 Elasticsearch 忽略，
除非您刪除 `es.networkaddress.cache.ttl` 和 `es.networkaddress.cache.negative.ttl`。
