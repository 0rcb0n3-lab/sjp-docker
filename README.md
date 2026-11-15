# sjp-docker

Готовые Docker-образы для [StaticJinjaPlus](https://github.com/MrDave/StaticJinjaPlus) — генератора статических сайтов на Jinja2.

Здесь лежат только сборочные файлы. Код приложения не копируется в репозиторий: при сборке нужный релиз или коммит подтягивается прямо с GitHub, распаковывается в образ и сразу готов к запуску.

## Доступные образы

| Файл | Тег | Версия приложения | База | Проверка хеша |
|------|-----|-------------------|------|---------------|
| `Dockerfile.ver010` | `0.1.0` | `0.1.0` | `ubuntu:26.04` | да |
| `Dockerfile.ver011` | `0.1.1` | `0.1.1` | `ubuntu:26.04` | да |
| `Dockerfile.slim010` | `0.1.0-slim` | `0.1.0` | `python:3.14.10-slim-bookworm` | да |
| `Dockerfile.slim011` | `0.1.1-slim` | `0.1.1` | `python:3.14.10-slim-bookworm` | да |
| `Dockerfile.dev` | `develop` | ветка `main` | `ubuntu:26.04` | нет |
| `Dockerfile.dev-slim` | `develop-slim` | ветка `main` | `python:3.14.10-slim-bookworm` | нет |
| `Dockerfile.ubuntu` | `latest` | задаётся через `APP_VERSION` | `ubuntu:26.04` | нет |
| `Dockerfile.slim` | `slim` | задаётся через `APP_VERSION` | `python:3.14.10-slim-bookworm` | нет |

Разница между вариантами:

- **`ver0xx` / `slim0xx`** — фиксированные релизы. Скачиваемый архив сверяется с контрольной суммой, поэтому сборка воспроизводима.
- **`dev` / `dev-slim`** — теги `develop` / `develop-slim`. Всегда свежий `main`. Хеш не проверяется: содержимое ветки меняется с каждым коммитом.
- **`ubuntu` / `slim`** — теги `latest` / `slim`. Универсальные: версия приложения и база переопределяются аргументами сборки, так что один файл закрывает любые релизы. Под тегами `latest` / `slim` собирают последний релиз — версию указывают вручную через `--build-arg`.

## Сборка

### Универсальные образы (`latest` / `slim`)

Версия по умолчанию — `main`. Чтобы собрать последний релиз под тегами `latest` / `slim`, передайте нужную версию через `APP_VERSION`:

```commandline
docker build -f Dockerfile.ubuntu --build-arg APP_VERSION=0.1.1 -t sjp:latest .
docker build -f Dockerfile.slim   --build-arg APP_VERSION=0.1.1 -t sjp:slim .
```

Базовый образ тоже переопределяется через `BASE_IMAGE` — полезно, если нужна другая версия дистрибутива:

```commandline
docker build -f Dockerfile.ubuntu --build-arg BASE_IMAGE=ubuntu:22.04 -t sjp:ubuntu22 .
docker build -f Dockerfile.slim   --build-arg BASE_IMAGE=python:3.11-slim-bookworm -t sjp:slim311 .
```

Оба аргумента можно совмещать:

```commandline
docker build -f Dockerfile.ubuntu \
  --build-arg BASE_IMAGE=ubuntu:22.04 \
  --build-arg APP_VERSION=0.1.1 \
  -t sjp:0.1.1-ubuntu22 .
```

### Фиксированные релизы

```commandline
docker build -f Dockerfile.ver010  -t sjp:0.1.0 .
docker build -f Dockerfile.slim010 -t sjp:0.1.0-slim .
docker build -f Dockerfile.ver011  -t sjp:0.1.1 .
docker build -f Dockerfile.slim011 -t sjp:0.1.1-slim .
```

### Последний main (`develop` / `develop-slim`)

```commandline
docker build -f Dockerfile.dev      -t sjp:develop .
docker build -f Dockerfile.dev-slim -t sjp:develop-slim .
```

## Запуск

Образ запускает рендер шаблонов и завершает работу:

```commandline
docker run --rm sjp:<тег>
```

Свои шаблоны удобно прокидывать томом. По умолчанию приложение ищет их в `./templates`, а результат кладёт в `./build`:

```commandline
docker run --rm -v "$PWD/templates:/app/templates" -v "$PWD/build:/app/build" sjp:<тег>
```

## Переменные контекста

Любая переменная окружения с префиксом `SJP_` становится доступной в шаблонах. Например, `SJP_SITE_NAME=demo` даст в шаблоне `{{ site_name }}`:

```commandline
docker run --rm -e SJP_SITE_NAME=demo sjp:<тег>
```
