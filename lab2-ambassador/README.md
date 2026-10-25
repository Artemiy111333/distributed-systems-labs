# Лабораторная работа №2. Паттерн Ambassador

## Цель работы

Исследование паттерна Ambassador для организации доступа к шардированному хранилищу Redis и маршрутизации HTTP-трафика между рабочей и экспериментальной версиями сервиса.

## Часть I. Шардирование Redis

Хранилище представлено тремя репликами `sharded-redis-0`, `sharded-redis-1`, `sharded-redis-2`, созданными объектом `StatefulSet`. Сервис `redis` с параметром `clusterIP: None` предоставляет стабильные DNS-имена экземпляров. Twemproxy использует алгоритм `ketama` и хеширование `fnv1a_64` для выбора сервера хранения.

### Развёртывание

```bash
kind create cluster --name redis --wait 120s
kubectl config use-context kind-redis
docker pull --platform linux/amd64 redis:7
docker image save --platform linux/amd64 --output /tmp/redis-amd64.tar redis:7
kind load image-archive /tmp/redis-amd64.tar --name redis
kubectl apply -f redis-service.yaml
kubectl apply -f redis-shards.yaml
kubectl rollout status statefulset/sharded-redis --timeout=180s

docker pull --platform linux/amd64 anchorfree/twemproxy:latest
docker image save --platform linux/amd64 --output /tmp/twemproxy-amd64.tar anchorfree/twemproxy:latest
kind load image-archive /tmp/twemproxy-amd64.tar --name redis
kubectl create configmap twem-config --from-file=nutcracker.yaml --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f twemproxy-ambassador-pod.yaml
```

### Проверка

```bash
kubectl get statefulsets
kubectl get pods -o wide
kubectl get services
kubectl logs ambassador-example
```

В рабочем состоянии присутствуют три Pod с Redis и Pod `ambassador-example` с Twemproxy. Конфигурация Twemproxy монтируется из `ConfigMap` в `/etc/config/nutcracker.yaml`.

## Часть II. Разделение HTTP-трафика

В качестве Ambassador используется Nginx. В конфигурации `nginx.conf` заданы два backend-сервиса — `web` и `experiment` — с отношением весов 9:1 и алгоритмом `ip_hash`. Основная версия возвращает `PRODUCTION`, экспериментальная — `EXPERIMENT`.

### Развёртывание

```bash
docker pull --platform linux/amd64 nginx:stable
docker image save --platform linux/amd64 --output /tmp/nginx-amd64.tar nginx:stable
kind load image-archive /tmp/nginx-amd64.tar --name redis
kubectl apply -f experiment-backends.yaml
kubectl rollout status deployment/web --timeout=180s
kubectl rollout status deployment/experiment --timeout=180s
kubectl create configmap experiment-config --from-file=nginx.conf --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f experiment-ambassador-pod.yaml
```

### Проверка

```bash
kubectl get pods
kubectl get services
kubectl port-forward pod/experiment-example 8082:80
```

HTTP-запрос из отдельного терминала:

```bash
curl http://localhost:8082/
```

Закрепление `ip_hash` означает, что запросы одного клиентского IP могут постоянно попадать в один backend. Отношение весов 9:1 само по себе не подтверждается серией обращений с одного IP.

## Исходные материалы

https://github.com/brendandburns/designing-distributed-systems/tree/master/ambassadors

## Результаты выполнения

Развёрнуты три экземпляра Redis, headless Service и Pod `ambassador-example` с Twemproxy. Команды `SET` и `GET` через порт 6380 выполнены успешно. Дополнительно созданы сервисы `web` и `experiment`, Nginx Ambassador запущен, проверка `nginx -t` прошла успешно. HTTP-запрос через Ambassador вернул `PRODUCTION`. Настройка весов 9:1 выполнена; статистическое распределение запросов между различными клиентскими IP отдельно не измерялось.

Подтверждающие материалы: [запуск Redis](screenshots/02-redis-replicas.png), [Twemproxy](screenshots/03-ambassador-ready.png), [SET/GET](screenshots/04-twemproxy-set-get.png), [backend-сервисы](screenshots/05-experiment-backends.png), [Nginx Ambassador](screenshots/06-experiment-ambassador.png).
