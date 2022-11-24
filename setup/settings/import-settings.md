# [重要設定](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)

Elasticsearch 只需很少的配置即可開始使用，
但在 production 環境中使用 cluster 之前必須考慮一些事項：

* Path settings
* Cluster name setting
* Node name setting
* Network host settings
* Discovery settings
* Heap size settings
* JVM heap dump path setting
* GC logging settings
* Temporary directory settings
* JVM fatal error log setting
* Cluster backups

## Path settings

Elasticsearch 將您索引的數據放在 `data` 目錄。 
Elasticsearch 將自己的應用程序日誌寫入 `log` 目錄，其中包含有關 cluster 運行狀況和操作的信息。

對於 [macOS .tar.gz](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)、[Linux .tar.gz](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html) 和 [Windows .zip](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html) 安裝，
`data` 和 `logs` 預設是 `$ES_HOME` 的子目錄。
但是，`$ES_HOME` 中的文件可能會在升級過程中被刪除。

在 production 環境中，我們強烈建議您將 elasticsearch.yml 中的 `path.data` 和 `path.logs` 設置為 `$ES_HOME` 之外的位置。
預設情況下，Docker、Debian 和 RPM 安裝將 data 和 log 寫入 `$ES_HOME` 之外的位置。

### Warning

不要修改 data 目錄中的任何內容或運行可能會干擾其內容的進程。
如果 Elasticsearch 以外的其他東西修改了 data 目錄的內容，那麼 Elasticsearch 可能會失敗，報告損壞或其他數據不一致，或者在默默丟失一些數據的情況下似乎可以正常工作。
不要嘗試對 data 目錄進行文件系統備份；沒有支持的方式來恢復這樣的備份。
相反，請使用 [快照和還原](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html) 來安全地進行備份。

不要在 data 目錄上運行病毒掃描程序。
病毒掃描程序可以阻止 Elasticsearch 正常工作，並可能修改 data 目錄的內容。
data 目錄不包含可執行文件，因此病毒掃描只會發現誤報 (false positives)。

## [Multiple data paths](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#_multiple_data_paths)

略

## [Migrate from multiple data paths](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#mdp-migrate)

略

## Cluster name setting

只有當一個 node 與 cluster 中的所有其他 nodes 共享其 `cluster.name` 時，它才能加入 cluster。  
預設名稱是 `elasticsearch`，但您應該將其更改為描述 cluster 用途的適當名稱。

    cluster.name: logging-prod
    
## Node name setting

Elasticsearch 使用 `node.name` 作為 Elasticsearch 特定實例的人類可讀標識符 (human-readable identifier)。  
此名稱包含在許多 API 的 response 中。  
node 名稱預設為 Elasticsearch 啟動時機器的主機名，但可以在 `elasticsearch.yml` 中顯式配置：

    node.name: prod-data-2
    
## Network host setting

預設情況下，Elasticsearch 只綁定 `127.0.0.1` 和 `[::1]` 等 loopback 地址。  
這足以在單個服務器上運行一個或多個 nodes 的 cluster 以進行開發和測試，但 [彈性 production cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability-cluster-design.html) 必須涉及其他服務器上的 nodes。  
有許多 [網絡設置](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html)，但通常您只需要配置 `network.host`：

    network.host: 192.168.1.10

### 重要

