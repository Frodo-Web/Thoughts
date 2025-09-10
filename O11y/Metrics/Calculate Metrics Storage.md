# Calculate Storage
Original Article - https://docs.victoriametrics.com/guides/understand-your-setup-size/ <br>
Виртуалка vmstorage с virtio_blk driver <br>
```
ls /sys/block/vda/device/driver/module/drivers/virtio\:virtio_blk/
```
<img width="1890" height="660" alt="image" src="https://github.com/user-attachments/assets/ae24023f-8cd0-449a-af8d-0bb3a25ecb64" />
NVMe passthrough is a virtualization technique that directly assigns a physical NVMe SSD and its PCIe controller to a virtual machine (VM), bypassing the host system's hypervisor to offer the VM dedicated, high-performance access to the drive <br>
State on 1st server:
```
113G	/var/lib/victoria-metrics-data/indexdb
279G	/var/lib/victoria-metrics-data/data
```
State on 2nd server:
```
46G  /var/lib/victoria-metrics-data/indexdb
79G  /var/lib/victoria-metrics-data/data
```

Тут видно что у второго довольно большой индекс по отношению к data. Это говорит о high churn rate - когда старые time series постоянно заменяются новыми (новые values в лейблах к примеру) - Например пересоздание подов генерирует такие события <br>

Важно не путать high churn rate с high cardinality, high cardinality - это когда много АКТИВНЫХ time series. Они не обяазтельно должны заменять старые <br>

Unique timeseries stored in database
```
curl http://server1/api/v1/series/count
..
{"status":"success","data":[349613065]}

curl http://server3:8428/api/v1/series/count
..
{"status":"success","data":[293375582]}

Разница 16%. У первого ретеншн 1 месяц, у третьего 4 месяца. 
```
Ingestion rate
```
// Ingestion rate before replication
sum(rate(vm_vminsert_metrics_read_total[24h]))

sum(rate(vm_rows_inserted_total[24h]))
sum(rate(vm_rows_inserted_total{instance=~".*kubemon.*"}[24h]))
sum(rate(vm_rows_inserted_total{app_name=~".*prom.*"}[24h]))



// These gets us 470 000 samples per second for 1 cluster - вырос с 420 до 470 за месяц
// 12299 for 2nd
// 12360 for 3rd
// 1130000 for 4th
```
This gives us average sample size after compression
```
sum(vm_data_size_bytes) / sum(vm_rows{type!~"indexdb.*"})
sum(vm_data_size_bytes{instance=~".*kubemon.*"}) / sum(vm_rows{type!~"indexdb.*", instance=~".*kubemon.*"})


// This gets us 0.33
// 1.45 to 1.10 for 2nd - we can diagnose high cardinality issues probably
// 1.7 to 1.2
```
Formula
```
Bytes Per Sample * Ingestion rate * Replication Factor * (Retention Period in Seconds +1 Retention Cycle(day or month)) * 1.2 (recommended 20% of dree space for merges ) 
```
Calculation example
```
(0.33 byte-per-sample * 470000 time series * 1 replication factor * 2592000 seconds) * 1.2 ) / 2^30 = 450 GB
(0.33 byte-per-sample * 520000 time series * 1 replication factor * 2592000 seconds) * 1.2 ) / 2^30 = 500 GB - если ingestion рейт опять подрастёт на 50к то столько нужно на следующий месяц. 50 ГБ в месяц
(1.43 byte-per-sample * 12299 time series * 1 replication factor * (2592000 seconds * 4)) * 1.2 ) / 2^30 = 205 GB
(0.43 * 1130000 * (2592000 * 6) * 1 * 1.2) / 1073741824 = 8.4 TB
```
