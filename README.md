<picture><source media="(max-width: 600px)" srcset="assets/hero-m.svg"><img src="assets/hero.svg" width="100%" alt="Максим — backend и infrastructure инженер. Пишу бэкенд на Python. Настраиваю серверы так, чтобы сервисы не падали. В одиночку запустил и веду свою PaaS Причал — prichal.tech. Сейчас в коммерции веду команду ИИ-агентов."></picture>

<picture><source media="(max-width: 600px)" srcset="assets/numbers-m.svg"><img src="assets/numbers.svg" width="100%" alt="Коммерция: 9 микросервисов в проде. Причал: более 2 400 автотестов и около 150 API-эндпоинтов. Инфраструктура: 4 ноды Kubernetes в проде."></picture>

<a href="https://prichal.tech"><picture><source media="(max-width: 600px)" srcset="assets/flow-m.svg"><img src="assets/flow.svg" width="100%" alt="Причал — prichal.tech: git push → определение стека → сборка без привилегий → песочница gVisor в Kubernetes → сайт онлайн"></picture></a>

<a href="https://prichal.tech"><picture><source media="(max-width: 600px)" srcset="assets/facts-m.svg"><img src="assets/facts.svg" width="100%" alt="Чужой код — в песочнице gVisor. Сеть закрыта по умолчанию. Сборка без root."></picture></a>

<details>
<summary><b>Как это устроено внутри</b> — для тех, кто любит детали</summary>

<br>

**Пользовательский код исполняется в песочнице, а не на ядре ноды.**
Threat model — недоверенный код: пользователь не имеет доступа к манифестам и к
работающим контейнерам. Все нагрузки тенантов идут через **gVisor (`runsc`)**;
это не опция, а граница безопасности. PodSecurity — `baseline`, потому что при
gVisor-границе `restricted` ломает половину пользовательских образов, ничего
не добавляя к изоляции; capabilities сбрасываются точечно.

**Сеть — default-deny.**
Cilium, egress-контроль, сетевые политики между тенантами. Приёмка кластера
построена на **негативных проверках**: чеклист подтверждает не «оно работает»,
а «чужой код НЕ может выйти за границу» — и прогоняется после каждого изменения
CNI, gVisor или политик.

**Сборка образов без Docker-демона и без привилегий.**
Rootless BuildKit внутри gVisor, в отдельном неймспейсе с ResourceQuota.
Собранные образы уезжают в приватный registry неизменяемыми тегами — деплой
всегда воспроизводим, «тот же тег с другим содержимым» невозможен.

**Обновления платформы не ломают прод.**
Миграции БД — отдельный gated Job под advisory lock, а не гонка в entrypoint
нескольких реплик. Стратегия Recreate, неизменяемые теги образов, один скрипт
обновления.

**Ёмкость считается до планирования, а не после отказа.**
Preflight понодно и статус `pending_capacity` вместо ложного `failed`: когда
в кластере нет места, пользователь видит правду, а не «деплой сломался».

**Топология:** 4 ноды в VPC, один публичный IPv4 на NAT-шлюзе, отдельная нода под
БД с local NVMe и тейнтом, Longhorn под тома, CloudNativePG — Postgres на пользователя.
k3s с ручным hardening до дефолтов RKE2; миграция на RKE2 — по мере роста.

**Масштаб:** ~150 REST-эндпоинтов, ~39 000 строк Python, ~1 950 тестов на pytest и ~470 на фронтенде,
6 асинхронных воркеров поверх Redis-очереди с FIFO-гарантией.
Python / FastAPI · React 19 / TypeScript · Kubernetes.

> Код закрыт — платформа коммерческая. Готов провести по архитектуре и показать
> живой деплой на созвоне.

</details>

<picture><source media="(max-width: 600px)" srcset="assets/experience-m.svg"><img src="assets/experience.svg" width="100%" alt="Коммерческий опыт: сейчас — AI-инженерная система на ИИ-агентах; окт 2025 — мар 2026 — backend-разработчик, 9 микросервисов."></picture>

<details>
<summary><b>Что именно я сделал</b></summary>

<br>

Спроектировал и в одиночку реализовал распределённое ядро из **9 микросервисов**, связанных очередями RabbitMQ:
бот на aiogram, FastAPI и 6 асинхронных воркеров, оркестратор двухуровневых workflow
с политиками ошибок на каждом этапе. 22 модели SQLAlchemy, 65 миграций, 33 эндпоинта.
Выгрузка в InSales и WooCommerce.

Чинил корневые причины, а не симптомы:

- падавшие фоновые задачи — PgBouncer'ом и разделением async/sync-подключений, а не ретраями;
- зависшие консьюмеры RabbitMQ — разбором логики `ack`;
- гонки за общий файл сессии Telethon — распределённым локом на Redis.

