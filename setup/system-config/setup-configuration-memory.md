# [Disable swapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html)

大多數 操作系統 (operating systems) 會盡可能嘗試將更多的 memory 用於 file system caches，  
並急切地 swap out 未使用的 application memory。   
這可能導致部分 JVM heap 甚至其可執行頁面被 swap 到磁盤。

Swapping 對 性能 和 節點穩定性 非常不利，應該不惜一切代價避免。  
它可能導致 垃圾收集 持續幾分鐘而不是幾毫秒，  
並且可能導致 節點回應緩慢 甚至 與集群斷開連接。  
在彈性分佈式系統中，讓操作系統 kill 節點更有效率。

有三種方法可以 disable swapping。  
首選選項是 完全 disable swap。  
如果這不是一個選項，  
傾向 最小化交換 (minimizing swappiness) 或 內存鎖定 (memory locking)，取決於您的環境。

## Disable all swap files

通常 Elasticsearch 是在一個盒子上運行的唯一服務，它的 memory 使用由 JVM 選項控制。  
應該不需要啟用 swap。

在 Linux 系統上，您可以執行以下命令 臨時禁用 swap：

    sudo swapoff -a

這不需要重新啟動 Elasticsearch。

要永久禁用它，您需要編輯 `/etc/fstab` 文件並註釋掉任何包含單詞 `swap` 的行。

## Configure `swappiness`

Linux 系統上可用的另一個選項是確保將 sysctl 值 `vm.swappiness` 設置為 1。
這降低了 kernel 傾向 swap 的程度，並且在正常情況下不應導致交換，
同時仍允許整個系統在緊急情況下進行 swap。

## Enable `bootstrap.memory_lock`

另一種選擇是在 Linux/Unix 系統上使用 [mlockall](http://opengroup.org/onlinepubs/007908799/xsh/mlockall.html)，
或在 Windows 上使用 [VirtualLock](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366895%28v=vs.85%29.aspx)，
嘗試將 process address space 鎖定到 RAM 中，
防止任何 Elasticsearch heap memory 被 swapped out。

(略)
