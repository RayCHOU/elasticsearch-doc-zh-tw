# Search cancellation

您可以使用 [任務管理 API](https://www.elastic.co/guide/en/elasticsearch/reference/8.6/tasks.html#task-cancellation) 
取消搜索請求。  
Elasticsearch 還會在客戶端的 HTTP 連接關閉時自動取消搜索請求。  
我們建議您將客戶端設置為在「搜索請求中止或超時」時 關閉 HTTP 連接。
