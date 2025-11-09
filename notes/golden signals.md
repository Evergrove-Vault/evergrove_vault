---
aliases:
  - 4 metrics
---
[[MOC DEVOps]]
[[MOC Observability]]
# Золотое правило (4 метрики = «Golden Signals»)

Коротко: в первую очередь **измеряйте четыре вещи** — **latency (задержки)**, **traffic (нагрузка/трафик)**, **errors (ошибки)** и **saturation (насыщение ресурсов)**. Это — проверенная практика (Google SRE / “Golden Signals”) и даёт максимальную отдачу при минимальном наборе метрик.

Ниже — подробное объяснение для каждой метрики + практические примеры (PromQL / alert rules / runbook-идеи). Значения/пороговые уровни зависят от приложения — я дам шаблоны и рекомендации, которые можно адаптировать.

---

# 1) Latency — задержки (p50/p95/p99)

**Что это:** время ответа на запросы (обычно измеряют пер-эндпойнт / per-route).  
**Почему важно:** пользователи замечают задержки раньше, чем падения доступности; рост p95/p99 обычно — первый сигнал проблемы.

**Что собирать:**

- Гистограммы (Histogram) или Summary для времени ответа (`http_request_duration_seconds_bucket`).
    
- Метки: `route`, `method`, `status`, `instance` — аккуратно с cardinality.
    

**Примеры PromQL:**

- p95 по всем запросам за 5m:
    

`histogram_quantile(0.95,   sum(rate(http_request_duration_seconds_bucket{job="myapp"}[5m])) by (le) )`

- среднее время (если нет гистограммы):
    

`rate(http_request_duration_seconds_sum{job="myapp"}[5m]) / rate(http_request_duration_seconds_count{job="myapp"}[5m])`

**Пример alert rule (Prometheus):**

`- alert: HighLatencyP95   expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{job="myapp"}[5m])) by (le)) > 1   for: 5m   labels:     severity: page   annotations:     summary: "p95 latency > 1s for myapp"     description: "p95 latency = {{ $value }}s for 5m. Check slow handlers, DB, external calls."`

**Runbook (первые шаги):**

1. Посмотреть панели по p95/p99 по route.
    
2. Отфильтровать по недавно задеплоенным версиям/инстансам.
    
3. Проверить длительные внешние вызовы (DB, API), GC-паузы, CPU-сатурацию.
    

---

# 2) Traffic — трафик / throughput

**Что это:** количество запросов в секунду (QPS), объемы данных, транзакций.  
**Почему важно:** помогает интерпретировать ошибки/латентность (высокий трафик может объяснить рост задержек), и служит входом в автоскейлинг/планирование.

**Что собирать:**

- Счётчики запросов (`http_requests_total`) по `route`, `status`, `instance`.
    
- Также полезно: конверсии/бизнес-метрики (orders/sec), размер payloads.
    

**Примеры PromQL:**

- QPS (1m):
    

`sum(rate(http_requests_total{job="myapp"}[1m]))`

- QPS по route:
    

`sum(rate(http_requests_total{job="myapp"}[route="/api/pay"] [1m])) by (route)`

**Пример alert (внезапный всплеск):**

`- alert: TrafficSpike   expr: sum(rate(http_requests_total{job="myapp"}[1m])) > 3 * sum(rate(http_requests_total{job="myapp"}[1h])) # грубая аномалия   for: 2m   labels: { severity: page }   annotations:     summary: "Traffic spike detected"`

**Runbook:**

1. Сравнить со стандартным дневным/недельным профилем.
    
2. Проверить кампании/cron jobs/новую маркетинговую активность.
    
3. Масштабировать frontend/backend, включить rate-limiting, откатить изменения если нужно.
    

---

# 3) Errors — ошибки (rate)

**Что это:** процент или абсолютное число ошибочных ответов (5xx/4xx), исключений в приложении.  
**Почему важно:** высокий процент ошибок = прямая деградация функциональности.

**Что собирать:**

- Счётчики ошибок (`http_requests_total{status=~"5.."}`), логируемые исключения, бизнес-LOST событий.
    
