# Лабораторная №2 — Ambassador

Основа: https://github.com/brendandburns/designing-distributed-systems (каталог `ambassadors`). В методичке две части: шардированный Redis и экспериментальное разделение трафика 90/10.

## Часть I. Redis / Twemproxy

```bash
kind create cluster --name redis --wait 120s
kubectl config use-context kind-redis
docker pull redis:7
kind load docker-image redis:7 --name redis
kubectl apply -f redis-service.yaml
kubectl apply -f redis-shards.yaml
kubectl rollout status statefulset/sharded-redis --timeout=180s
kubectl get pods
# Сделайте 01-redis-ready.png

docker pull anchorfree/twemproxy:latest
kind load docker-image anchorfree/twemproxy:latest --name redis
kubectl create configmap twem-config --from-file=nutcracker.yaml --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f twemproxy-ambassador-pod.yaml
kubectl get pods
kubectl logs ambassador-example
# Сделайте 02-ambassador-ready.png
```

Важное: если `kubectl rollout status` завершается таймаутом, диагностируйте `kubectl describe pod sharded-redis-0` и `kubectl get pods -o wide` до следующего шага. Для `anchorfree/twemproxy:latest` явно задано `imagePullPolicy: IfNotPresent`.

## Часть II. Nginx как Ambassador для 90/10

```bash
docker pull nginx:stable
kind load docker-image nginx:stable --name redis
kubectl apply -f experiment-backends.yaml
kubectl rollout status deployment/web --timeout=180s
kubectl rollout status deployment/experiment --timeout=180s
kubectl create configmap experiment-config --from-file=nginx.conf --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f experiment-ambassador-pod.yaml
kubectl get pods
kubectl get svc
# Сделайте 03-experiment.png
kubectl port-forward pod/experiment-example 8082:80
```

В другом терминале: `curl http://localhost:8082/`. Ответ: `PRODUCTION` или `EXPERIMENT`. Использованный в методичке `ip_hash` закрепляет один IP за одним backend, поэтому повторные запросы с одного клиента **не** демонстрируют статистику 90/10. Вес 9:1 характеризует целевое распределение по различным клиентским IP, а не гарантию 9 из каждых 10 запросов.
