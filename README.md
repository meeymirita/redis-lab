# Redis Lab — кэш, локи, rate limit, Streams

![Redis](redis.png)

**Статус: ⚪ методичка готова, прохождение впереди.**
**Сложность: средняя.** Нужен тот же бэкграунд, что и для RabbitMQ-лабы (Laravel, Docker), домен заказов переиспользуется. Рекомендуется после RabbitMQ Lab: методичка постоянно сравнивает Streams с брокером.

## О чём

Redis как кэш, хранилище сессий, примитив синхронизации и брокер событий — одновременно, на кусочке той же системы заказов. Лаба специально показывает, где каждая из этих ролей "подводит" (что будет при рестарте без AOF, при отвале Pub/Sub-подписчика, при гонке за один и тот же лок).

## Стек

Laravel 13 + PostgreSQL 17 + Redis 7.

## Формат

Методичка [`Redis_Lab_Plan.html`](Redis_Lab_Plan.html) — открывается в браузере, прогресс по чекбоксам сохраняется локально.

## Что внутри (3 сессии)

- **Сессия 1** — docker-compose и `redis.conf`, Laravel + `.env`, миграции; **Cache-Aside** для карточки товара (`ProductRepository`); сессии в Redis (`SESSION_DRIVER=redis`); `StreamPublisher` — первый producer в Redis Streams; первый consumer (happy path)
- **Сессия 2** — **атомарный Lua-скрипт** в `StockReservationService` (проверка остатка и списание одной командой), чтобы не продать один товар дважды; **rate limiter** (sliding window); competing consumers + нагрузочный тест; crash-тест на **PEL** (Pending Entries List) и идемпотентность
- **Сессия 3** — retry через `XAUTOCLAIM`; ручной DLQ-поток; приоритет очереди через `ZSET`; Pub/Sub-дашборд в реальном времени; "Production Hell" — комплексный сценарий без подсказок

Логика подачи материала зеркалит RabbitMQ-лабу (архитектура → сборка по шагам → "под капотом" → что почитать перед следующим шагом), но через призму структур данных Redis вместо AMQP.

---

Часть сборного репозитория лабораторных работ — [submodule-group-lab](https://github.com/meeymirita/submodule-group-lab).
