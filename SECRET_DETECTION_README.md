# Интеграция обнаружения секретов в CI/CD

## Описание

В проект интегрирован **Gitleaks** — инструмент обнаружения секретов (пароли, API-ключи, токены и т.п.) в коде и истории Git.

**Порядок выполнения pipeline:**
1. **Секреты** — сканирование запускается первым
2. **Сборка** — выполняется только при отсутствии секретов
3. При обнаружении секретов pipeline останавливается, сборка не выполняется

---

## GitLab CI

Файл: `.gitlab-ci.yml`

**Этапы:**
- `security` — Gitleaks сканирует весь репозиторий и полную историю (`GIT_DEPTH: 0`)
- `build` — Maven-сборка (запускается только после успешного security)

**Артефакты при ошибке:** `gitleaks-report.json`

---

## GitHub Actions

Файл: `.github/workflows/ci.yml`

**Jobs:**
- `secret-detection` — первым запускается Gitleaks
- `build` — сборка выполняется после `secret-detection` (`needs: secret-detection`)

---

## Конфигурация Gitleaks

Файл `.gitleaks.toml` добавляет правила для Java:
- пароли в properties/конфигах;
- AWS Access Keys;
- приватные ключи (RSA, EC и т.д.).

---

## Использование

### GitLab
Скопируйте `.gitlab-ci.yml` и `.gitleaks.toml` в корень репозитория GitLab. Job `secret-detection` выполнится при каждом push/merge.

### GitHub
Скопируйте папку `.github/workflows/` и файл `.gitleaks.toml` в корень репозитория GitHub. Workflow запускается при push и pull request.

---

## Локальный запуск

```bash
# Сканирование текущей директории
gitleaks detect --source . --verbose

# Или через Docker
docker run --rm -v "${PWD}:/path" zricethezav/gitleaks:latest detect --source /path --no-banner
```
