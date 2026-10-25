# Лабораторная №4 — Replicated Service

Три этапа методички: 3 nginx-replica → Service, 2 Varnish-cache → Service, 4 nginx с TLS → Service.

## I. Приложение и Service

```bash
# Сделайте скриншоты лабораторной №3, затем удалите кластер adapter
kind delete cluster --name adapter
kind create cluster --name replica --wait 120s
kubectl config use-context kind-replica
docker pull nginx:stable
docker pull varnish:stable
kind load docker-image nginx:stable varnish:stable --name replica
kubectl apply -f dictionary-deploy.yaml
kubectl apply -f dictionary-service.yaml
kubectl rollout status deployment/dictionary-server --timeout=180s
kubectl get pods
kubectl get svc
# Сохраните 01-replicas.png и 02-services.png
kubectl port-forward svc/dictionary-server-service 8081:80
```

В другом терминале: `curl http://localhost:8081/` (страница nginx). Остановите `port-forward`.

## II. Varnish

```bash
kubectl create configmap varnish-config --from-file=default.vcl --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f varnish-deploy.yaml
kubectl apply -f varnish-service.yaml
kubectl rollout status deployment/varnish-cache --timeout=180s
kubectl get pods
kubectl port-forward svc/varnish-service 8080:80
```

В другом терминале: `curl -I http://localhost:8080/` и `curl http://localhost:8080/`. Сделайте `03-varnish.png`.

Если `Pending` из-за нехватки памяти, уменьшите **оба** значения в `varnish-deploy.yaml` с `2Gi`/`malloc,2G` до `256Mi`/`malloc,256M`; это отступление от примера методички необходимо отметить в отчёте.

## III. HTTPS

```bash
# Убедитесь, что предыдущий port-forward закрыт
openssl req -x509 -nodes -newkey rsa:2048 -days 365 -keyout server.key -out server.crt -subj '/CN=localhost' -addext 'subjectAltName=DNS:localhost'
kubectl create secret tls ssl --cert=server.crt --key=server.key
kubectl create configmap nginx-conf --from-file=nginx.conf --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f nginx-deploy.yaml
kubectl apply -f nginx-service.yaml
kubectl rollout status deployment/nginx-ssl --timeout=180s
kubectl get pods
kubectl get svc
kubectl port-forward svc/nginx-service 8443:443
```

В другом терминале: `curl -k https://localhost:8443/` и `curl -k -I https://localhost:8443/`. Сделайте `04-https.png`.

Браузер может предупреждать о самоподписанном сертификате. В KIND для `type: LoadBalancer` обычно сохраняется `EXTERNAL-IP: <pending>` — для лабораторной используется port-forward.

**Важно:** `server.key` и `server.crt` не публиковать на GitHub. Из папки GitHub исключаются правилами `.gitignore`.
