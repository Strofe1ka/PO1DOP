# PO1 DOP — Интеграция обнаружения секретов в CI/CD

Лабораторная работа: внедрение Gitleaks в конвейеры GitLab и GitHub.

## Требования задания

- ✅ Инструмент поиска секретов внедрён в репозиторий
- ✅ Поиск секретов выполняется **ДО** сборки приложения
- ✅ Сборка **останавливается** при обнаружении секретов

## Используемый инструмент: Gitleaks

[Gitleaks](https://github.com/gitleaks/gitleaks) — open-source инструмент для обнаружения hardcoded секретов (пароли, API-ключи, токены) в коде. Интеграция через [gacts/gitleaks](https://github.com/gacts/gitleaks) (лицензия не требуется).

## Структура проекта

```
.
├── .gitlab-ci.yml          # Конфигурация GitLab CI
├── .github/workflows/
│   └── ci.yml             # Конфигурация GitHub Actions
├── .gitleaks.toml         # Конфигурация Gitleaks
├── pom.xml                # Maven
├── src/main/java/ru/po1dop/
│   └── App.java
└── README.md
```

## GitLab CI

Файл `.gitlab-ci.yml` настроен так:

1. **Этап `security`** — запускается первым, выполняет Gitleaks
2. **Этап `build`** — выполняется только после успешной проверки секретов
3. При обнаружении секретов `allow_failure: false` гарантирует падение пайплайна

### Использование

Скопируйте `.gitlab-ci.yml` в корень вашего репозитория GitLab. Адаптируйте job `build` под ваш проект (образ Docker, команды сборки).

## GitHub Actions

Файл `.github/workflows/ci.yml` настроен так:

1. **Job `detect-secrets`** — запускается первым, выполняет Gitleaks
2. **Job `build`** — имеет `needs: detect-secrets`, выполняется только после успешной проверки
3. При обнаружении секретов Gitleaks возвращает exit code 1 → workflow падает

### Использование

Скопируйте папку `.github/workflows/` в корень вашего репозитория GitHub. Адаптируйте job `build` под ваш проект.

## Локальная проверка

Перед push можно проверить репозиторий локально:

```bash
# Через Docker (Gitleaks)
docker run --rm -v "$(pwd):/path" zricethezav/gitleaks:latest detect --source /path --verbose
```

## Что обнаруживает Gitleaks

- API ключи (AWS, Google, Azure и др.)
- Пароли и токены в коде
- Приватные ключи SSH
- Секреты в переменных окружения

## Быстрый старт

```bash
# Локально
mvn clean package
java -jar target/secret-detection-demo-1.0.0.jar
```

## Как протестировать обнаружение секретов

1. **Сценарий «всё ок»**: сделайте push — пайплайн должен пройти (detect-secrets → build).
2. **Сценарий «секрет найден»**: создайте файл `secrets.txt` с любым AWS-ключом в формате `AKIA` + 16 символов (не используйте ключи, оканчивающиеся на `EXAMPLE` — они игнорируются). Добавьте в git и сделайте push — пайплайн упадёт на этапе detect-secrets, сборка не запустится.

## Исключение ложных срабатываний

Если Gitleaks находит «секрет» в тестовых данных или примерах — добавьте исключения в `.gitleaks.toml`.