當您設定了 `network.host`，
Elasticsearch 假定您正在從 開發模式 轉移到 生產模式 (production mode)，
並將許多系統啟動檢查從警告升級為異常 (exceptions)。
查看 [開發和生產模式](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html#dev-vs-prod) 之間的差異。

## 發現、形成 cluster 的設定

在投入 production 之前配置兩個重要的 discovery 和 cluster formation 設置，
以便 cluster 中的 nodes 可以相互發現並選舉一個主節點 (master node)。

### `discovery.seed_hosts`

開箱即用，無需任何網絡配置，Elasticsearch 將綁定到可用的 loopback 地址並掃描 local ports `9300` 到 `9305` 以連接同一服務器上運行的其他節點。
此行為無需進行任何配置即可提供 auto-clustering 體驗。

當您想與其他主機上的 nodes 形成 cluster 時，請使用靜態 `discovery.seed_hosts` 設置。
此設置提供 cluster 中其他 nodes 的列表，這些 nodes 符合主節點資格，並且可能處於活動狀態且可聯繫以播種發現過程 (seed the [discovery process](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-hosts-providers.html))。
此設置接受 cluster 中所有符合主節點條件的 nodes 的 YAML sequence 或地址 array。
每個地址可以是 IP 地址，也可以是通過 DNS 解析為一個或多個 IP 地址的主機名。

```yaml
discovery.seed_hosts:
  - 192.168.1.10:9300
  - 192.168.1.11  # 註1
  - seeds.mydomain.com # 註2
  - [0:0:0:0:0:ffff:c0a8:10c]:9301 # 註3
```

* 註1：port 預設為 9300，可以被覆寫。
* 註2：如果一個主機名解析為多個 IP 地址，該節點將嘗試在所有已解析地址上發現其他節點。
* 註3：IPv6 地址必須用方括號括起來。

如果您的主合格節點 (master-eligible nodes) 沒有固定的名稱或地址，
請使用 替代主機提供程序 ([alternative hosts provider](https://www.elastic.co/guide/en/elasticsearch/reference/current/discovery-hosts-providers.html#built-in-hosts-providers)) 來動態查找它們的地址。

### [`cluster.initial_master_nodes`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#initial_master_nodes)

(略)

## Heap size settings

預設情況下，Elasticsearch 會根據節點的[角色](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#node-roles)和 total memory 自動設置 JVM heap size。
我們建議大多數生產環境使用默認大小。

如果需要，您可以通過手動[設置 JVM heap size](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-heap-size) 來覆蓋默認大小。

## JVM heap dump path setting

默認情況下，Elasticsearch 將 JVM 配置為將 內存不足異常 (out of memory exceptions) 時的 heap dump 到預設的 data 目錄。  
如果是用 RPM 和 Debian package 安裝，data 目錄是 `/var/lib/elasticsearch`。  
如果是用 Linux 和 MacOS 以及 Windows distributions，data 目錄位於 Elasticsearch 安裝的根目錄下。

如果此路徑不適合接收 heap dumps，
請修改 `jvm.options` 中的 `-XX:HeapDumpPath=...` 條目：

* 如果指定目錄，JVM 將根據 running instance 的 PID 為 heap dump 產生檔名。
* 如果指定固定檔名而不是目錄，則當 JVM 需要針對 out of memory exception 執行 heap dump 時，該文件不得存在。否則，heap dump 將失敗。

## [GC logging settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#gc-logging)

默認情況下，Elasticsearch 啟用垃圾收集 (GC) 日誌。  
這些在 `jvm.options` 中配置並輸出到與 Elasticsearch 日誌相同的默認位置。  
默認配置每 64 MB 輪換一次日誌，最多可消耗 2 GB 磁盤空間。

(略)

## [Temporary directory settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#es-tmpdir)

默認情況下，Elasticsearch 使用 startup script 在系統臨時目錄下創建的私有臨時目錄。

(略)

## [JVM fatal error log setting](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#error-file-path)

默認情況下，Elasticsearch 將 JVM 配置為將 fatal error logs 寫到預設的日誌目錄。  
在 RPM 和 Debian packages 上，這個目錄是 `/var/log/elasticsearch`。
在 Linux 和 MacOS 以及 Windows 發行版上，日誌目錄位於 Elasticsearch 安裝的根目錄下。

(略)

## Cluster backups

如果發生災難，快照 (snapshots) 可以防止永久性數據丟失。  
[快照生命週期管理](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html#automate-snapshots-slm) 是定期備份 cluster 的最簡單方法。
有關詳細信息，請參閱 [創建快照](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html)。

### 警告

**拍攝快照是備份 cluster 的唯一可靠且受支持的方式。**  
您不能通過複製其節點的 data 目錄來備份 Elasticsearch 集群。  
不支持從 文件系統級備份 (filesystem-level backup) 中恢復任何數據的方法。  
如果您嘗試從此類備份中恢復集群，它可能會因報告損壞或丟失文件或其他數據不一致而失敗，或者它可能看起來似乎成功了卻默默地丟失了一些數據。
