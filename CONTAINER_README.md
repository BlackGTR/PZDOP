# Интеграция проверки безопасности контейнеров

## Описание

В pipeline добавлена сборка Docker-образа и его **проверка безопасности до публикации** в реестр. Используется **Trivy**.

**Порядок выполнения:**
1. Сборка Docker-образа
2. Сканирование Trivy (уязвимости HIGH, CRITICAL)
3. **Публикация в реестр только при отсутствии уязвимостей** — при их обнаружении pipeline падает, образ не публикуется

---

## GitLab

**Этап:** `container`

- **Registry:** GitLab Container Registry (`registry.gitlab.com/<namespace>/<project>`)
- **Подключение:** автоматически через `CI_REGISTRY_*`
- **Теги:** `$CI_COMMIT_SHA`, `latest`

Образ: `$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA`

---

## GitHub

**Job:** `container` (Build, Scan & Push Image)

- **Registry:** GitHub Container Registry (`ghcr.io/<owner>/<repo>`)
- **Подключение:** через `GITHUB_TOKEN` (разрешение `packages: write`)
- **Теги:** SHA коммита, `latest`

---

## Файлы

- `Dockerfile` — мультистадийная сборка Java (Maven → JRE)
- `src/main/java/ru/pzdop/Application.java` — минимальный Main

---

## Trivy

- **Уровни:** CRITICAL, HIGH
- **`--ignore-unfixed`:** только уязвимости с доступным исправлением
- **`--exit-code 1`:** при обнаружении уязвимостей pipeline завершается с ошибкой

---

## Если pipeline падает на Trivy

1. Обновить базовый образ в Dockerfile
2. Перейти на более новый дистрибутив (например, Alpine → slim)
3. Добавить в `.trivyignore` (временно) для конкретных CVE, если это обосновано
