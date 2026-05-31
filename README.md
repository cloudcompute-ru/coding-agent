# coding-agent — свой ИИ-агент для кода на своём GPU

Self-hosted OpenAI-совместимый API на собственной (или арендованной) видеокарте для агентного программирования. Подключается в наше расширение для VS Code, а также в любой другой клиент, говорящий по протоколу OpenAI — **[Cline](https://github.com/cline/cline)**, **[Continue.dev](https://continue.dev)**, **[Aider](https://aider.chat)**, OpenAI SDK, raw `curl` и т.д.

Запускает **[Qwen2.5-Coder-32B Instruct](https://huggingface.co/Qwen/Qwen2.5-Coder-32B-Instruct-AWQ)** (AWQ INT4, ~22 ГБ VRAM) на **[vLLM](https://github.com/vllm-project/vllm)** и публикует через бесплатный **Cloudflare quick tunnel**, который даёт публичный HTTPS-URL без настройки DNS и сертификатов.

Полезно когда:

- работаете с кодом, который не хочется отправлять в коммерческие сервисы;
- хотите попробовать open-source модели в реальном агентном рабочем процессе;
- ищете альтернативу с оплатой только за фактическую работу видеокарты, которую можно расшарить на всю команду через один эндпоинт.

## Что нужно

Linux с одной NVIDIA GPU от **24 ГБ VRAM** (RTX 4090, RTX A6000, A100 80 ГБ, H100 — все подходят) и актуальный CUDA-драйвер. Qwen2.5-Coder-32B AWQ умещается в 24 ГБ с запасом на KV-кеш; для меньших карт (12–16 ГБ) переключите `MODEL_ID` на `Qwen/Qwen2.5-Coder-7B-Instruct-AWQ`.

## Запуск

```bash
git clone https://github.com/cloudcompute-ru/coding-agent.git
cd coding-agent
bash provision.sh
```

Через 5–10 минут скрипт выведет подключение:

```
[cc-provision] provisioning complete
[cc-provision]   tunnel: https://three-random-words.trycloudflare.com
[cc-provision]   api key: sk-... (owned by the customer app)
[cc-provision]   model name: qwen2.5-coder-32b
```

Вставьте их в настройки кастомной модели вашего клиента:

| Поле        | Что вставить                                                  |
| ----------- | ------------------------------------------------------------- |
| Base URL    | `<tunnel URL>/v1` (например `https://...trycloudflare.com/v1`) |
| API Key     | строка `sk-...`                                               |
| Model       | `qwen2.5-coder-32b`                                            |

URL туннеля меняется при каждом перезапуске контейнера — особенность бесплатного quick tunnel. Для стабильного URL поднимите named-tunnel через свой Cloudflare-аккаунт, либо используйте one-click флоу на cloudcompute.ru (см. ниже).

### Управление API-ключом

При запуске через cloudcompute.ru API-ключ генерирует **сам сервис** (приложение) и передаёт его скрипту через переменную `CC_VLLM_API_KEY` — ключ виден в панели управления сразу, ещё до старта vLLM. При ручном запуске (без приложения) скрипт сам сгенерирует одноразовый ключ вида `sk-cc-<hex>`.

### Переменные окружения

| Переменная             | По умолчанию                          | Что задаёт                                                       |
| ---------------------- | ------------------------------------- | ---------------------------------------------------------------- |
| `CC_VLLM_API_KEY`      | (генерируется приложением / fallback) | API-ключ, который требует vLLM на каждом запросе.                |
| `CC_SERVED_MODEL_NAME` | `qwen2.5-coder-32b`                   | Имя модели, которое клиенты отправляют в `model:`.               |
| `MODEL_ID`             | `Qwen/Qwen2.5-Coder-32B-Instruct-AWQ` | HuggingFace ID модели для скачивания и сервинга.                 |
| `VLLM_PORT`            | `8000`                                | Внутренний порт vLLM. Туннель всегда указывает сюда.             |
| `WORKDIR`              | `/workspace`                          | Каталог для HF-кеша и runtime-состояния.                         |

## Что внутри

- `provision.sh` — ставит vLLM + cloudflared, скачивает Qwen2.5-Coder, поднимает OpenAI-совместимый сервер на `0.0.0.0:8000`, открывает Cloudflare-туннель и сообщает готовое подключение приложению.

## Про cloudcompute.ru

Этот репозиторий поддерживает [cloudcompute.ru](https://cloudcompute.ru) — российский GPU-хостинг с почасовой оплатой. Если не хочется самостоятельно арендовать видеокарту и поднимать контейнер, one-click флоу на cloudcompute.ru — это тот же скрипт, запущенный автоматически: подбор подходящей видеокарты, оплата по факту работы, готовая карточка с настройками подключения через 5–10 минут.

## Лицензии

Скрипты и конфигурация — MIT (см. `LICENSE`). Модель Qwen2.5-Coder-32B распространяется под **Qwen License** (Alibaba) — коммерческое использование разрешено до 100M MAU. vLLM — Apache 2.0, cloudflared — Apache 2.0. Этот репозиторий устанавливает их в runtime, но не модифицирует и не перераспространяет.
