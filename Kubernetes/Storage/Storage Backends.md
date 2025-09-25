# Storage Backends
## Terms
Erasure coding - a technique used in data protection and data storage that distributes redundant information across multiple storage nodes, allowing the system to recover from the loss of data without the need for a complete copy of each piece

## Longhorn
- Эффективные block-change detection бекапы в S3, NFS, Cloud из коробки, по 500 ГБ тома
- Есть поддержка репликации данных на других ноды
- Инкрементальные снапшоты
- Не поддерживает erasure coding
- RWX
- Ставится в куб
- Hyper-converged solution
- Есть жалобы на write perfomance у распределлёных кластеров БД, но возможно тут можно репликации избежать
- Может порождать репликацию репликации, например кластер Patroni, etcd.  I hear you like distributed replication so we put distributed replication on your distributed replication so you can replicate your replications while you distribute your distributions.. 

## Rook + CephFS
- Build on top of распредленное объектное хранилище RADOS
- Поддерживает erasure coding
- RWX
- Если сломается, тяжело восстанавливать
- Есть RDB, есть file storage, есть возможность S3

## Linstor
- Оркестрация блочкой в датацентре и не только

## NFS с провижинером
- Простота настройки
- Нет распределения
- RWX

## OpenEBS Mayastor
- Есть поддержка репликации данных на других ноды

## FUSE FS implementation over S3 
- GeeseFS от яндекса самое норм - больше возможностей, s3fs имеет плохие отзывы
- Конечно производительность будет желать лучшего

## Общее
- Базы могут не понимать на каком типе сторадж работают, и ожидать большего - это может плохо сказываться.
