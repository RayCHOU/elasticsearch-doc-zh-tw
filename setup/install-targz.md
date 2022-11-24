# [Install Elasticsearch from archive on Linux or MacOS](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)

Elasticsearch is available as a .tar.gz archive for Linux and MacOS.

最新版本可以在 [Download Elasticsearch](https://www.elastic.co/downloads/elasticsearch) 找到。
其他版本可以在 [Past Releases](https://www.elastic.co/downloads/past-releases) 找到。

## Download and install archive for MacOS

下載

    curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.5.2-darwin-x86_64.tar.gz

比較 下載的 `.tar.gz` 壓縮檔的 SHA 與公開的 checksum：

    curl https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.5.2-darwin-x86_64.tar.gz.sha512 | shasum -a 512 -c - 

應該要顯示：`elasticsearch-{version}-darwin-x86_64.tar.gz: OK`.

進入 `$ES_HOME` 目錄

    cd elasticsearch-8.5.2/

## Run Elasticsearch from the command line

啟動 Elasticsearch

    ./bin/elasticsearch

預設 HTTP prot 9200.

## 檢查 Elasticsearch 有在執行

    curl --cacert $ES_HOME/config/certs/http_ca.crt -u elastic https://localhost:9200 

必須使用 https

--cacert 參數： http_ca.crt certificate 路徑

## Run as a Daemon

    ./bin/elasticsearch -d -p pid

* -d 表示 daemon
* -p 將 process ID 記錄在檔案裡

Log 在 `$ES_HOME/logs/` 目錄

結束 Elasticsearch

    pkill -F pid

## Configure Elasticsearch on the command line

Elasticsearch 預設由 `$ES_HOME/config/elasticsearch.yml` 載入 configuration。
這個 config 檔案格式在這裡說明： [Configuring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).

任何可以在 config 檔案設定的也可以在 command line 使用 `-E` 語法設定如下：

    ./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1

## Next steps

現在你有了一個 Elasticsearch 測試環境。
在你開始認真開發或使用 Elasticsearch 進入 production 之前，
你必須做一些額外的設定：

* Learn how to [configure Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).
* Configure [important Elasticsearch settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html).
* Configure [important system settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html).
