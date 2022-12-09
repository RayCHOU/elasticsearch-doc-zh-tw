# [Scripting](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)

通過腳本，您可以 evaluate Elasticsearch 中的自定義表達式。  
例如，您可以使用腳本將計算值作為 field 返回或 evaluate query 的自定義分數。

預設的腳本語言是 [Painless](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html)。  
其他 `lang` plugins 可用於運行用其他語言編寫的腳本。  
您可以在腳本運行的任何地方指定腳本的語言。

## Available scripting languages

Painless 專為 Elasticsearch 構建，可用於腳本 API 中的任何目的，並提供最大的靈活性。  
其他語言不太靈活，但可以用於特定目的。

| Language | Sandboxed | Required plugin | Purpose |
| -------- | --------- | --------------- | ------- |
| [`painless`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html) | Yes | Built-in | Purpose-built for Elasticsearch |
| [`expression`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-expression.html) | Yes | Built-in | Fast custom ranking and sorting |
| [`mustache`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-template.html) | Yes | Built-in | Templates |
| [`java`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-engine.html) | No | You write it! | Expert API |

## Next

* [Painless scripting language](painless.md)
* [How to write scriptsedit](how-to-write-scripts.md)
* [Access fields in a document](script-fields-api.md)
* [Common scripting use cases](common-use-cases.md)
* [Accessing document fields and special variables](scripting-fields.md)
* [Scripting and security](scripting-security.md)
* [Lucene expressions language](scripting-expression.md)
