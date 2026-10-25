# Лабораторные работы по распределённым системам

Репозиторий содержит конфигурации и команды для исследования контейнерных паттернов Sidecar, Ambassador, Adapter и Replicated Service.

## Программное окружение

- Ubuntu 24.04 LTS (VMware Workstation)
- Docker Engine и Go
- Kubernetes, kubectl, KIND
- Nginx, Redis, Twemproxy, Prometheus, Redis Exporter, Varnish

## Содержание

| № | Тема | Основные компоненты |
|---|---|---|
| [1](lab1-sidecar/) | Sidecar | Topz, Docker, пространство PID |
| [2](lab2-ambassador/) | Ambassador | Redis StatefulSet, Twemproxy, Nginx |
| [3](lab3-adapter/) | Adapter | Redis, Redis Exporter, Prometheus |
| [4](lab4-replicated-service/) | Replicated Service | Nginx, Kubernetes Service, Varnish, TLS |

Каждый каталог содержит исходные конфигурационные файлы, описание архитектуры и команды запуска и проверки. Материалы выполнения размещаются в каталогах `screenshots/`.

## Источники

- Brendan Burns, *Designing Distributed Systems*: https://github.com/brendandburns/designing-distributed-systems
- Проект Topz: https://github.com/brendandburns/topz
- Документация Kubernetes: https://kubernetes.io/docs/
- Документация Docker: https://docs.docker.com/

## Результаты выполнения

| Работа | Итог проверки | Материалы |
|---|---|---|
| №1 Sidecar | Два Docker-контейнера; `/topz` показывает процессы | [Скриншоты](lab1-sidecar/screenshots/) |
| №2 Ambassador | Три Redis, Twemproxy (`SET`/`GET`), HTTP-маршрутизация через Nginx | [Скриншоты](lab2-ambassador/screenshots/) |
| №3 Adapter | Метрика `redis_up 1`; оба Target Prometheus в состоянии `UP` | [Скриншоты](lab3-adapter/screenshots/) |
| №4 Replicated Service | Девять работающих Pod; Varnish и HTTPS возвращают `200 OK` | [Скриншоты](lab4-replicated-service/screenshots/) |
