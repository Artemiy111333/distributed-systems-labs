## №1 Sidecar
- `screenshots/01-topz-result.png`: в одном кадре `docker ps` (два контейнера `Up`) и `curl http://localhost:8080/topz` (процессы `tail` и `/server`). Уже показанный в переписке результат успешно проверен, нужно сохранить PNG.

## №2 Ambassador
- `screenshots/01-redis-ready.png`: `kubectl get pods -o wide` — три `sharded-redis-*` со статусом `Running`.
- `screenshots/02-ambassador-ready.png`: четыре Pod: `ambassador-example 1/1 Running`, три Redis `1/1 Running` (как в образце преподавателя).
- `screenshots/03-experiment.png`: `kubectl get pods` + `kubectl get svc` + подтверждение работы `experiment-example` и backend-сервисов (вторая часть методички).

## №3 Adapter
- `screenshots/01-pods.png`: `adapter-example 2/2 Running`, `prometheus` Pod `1/1 Running`.
- `screenshots/02-metrics.png`: ответ `curl http://localhost:9121/metrics` с `redis_up 1`.
- `screenshots/03-targets-up.png`: Prometheus -> Status -> Target health, `prometheus` и `redis` в состоянии `UP` (как в образце преподавателя).

## №4 Replicated Service
- `screenshots/01-replicas.png`: `kubectl get pods`, три dictionary-server Pod `Running`.
- `screenshots/02-services.png`: `kubectl get svc` и доступность основного сервиса.
- `screenshots/03-varnish.png`: два varnish-cache Pod `Running`, `curl -I http://localhost:8080/` через Varnish.
- `screenshots/04-https.png`: четыре nginx-ssl Pod `Running` и `curl -k https://localhost:8443/` с ответом nginx.
