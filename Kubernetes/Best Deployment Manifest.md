# Best Deployment Manifest

## Deployment должен иметь
 - Реквесты (Лимиты?)
 - Readiness, liveness пробы

## Fault Tolerance
Для отказоусточивости используют:
 - Multiple replicas (usually >=2)
 - Update Strategy, Pod Disruption Budget (PDB)
 - topologySpreadConstraints (распределение по нодам, распределение по зонам)
 - Node Affinity / Taints & Tols to control scheduling
 - Anti-affinity rules (podAntiAffinity) to spread pods across nodes

## Rollouts & Rollbacks
Control how updates happen:

Rolling update strategy with sane maxSurge and maxUnavailable
## Security
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
 - Run as non-root user (runAsNonRoot: true, userID)
 - Use least-privilege service accounts
 - Restrict capabilities (drop unnecessary Linux capabilities)
 - Image security : Use signed or trusted images, avoid latest tags
 - Use Kubernetes Secrets or external secret managers
 - Pod Security Admission (PSA) 


