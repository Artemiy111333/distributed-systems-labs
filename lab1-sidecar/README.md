# Лабораторная работа №1. Паттерн Sidecar

## Цель работы

Исследование совместного использования пространства идентификаторов процессов несколькими контейнерами Docker на примере сервиса Topz.

## Архитектура

Основной контейнер `topz-target` выполняет команду `tail -f /dev/null`. Дополнительный контейнер `topz-sidecar` запускает HTTP-сервис Topz и присоединяется к PID-пространству основного контейнера параметром `--pid=container`. Информация о процессах предоставляется по адресу `/topz` на порту 8080.

Исходный код Topz: https://github.com/brendandburns/topz

## Сборка

```bash
git clone https://github.com/brendandburns/topz.git
cd topz
go mod download
go build -o main ./cmd/server
docker build -t topz-app .
```

Dockerfile содержит базовый образ `debian:12-slim` и копирование скомпилированного исполняемого файла в `/server`.

## Запуск контейнеров

```bash
docker run -d --name topz-target topz-app tail -f /dev/null
APP_ID=$(docker inspect -f '{{.Id}}' topz-target)
docker pull brendanburns/topz:db0fa58
docker run -d --name topz-sidecar --pid=container:${APP_ID} -p 8080:8080 brendanburns/topz:db0fa58 /server -addr 0.0.0.0:8080
```

## Проверка

```bash
docker ps
curl http://localhost:8080/topz
```

## Результат

В выводе `/topz` отображены процессы `tail -f /dev/null` и `/server -addr 0.0.0.0:8080`. Таким образом, вспомогательный контейнер получил доступ к сведениям о процессах контейнера приложения через общее пространство PID.
