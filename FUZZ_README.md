# Интеграция фаззинг-тестирования в CI/CD

## Описание

В pipeline интегрирован **ffuf** (Fuzz Faster U Fool) — инструмент фаззинг-тестирования веб-приложений. Выполняется после сборки, по аналогии с DAST.

**Что делает ffuf:**
- Сканирует пути (directories, endpoints) по wordlist
- Находит скрытые страницы, API, панели администрирования
- Обнаруживает уязвимые или незадокументированные маршруты

---

## GitLab CI

Этап `fuzzing` добавлен после `build`.

**Настройка:**
1. Settings → CI/CD → Variables
2. Добавить переменную `FUZZ_TARGET_URL` (например: `https://staging.example.com`)
3. Рекомендуется: пометить как **Protected** и **Masked**, если URL содержит чувствительные данные

**Без URL:** этап выполняется, но пропускает сканирование и завершается успешно.

**Артефакты:** `fuzz-report.json` — список найденных endpoints.

---

## GitHub Actions

Job `fuzzing` добавлен после `secret-detection`.

**Настройка:**
1. Settings → Secrets and variables → Actions
2. Вкладка **Variables** → New repository variable
3. Имя: `FUZZ_TARGET_URL`, значение: URL развёрнутого приложения (например: `https://staging.example.com`)

**Без URL:** job выполняется, сканирование пропускается.

**Артефакты:** `fuzz-report` — содержит `fuzz-report.json`.

---

## Wordlist

Файл `fuzz-wordlist.txt` — минимальный набор типичных путей для Java/Spring и веб-приложений:
`admin`, `api`, `login`, `health`, `actuator`, `swagger` и т.п.

Можно дополнить своими путями или подключить [SecLists](https://github.com/danielmiessler/SecLists).

---

## Важно

- Запускайте фаззинг только по тестовым/staging окружениям
- Не направляйте фаззинг на production — возможны лишняя нагрузка и срабатывание защиты
- Ограничение по времени: 120 секунд (`-maxtime 120`)