- Разделять 4xx и 5xx (4xx часто — клиентская проблема; 5xx — серверная).
    

**Примеры PromQL:**

- Доля 5xx за 5m:
    

`sum(rate(http_requests_total{job="myapp", status=~"5.."}[5m])) / sum(rate(http_requests_total{job="myapp"}[5m]))`

- Альтернатива (в процентах):
    

`100 * ( sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) )`

**Пример alert:**

`- alert: HighErrorRate   expr: ( sum(rate(http_requests_total{job="myapp", status=~"5.."}[5m])) / sum(rate(http_requests_total{job="myapp"}[5m])) ) > 0.01   for: 5m   labels: { severity: page }   annotations:     summary: "Error rate > 1% for myapp"`

**Runbook:**

1. Посмотреть какие endpoints/версия дают ошибки.
    
2. Проверить логи (traceId), недавние деплои.
    
3. Если ошибки связаны с внешними сервисами — проверить их статус/таймауты.
    
4. По необходимости — включить circuit breaker или откат.
    

---

# 4) Saturation — насыщение ресурсов

**Что это:** насколько «заполнен» ресурс — CPU, RAM, диск, connection pools, file descriptors, очередь задач.  
**Почему важно:** saturation приводит к росту latency и ошибок — это ресурсный лимит.

**Что собирать:**

- CPU% (container/node), memory used / limit, disk usage %, connection-pool usage, thread count, queue depth.
    

**Примеры метрик/PromQL (шаблоны):**

- CPU (container):
    

`avg(rate(container_cpu_usage_seconds_total{job="kubelet", pod=~"myapp-.*"}[5m])) by (pod) * 100`

- Memory % (container):
    

`sum(container_memory_usage_bytes{pod=~"myapp-.*"}) by (pod) / sum(container_spec_memory_limit_bytes{pod=~"myapp-.*"}) by (pod) * 100`

- Connection pool saturation (пример):
    

`sum(app_db_pool_in_use{app="myapp"}) / sum(app_db_pool_size{app="myapp"})`

**Пример alert (память):**

`- alert: HighMemorySaturation   expr: sum(container_memory_usage_bytes{pod=~"myapp-.*"}) by (pod) / sum(container_spec_memory_limit_bytes{pod=~"myapp-.*"}) by (pod) > 0.85   for: 5m   labels: { severity: page }   annotations:     summary: "Memory > 85% on some pods"`

**Runbook:**

1. Посмотреть какой компонент/контейнер перегружен.
    
2. Проверить OOM, GC-логи, утечки памяти.
    
3. Увеличить ресурсы / горизонтально масштабировать / оптимизировать запросы / починить утечку.
    

---

# SLO / SLI / Alerting — как связывать метрики с операциями

- **SLI (Service Level Indicator)** — конкретная метрика (напр., p95 < 300 ms, availability = 99.9%).
    
- **SLO (Service Level Objective)** — целевой уровень SLI (напр., 99.9% за 30 дней).
    
- **Error budget** = 1 − SLO. Если бюджет исчерпан → строгие меры (откат, ограничение новых фич).
    

**Пример SLI/SLO:**

- SLI: fraction of successful requests with p95 < 500ms.
    
- SLO: 99.9% таких запросов за 30d.
    

**Алёртинг-стратегия (рекомендация):**

- Page (пейджить) — критичные события: объем 5xx, p95 сильно > SLO, saturation, потеря error budget.
    
- Ticket (ops) — тренды (p95 растёт, но ещё в пределах error budget).
    
- Observe-only — мелкие аномалии/информативные.

---
# Полезные советы / антипаттерны

- Не делать метрики по  всему подряд: **избегайте высокой cardinality** (не ставьте user_id как метку на каждую метрику).
    
- Используйте **гистограммы** для latency — их легче агрегировать и получать p95/p99.
    
- Разделяйте **инфраструктурные** и **бизнес** метрики (оба важны).
    
- Настраивайте **tuning** алертов: слишком чувствительные → шум, неподходящие → пропустят инцидент.
    
- Вводите **SLO-ориентированное алертирование** (alert на исчерпание error budget важнее, чем на единичные мелкие всплески).