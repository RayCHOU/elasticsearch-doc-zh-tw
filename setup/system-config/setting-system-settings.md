# [Configuring system settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html)

在哪裡配置 system settings 取決於您用於安裝 Elasticsearch 的 package 以及您使用的操作系統。

如果是使用 `.zip` 或 `.tar.gz` packages，可以如下方式配置 system settings： 
* 暫時性的使用 [`ulimit`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#ulimit)。 
* 永久性的在 [`/etc/security/limits.conf`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#limits.conf)

如果是使用 RPM 或 Debian packages 安裝，  
大多數 系統設置 都是在 [system configuration file](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#sysconfig) 中設置的。 
但是，使用 systemd 的系統要求在 [systemd configuration file](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#systemd) 中指定 系統限制 (system limits)。

## Sysconfig file

使用 RPM 或 Debian packages 時，可以在 system configuration file 中指定環境變數，該文件位於：

| package | 系統設定檔位置 |
| ------- | ------------ |
| RPM     | /etc/sysconfig/elasticsearch |
| Debian  | /etc/default/elasticsearch |

但是，需要通過 [systemd](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#systemd) 指定 system limits。

## Systemd configuration

在使用 [systemd](https://en.wikipedia.org/wiki/Systemd) 的系統上使用 RPM 或 Debian packages 時，  
必須通過 systemd 指定 system limits。

systemd servie file (`/usr/lib/systemd/system/elasticsearch.service`) 包含了預設會被使用的 limits。

要覆蓋它們，請添加一個名為 `/etc/systemd/system/elasticsearch.service.d/override.conf` 的文件（或者，您可以執行 `sudo systemctl edit elasticsearch` 在預設編輯器中自動打開該文件）。

在此文件中設置任何更改，例如：

```
[Service]
LimitMEMLOCK=infinity
```

完成後，執行以下命令 reload 單元：

```
sudo systemctl daemon-reload
```
