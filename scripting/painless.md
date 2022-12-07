# [Painless scripting language](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html)

Painless 是一種專為 Elasticsearch 設計的高性能、安全的腳本語言。  
在 Elasticsearch 支持腳本的任何地方，您都可以使用 Painless 安全地編寫 inline 和 stored scripts。

Painless 提供了圍繞以下核心原則的眾多功能：

* **安全性**：確保 cluster 的安全性至關重要。 為此，Painless 使用了細粒度 (find-grained) 的允許列表，其粒度細到 class 的成員。 任何不屬於允許列表的內容都會導致編譯錯誤。 有關每個腳本上下文的可用 classes、methods 和 fields 的完整列表，請參閱 [Painless API Reference](https://www.elastic.co/guide/en/elasticsearch/painless/8.5/painless-api-reference.html)。
* **性能**：Painless 直接編譯成 JVM bytecode，以利用 JVM 提供的所有可能的優化。 此外，Painless 通常會避免在運行時需要額外較慢檢查的功能。
* **簡單性**：Painless 實現了一種語法，任何有基本編碼經驗的人都自然熟悉。 Painless 使用 Java 語法的子集和一些額外的改進來增強可讀性並刪除樣板 (boilerplate)。

## Start scripting

準備好開始使用 Painless 編寫腳本了嗎？ 了解如何 [編寫您的第一個腳本](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html)。

如果您已經熟悉 Painless，請參閱 [Painless 語言規範](https://www.elastic.co/guide/en/elasticsearch/painless/8.5/painless-lang-spec.html) 以獲取有關 Painless 語法和語言功能的詳細說明。
