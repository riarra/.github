# riarra/.github

Organization-wide defaults и reusable workflows для всех репозиториев в `riarra/*`.

GitHub автоматически распознаёт repo с именем `.github` в organization: файлы из `profile/` идут на landing page org, `.github/` директория работает как fallback для repos которые не имеют своей.

## Что здесь живёт

### `profile/`
- `README.md` — страница которую видят посетители https://github.com/riarra

### `.github/workflows/` — reusable workflows

Все имя файлов начинаются с `_` (convention для reusable). Вызываются из caller workflows в других repos через `uses:` + `secrets: inherit`.

| Workflow | Событие caller'а | Что делает |
|---|---|---|
| `_notify-push.yml` | `push` | Отправляет в TG группу сообщение про новый commit |
| `_notify-issue.yml` | `issues` | Уведомление про создание/закрытие/назначение issue |
| `_notify-pr.yml` | `pull_request` | Уведомление про PR (открыт/смёржен/закрыт/ready for review) |

### Caller example

Любой repo в org подключается так — создать в нём `.github/workflows/notify.yml`:

```yaml
name: Notifications
on:
  push:
    branches: [main]
  issues:
    types: [opened, closed, reopened, assigned, labeled]
  pull_request:
    types: [opened, closed, reopened, ready_for_review]

jobs:
  push:
    if: github.event_name == 'push'
    uses: riarra/.github/.github/workflows/_notify-push.yml@main
    secrets: inherit
  issue:
    if: github.event_name == 'issues'
    uses: riarra/.github/.github/workflows/_notify-issue.yml@main
    secrets: inherit
  pr:
    if: github.event_name == 'pull_request'
    uses: riarra/.github/.github/workflows/_notify-pr.yml@main
    secrets: inherit
```

## Требуемые org-level secrets

| Secret | Назначение |
|---|---|
| `NOTIFY_BOT_TOKEN` | Telegram Bot API token (бот @rilhom_bot или отдельный notify-bot) |
| `NOTIFY_CHAT_ID` | ID группы «Разработка Riarra IT» |

Устанавливаются один раз в `Organization → Settings → Secrets and variables → Actions → New organization secret`.

Scope: `Selected repositories` — указать repos которые могут их использовать. При создании нового repo в org — не забыть добавить.

## Архитектурное решение

См. `riarra/app` ADR-003 (будет добавлено при миграции) или notes в Claude memory.

Ключевые тезисы:
- Один workflow — одно место редактирования (DRY)
- Org-level secrets — ротация в одной точке
- HTML-escape всего user-controlled input (injection-safe per GitHub security guide)
- Русские имена (Ильхом/Вадим/Руслан) — читаемо для команды
