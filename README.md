<p align="center">
  <img src="assets/hero.svg" width="100%" alt="Максим — Backend / Infra инженер. Пишу бэкенд на Python, строю свою PaaS на Kubernetes, коммерческий опыт: AI-инженерия и e-commerce.">
</p>

<p align="center">
  <a href="https://t.me/raise0x"><img src="https://img.shields.io/badge/Telegram-@raise0x-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white" alt="Telegram"></a>
  <a href="https://prichal.tech"><img src="https://img.shields.io/badge/Prichal-prichal.tech-0a0b0d?style=for-the-badge&labelColor=3fb950&logoColor=white" alt="prichal.tech"></a>
  <a href="mailto:maks.demkin87@gmail.com"><img src="https://img.shields.io/badge/Email-написать-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"></a>
  <img src="https://img.shields.io/badge/Открыт_к_работе-Backend_·_Platform_·_DevOps-3fb950?style=for-the-badge" alt="Открыт к работе">
</p>

<br>

<p align="center">
  <a href="https://prichal.tech"><img src="assets/prichal.svg" width="100%" alt="Prichal (prichal.tech): git push → сборка без root → registry → Kubernetes"></a>
</p>

**[Prichal](https://prichal.tech)** — PaaS для деплоя из GitHub в один клик, уже работает: **[prichal.tech](https://prichal.tech)**. Один разработчик, ~6 месяцев: бэкенд, фронт, инфраструктура, эксплуатация кластера.

<details>
<summary><b>Как это устроено внутри →</b></summary>

<br>

**Пользовательский код исполняется в песочнице, а не на ядре ноды.**
Threat model — недоверенный код: у пользователя нет доступа к манифестам и к работающим
контейнерам. Все нагрузки тенантов идут через **gVisor (`runsc`)** — это граница безопасности,
а не опция. PodSecurity — `baseline`: при gVisor-границе `restricted` ломает половину
пользовательских образов, ничего не добавляя к изоляции; capabilities сбрасываются точечно.

**Сеть — default-deny.**
Cilium, egress-контроль, сетевые политики между тенантами. Приёмка кластера построена на
**негативных проверках**: чеклист подтверждает не «оно работает», а «чужой код НЕ может
выйти за границу» — и прогоняется после каждого изменения CNI, gVisor или политик.

**Сборка образов без Docker-демона и без привилегий.**
Rootless BuildKit внутри gVisor, в отдельном неймспейсе с ResourceQuota.
Образы уезжают в приватный registry неизменяемыми тегами — деплой всегда воспроизводим.

**Обновления платформы не ломают прод.**
Миграции БД — отдельный gated Job под advisory lock, а не гонка в entrypoint нескольких
реплик. Стратегия Recreate, неизменяемые теги, один скрипт обновления.

**Ёмкость считается до планирования, а не после отказа.**
Preflight понодно и статус `pending_capacity` вместо ложного `failed`.

**Топология:** 4 ноды в VPC, один публичный IPv4 на NAT-шлюзе, отдельная нода под БД с
local NVMe и тейнтом, Longhorn под тома, CloudNativePG — Postgres на пользователя.
k3s с ручным hardening до дефолтов RKE2.

**В цифрах:** 75 REST-эндпоинтов · ~18 800 строк Python · 180 тестов на pytest ·
6 асинхронных воркеров поверх Redis-очереди с FIFO и честным распределением конкурентности.
Python / FastAPI · React 19 / TypeScript · Kubernetes.

> Код закрыт — платформа коммерческая. Готов провести по архитектуре и показать живой деплой на созвоне.

</details>

<br>

<p align="center">
  <img src="assets/ai.svg" width="100%" alt="AI-инженерия: я ставлю задачи и координирую, ИИ-агенты пишут код, тестируют и анализируют, на выходе — работающий продукт">
</p>

**AI-инженерия — моя текущая коммерческая работа.** Строю управляемую AI-инженерную систему: ИИ-агенты
программируют, тестируют и проводят технический анализ, а я ставлю им задачи, координирую их и отвечаю
за то, чтобы конечный продукт реально работал.

<br>

<p align="center">
  <img src="assets/work.svg" width="100%" alt="Коммерческий проект: Telegram-каналы → 9 сервисов на RabbitMQ + LLM → интернет-магазины">
</p>

**Контракт, NDA.** Один спроектировал и написал распределённую систему, которая заменила ручной труд менеджера.

<details>
<summary><b>Подробнее о проекте →</b></summary>

<br>

Система забирает товарные посты из Telegram-каналов поставщиков, обогащает данные через LLM
и выгружает готовые карточки в магазины на InSales и WooCommerce.

Ядро — **9 сервисов на RabbitMQ**: бот на aiogram, FastAPI и 6 асинхронных воркеров,
оркестратор двухуровневых workflow с политиками ошибок на каждом этапе.
22 модели SQLAlchemy, 65 миграций, 33 эндпоинта.

Чем горжусь больше кода — чинил корневые причины:
- падавшие фоновые задачи — не ретраями, а PgBouncer'ом и разделением async/sync-подключений;
- зависшие консьюмеры RabbitMQ — разбором логики `ack`;
- гонки за общий файл сессии Telethon — распределённым локом на Redis.

`Python 3.13` `FastAPI` `RabbitMQ / aio-pika` `PostgreSQL + PgBouncer` `Redis`
`OpenAI API` `Prometheus / Loki` `Docker Compose (17 сервисов)`

</details>

---

### Ещё

- **[global-rate-limiter](https://github.com/0xraisex/global-rate-limiter)** — распределённый rate limiting: Envoy + gRPC token bucket на Redis, логи в ClickHouse, детектор аномалий.
- **[ecdsa](https://github.com/0xraisex/ecdsa)** — ECDSA secp256k1 на Rust, с нуля.

### Стек

`Python` `FastAPI` `async SQLAlchemy` `Rust` · `Kubernetes` `Cilium` `gVisor`
`Longhorn` `CloudNativePG` `BuildKit` · `PostgreSQL` `Redis` `RabbitMQ` `ClickHouse` ·
`Linux` `nftables` `cloud-init` `GitHub Actions`

---

<p align="center"><i>18 лет. Ищу команду, где инфраструктура — не накладные расходы, а часть продукта.</i></p>
