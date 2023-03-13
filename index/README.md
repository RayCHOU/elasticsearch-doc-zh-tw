# [Index modules](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/index-modules.html)

Index Modules are modules created per index and control all aspects related to an index.

索引模塊是為每個索引創建的模塊，並控制與索引相關的所有方面。

## Index Settings

Index level settings can be set per-index. Settings may be:

可以為每個索引設置索引級別設置。 設置可能是：

* **static**: They can only be set at index creation time or on a [closed index](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-open-close.html).
* **dynamic**: They can be changed on a live index using the [update-index-settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-update-settings.html) API.

* **靜態**：它們只能在 索引創建時 或 在[已關閉的索引](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-open-close.html)上 設置。
* **動態**：可以使用 [update-index-settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-update-settings.html) API 在 實時索引(live index) 上更改它們。

**WARNING**: Changing static or dynamic index settings on a closed index could result in incorrect settings that are impossible to rectify without deleting and recreating the index.

**警告**：更改 關閉索引上的 靜態或動態索引設置 可能會導致不正確的設置，如果不 刪除並重新創建索引 就無法糾正這些設置。

### Static index settings

Below is a list of all *static* index settings that are not associated with any specific index module:

以下是與任何 特定 索引模塊 無關的所有 *靜態*索引設置 的列表：

#### `index.number_of_shards`

The number of primary shards that an index should have. Defaults to `1`. This setting can only be set at index creation time. It cannot be changed on a closed index.

索引應具有的 primary shards 數量。 預設為 `1`。  
此設置只能在創建索引時設置。 它不能在已關閉的索引上更改。


NOTE: The number of shards are limited to 1024 per index. 
This limitation is a safety limit to prevent accidental creation of indices that can destabilize a cluster due to resource allocation. 
The limit can be modified by specifying `export ES_JAVA_OPTS="-Des.index.max_number_of_shards=128"` system property on every node that is part of the cluster.

注意：每個索引的分片數量限制為 1024 個。  
此限制是一個安全限制，以防止意外創建索引，這些索引可能因資源分配而破壞集群的穩定性。  
可以通過在集群中的每個節點上指定 `export ES_JAVA_OPTS="-Des.index.max_number_of_shards=128"` 系統屬性來修改限制。

#### `index.number_of_routing_shards`

Integer value used with `index.number_of_shards` to route documents to a primary shard. See [`_routing` field](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/mapping-routing-field.html).

與 `index.number_of_shards` 一起使用的整數值，用於將文檔路由到 primary shard。 請參見 [`_routing` field](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/mapping-routing-field.html)

Elasticsearch uses this value when [splitting](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-split-index.html) an index. 
For example, a 5 shard index with `number_of_routing_shards` set to `30` (`5 x 2 x 3`) could be split by a factor of `2` or `3`. 
In other words, it could be split as follows:

Elasticsearch 在 [splitting](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-split-index.html) 索引時使用此值。  
例如，`number_of_routing_shards` 設置為 `30`（`5 x 2 x 3`）的 5 分片索引可以按 `2` 或 `3` 的因子拆分。
換句話說，它可以拆分如下：

* `5` → `10` → `30` (split by 2, then by 3)
* `5` → `15` → `30` (split by 3, then by 2)
* `5` → `30` (split by 6)

This setting’s default value depends on the number of primary shards in the index. 
The default is designed to allow you to split by factors of 2 up to a maximum of 1024 shards.

此設置的預設值取決於索引中 primary shards 的數量。  
預設允許您按 2 的因子拆分，最多 1024 個分片。

