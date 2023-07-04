# Token graphs

https://www.elastic.co/guide/en/elasticsearch/reference/current/token-graphs.html

當 [tokenizer](analyzer-anatomy.md#analyzer-anatomy-tokenizer) 將 text 轉換為 stream of tokens 時，它還會記錄以下內容：

* 每個令牌在 stream 中的 `position`
* `positionLength`，token 跨越的位置數

使用這些，您可以為 stream 創建 [directed acyclic graph (有向無環圖)](https://en.wikipedia.org/wiki/Directed_acyclic_graph)，
稱為 token graph。 
在 token graph 中，每個位置代表一個 node。 
每個 token 代表一條 edge 或 arc，指向下一個位置。

![token-graph-1](https://github.com/RayCHOU/elasticsearch-doc-zh-tw/assets/1132241/82784550-2de2-4a6b-8337-c27a20b7e12a)

## Synonyms

https://www.elastic.co/guide/en/elasticsearch/reference/current/token-graphs.html#token-graphs-synonyms

某些 token filters 可以將新 tokens（例如同義詞）添加到現有 token stream 中。 
這些同義詞通常與現有 token 具有相同的位置。

在下圖中，`quick` 及其同義詞 `fast` 的位置均為 `0`。它們跨越相同的位置。

![token-graph-2](https://github.com/RayCHOU/elasticsearch-doc-zh-tw/assets/1132241/f1d57da0-5bdd-4923-83e8-441e7fe86cf3)

## Multi-position tokens

https://www.elastic.co/guide/en/elasticsearch/reference/current/token-graphs.html#token-graphs-multi-position-tokens

某些 token filters 可以添加跨越多個位置的 tokens。 
這些可以包括 多詞同義詞 的 tokens，例如使用 "atm" 作為 "automatic teller machine" 的同義詞。

然而，只有某些 token filters（稱為 _graph token filters_）能夠準確記錄 多位置 tokens 的 `positionLength`。 
這些 filters 包括：

* [`synonym_graph`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-graph-tokenfilter.html)
* [`word_delimiter_graph`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-word-delimiter-graph-tokenfilter.html)

某些 tokenizers（例如 [`nori_tokenizer`](https://www.elastic.co/guide/en/elasticsearch/plugins/8.8/analysis-nori-tokenizer.html)）
也可以準確地將 compound tokens 分解為 多位置 tokens。

在下圖中，`domain name system` 及其同義詞 `dns` 的位置均為 `0`。但是，`dns` 的 `positionLength` 為 `3`。圖中的其他 tokens 的 `positionLength` 為預設值 `1`。

![token-graph-3](https://github.com/RayCHOU/elasticsearch-doc-zh-tw/assets/1132241/6df888cd-712b-4af1-a9df-eb4f519e12b8)

### Using token graphs for search

https://www.elastic.co/guide/en/elasticsearch/reference/current/token-graphs.html#token-graphs-token-graphs-search

[索引時](analysis-index-search-time.md)會忽略 `positionLength` 屬性，並且不支持包含「multi-position tokens」的 token graphs。

但是，queries（例如 [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) 
或 [`match_phrase`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html) query）
可以使用這些圖從單個 query string生成多個 sub-queries。

#### Example

用戶使用 `match_phrase` query 運行以下短語的搜索：

    domain name system is fragile

在 [search analysis](analysis-index-search-time.md) 過程中，
`dns`（`domain name system` 的同義詞）被添加到 query string的 token stream 中。 
`dns` token 的 `positionLength` 為 `3`。

![token-graph-4](https://github.com/RayCHOU/elasticsearch-doc-zh-tw/assets/1132241/06bcea7a-1363-4cb4-96cb-511e530e301d)

The match_phrase query uses this graph to generate sub-queries for the following phrases:

`match_phrase` query 使用此圖生成以下短語的 sub-queries：

    dns is fragile
    domain name system is fragile

這意味著 這個 query 會 match 包含 `dns is fragile` 或 `domain name system is fragile` 的文檔。

### Invalid token graphs

以下 token filters 可以添加跨越多個位置的 tokens，但僅記錄預設的 `positionLength` `1`：

* [`synonym`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html)
* [`word_delimiter`](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-word-delimiter-tokenfilter.html)

這意味著這些 filters 將為包含此類 tokens 的 streams 生成無效的 token graphs。

下圖中，`dns` 是 `domain name system` 的 multi-position synonym。 
但是，`dns` 的 `positionLength` 預設為 `1`，導致圖表無效。

![token-graph-5](https://github.com/RayCHOU/elasticsearch-doc-zh-tw/assets/1132241/3b47c871-34b5-40da-963c-1822bf81fd0e)

避免使用無效的 token graphs 進行搜索。 
無效的圖表可能會導致意外的搜索結果。
