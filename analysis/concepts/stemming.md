# Stemming (詞幹提取)

資訊來源（英文版）： https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html

詞幹提取 是將單詞還原為其 詞根形式 的過程。 
這確保了搜索過程中單詞的變體匹配。

例如，`walking` 和 `walked` 可以源於同一個詞根：`walk`。 
如果做了「詞幹提取」，任一單詞的出現都會與搜索中的另一個單詞匹配。

詞幹提取 與語言相關，但通常涉及從單詞中刪除前綴和後綴。

在某些情況下，詞幹提取後的 詞根形式 可能不是真正的詞。 
例如，`jumping` 和 `jumpiness` 都可以被詞幹化 `jumpi`。 
雖然 `jumpi` 不是一個真正的英語單詞，但它對於搜索來說並不重要； 如果一個單詞的所有變體都簡化為相同的詞根形式，它們將正確匹配。

## Stemmer token filters

資訊來源（英文版）： https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html#stemmer-token-filters

在 Elasticsearch 中，詞幹分析由 stemmer token filters 處理。 
這些 token filters 可以根據 詞幹提取 的方式進行分類：

* [Algorithmic stemmers(算法詞幹提取器)](#algorithmic-stemmers)，
  根據一組規則對單詞進行詞幹提取
* [Dictionary stemmers(詞典詞幹提取器)](#dictionary-stemmers)，
  通過在詞典中查找單詞來提取詞幹

由於詞幹分析會更改 tokens，因此我們建議在[索引和搜索分析](analysis-index-search-time.md)期間使用相同的 stemmer token filters。

## Algorithmic stemmers

https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html#algorithmic-stemmers

算法詞幹分析器 對每個單詞應用一系列規則，將其還原為詞根形式。 
例如，英語的 算法詞幹分析器 可以從復數單詞的末尾刪除 -s 和 -es 後綴。

算法詞幹提取器 有一些優點：

* 它們幾乎不需要設置，並且通常開箱即用。
* 他們使用很少的 memory。
* 它們通常比 字典詞幹分析器 更快。

然而，大多數 算法詞幹提取器 僅針對單詞的現有文字做某些改變。 
這意味著它們可能無法很好地處理不包含其詞根形式的不規則單詞，例如：

* `be`, `are`, 和 `am`
* `mouse` 和 `mice`
* `foot` 和 `feet`

porter_stem, our recommended algorithmic stemmer for English.
snowball, which uses Snowball-based stemming rules for several languages.

以下 token filters 使用 算法詞幹提取：

* [stemmer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html)，它為多種語言提供 算法詞幹提取，其中一些還帶有其他變體。
* [kstem](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-kstem-tokenfilter.html)，一個英語詞幹分析器，它將 算法詞幹提取 與 內置詞典 相結合。
* [porter_stem](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-porterstem-tokenfilter.html)，我們推薦的英語算法詞幹分析器。
* [snowball](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-snowball-tokenfilter.html)，它對多種語言使用基於 [Snowball](https://snowballstem.org/) 的詞幹規則。

## Dictionary stemmers

https://www.elastic.co/guide/en/elasticsearch/reference/current/stemming.html#dictionary-stemmers

詞典詞幹分析器 在提供的詞典中查找單詞，用詞典中的 詞幹詞 替換 未詞幹化的變體。

理論上，字典詞幹分析器 非常適合於：

* 詞幹化 不規則單詞
* 區分 拼寫相似 但 概念上不相關 的單詞，例如：
  * `organ` 和 `organization`
  * `broker` 和 `broken`

在實務上，算法詞幹分析器 通常優於 字典詞幹分析器。 
這是因為字典詞幹分析器有以下缺點：

* __詞典質量__  
  字典詞幹分析器 的好壞取決於它的字典。
  為了發揮良好作用，這些詞典必須包含大量單詞，定期更新，並隨著語言趨勢而變化。
  通常，當字典面世時，它是不完整的，並且其中的一些條目已經過時。
* __尺寸和性能__  
  字典詞幹分析器 必須將字典中的所有單詞、前綴和後綴加載到記憶體中。
  這會使用大量 RAM。
  低質量的字典在刪除前綴和後綴方面也可能效率較低，這會顯著減慢詞幹提取過程。
