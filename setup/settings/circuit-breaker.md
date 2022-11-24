# [Circuit breaker settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker.html)

Elasticsearch 包含多個 斷路器，用於防止操作導致 OutOfMemoryError。  
每個 斷路器 都指定了它可以使用的 memory 的限制。  
此外，還有一個 父級別 (parent-level) 的斷路器，用於指定 跨所有斷路器 可使用的 memory 總量。

除非另有說明，否則可以使用 [cluster-update-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API 在 live cluster 上動態更新這些設置。

有關 斷路器錯誤 的信息，請參閱 [斷路器錯誤](https://www.elastic.co/guide/en/elasticsearch/reference/current/circuit-breaker-errors.html)。

(略)
