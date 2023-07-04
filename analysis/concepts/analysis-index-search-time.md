# Index and search analysis

https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-index-search-time.html

文本分析 發生在兩個時間點：

* Index time (索引時)
  * 當文檔被索引時，所有 [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) field 的值都會被分析。
* Search time (搜尋時)
  * 在 `text` field 上運行 [全文搜索](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html) 時，
    將分析 query string串（用戶要搜索的文字）。
  * Search time 也稱為 query time。

各個時間點使用的 analyzer 或 分析規則集 分別稱為 index analyzer 或 search analyzer。

## How the index and search analyzer work together

在大多數情況下，索引和搜索時應使用相同的分析器。 
這可確保 field 的值和 query string 更改為相同形式的 tokens。 
反過來，這可以確保 tokens 在搜索過程中按預期匹配。

### Example

使用 `text` field 中的以下值對文檔進行索引：

    The QUICK brown foxes jumped over the dog!

field 的 index analyzer 將值轉換為 tokens 並對其進行標準化。 在本例中，每個 token 代表一個單詞：

    [ quick, brown, fox, jump, over, dog ]

然後將這些 tokens 編入索引。

隨後，用戶在同一 `text` field 中搜索：

    "Quick fox"

用戶希望此搜索與之前索引的句子匹配，`The QUICK Brown Foxes Jump over the Dog!`。

但是，query string 不包含 文檔原始文本中使用的確切單詞：

* `Quick` vs `QUICK`
* `fox` vs `foxes`

To account for this, the query string is analyzed using the same analyzer. This analyzer produces the following tokens:
為了解決這個問題，使用相同的分析器來分析 query string。 該分析器生成以下 tokens：

    [ quick, fox ]

To execute the search, Elasticsearch compares these query string tokens to the tokens indexed in the text field.

為了執行搜索，Elasticsearch 會將這些 query string tokens 與 `text` field 中索引的 tokens 進行比較。
		
| Token  | Query string | `text` field |
| ------ | ------------ | ------------ |
| `quick` | X | X |
| `brown` |   | X |
| `fox`   | X | X |
| `jump`  |   | X |
| `over`  |   | X |
| `dog`   |   | X |

由於 field value 和 query string 的分析方式相同，因此它們創建了類似的 tokens。 
`quick` 和 `fox` 這兩個 tokens 完全匹配。 
這意味著這個搜索 會 match 到包含 `"The QUICK brown foxes jumped over the dog!"` 的文檔，正如用戶所期望的那樣。

## When to use a different search analyzer

雖然不太常見，但有時在 索引 和 搜索 時
[使用不同的 analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/specify-analyzer.html#specify-search-analyzer) 是有意義的。 
為此，Elasticsearch 允許您指定單獨的 search analyzer。

一般來說，只有在「欄位值 和 查詢字串 使用相同形式的 tokens」會產生意想不到的或無關的搜索結果時，才應指定單獨的 search analyzer。

### Example

A document is added to the search engine’s index; this document contains one such word in a text field:

Elasticsearch 用於創建一個搜索引擎，僅匹配 以某個提供的前綴開頭的單詞。 
例如，搜索 `tr` 應回傳 `tram` 或 `trope`，但絕不會回傳 `taxi` 或 `bat`。

文檔被添加到搜索引擎的索引中； 該文檔的 `text` field 中包含一個這樣的單詞：

    "Apple"

該 field 的 index analyzer 將 value 轉換為 tokens 並對其進行標準化。 
在本例中，每個 token 代表該單詞的一個潛在前綴：

    [ a, ap, app, appl, apple ]

然後將這些 tokens 編入索引。

隨後，用戶在同一 `text` field 中搜索：

    "appli"

User 希望此搜索僅 match 以 appli 開頭的單詞，例如 Appliance 或 application。 
這個搜索不應 match 到 apple。

但是，如果使用 index analyzer 來分析這個 query string，它將產生以下 tokens：

    [ a, ap, app, appl, appli ]

當 Elasticsearch 將這些 query string tokens 與為 apple 建立索引的 tokens 進行比較時，它會找到多個匹配項。

| Token | appli | apple |
| ----- | ----- | ----- |
| a     | X | X |
| ap    | X | X |
| app   | X | X |
| appl  | X | X |
| appli | X |   |

這意味著搜索將 錯誤地 找到 `apple`。 
不僅如此，它還可以匹配任何以 `a` 開頭的單詞。

要解決此問題，您可以為 `text` field 上使用的 query string 指定不同的 search analyzer。

在本例中，您可以指定一個 search analyzer 來生成單個 token 而不是一組前綴：

    [ appli ]

這個 query string token 僅匹配以 `appli` 開頭的單詞的 token，這更符合用戶的搜索期望。
