# Лаб. П08 — Docker Compose: многоконтейнерные сервисы

Два compose-стека:

1. **`flask-redis/`** — веб-приложение Flask со счётчиком посещений в Redis.
2. **`monitoring/`** — Prometheus + Grafana + blackbox_exporter: мониторинг
   доступности Flask-приложения и внешних сайтов.

```
 user ──:8000──► Flask ──:6379──► Redis
                  ▲ http probe
 admin ─:3000─► Grafana ──► Prometheus(:9090) ──► Blackbox exporter(:9115)
```

## Содержимое

```
.
├── flask-redis/
│   ├── app.py, requirements.txt, dockerfile
│   └── compose.yaml                      # web (build) + redis:alpine
├── monitoring/
│   ├── compose.yaml                      # prometheus, grafana, blackbox
│   ├── prometheus/prometheus.yml         # self-scrape + blackbox-http job
│   └── grafana/
│       ├── provisioning/datasources/     # источник данных Prometheus
│       ├── provisioning/dashboards/      # провайдер дашбордов
│       └── dashboards/blackbox-13659.json  # дашборд 13659 (grafana.com), готовый
└── screenshots/
```

## Запуск (VM Ubuntu 24.04, vm1 = 192.168.227.15)

```bash
sudo apt install -y docker.io docker-compose-v2

# 1. Flask + Redis
cd flask-redis && docker compose up -d --build
curl 127.0.0.1:8000        # счётчик растёт на каждый запрос

# 2. Мониторинг
cd ../monitoring && docker compose up -d
```

| Сервис | Адрес |
|---|---|
| Flask | http://192.168.227.15:8000 |
| Prometheus | http://192.168.227.15:9090 |
| Grafana | http://192.168.227.15:3000 (`admin` / `grafana`) |
| Blackbox exporter | http://192.168.227.15:9115 |

## Нюансы

- В `prometheus.yml` Flask опрашивается по IP VM (`192.168.227.15:8000`);
  при другом адресе его нужно заменить.
- Дашборд 13659 подключён через provisioning (в JSON подставлен uid источника
  данных `prometheus`), поэтому вручную импортировать его не нужно.
- Данные Prometheus и Grafana хранятся в именованных volumes.

## Скриншоты

1. `screenshots/01-flask-browser.png` — Flask-приложение со счётчиком в браузере (`:8000`).
2. `screenshots/02-prometheus-targets.png` — Prometheus: все цели (`prometheus`, `blackbox-http`) в состоянии UP.
3. `screenshots/03-grafana-dashboard.png` — Grafana: дашборд 13659 (Blackbox Exporter, HTTP prober).
4. `screenshots/04-compose-ps.png` — `docker compose ps` обоих стеков и ответы Flask.
