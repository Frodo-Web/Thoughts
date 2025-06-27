# Где хранить метрики
## VictoriaMetrics
### Дано
Локальный диск (HDD/SSD). Object storage — опционально (для бэкапов). Быстрые локальные чтения/записи, высокая компрессия. <br>
VictoriaMetrics uses its own custom, high-performance, append-only block storage format , similar to Prometheus. This means:
 - It writes time series data to disk in optimized blocks.
 - It expects fast, low-latency disk access — like SSDs or local volumes.
 - It is not designed for object storage (like S3) out of the box for active data.
### Мысли
Раз она блочка то деплоить сторадж виктории в кубер это дичь. <br>
В теории можно через S3 CSI Driver подмонтировать раздел и лить метрики туда, но S3 имеет большие задержки, а виктория ожидает быстрые диски, это может зааффектить.
