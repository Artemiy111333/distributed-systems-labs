# Лабораторная работа №4. Реплицированные сервисы с распределением нагрузки

## Цель работы

Развёртывание реплицированного HTTP-сервиса в Kubernetes с балансировкой запросов, промежуточным кэширующим слоем Varnish и отдельным TLS-шлюзом Nginx.

## Архитектура

Сервис состоит из трёх слоёв:

1. `dictionary-server`: три реплики Nginx, доступные через Kubernetes Service `dictionary-server-service`.
2. `varnish-cache`: две реплики Varnish с backend `dictionary-server-service:80`, доступные через `varnish-service`.
3. `nginx-ssl`: четыре реплики Nginx, принимающие HTTPS на порту 443 и передающие запросы в `varnish-service`.

## Часть I. Реплицированный HTTP-сервис

```bash
kind create cluster --name replica --wait 120s
kubectl config use-context kind-replica
docker pull --platform linux/amd64 nginx:stable
docker pull --platform linux/amd64 varnish:stable
docker image save --platform linux/amd64 --output /tmp/replica-images.tar nginx:stable varnish:stable
kind load image-archive /tmp/replica-images.tar --name replica
kubectl apply -f dictionary-deploy.yaml
kubectl apply -f dictionary-service.yaml
kubectl rollout status deployment/dictionary-server --timeout=180s
kubectl get pods
kubectl get services
```

Проброс порта сервиса:

```bash
kubectl port-forward svc/dictionary-server-service 8081:80
```

Проверка HTTP из другого терминала:

```bash
curl http://localhost:8081/
```

## Часть II. Кэширующий слой Varnish

```bash
kubectl create configmap varnish-config --from-file=default.vcl --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f varnish-deploy.yaml
kubectl apply -f varnish-service.yaml
kubectl rollout status deployment/varnish-cache --timeout=180s
kubectl get pods
kubectl port-forward svc/varnish-service 8080:80
```

Проверка HTTP в отдельном терминале:

```bash
curl -I http://localhost:8080/
curl http://localhost:8080/
```

Для каждой реплики Varnish в манифесте установлено требование `memory: 2Gi`, а размер кэша задан параметром `malloc,2G`.

## Часть III. TLS-шлюз Nginx

Для локальной проверки применяется самоподписанный сертификат с именем `localhost`. В production-среде такой сертификат не используется.

```bash
openssl req -x509 -nodes -newkey rsa:2048 -days 365 -keyout server.key -out server.crt -subj '/CN=localhost' -addext 'subjectAltName=DNS:localhost'
kubectl create secret tls ssl --cert=server.crt --key=server.key
kubectl create configmap nginx-conf --from-file=nginx.conf --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f nginx-deploy.yaml
kubectl apply -f nginx-service.yaml
kubectl rollout status deployment/nginx-ssl --timeout=180s
kubectl get pods
kubectl get services
kubectl port-forward svc/nginx-service 8443:443
```

Проверка HTTPS из отдельного терминала:

```bash
curl -k https://localhost:8443/
curl -k -I https://localhost:8443/
```

Сервис `nginx-service` имеет тип `LoadBalancer`. В локальном кластере KIND без отдельной реализации внешнего балансировщика значение `EXTERNAL-IP` может оставаться `pending`. Для обращения к сервису используется `kubectl port-forward`.

## Результаты выполнения

Работают три Pod `dictionary-server`, два Pod `varnish-cache` и четыре Pod `nginx-ssl` — все в состоянии `1/1 Running`. HTTP-запрос через Varnish возвращает `200 OK` и заголовок `Via: 1.1 varnish`. Проверка конфигурации SSL-шлюза (`nginx -t`) прошла успешно; HTTPS-запрос через порт 8443 возвращает `HTTP/1.1 200 OK` и страницу Nginx.

Подтверждающие материалы: [реплики Nginx](screenshots/01-nginx-replicas.png), [слой Varnish](screenshots/02-varnish-cache.png), [HTTPS-шлюз](screenshots/03-nginx-https.png).
