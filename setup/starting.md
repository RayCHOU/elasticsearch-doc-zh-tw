# [Starting Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html)

啟動 Elasticsearch 的方法因您的安裝方式而異。

## Archive packages (`.tar.gz`)

如果您使用 `.tar.gz` 安裝 Elasticsearch，則可以從 command line 啟動 Elasticsearch。

### Run Elasticsearch from the command line

在 command line 執行以下命令 啟動 Elasticsearch：

    ./bin/elasticsearch

### Run as a daemon

要將 Elasticsearch 執行為一個 daemon，請在 command line 中指定 `-d`，並使用 `-p` 選項將 process ID 記錄在文件中：

Log 訊息可以在 `$ES_HOME/logs/` 目錄中找到。

要關閉 Elasticsearch，請終止 pid 文件中記錄的 process ID：

    pkill -F pid

## [Archive packages (`.zip`)](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-zip)

(略)

## [Debian packages](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-deb)

### Running Elasticsearch with systemd

要將 Elasticsearch 配置為在系統啟動時自動啟動，請運行以下命令：

    sudo /bin/systemctl daemon-reload
    sudo /bin/systemctl enable elasticsearch.service

Elasticsearch 可以按如下方式啟動和停止：

    sudo systemctl start elasticsearch.service
    sudo systemctl stop elasticsearch.service

這些命令不提供關於 Elasticsearch 是否成功啟動的反饋。相反的，此信息將寫入位於 `/var/log/elasticsearch/` 的日誌文件中。

## [Docker images](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-docker)

略

## [RPM packages](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#start-rpm)

略
