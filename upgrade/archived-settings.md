# [Archived settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/archived-settings.html)

如果您將具有已被棄用 (deprecated) 的「persistent cluster setting」的 cluster 升級到不再支持該設置的版本，  
Elasticsearch 會自動將該 setting 存檔。  
同樣，如果您升級包含具有不受支持的 index setting 的索引的 cluster，Elasticsearch 會將該 index setting 歸檔。

我們建議您在升級後刪除所有被歸檔的 settings。  
被歸檔的 settings 被視為無效，可能會影響您配置其他設置的能力。

被歸檔的 settings 會以 `archived.` 前綴開頭。

## Archived cluster settings

使用以下 [cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) 請求檢查被歸檔的 cluster settings。  
如果請求返回一個 empty object ({ })，則沒有被歸檔的 cluster settings。

```http
GET _cluster/settings?flat_settings=true&filter_path=persistent.archived*
```

要刪除任何被歸檔的 cluster settings，請使用以下 [cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) 請求。

```http
PUT _cluster/settings
{
  "persistent": {
    "archived.*": null
  }
}
```

Elasticsearch 不會歸檔臨時 cluster settings 或 `elasticsearch.yml` 中的設置。  
如果一個節點在 `elasticsearch.yml` 中包含不受支持的設置，它將在啟動時返回錯誤。

## Archived index settings

重要：  
在升級之前，請從索引和 component templates 中刪除任何不受支持的索引設置。 
在升級過程中，Elasticsearch 不會在 templates 中歸檔不受支持的索引設置。  
嘗試使用包含不受支持的索引設置的模板將失敗並返回錯誤。  
這包括自動化操作，例如 ILM rollover 操作。

被歸檔的 index settings 不會影響索引的 configuration 或大多數索引操作，例如索引或搜索。  
但是，您需要先刪除它們，然後才能為索引配置其他 settings，例如 `index.hidden`。

使用以下 [get index settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-settings.html) 請求來獲取具有 archived settings 的列表索引。  
如果返回一個 empty object ({ })，則沒有 archived index settings。

```http
GET */_settings?flat_settings=true&filter_path=**.settings.archived*
```

要刪除任何 archived index settings，請使用以下 [indices update settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html) 請求。

```http
PUT /my-index/_settings
{
  "archived.*": null
}
```
