# [Install Elasticsearch with Debian Package](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)

Debian package for Elasticsearch 可用來安裝 Elasticsearch 於任何 Debian-based system 例如  Debian 以及 Ubuntu.

## Import the Elasticsearch PGP Key

Download and install the public signing key:

    $ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

## Installing from the APT repository

可能需先安裝 `apt-transport-https` package:

    $ sudo apt-get install apt-transport-https

Save the repository definition to `/etc/apt/sources.list.d/elastic-8.x.list`:

    $ echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

安裝 Elasticsearch Debian package:

    $ sudo apt-get update && sudo apt-get install elasticsearch

## Download and install the Debian package manually

https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html#install-deb 

The Debian package for Elasticsearch v8.5.0 可由網站下載並安裝如下：
```
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.5.0-amd64.deb
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.5.0-amd64.deb.sha512
$ shasum -a 512 -c elasticsearch-8.5.0-amd64.deb.sha512 
$ sudo dpkg -i elasticsearch-8.5.0-amd64.deb
```

## Running Elasticsearch with `systemd`

執行以下命令，設定 Elasticsearch 在系統重開時自動啟動：

    $ sudo /bin/systemctl daemon-reload
    $ sudo /bin/systemctl enable elasticsearch.service

Elasticsearch 可如下啟動或停止：

    $ sudo systemctl start elasticsearch.service
    $ sudo systemctl stop elasticsearch.service

這些命令不會說啟動是否成功。
這些資訊會寫到 log 裡：`/var/log/elasticsearch/`.

## Check that Elasticsearch is running

你可以傳送一個 HTTPS request 到 `localhost` port `9200` 來檢查 Elasticsearch 是否有執行：

    $ curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic https://localhost:9200

## Configuring Elasticsearch

`/etc/elasticsearch` 目錄裡包含 Elasticsearch 預設的 runtime configuration.
這個目錄以及裡面全部檔案的擁有者，都在 package 安裝時設為 `root:elasticsearch`.

Elasticsearch 預設由 `/etc/elasticsearch/elasticsearch.yml` 載入 configuration.
這個設定檔的格式說明於：[Configuring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).

Debian package 還有一個系統設定檔 (`/etc/default/elasticsearch`), 
可設定下列參數：

| 參數名稱       | 說明                                 |
| ------------- | ----------------------------------- |
| ES_JAVA_HOME  | Set a custom Java path to be used.  |
| ES_PATH_CONF  | Configuration file directory (which needs to include elasticsearch.yml, jvm.options, and log4j2.properties files); defaults to /etc/elasticsearch. |
| ES_JAVA_OPTS  | Any additional JVM system properties you may want to apply. |
| RESTART_ON_UPGRADE | Configure restart on package upgrade, defaults to false. This means you will have to restart your Elasticsearch instance after installing a package manually. The reason for this is to ensure, that upgrades in a cluster do not result in a continuous shard reallocation resulting in high network traffic and reducing the response times of your cluster. |

## Directory layout of Debian package

Debian package 把 config, logs, 以資料目錄放在適當的地方：

| Type | Description | Default Location | Setting |
| ---- | ----------- | ---------------- | ------- |
| home | Elasticsearch home directory or `$ES_HOME` | `/usr/share/elasticsearch` |
| bin  | Binary scripts including `elasticsearch` to start a node and elasticsearch-plugin to install plugins | `/usr/share/elasticsearch/bin` |
| conf | Configuration files including `elasticsearch.yml` | `/etc/elasticsearch` | [`ES_PATH_CONF`](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#config-files-location) |
| conf | Environment variables including heap size, file descriptors. | `/etc/default/elasticsearch` |
| conf | Generated TLS keys and certificates for the transport and http layer. | `/etc/elasticsearch/certs` |
| data | The location of the data files of each index / shard allocated on the node. | `/var/lib/elasticsearch` | `path.data` |
| jdk  | The bundled Java Development Kit used to run Elasticsearch. Can be overridden by setting the `ES_JAVA_HOME` environment variable in `/etc/default/elasticsearch`. | `/usr/share/elasticsearch/jdk`
| logs | Log files location. | `/var/log/elasticsearch` | `path.logs` |
| plugins | Plugin files location. Each plugin will be contained in a subdirectory. | `/usr/share/elasticsearch/plugins` |
| repo | Shared file system repository locations. Can hold multiple locations. A file system repository can be placed in to any subdirectory of any directory specified here. | Not configured | `path.repo` |

## Next steps

現在你有了一個 Elasticsearch 測試環境。
在你開始認真開發或使用 Elasticsearch 進入 production 之前，
你必須做一些額外的設定：

* Learn how to [configure Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).
* Configure [important Elasticsearch settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html).
* Configure [important system settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html).
