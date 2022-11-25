# [Networking](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html)

每個 Elasticsearch node 都有兩個不同的 network interfaces。  
客戶端使用其 [HTTP interface](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#http-settings) 向 Elasticsearch 的 REST API 發送請求，  
但 nodes 使用 [transport interface](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#transport-settings) 與其他 nodes 通信。  
transport interface 還用於與 [remote clusters](https://www.elastic.co/guide/en/elasticsearch/reference/current/remote-clusters.html) 的通信。

您可以使用 `network.*` 設置同時配置這兩個 interfaces。  
如果您有一個更複雜的網絡，  
您可能需要使用 `http.*` 和 `transport.*` 設置獨立配置 interface。  
在可能的情況下，使用適用於兩個接口的 `network.*` 設置來簡化您的配置並減少重複。

默認情況下，Elasticsearch 只綁定到 `localhost`，這意味著它不能被遠程訪問。
這種配置對於由一個或多個節點組成的本地開發集群來說已經足夠了，這些節點都在同一台主機上運行。
要形成跨多個主機的集群，或者遠程客戶端可以訪問的集群，您必須調整一些 [網絡設置](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#common-network-settings)，例如 `network.host`。

配置 Elasticsearch 綁定到非本地地址會將一些警告轉換為 fatal exception。  
如果節點在配置其網絡設置後拒絕啟動，那麼您必須在繼續之前解決記錄的異常。

## 常用網路設定

大多數用戶只需要配置以下網絡設置。

### `network.host`

(Static, string) 為 HTTP 和 transport traffic 設置此節點的地址。
該節點將綁定到該地址，並將其用作其發布地址。
接受 IP 地址、主機名或 [特殊值](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#network-interface-values)。

預設為 `_local_`

### `http.port`

(Static, integer) 為 HTTP 客戶端通信綁定的 port。接受單個值或範圍。如果指定了一個範圍，節點將綁定到該範圍內的第一個可用 port。

預設為 `9200-9300`.

### `transport.port`

(Static, integer) 為節點之間的通信綁定的 port。接受單個值或範圍。如果指定了一個範圍，節點將綁定到該範圍內的第一個可用 port。
在每個 master-eligible node 上將此設置設置為單個端口，而不是一個範圍。

Defaults to `9300-9400`.

## Special values for network addresses

您可以使用以下特殊值 將 Elasticsearch 配置為自動確定其地址。
在配置 network.host、network.bind_host、network.publish_host 以及 HTTP 和 transport interfaces 的相應設置時使用這些值。

| value | 描述 |
| ----- | ---- |
| `_local_`  | 系統上的任何 loopback 地址，例如 `127.0.0.1`。 |
| `_site_`   | 系統上的任何 site-local 地址，例如 `192.168.0.1`。 |
| `_global_` | 系統上任何 globally-scoped 地址，例如 `8.8.8.8`。 |
| `_[networkInterface]_` | 使用名為 `[networkInterface]` 的 network interface 的地址。例如，如果您希望使用名為 `en0` 的接口地址，則設置 `network.host: _en0_`。 |
| `0.0.0.0` | 所有可用 network interfaces 的地址。 |

(略)
