# Ceph
This is my opinion, but I rather have ceph external for any large k8s deployments, or multiple k8s cluster. Easier to manage and not dependant on k8s itself. But downside is that gets more complex as now need to manage another system and then networking needs to be more involved
If it was smallish sure since less things to go wrong
## RBD-mirroring
Интересно можно сделать так
```
┌────────────────┐     ┌────────────────┐
│     Site A     │     │     Site B     │
│ ┌────────────┐ │     │ ┌────────────┐ │
│ │   Node 1   │ │     │ │   Node 1   │ │
│ │            │ │     │ │            │ │
│ │            │>──>──>┼─┼─RBD Mirror │ │
│ └────────────┘ │     │ └────────────┘ │
│ ┌────────────┐ │     │ ┌────────────┐ │
│ │   Node 2   │ │     │ │   Node 2   │ │
│ └────────────┘ │     │ └────────────┘ │
│ ┌────────────┐ │     │ ┌────────────┐ │
│ │   Node 3   │ │     │ │   Node 3   │ │
│ └────────────┘ │     │ └────────────┘ │
└────────────────┘     └────────────────┘
```
Чтобы например был Cluster Mesh, в обоих Ceph, в один Ceph идёт зеркалирование. <br>
Но оба Ceph доступны на чтение, а один работает на запись <br>
Но это хрень наверно, я думаю nvme устройства оптимизированы на то что параллельно могут обрабатывать чтение и запись, какие нибудь буферы, очереди и так далее работают
