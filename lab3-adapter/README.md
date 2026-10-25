# Лабораторная работа №3. Паттерн Adapter

## Цель работы

Исследование экспорта метрик Redis в формате Prometheus с использованием дополнительного контейнера Redis Exporter.

## Архитектура

Pod `adapter-example` содержит Redis и Redis Exporter. Контейнеры Pod используют общее сетевое пространство, поэтому экспортер подключается к Redis по адресу `redis://localhost:6379` и публикует метрики на порту 9121. Prometheus обращается к сервису `adapter-example:9121`, а собственные метрики получает через `localhost:9090`.

## Развёртывание

```bash
kind create cluster --name adapter --wait 120s
kubectl config use-context kind-adapter
docker pull --platform linux/amd64 prom/prometheus:v3.8.0
docker pull --platform linux/amd64 redis:7.4-alpine
docker pull --platform linux/amd64 oliver006/redis_exporter:v1.76.0
docker image save --platform linux/amd64 --output /tmp/adapter-images.tar prom/prometheus:v3.8.0 redis:7.4-alpine oliver006/redis_exporter:v1.76.0
kind load image-archive /tmp/adapter-images.tar --name adapter
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus.yaml
kubectl apply -f pod-redis.yaml
kubectl apply -f service-redis.yaml
kubectl rollout status deployment/prometheus --timeout=180s
```

## Проверка Pod и Service

```bash
kubectl get pods
kubectl get services
kubectl logs adapter-example -c adapter
```

Метрики доступны при пробросе порта:

```bash
kubectl port-forward svc/adapter-example 9121:9121
```

Запрос из отдельного терминала:

```bash
curl -s http://localhost:9121/metrics | grep '^redis_up'
```

Значение `redis_up 1` означает успешное подключение экспортера к Redis.

## Проверка Prometheus

```bash
kubectl port-forward deployment/prometheus 9090:9090
```

Адрес интерфейса: `http://localhost:9090/targets`. В разделе `Status → Target health` контролируются две цели: `prometheus` и `redis`. Состояние `UP` означает успешный опрос соответствующей цели.

## Используемые компоненты

- Prometheus: https://prometheus.io/
- Redis Exporter: https://github.com/oliver006/redis_exporter

## Результаты выполнения

Pod `adapter-example` запущен с двумя готовыми контейнерами (`2/2 Running`), Pod Prometheus — `1/1 Running`. Метрика `redis_up` равна `1`. В интерфейсе Prometheus обе цели `prometheus` и `redis` имеют состояние `UP`.

Подтверждающие материалы: [состояние Pod и Service](screenshots/01-pods-services.png), [метрики Redis](screenshots/02-redis-metrics.png), [Prometheus Target health](screenshots/03-prometheus-targets.png).
