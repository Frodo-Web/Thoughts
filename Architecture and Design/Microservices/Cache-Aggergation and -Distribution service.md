# Cache Aggregation and Distribution Serivce
Группа микросервисов использующих различные Data Source (DBs, In memory Caches, data storages) аггрегирующих определенную инфу, собирающую горячий кеш и распространяющих этот кеш на различные другие микросервисы <br>
Кеш кладётся локально с работающим кодом, размер кеша маленький. Это могут быть какие то SQLite, GeoIP и прочие. <br>
Может быть подразумевать отдельную службу под кеш. <br>
Флоу: <br>
CPU cache of executing program -> Disk cache -> Local cache service via Unix Domain Socket (for example) -> External Cache systems (like in memory DB) -> Actual Databases or data storages (Like S3, Ceph) <br>
И интересно оценить как это может повлиять на конечную стоимость. Если часть инфраструктуры храним локально, а другую в облаке. За счёт кеш систем снижаем облачные расходы. <br>
Интересно рассматривать такое как Переходный момент в стартапе, от облачной инфры на Self Hosted <br>
Это какая то Subscription Cache Model, есть ещё TTL <br>
