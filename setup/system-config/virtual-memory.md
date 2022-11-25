# Virtual memory

Elasticsearch 默認使用 [mmapfs](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-store.html#mmapfs) 目錄來存儲其索引。 
默認操作系統對 mmap 計數的限制可能太低，這可能會導致 out of memory exceptions。

在 Linux 上，您可以通過以 root 身份運行以下命令來增加提高限制：

    sysctl -w vm.max_map_count=262144

要永久設置此值，請更新 `/etc/sysctl.conf` 中的 `vm.max_map_count` 設置。
要在重新啟動後進行驗證，請運行 `sysctl vm.max_map_count`。

RPM 和 Debian packages 將自動配置此設置。 無需進一步配置。
