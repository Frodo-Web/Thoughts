# Calculate Storage
Original Article - https://docs.victoriametrics.com/guides/understand-your-setup-size/ <br>
State on 2nd server:
```
46G  /var/lib/victoria-metrics-data/indexdb
79G  /var/lib/victoria-metrics-data/data
```
Total series
```
/api/v1/series/count
..
{
  "status": "success",
  "data": [236108482]
}
```
```
// Ingestion rate before replication
sum(rate(vm_vminsert_metrics_read_total[24h]))

sum(rate(vm_rows_inserted_total[24h]))
sum(rate(vm_rows_inserted_total{instance=~".*kubemon.*"}[24h]))


// These gets us 470 000 samples per second for 1 cluster - вырос с 420 до 470 за месяц
// 12299 for 2nd
// 12360 for 3rd
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
```
