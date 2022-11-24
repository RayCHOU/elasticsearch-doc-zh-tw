# [Secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)

有些設置是敏感的，依靠文件系統權限來保護它們的值是不夠的。  
對於這個用例，Elasticsearch 提供了一個 密鑰庫 (keystore) 和 [`elasticsearch-keystore` 工具](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-keystore.html) 來管理密鑰庫中的設置。

重要

    只有一些設置被設計為從密鑰庫中讀取。
    但是，密鑰庫沒有驗證來阻止不受支持的設置。
    向密鑰庫添加不受支持的設置會導致 Elasticsearch 無法啟動。
    要查看密鑰庫中是否支持設置，請在設置參考中查找 “Secure” qualifier。

對 keystore 的所有修改只有在重啟 Elasticsearch 後才會生效。

這些設置，就像 `elasticsearch.yml` 配置文件中的常規設置一樣，
需要在集群中的每個節點上指定。
目前，所有安全設置 (secure settings) 都是特定於節點 (node-specific) 的設置，
在每個節點上必須具有相同的值。

(略)
