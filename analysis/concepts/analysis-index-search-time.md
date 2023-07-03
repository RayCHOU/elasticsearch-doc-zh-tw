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

