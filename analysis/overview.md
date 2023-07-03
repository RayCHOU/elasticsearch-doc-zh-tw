# Text analysis overview

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-overview.html

文本分析 使 Elasticsearch 能夠執行 全文搜索，搜索回傳所有 相關結果 而不僅僅是 完全匹配。

如果您搜索 `Quick fox jumps`，您可能需要包含 `A quick brown fox jumps over the lazy dog` 的文檔，
並且您可能還需要包含相關單詞（例如 `fast fox` 或 `foxes leap`）的文檔。

## Tokenization

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-overview.html#tokenization

分析通過 tokenization 使全文搜索成為可能：將文本分解成更小的塊，稱為 tokens。 
在大多數情況下，這些 tokens 是單獨的單詞。

如果您將短語 `the quick brown fox jumps` 作為單個字串編制索引，並且用戶搜索 `quick fox`，則不會將其視為 match。 
但是，如果您對短語進行 tokenize 並單獨索引每個單詞，則可以單獨查找 query string 中的術語。 
這意味著可以通過搜索 `quick fox`、`fox brown` 或其他變體來匹配它們。

## Normalization

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-overview.html#normalization

Tokenization 可以對各個術語進行匹配，但每個 token 仍然按字面意思進行匹配。 這意味著：

* 搜索 `Quick` 不會匹配 `quick`，即使您可能希望其中一個術語與另一個術語匹配
* 儘管 `fox` 和 `foxes` 具有相同的詞根，但搜索 `foxes` 不會匹配 `fox`，反之亦然。
* 搜索 `jumps` 不會匹配 `leaps`。 雖然它們不共享詞根，但它們是同義詞並且具有相似的含義。

為了解決這些問題，文本分析 可以將這些 tokens 標準化為標準格式。 
這允許您匹配與 search terms 不完全相同但足夠相似以仍然相關的 tokens。 例如：

* `Quick` 可以小寫：`quick`。
* `foxes` 可以被 詞幹化，或者簡化為其詞根：`fox`。
* `jump` 和 `leap` 是同義詞，可以作為單個單詞進行索引：`jump`。

為了確保 search terms 按預期匹配這些詞，您可以對 query string 應用相同的 tokenization 和 normalization 規則。 
例如，對 `Foxes leap` 的搜索可以標準化為對 `fox jump` 的搜索。

## Customize text analysis

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-overview.html#analysis-customization

文本分析 由 analyzer 執行，analyzer 是一組控制整個過程的規則。

Elasticsearch 包含一個預設的 analyzer，稱為 standard analyzer，它開箱即用，適用於大多數用例。

如果您想定制搜索體驗，可以選擇不同的 built-in analyzer，甚至配置自定義分析器。 
custom analyzer 使您可以控制分析過程的每個步驟，包括：

* tokenization 之前對文本的更改
* 文本如何轉換為 tokens
* 在 indexing 或 搜索 之前對 tokens 進行 normalization 更改