`Python 3.13` `FastAPI` `RabbitMQ / aio-pika` `PostgreSQL + PgBouncer` `Redis`
`OpenAI API` `Prometheus / Loki` `Docker Compose (17 сервисов)`

</details>

<picture><source media="(max-width: 600px)" srcset="assets/projects-m.svg"><img src="assets/projects.svg" width="100%" alt="Другие проекты"></picture>

<a href="https://github.com/0xRaiseX/deploy-platform-showcase"><picture><source media="(max-width: 600px)" srcset="assets/p-deploy-platform-showcase-m.svg"><img src="assets/p-deploy-platform-showcase.svg" width="100%" alt="deploy-platform-showcase — Открытый срез кода Причала: как устроены сборка, деплой и живые логи."></picture></a>

<a href="https://github.com/0xRaiseX/global-rate-limiter"><picture><source media="(max-width: 600px)" srcset="assets/p-global-rate-limiter-m.svg"><img src="assets/p-global-rate-limiter.svg" width="100%" alt="global-rate-limiter — Ограничение запросов для всего трафика: Envoy, gRPC, Redis, аналитика в ClickHouse."></picture></a>

<a href="https://github.com/0xRaiseX/super-octo-bassoon"><picture><source media="(max-width: 600px)" srcset="assets/p-super-octo-bassoon-m.svg"><img src="assets/p-super-octo-bassoon.svg" width="100%" alt="super-octo-bassoon — Первый прототип платформы: деплой Docker-образов в Kubernetes одной кнопкой."></picture></a>

<a href="https://github.com/0xRaiseX/tender-tracker"><picture><source media="(max-width: 600px)" srcset="assets/p-tender-tracker-m.svg"><img src="assets/p-tender-tracker.svg" width="100%" alt="tender-tracker — Учёт тендеров с журналом статусов, который нельзя подделать на уровне базы."></picture></a>

<a href="https://github.com/0xRaiseX/album-catalog"><picture><source media="(max-width: 600px)" srcset="assets/p-album-catalog-m.svg"><img src="assets/p-album-catalog.svg" width="100%" alt="album-catalog — Каталог музыки на Django: исполнители, альбомы и треклисты."></picture></a>

<a href="https://github.com/0xRaiseX/prichal-sample"><picture><source media="(max-width: 600px)" srcset="assets/p-prichal-sample-m.svg"><img src="assets/p-prichal-sample.svg" width="100%" alt="prichal-sample — Приложение-образец: нажал «Deploy» — через минуту оно в сети."></picture></a>

<a href="https://github.com/0xRaiseX/simple-app"><picture><source media="(max-width: 600px)" srcset="assets/p-simple-app-m.svg"><img src="assets/p-simple-app.svg" width="100%" alt="simple-app — REST API с Docker, CI и развёртыванием через Ansible."></picture></a>

<a href="https://github.com/0xRaiseX/ecdsa"><picture><source media="(max-width: 600px)" srcset="assets/p-ecdsa-m.svg"><img src="assets/p-ecdsa.svg" width="100%" alt="ecdsa — Цифровые подписи на кривой secp256k1, написано на Rust."></picture></a>

<a href="https://github.com/0xRaiseX/avtokran-pskov"><picture><source media="(max-width: 600px)" srcset="assets/p-avtokran-pskov-m.svg"><img src="assets/p-avtokran-pskov.svg" width="100%" alt="avtokran-pskov — Лендинг услуги аренды автокрана."></picture></a>

<details>
<summary><b>Ранние проекты</b> — с них всё началось</summary>

<br>

- **[oxi-core](https://github.com/0xRaiseX/oxi-core)** — ядро автоторговли на криптобирже: данные по WebSocket, ордера в реальном времени.
- **[api-manager-exchanges](https://github.com/0xRaiseX/api-manager-exchanges)** — арбитраж ставок финансирования между криптобиржами.
- **[discord-economy-bot](https://github.com/0xRaiseX/discord-economy-bot)** — Discord-бот с экономикой и магазином ролей.

</details>

<picture><source media="(max-width: 600px)" srcset="assets/stack-m.svg"><img src="assets/stack.svg" width="100%" alt="Стек: Python, FastAPI, Django, Rust, Kubernetes, Cilium, gVisor, PostgreSQL, Redis, RabbitMQ, ClickHouse, ИИ-агенты, LLM"></picture>

<a href="https://t.me/raise0x"><picture><source media="(max-width: 600px)" srcset="assets/contact-m.svg"><img src="assets/contact.svg" width="100%" alt="Написать в Telegram: @raise0x"></picture></a>

<p align="center"><sub><a href="https://prichal.tech">prichal.tech</a> · <a href="https://t.me/raise0x">t.me/raise0x</a> · maks.demkin87@gmail.com</sub></p>
