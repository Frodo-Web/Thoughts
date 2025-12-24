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

К чему приведёт Liveness проба на коннект к СУБД?

## Startup