**NOTE**: In Elasticsearch 7.0.0 and later versions, this setting affects how documents are distributed across shards. 
When reindexing an older index with custom routing, you must explicitly set `index.number_of_routing_shards` to maintain the same document distribution. 
See the [related breaking change](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/breaking-changes-7.0.html#_document_distribution_changes).

**注意**：在 Elasticsearch 7.0.0 及更高版本中，此設置會影響文檔在分片之間的分佈方式。  
當使用 自定義路由 重新索引舊索引時，您必須顯式設置 `index.number_of_routing_shards` 以保持相同的文檔分佈。 請參閱 [相關的重大更改](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/breaking-changes-7.0.html#_document_distribution_changes)。

#### `index.codec`

The `default` value compresses stored data with LZ4 compression, but this can be set to `best_compression` which uses [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) for a higher compression ratio, at the expense of slower stored fields performance. 
If you are updating the compression type, the new one will be applied after segments are merged. 
Segment merging can be forced using [force merge](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-forcemerge.html).

預設為 `default` 使用 LZ4 壓縮來壓縮存儲的數據，但這可以設置為 `best_compression`，它使用 [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) 以獲得更高的壓縮率，但代價是存儲字段性能較慢。
如果您正在更新壓縮類型，新的壓縮類型將在 segments 合併後應用。
可以使用 [強制合併](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-forcemerge.html) 強制 segment 合併。

#### `index.routing_partition_size`

The number of shards a custom routing value can go to. 
Defaults to 1 and can only be set at index creation time. 
This value must be less than the `index.number_of_shards` unless the `index.number_of_shards` value is also 1. 
See Routing to an index partition for more details about how this setting is used.

自定義 [routing](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/mapping-routing-field.html) 值可以轉到的分片數。  
預設為 1，只能在創建索引時設置。  
此值必須小於 `index.number_of_shards` 除非 `index.number_of_shards` 值也為 1。  
有關如何使用此設置的更多詳細信息，請參閱 [Routing to an index partition](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/mapping-routing-field.html#routing-index-partition)。

#### `index.soft_deletes.enabled`

Deprecated in 7.6.0. Creating indices with soft-deletes disabled is deprecated and will be removed in future Elasticsearch versions.

在 7.6.0 中棄用。 「創建禁用 soft-deletes 的索引」已被棄用，並將在未來的 Elasticsearch 版本中刪除。

Indicates whether soft deletes are enabled on the index. 
Soft deletes can only be configured at index creation and only on indices created on or after Elasticsearch 6.5.0. Defaults to `true`.

指示是否對索引啟用 soft deletes。  
Soft deletes 只能在 創建索引時 配置，並且只能在 Elasticsearch 6.5.0 或之後創建的索引上配置。  
預設為 `true`。

#### `index.soft_deletes.retention_lease.period`

The maximum period to retain a shard history retention lease before it is considered expired. 
Shard history retention leases ensure that soft deletes are retained during merges on the Lucene index. 
If a soft delete is merged away before it can be replicated to a follower the following process will fail due to incomplete history on the leader. 
Defaults to `12h`.

在被視為過期之前保留 分片歷史保留租約 的最長時間。  
分片歷史保留租約 確保在合併 Lucene 索引期間保留 soft deletes。  
如果 soft deletel 在可以復製到跟隨者之前被合併掉，則由於領導者的歷史記錄不完整，以下過程將失敗。  
預設為 `12h`。

#### `index.load_fixed_bitset_filters_eagerly`

Indicates whether cached filters are pre-loaded for nested queries. Possible values are `true` (default) and `false`.

指示是否為 nested queryies 預加載 [cached filters](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/query-filter-context.html)。  
可能的值為 `true`（預設值）和 `false`。

#### `index.shard.check_on_startup`

**WARNING**: Expert users only. 
This setting enables some very expensive processing at shard startup and is only ever useful while diagnosing a problem in your cluster. 
If you do use it, you should do so only temporarily and remove it once it is no longer needed.

**警告**：僅限專家用戶。  
此設置在分片啟動時啟用一些非常昂貴的處理，並且僅在診斷集群中的問題時才有用。  
如果您確實要使用它，您應該只是暫時使用它，並在不再需要時將其移除。

Elasticsearch automatically performs integrity checks on the contents of shards at various points during their lifecycle. 
For instance, it verifies the checksum of every file transferred when recovering a replica or taking a snapshot. 
It also verifies the integrity of many important files when opening a shard, which happens when starting up a node and when finishing a shard recovery or relocation. 
You can therefore manually verify the integrity of a whole shard while it is running by taking a snapshot of it into a fresh repository or by recovering it onto a fresh node.

Elasticsearch 在 分片生命週期 的不同時間點自動對 分片內容 執行 完整性檢查。  
例如，它會在 恢復副本 或 拍攝快照 時 驗證傳輸的每個文件的 checksum。  
它還會在打開分片時驗證許多重要文件的完整性，這發生在啟動節點和完成 分片恢復 或 重定位 時。  
因此，您可以在運行時手動驗證整個分片的完整性，方法是將它的快照拍攝到一個新的存儲庫中，或者將它恢復到一個新的節點上。

This setting determines whether Elasticsearch performs additional integrity checks while opening a shard. 
If these checks detect corruption then they will prevent the shard from being opened. 
It accepts the following values:

此設置確定 Elasticsearch 在打開分片時是否執行額外的完整性檢查。  
如果這些檢查檢測到損壞，那麼它們將阻止打開分片。  
它接受以下值：

* `false`: Don’t perform additional checks for corruption when opening a shard. This is the default and recommended behaviour.
* `checksum`: Verify that the checksum of every file in the shard matches its contents. This will detect cases where the data read from disk differ from the data that Elasticsearch originally wrote, for instance due to undetected disk corruption or other hardware failures. These checks require reading the entire shard from disk which takes substantial time and IO bandwidth and may affect cluster performance by evicting important data from your filesystem cache.
* `true`: Performs the same checks as checksum and also checks for logical inconsistencies in the shard, which could for instance be caused by the data being corrupted while it was being written due to faulty RAM or other hardware failures. These checks require reading the entire shard from disk which takes substantial time and IO bandwidth, and then performing various checks on the contents of the shard which take substantial time, CPU and memory.

* `false`：打開分片時不執行額外的損壞檢查。 這是預設和推薦的行為。
* `checksum`：驗證分片中每個文件的 checksum 是否與其內容相匹配。 這將檢測從磁盤讀取的數據與 Elasticsearch 最初寫入的數據不同的情況，例如由於未檢測到的磁盤損壞或其他硬件故障。 這些檢查需要從磁盤讀取整個分片，這需要大量時間和 IO bandwidth，並且可能會通過從 filesystem cache 中逐出重要數據來 影響集群性能。
* `true`：執行與 checksum 相同的檢查，還檢查分片中的邏輯不一致，例如，這可能是由於 RAM 故障或其他硬件故障導致數據在寫入時被損壞造成的。 這些檢查需要從磁盤讀取整個分片，這需要大量的時間和 IO bandwidth，然後對分片的內容執行各種檢查，這需要大量的時間、CPU 和 memory。

