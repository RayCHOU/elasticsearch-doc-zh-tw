# [使用 field API 訪問文檔中的 fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/script-fields-api.html)

警告：field API 仍在開發中，應被視為測試版功能。  
API 可能會發生變化，並且此迭代(iteration)可能不是最終狀態。 有關功能狀態，請參閱 [#78920](https://github.com/elastic/elasticsearch/issues/78920)。

使用 `field` API 訪問文件的 fields：

```javascript
field('my_field').get(<default_value>)
```

該 API 從根本上改變了您在 Painless 中訪問文檔的方式。 以前，您必須使用要訪問的 field 名稱訪問 doc map：

```javascript
doc['my_field'].value
```

以這種方式訪問 document fields 不會處理缺失值或缺失映射，這意味著要編寫強健的 Painless 腳本，您需要包含邏輯來檢查 field 和值是否都存在。

相反，使用 `field` API，這是在 Painless 中訪問文檔的首選方法。 `field` API 處理缺失值，並將演變為對 `_source` 和 `doc_values` 的抽象訪問。

NOTE: 某些 fields 與 `field` API 尚不兼容，例如 `text` 或 `geo` fields。 繼續使用 `doc` 訪問 `field` API 不支持的 field 類型。

`field` API 返回一個 `Field` object，該物件迭代具有多個值的 fields，通過 `get(<default_value>)` 方法以及類型轉換和輔助方法提供對基礎值的訪問。

`field` API 返回您指定的默認值，無論該 field 是否存在或是否具有當前文檔的任何值。  
這意味著 `field` API 可以處理缺失值而不需要額外的邏輯。  
對於 `keyword` 等引用類型，默認值可以為 `null`。  
對於 `boolean` 或 `long` 等基本類型，默認值必須是匹配的基本類型，例如 `false` 或 `1`。

## Convenient, simpler access

您可以包含 `$` 快捷方式，而不是使用 `get()` 方法顯式呼叫 `field` API。  
只需包含 `$` 符號、field 名稱和默認值，以防字段沒有值：

```javascript
$('field', <default_value>)
```

借助這些增強的功能和簡化的語法，您可以編寫更短、更簡單且更易於閱讀的腳本。  
例如，以下腳本使用過時的語法來確定索引文檔中兩個複雜日期時間值之間的毫秒差值：

```javascript
if (doc.containsKey('start') && doc.containsKey('end')) {
   if (doc['start'].size() > 0 && doc['end'].size() > 0) {
       ZonedDateTime start = doc['start'].value;
       ZonedDateTime end = doc['end'].value;
       return ChronoUnit.MILLIS.between(start, end);
   } else {
       return -1;
   }
} else {
   return -1;
}
```

使用 `field` API，您可以更簡潔地編寫相同的腳本，而無需額外的邏輯來確定 fields 是否存在，然後再對它們進行操作：

```javascript
ZonedDateTime start = field('start').get(null);
ZonedDateTime end = field('end').get(null);
return start == null || end == null ? -1 : ChronoUnit.MILLIS.between(start, end)
```

## Supported mapped field types

下表指示 `field` API 支持的映射 field 類型。 對於每個受支持的類型，列出了 `field` API（來自 `get` 和 `as<Type>` 方法）和 `doc` 映射（來自 `getValue` 和 `get` 方法）返回的值。

NOTE: `fields` API 目前不支持某些 fields，但您仍然可以通過 doc map 訪問這些 fields。有關受支持 fields 的最新列表，請參閱 [#79105](https://github.com/elastic/elasticsearch/issues/79105)。

(略)
