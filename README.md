# template-docker-mlflow-s3

Шаблон учебного проекта для трекинга экспериментов в MLflow с хранением артефактов в MinIO (S3-compatible).

## Что внутри

- `docker-compose.yml` — поднимает `Postgres + MLflow + MinIO`.
- `1_mlflow_catboost_tutorial.ipynb` — классическое ML (CatBoost) + логирование в MLflow.
- `2_mlflow_cnn_tutorial.ipynb` — простая CNN (PyTorch) + логирование.
- `3_mlflow_bert_tiny_tutorial.ipynb` — маленький BERT/DistilBERT сценарий + логирование.
- `administration.ipynb` — вспомогательные административные проверки.

## Требования

- Python `3.12+`
- [uv](https://docs.astral.sh/uv/)
- Docker + Docker Compose

## Быстрый старт

1. Установить Python-зависимости и поднять виртуальное окружение:

```bash
uv sync
```

2. Активировать окружение (опционально, если запускаете команды через `uv run`):

```bash
source .venv/bin/activate
```

3. Поднять инфраструктуру:

```bash
docker compose up -d --build
```

4. Проверить, что контейнеры запущены:

```bash
docker compose ps
```

## Доступы и URL

- MLflow UI: `http://localhost:5050`
- MinIO API: `http://localhost:9000`
- MinIO Console: `http://localhost:9001`
- Postgres: `localhost:5432`

Из `.env`:

- MinIO user: `admin`
- MinIO password: `password`
- Bucket: `mlflow-bucket`

## Запуск ноутбуков

Открыть Jupyter (например):

```bash
uv run jupyter lab
```

Затем запускать по порядку:

1. `1_mlflow_catboost_tutorial.ipynb`
2. `2_mlflow_cnn_tutorial.ipynb`
3. `3_mlflow_bert_tiny_tutorial.ipynb`

Во всех ноутбуках показывается полный цикл:

- обучение модели,
- логирование `params/metrics/artifacts` в MLflow,
- регистрация модели в Model Registry,
- назначение production alias `prd`,
- загрузка модели по `models:/<name>@prd`.

## Как проверить, что артефакты попали в MinIO

1. Откройте MLflow UI и найдите run.
2. В run проверьте `artifact_uri`.
3. В MinIO Console (`localhost:9001`) проверьте объекты в `mlflow-bucket`.

Для текущей конфигурации (`--serve-artifacts` + `--artifacts-destination`) новые артефакты должны попадать в S3-путь внутри бакета `mlflow-bucket`.

## Полезные команды

Остановить окружение:

```bash
docker compose down
```

Остановить и удалить тома:

```bash
docker compose down -v
```

Посмотреть логи MLflow:

```bash
docker compose logs -f mlflow-service
```

## Troubleshooting

### 1) `Could not connect to the endpoint URL ... minio:9000`

Причина: имя `minio` резолвится внутри docker-сети, но не на хосте.  
Решение: использовать проксирование артефактов через MLflow (`mlflow-artifacts:/`) и убедиться, что сервер запущен с `--serve-artifacts` и `--artifacts-destination`.

### 2) `experiment 0 ... deleted`

Причина: попытка писать в удаленный default experiment.  
Решение: запускать `mlflow.start_run(experiment_id=...)` с явным experiment id.

### 3) Ошибки загрузки tokenizer для BERT

Если `AutoTokenizer` падает, проверьте установку зависимостей:

```bash
uv sync
```

и перезапустите kernel в Jupyter.

### 4) Ошибки при первом запуске BERT/DistilBERT

Нужен доступ в интернет для загрузки модели с Hugging Face.

## Примечание по версиям MLflow

В `docker-compose.yml` используется образ MLflow `v3.12.0`. Следите, чтобы версия клиента `mlflow` в Python-окружении была совместима с сервером.
