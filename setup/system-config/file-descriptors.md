# File Descriptors

Elasticsearch 使用了大量的 file descriptors 或 file handles。 
File descriptors 用完可能是災難性的，並且很可能會導致數據丟失。 
確保將運行 Elasticsearch 的用戶的「打開 files descriptors 數量限制」增加到 65,536 或更高。

對於 `.zip` 和 `.tar.gz` packages，在啟動 Elasticsearch 之前將 [`ulimit -n 65535`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#ulimit) 設置為 root，或者在 [`/etc/security/limits.conf`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#limits.conf) 中將 `nofile` 設置為 `65535`。

在 macOS 上，您還必須將 JVM 選項 `-XX:-MaxFDLimit` 傳遞給 Elasticsearch，
以便它使用更高的 file descriptor 限制。

RPM 和 Debian packages 已經將 file descriptors 的最大數量默認為 65535，
不需要進一步配置。

您可以使用 Nodes stats API 檢查為每個節點配置的 `max_file_descriptors`：

```http
GET _nodes/stats/process?filter_path=**.max_file_descriptors
```
