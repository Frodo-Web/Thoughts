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


// These gets us 940 000 samples per second for 1 cluster
// 12299 for 2nd
```
This gives us average sample size after compression
```
sum(vm_data_size_bytes) / sum(vm_rows{type!~"indexdb.*"})

// This gets us 0.3
// 1.45 to 1.10 for 2nd - we can diagnose high cardinality issues probably
```
Formula
```
Bytes Per Sample * Ingestion rate * Replication Factor * (Retention Period in Seconds +1 Retention Cycle(day or month)) * 1.2 (recommended 20% of dree space for merges ) 
```
Calculation example
```
(1 byte-per-sample * 940000 time series * 1 replication factor * 2592000 seconds) * 1.2 ) / 2^30 = 381 GB
(1 byte-per-sample * 24749 time series * 1 replication factor * 2592000 seconds) * 1.2 ) / 2^30 = 381 GB

```
