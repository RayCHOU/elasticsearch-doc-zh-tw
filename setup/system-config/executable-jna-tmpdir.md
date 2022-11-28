# [Ensure JNA temporary directory permits executables](https://www.elastic.co/guide/en/elasticsearch/reference/current/executable-jna-tmpdir.html)

Elasticsearch 使用 Java Native Access (JNA) library 和另一個名為 `libffi` 的 library 來執行一些 platform-dependent 的 native code。 
在 Linux 上，支持這些 libraries 的 native code 在 runtime 被提取到一個臨時目錄中，
然後映射到 Elasticsearch 地址空間中的可執行頁面中。 
這要求底層文件不在使用 `noexec` 選項掛載的 filesystem 上。

預設情況下，Elasticsearch 將在 `/tmp` 中創建其臨時目錄。 
但是，一些強化的 Linux 安裝 預設使用 `noexec` 選項掛載 `/tmp`。 
這會阻止 JNA 和 libffi 正常工作。 
例如，在啟動時，JNA 可能無法加載並出現 `java.lang.UnsatisfiedLinkerError` exception 或類似於 `failed to map segment from shared object` 的訊息，
或者 `libffi` 可能會報告諸如 `failed to allocate closure` 之類的消息。 
請注意，JVM 不同版本的 exception 訊息 可能不同。 
此外，依賴於通過 JNA 執行 native code 的 Elasticsearch 組件可能會失敗，並顯示訊息表明這是 `because JNA is not available`。

要解決這些問題，請從 `/tmp` 文件系統中刪除 `noexec` 選項，
或者通過設置 [`$ES_TMPDIR`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#es-tmpdir) 環境變量來配置 Elasticsearch 為其臨時目錄使用不同的位置。 
例如：

    export ES_TMPDIR=/usr/share/elasticsearch/tmp

如果您需要更好地控制這些臨時文件的位置，
您還可以使用 [JVM flag](https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-options) `-Djna.tmpdir=<path>` 配置 JNA 使用的路徑，
並且可以通過設置 `LIBFFI_TMPDIR` 環境變量 來指定 `libffi` 使用的暫存檔路徑。 
未來版本的 Elasticsearch 可能需要額外的配置，因此您應該盡可能設置 `ES_TMPDIR`。
