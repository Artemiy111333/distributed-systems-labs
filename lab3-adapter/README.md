# Лабораторная №3 — Adapter + Prometheus

Основа: лабораторная методичка (Adapter Redis Exporter -> Prometheus).

```bash
# Сначала сохраните скриншоты лабораторной №2
kind delete cluster --name redis
kind create cluster --name adapter --wait 120s
kubectl config use-context kind-adapter

docker pull prom/prometheus:v3.8.0
docker pull redis:7.4-alpine
docker pull oliver006/redis_exporter:v1.76.0
kind load docker-image prom/prometheus:v3.8.0 redis:7.4-alpine oliver006/redis_exporter:v1.76.0 --name adapter
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus.yaml
kubectl apply -f pod-redis.yaml
kubectl apply -f service-redis.yaml
kubectl rollout status deployment/prometheus --timeout=180s
kubectl get pods
# Сделайте 01-pods.png
```

Проверка метрик:

```bash
kubectl port-forward svc/adapter-example 9121:9121
```

В другом терминале: `curl -s http://localhost:9121/metrics | grep '^redis_up'`. Ожидается `redis_up 1`. Скриншот `02-metrics.png`.

Остановите предыдущий `port-forward` (`Ctrl+C`) и запустите:

```bash
kubectl port-forward deployment/prometheus 9090:9090
```

В браузере **в Ubuntu** откройте `http://localhost:9090/targets` → `Status / Target health`: два Target `prometheus`, `redis` в состоянии `UP`. Скриншот `03-targets-up.png`.

Если `DOWN`: `kubectl logs adapter-example -c adapter`, `kubectl get svc`, `kubectl logs deployment/prometheus`, проверить ответ `/metrics`.
