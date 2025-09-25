# Ceph
## External vs Internal Ceph for k8s
This is my opinion, but I rather have ceph external for any large k8s deployments, or multiple k8s cluster. Easier to manage and not dependant on k8s itself. But downside is that gets more complex as now need to manage another system and then networking needs to be more involved
If it was smallish sure since less things to go wrong <br>

### External Ceph Cluster
#### Pros ✅
- Performance isolation: Ceph workloads don't compete with your application workloads for CPU, memory, and I/O
- Simpler troubleshooting: Storage issues are isolated from Kubernetes issues
- Better resource control: Dedicated hardware/resources for storage
- Mature operations: Well-established operational patterns and tooling
- Easier upgrades: Can upgrade Ceph independently of Kubernetes
- Predictable performance: No noisy neighbor problems from application pods
- Better for large-scale: More suitable for production environments with significant storage needs

#### Cons ❌
- Additional infrastructure: Need separate servers/machines for Ceph
- Higher operational overhead: Managing two separate systems
- Network complexity: Additional network considerations between K8s and Ceph
- Cost: Additional hardware/cloud instances required

### Ceph-in-Kubernetes (via Rook)
#### Pros ✅
- Unified management: Everything managed through Kubernetes
- Resource efficiency: Can share infrastructure (good for dev/test/small prod)
- Automated operations: Rook handles much of Ceph's complexity
- Faster deployment: Quick to get up and running
- Cost-effective: No additional infrastructure needed
- GitOps friendly: Entire setup can be version-controlled
#### Cons ❌
- Resource contention: Ceph pods compete with application pods
- Complex debugging: Storage and compute issues can be intertwined
- Limited scalability: Performance bottlenecks as cluster grows
- Upgrade complexity: Upgrading Ceph may impact applications
- Operational risk: Issues in Ceph can destabilize entire Kubernetes cluster
- Not ideal for high-performance workloads: Shared resources limit IOPS/throughput
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
