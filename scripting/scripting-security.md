# [Scripting and security](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-security.html)

Painless 和 Elasticsearch 實施安全層來構建安全運行腳本的縱深防禦策略。

Painless 使用 細粒度(fine-grained) 的白名單。  
任何不屬於許可名單的內容都會導致編譯錯誤。  
此功能是腳本縱深防禦策略中的第一層安全層。

第二層安全層是 [Java 安全管理器](https://www.oracle.com/java/technologies/javase/seccodeguide.html)。  
作為其啟動序列的一部分，Elasticsearch 使 Java 安全管理器 能夠限制 代碼部分 可以執行的操作。  
[Painless](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html) 使用 Java 安全管理器 作為額外的防禦層來防止腳本執行 寫入檔案 和 listening to sockets 等操作。

Elasticsearch 在 Linux 中使用 [seccomp](https://en.wikipedia.org/wiki/Seccomp)，在 macOS 中使用 [Seatbelt](https://www.chromium.org/developers/design-documents/sandbox/osx-sandboxing-design)，在 Windows 中使用 [ActiveProcessLimit](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684147) 作為額外的安全層，以防止 Elasticsearch forking 或運行其他 processes。

您可以修改以下 script settings 以限制允許運行的腳本類型，並控制腳本可以運行的可用 [contexts](https://www.elastic.co/guide/en/elasticsearch/painless/8.5/painless-contexts.html)。  
要在縱深防禦策略中實施其他層，請遵循 [Elasticsearch 安全原則](https://www.elastic.co/guide/en/elasticsearch/reference/current/es-security-principles.html)。

## Allowed script types setting

Elasticsearch 支持兩種腳本類型：`inline` 和 `stored`。  
默認情況下，Elasticsearch 配置為運行這兩種類型的腳本。  
要限制運行的腳本類型，請將 `script.allowed_types` 設置為 `inline` 或 `stored`。  
要阻止任何腳本運行，請將 `script.allowed_types` 設置為 `none`。

重要：如果您使用 Kibana，請將 `script.allowed_types` 設置為 `both` 或 `inline`。  
某些 Kibana 功能依賴於 inline scripts，如果 Elasticsearch 不允許 inline scripts，則無法按預期運行。

例如，要運行 inline scripts 但是不要 stored scripts：

```
script.allowed_types: inline
```

## Allowed script contexts setting

默認情況下，允許所有 script contexts。  
使用 `script.allowed_contexts` 設置指定允許的 contexts。  
要指定不允許任何 contexts，請將 `script.allowed_contexts` 設置為 `none`。

例如，允許 scripts 僅在 `scoring` 和 `update` contexts 中運行：

    script.allowed_contexts: score, update
