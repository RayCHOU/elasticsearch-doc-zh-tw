# [使用 field API 訪問文檔中的 fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/script-fields-api.html)

警告：field API 仍在開發中，應被視為測試版功能。  
API 可能會發生變化，並且此迭代(iteration)可能不是最終狀態。 有關功能狀態，請參閱 [#78920](https://github.com/elastic/elasticsearch/issues/78920)。

使用 `field` API 訪問文件的 fields：

    field('my_field').get(<default_value>)

This API fundamentally changes how you access documents in Painless. Previously, you had to access the doc map with the field name that you wanted to access:
