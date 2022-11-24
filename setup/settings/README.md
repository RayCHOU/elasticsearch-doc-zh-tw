# [Configuring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)

## Config files location

Elasticsearch 有三個設定檔:

* `elasticsearch.yml` for configuring Elasticsearch
* `jvm.options` for configuring Elasticsearch JVM settings
* `log4j2.properties` for configuring Elasticsearch logging

這些檔案放在 config 目錄裡，它們的預設位置看你是 archive distribution (tar.gz or zip) 或是 package distribution (Debian or RPM packages).

如果是用 archive distributions 安裝，預設 config 目錄位置在 `$ES_HOME/config`. 
可透過 `ES_PATH_CONF` 環境變數 變更目錄的位置如下：

    ES_PATH_CONF=/path/to/my/config ./bin/elasticsearch

或者，您可以通過 command line 或您的 shell profile `export` `ES_PATH_CONF` 環境變數。

如果是用 package distributeions 安裝，config 目錄位置預設為 `/etc/elasticsearch`。
config 目錄的位置也可以通過 `ES_PATH_CONF` 環境變數進行更改，
但請注意，在您的 shell 中設置它是不夠的。
相反的，這個變數來自 `/etc/default/elasticsearch`（for the Debian package）和 `/etc/sysconfig/elasticsearch`（for the RPM package）。
您需要相應地編輯這些文件之一中的 `ES_PATH_CONF=/etc/elasticsearch` 條目以更改配置目錄位置。

## Config file format

configuration 格式為 YAML。
以下是更改 data 和 log 目錄路徑的示例：

```yaml
path:
   data: /var/lib/elasticsearch
   logs: /var/log/elasticsearch
```

也可如下將設定扁平化：

```yaml
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

在 YAML 中，您可以將 non-scalar values 格式化為 sequence：

```yaml
discovery.seed_hosts:
  - 192.168.1.10:9300
  - 192.168.1.11
  - seeds.mydomain.com
```

雖然不太常見，但您也可以將 non-scalar values 格式化為 array：

```yaml
discovery.seed_hosts: ["192.168.1.10:9300", "192.168.1.11", "seeds.mydomain.com"]
```

## Environment variable substitution

configuration 檔案中使用 `${...}` 符號引用的環境變數將替換為環境變量的值。例如：

```yaml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

環境變數的值必須是簡單的字符。
使用逗號分隔的字串提供給 Elasticsearch 將被解析為列表的值。
例如，Elasticsearch 會將以下字符拆分為 `${HOSTNAME}` 環境變數的值列表：

    export HOSTNAME="host1,host2"

## Cluster and node setting types

Cluster 和 node 的設定，根據配置方式可分為以下兩類：

### Dynamic

您可以使用 [cluster update settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) 在運行中的 cluster 上配置和更新動態設置。
在未啟動或關閉的 node 上，您還可以使用 `elasticsearch.yml` 在本地端 配置動態設置。

使用 cluster update settings API 進行的更新可以是持久的 (persistent)，適用於 cluster 重啟，
也可以是瞬態的 (transient)，在 cluster 重啟後重置。
您還可以使用 API assign `null` 值給它們，來重置瞬態或持久設置。

如果您使用多種方法配置相同的設置，Elasticsearch 會按以下優先順序應用這些設置：

1. Transient setting
2. Persistent setting
3. `elasticsearch.yml` setting
4. Default setting value

例如，您可以應用 transient 設置來覆蓋 persistent 設置或 `elasticsearch.yml` 設置。
但是，對 `elasticsearch.yml` 設置的更改不會覆蓋已定義的瞬態或持久設置。

### Static

靜態設置只能使用 `elasticsearch.yml` 在未啟動或關閉的 node 上配置。

必須在 cluster 中的每個相關 node 上設置靜態設置。
