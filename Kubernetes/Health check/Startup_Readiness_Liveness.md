# Startup, Readiness, Liveness - Когда нужно использовать и какие пробы?
## Readiness
- Purpose: Determines when a pod is ready to receive traffic.
- Almost always use it if your app needs time to initialize or has external dependencies (e.g., database connections, caches).
- Without it, Kubernetes may send traffic to a pod before it’s actually ready, causing request errors.

Инициализация приложения, и инициализация взаимодействия, каналов связи с внешними зависимостями! Только после успеха на такой под должен идти трафик. То есть прежде чем под сможет работать, он должен успеть установить соединение с базой к примеру. Инициализировать код, закачать что нибудь из S3 и прочее. Сбилдить кеш. <br>
Приложуха не может пулять запросы в кеш или СУБД без установки соединения!

## Liveness
- Purpose: Determines if the app is running correctly. If it fails, Kubernetes kills and restarts the pod.
- Only if your app can get into a deadlocked or unrecoverable state that doesn’t crash the process.
- Caution: A poorly configured liveness probe can cause cascading restarts or outages (e.g., during temporary load spikes).

Например, приложенеие многозадачное, многопоточное. Вот одни потоки с аллокацией, освобождением памяти работают (которая лимитирована). Другие потоки с i/o (network, disk и другие). <br>
У приложения могут зависать потоки, например, по любым причинам. Поток много что умеет выполнять, и поэтому мы делаем отдельный поток - смотритель (watcher). Который отдаёт такую стату. И плюс ещё другие показатели по внутренней работе приложение - совокупность факторов даёт непрохождение Liviness пробы и приведёт к рестарту пода. <br>

⚠️ Use selectively—only when truly needed. Some teams avoid liveness probes altogether and rely on app self-termination or external monitoring instead.

### К чему приведёт Liveness проба на коннект к СУБД?
```
❌ 1. Confuses Application Health with Dependency Health
Liveness probes should only reflect whether the application itself is alive, not whether its dependencies (like databases) are available.
If the database is down but your app is otherwise running fine, Kubernetes will restart your healthy pod, which won’t fix the underlying DB issue and may even increase load (e.g., reconnections, bootstrap overhead).
🔁 Result: Unnecessary restarts → cascading failures, service disruption, or prolonged outages.

❌ 2. Amplifies Outages
Imagine a temporary database hiccup (e.g., network blip, failover, maintenance).
If all app pods fail liveness due to DB unavailability, Kubernetes kills and restarts them simultaneously.
When they restart, they all try to reconnect to the DB at once, potentially overwhelming it further.
📉 This turns a minor, transient issue into a full-blown cascading failure.

❌ 3. Violates Responsibility Boundaries
Liveness probe: “Is my process stuck or deadlocked?” → If yes, restart it.
Readiness probe: “Am I ready to serve traffic?” → If DB is down, return not ready; traffic stops flowing, but the pod stays alive.
External monitoring/alerting: “Is the database up?” → Handled separately by SRE/observability tools.
```

Вместо этого:
```
Handle DB failures gracefully in code:
Use retries, circuit breakers, or fallbacks.
Log errors and alert via monitoring (e.g., Prometheus), but don’t crash the pod.
```
## Startup
- Purpose: Indicates when the application has finished starting up. Disables liveness/readiness checks during startup.
- For slow-starting apps (e.g., JVM-based services, apps that load large datasets).
- Prevents premature killing by liveness probes or traffic routing by readiness probes.

То есть тут даётся довольно длительное время на запуск приложения. Игнорируются другие пробы, чтобы не перезапускать преждевременно поду. Лучше использовать когда время запуска тяжело спрогнозировать.

## Итог
Я бы сказал Startup проба - на внутреннюю инициализацию приложения, что может происходить долго у некоторых, Readiness проба - на инициализацию зависимостей приложения (установку соединения, кешей), Liveness проба - на нормальную жизнедеятельность самого приложения

## Приоритет
Readiness - чтобы начал пускать нагрузку на приложение <br>
Startup - долгое время запуска, иногда тяжело прогнозируемое <br>
Liveness - в случае, если приложение может сломаться без краша, тотального вылета (выхода с опред статус кодом) <br>
