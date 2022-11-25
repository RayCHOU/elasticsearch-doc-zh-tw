# Number of threads

Elasticsearch 使用多個 thread pools 來執行不同類型的操作。 
重要的是它能夠在需要時創建新 threads。 
確保 Elasticsearch 用戶可以創建的 threads 數量至少為 4096。

這可以通過在啟動 Elasticsearch 之前，以 root 身份設定 [`ulimit -u 4096`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#ulimit) 來完成，
或者通過在 [`/etc/security/limits.conf`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#limits.conf) 中將 `nproc` 設置為 `4096` 來完成。

在 `systemd` 下作為 services 運行的 package distributions 將自動為 Elasticsearch process 配置 threads 數量。無需額外配置。
