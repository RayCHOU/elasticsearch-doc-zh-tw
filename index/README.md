# [Index modules](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/index-modules.html)

Index Modules 是為每個索引創建的模塊，並控制與索引相關的所有方面。

## Index Settings

可以為每個索引設置 Index level settings。 可能的 settings：

* **靜態**：它們只能在 索引創建時 或 在[已關閉的索引](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-open-close.html)上 設置。
* **動態**：可以使用 [update-index-settings](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-update-settings.html) API 在 實時索引(live index) 上更改它們。

**警告**：在一個已關的索引上變更靜態或動態索引設置，可能會導致「必須刪除並重新創建索引才能糾正」的不正確 settings。

### Static index settings

以下是與任何特定的 index module 無關的所有 *靜態*索引設置 的列表：

#### `index.number_of_shards`

索引應具有的 primary shards 數量。 預設為 `1`。  
此設置只能在創建索引時設置。 它不能在已關閉的索引上更改。

注意：每個索引的分片數量限制為 1024 個。  
此限制是一個安全限制，以防止意外創建「可能因資源分配而破壞集群穩定性」的索引。  
可以通過在集群中的每個節點上指定 `export ES_JAVA_OPTS="-Des.index.max_number_of_shards=128"` 系統屬性來修改這個限制。

#### `index.number_of_routing_shards`

與 `index.number_of_shards` 一起使用的整數值，用於將文檔 route 到 primary shard。 請參見 [`_routing` field](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/mapping-routing-field.html)

Elasticsearch 在 [splitting](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-split-index.html) 索引時使用此值。  
例如，`number_of_routing_shards` 設置為 `30`（`5 x 2 x 3`）的 5 分片索引可以按 `2` 或 `3` 的因子拆分。
換句話說，它可以拆分如下：

* `5` → `10` → `30` (split by 2, then by 3)
* `5` → `15` → `30` (split by 3, then by 2)
* `5` → `30` (split by 6)

此設置的預設值取決於索引中 primary shards 的數量。  
預設允許您按 2 的因子拆分，最多 1024 個分片。

**注意**：在 Elasticsearch 7.0.0 及更新版本中，此設置會影響文檔在分片之間的分佈方式。  
當使用 自定義路由 重新索引舊索引時，您必須顯式設置 `index.number_of_routing_shards` 以保持相同的文檔分佈。 請參閱 [相關的重大更改](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/breaking-changes-7.0.html#_document_distribution_changes)。

#### `index.codec`

預設為 `default` 使用 LZ4 來壓縮存儲的資料，可以設置為 `best_compression`，它使用 [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE) 以獲得更高的壓縮率，但代價是較慢的 stored fields 性能。
如果您正在更新壓縮類型，新的壓縮類型將在 segments 合併後應用。
可以使用 [強制合併](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/indices-forcemerge.html) 強制 segment 合併。

#### `index.routing_partition_size`

The number of shards a custom routing value can go to. 

自定義 [routing](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/mapping-routing-field.html) 值可以轉到的分片數。  
預設為 1，只能在創建索引時設置。  
此值必須小於 `index.number_of_shards` 除非 `index.number_of_shards` 值也為 1。  
有關如何使用此設置的更多詳細信息，請參閱 [Routing to an index partition](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/mapping-routing-field.html#routing-index-partition)。

#### `index.soft_deletes.enabled`

在 7.6.0 中棄用。 「創建 soft-deletes disabled 的索引」已被棄用，並將在未來的 Elasticsearch 版本中刪除。

指示是否對索引啟用 soft deletes。  
Soft deletes 只能在 創建索引時 配置，並且只能在 Elasticsearch 6.5.0 或之後創建的索引上配置。  
預設為 `true`。

#### `index.soft_deletes.retention_lease.period`

在被視為過期之前保留 shard history retention lease 的最長時間。  
shard history retention leases 確保在合併 Lucene 索引時保留 soft deletes。  
如果 soft delete 在可以複製到跟隨者之前被合併掉 (merged away)，則由於領導者的歷史記錄不完整，後續 process 將失敗。  
預設為 `12h`。

#### `index.load_fixed_bitset_filters_eagerly`

Indicates whether cached filters are pre-loaded for nested queries.

指示是否為 nested queryies 預加載 [cached filters](https://www.elastic.co/guide/en/elasticsearch/reference/8.5/query-filter-context.html)。  
可能的值為 `true`（預設值）和 `false`。

#### `index.shard.check_on_startup`

**警告**：僅限專家用戶。  
此設置在分片啟動時啟用一些非常昂貴的處理，並且僅在診斷集群中的問題時才有用。  
如果您確實要使用它，您應該只是暫時使用它，並在不再需要時將其移除。

Elasticsearch 在 分片生命週期 的不同時間點自動對 分片內容 執行 完整性檢查。  
例如，它會在 恢復副本 或 拍攝快照 時 驗證傳輸的每個文件的 checksum。  
它還會在打開分片時驗證許多重要文件的完整性，這發生在啟動節點和完成 shard recovery 或 relocation 時。  
因此，您可以在運行時手動驗證整個分片的完整性，方法是將它的快照拍攝到一個新的存儲庫中，或者將它 recovering 到一個新的節點上。

此設置決定 Elasticsearch 在打開分片時是否執行額外的完整性檢查。  
如果這些檢查檢測到損壞，那麼它們將阻止打開分片。  
它接受以下值：

* `false`：打開分片時不執行額外的損壞檢查。 這是預設和推薦的行為。
* `checksum`：驗證分片中每個文件的 checksum 是否與其內容相匹配。 這將檢測「從磁碟讀取的數據」與「Elasticsearch 最初寫入的數據」不同的情況，例如由於未檢測到的磁碟損壞或其他硬件故障。 這些檢查需要從磁碟讀取整個分片，這需要大量時間和 IO bandwidth，並且可能會因為從 filesystem cache 中逐出重要數據而影響集群性能。
* `true`：執行與 checksum 相同的檢查，還檢查分片中的邏輯不一致，例如，這可能是由於 RAM 故障或其他硬件故障導致數據在寫入時被損壞造成的。 這些檢查需要從磁碟讀取整個分片，這需要大量的時間和 IO bandwidth，然後對分片的內容執行各種檢查，這需要大量的時間、CPU 和 memory。

