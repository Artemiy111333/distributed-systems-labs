# Лабораторная №1 — Sidecar (Topz)

**Цель:** исследовать паттерн Sidecar, предоставляющий интерфейс `/topz` для просмотра процессов основного контейнера.

Исходный проект: https://github.com/brendandburns/topz. Использован исходный код автора, не копируем весь проект в этот репозиторий.

## Воспроизведение

```bash
# команды выполнять от обычного пользователя Ubuntu
cd ~/distributed-labs/lab1/topz
go mod download
go build -o main ./cmd/server
docker build -t topz-app .
docker run -d --name topz-target topz-app tail -f /dev/null
APP_ID=$(docker inspect -f '{{.Id}}' topz-target)
docker pull brendanburns/topz:db0fa58
docker run -d --name topz-sidecar --pid=container:${APP_ID} -p 8080:8080 brendanburns/topz:db0fa58 /server -addr 0.0.0.0:8080
docker ps
curl http://localhost:8080/topz
```

Если контейнеры уже созданы, повторно `docker run` запускать не нужно. Для повторной проверки остановленные контейнеры запускают через `docker start topz-target topz-sidecar` (основной — первым).

**Ожидаемый результат:** в `/topz` перечислены `tail -f /dev/null` и процесс `/server`. Фактический результат выполнения подтверждён скриншотом, но PNG ещё нужно скопировать в `screenshots/`.
