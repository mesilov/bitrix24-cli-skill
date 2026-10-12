# Контракт транспорта b24cli

## Релиз и установка

Проверено 12.10.2026 на macOS arm64 с опубликованным
[релизом b24cli 0.1.0](https://github.com/mesilov/bitrix24-cli/releases/tag/0.1.0).
Он предоставляет 30 команд `task:*` и 5 локальных команд `connection:*`.
Бинарь включает runtime; PHP, Composer и Docker для запуска не нужны.

| Платформа | Релизный архив |
| --- | --- |
| Linux amd64 | [b24cli-linux-amd64.tar.gz](https://github.com/mesilov/bitrix24-cli/releases/download/0.1.0/b24cli-linux-amd64.tar.gz) |
| macOS arm64 | [b24cli-macos-arm64.tar.gz](https://github.com/mesilov/bitrix24-cli/releases/download/0.1.0/b24cli-macos-arm64.tar.gz) |
| Windows amd64 | [b24cli-windows-amd64.zip](https://github.com/mesilov/bitrix24-cli/releases/download/0.1.0/b24cli-windows-amd64.zip) |

Скачай архив своей платформы и
[SHA256SUMS того же релиза](https://github.com/mesilov/bitrix24-cli/releases/download/0.1.0/SHA256SUMS).
Например, через GitHub CLI для macOS arm64:

```sh
gh release download 0.1.0 --repo mesilov/bitrix24-cli \
  --pattern b24cli-macos-arm64.tar.gz --pattern SHA256SUMS \
  --dir ./b24cli-download
```

Проверь SHA-256 архива перед распаковкой. Распаковывай весь пакет и сохраняй
`build-info.json`, runtime/compression metadata и все license notices вместе
с executable. На Windows он называется `b24cli.exe`.
После установки `b24cli --version` должен показать `b24cli 0.1.0`.
Подробности установки и поддержки ОС — в
[README транспорта](https://github.com/mesilov/bitrix24-cli#установка).

## Учётные данные

Для разового вызова выбери один источник явно:
`--env-file PATH` для dotenv или `--profile NAME` для сохранённого профиля.
Эти selectors сильнее credentials из окружения и взаимоисключающие.
Ошибка выбранного источника не переключает CLI на другое подключение.

```sh
b24cli --env-file "/absolute/path/to/.env.local" \
  task:list --max-scan 100 --limit 10 --json
b24cli --profile NAME task:list --max-scan 100 --limit 10 --json
```

В PowerShell:

```powershell
b24cli.exe --env-file 'C:\Private Config\.env.local' task:list --max-scan 100 --limit 10 --json
```

Файл содержит `BITRIX24_WEBHOOK=…`, без shell-команд. Его разбирает CLI;
выполнять `source .env.local` не требуется. Не выводи файл или переменную с
webhook и не вставляй webhook в аргументы команды. `--config` предназначен
для несекретного JSON с ключами `timeout`, `apiPolicy`, `profile`.

`profile` внутри `--config` также выбирает сохранённый профиль;
`--profile` переопределяет его, а `--env-file` конфликтует с ним.
Для разового выбора файла не добавляй profile в runtime config.

Без явного selector CLI выбирает: process `BITRIX24_WEBHOOK`, затем alias
`BITRIX24_PHP_SDK_PLAYGROUND_WEBHOOK`, затем `B24CLI_ENV_FILE`,
`B24CLI_PROFILE`, сохранённый `defaultProfile`.
Одновременно непустые `B24CLI_ENV_FILE` и `B24CLI_PROFILE` без явного selector
дают configuration-error. Standalone-бинарь не ищет dotenv в cwd,
рядом с executable или внутри пакета. Рекомендуемый способ — явный selector.

`--profile default` в 0.1.0 означает профиль с alias `default`;
это не специальное имя для подключения из окружения.

## Сохранённые подключения и defaults

`connection:*` управляют локальной конфигурацией CLI и не вызывают портал.
Для разового чтения с `--env-file` создавать или переключать сохранённый
профиль не требуется. Меняй локальные профили, когда это входит в запрос.
Полная карта этих команд — в [commands.md](commands.md).

Каталог выбирается через `--config-dir PATH`, затем `B24CLI_CONFIG_DIR`.
По умолчанию Linux/macOS используют абсолютный `$XDG_CONFIG_HOME/b24cli`
либо `$HOME/.config/b24cli`; Windows — `%APPDATA%\b24cli`.
CLI хранит metadata в `config.json`, а webhook — отдельно в credential file.
Webhook хранится plaintext с ограниченными правами доступа.

`connection:add NAME` принимает скрытый ввод в TTY; для automation:

```sh
b24cli connection:add NAME --webhook-stdin -n < /secure/webhook.txt
```

Этот stdin содержит только webhook URL, без `BITRIX24_WEBHOOK=`.
`connection:add` не меняет defaultProfile. Для постоянного выбора служит
`connection:use NAME`; для разового — `--profile NAME`.
`connection:remove NAME` удаляет только локальный профиль и credential file,
очищая defaultProfile при совпадении. Отзыв webhook на портале не выполняется.

Выбранный профиль может иметь `default.task` с `groupId`, `creatorId`,
`responsibleId`, `allowsTimeTracking`. Defaults применяются только к
`task:add`; явные значения полей имеют приоритет. Через process credentials
или `--env-file` профильные defaults не применяются.
CLI не определяет автора/исполнителя по webhook. Скилл использует только
уже заданные пользователем значения; собственные defaults не вводит.

Команды используют права настроенного пользователя. При отказе доступа
сообщи ошибку CLI. Наличие команды не гарантирует доступ к данным
конкретного портала.

## Основные глобальные опции

| Опция | Контракт |
| --- | --- |
| `--json` | Один JSON envelope в stdout; диагностика в stderr. |
| `--plain` | TSV без заголовков; специальные символы экранируются. Взаимоисключается с `--json`. |
| `--timeout SECONDS` | HTTP timeout, по умолчанию 30 секунд, максимум 300. |
| `--config PATH` | Runtime JSON; `-` читает stdin. Credentials сюда не входят. |
| `--profile NAME` | Выбрать сохранённый профиль, сильнее credentials окружения. |
| `--env-file PATH` | Выбрать credential dotenv; нельзя смешивать с `--profile`. |
| `--config-dir PATH` | Указать каталог локальной конфигурации CLI. |
| `-n`, `--no-interaction` | Отключает интерактивные вопросы. |

Глобальные опции допустимы до и после имени команды.
Полный набор доступных опций уточняй через `b24cli help ИМЯ_КОМАНДЫ`.
`--` завершает разбор опций. `-q` скрывает полезный результат;
`--silent` скрывает также диагностику — не используй их для проверки результата.

## Текст, JSON и изменения

`--description-file`, `--text-file`, `--fields-file`, `--where-file`,
`--params-file` читают файл либо stdin (`-`). Используй файл для сложного
текста и JSON, чтобы кавычки, переводы строк и shell-подстановки не меняли ввод.
Один stdin нельзя одновременно использовать для нескольких источников.

`task:add` требует title; `creatorId` и `responsibleId` передаются явно либо
берутся из `default.task` выбранного профиля. Без них создание не проходит
валидацию. Basic flags
`--title`, `--creator`, `--responsible` и другие нельзя смешивать с advanced
режимом `--fields`/`--fields-file`. JSON использует имена полей, принимаемые
CLI, например `creatorId`. Допустимость полей проверяет CLI.
Если `allowsTimeTracking` не задан явно и отсутствует в выбранном профиле,
используется встроенный default `false`.

`task:update` отправляет только явно заданные поля; `--description=""`
очищает описание. Изменение статуса и произвольных полей в таком патче
не поддержано. Для участников есть отдельная команда.
Сроки передаются в ISO8601 с часовым поясом.

`--dry-run` доступен у команд записи/удаления задач и их ресурсов.
Локальные `connection:*` его не поддерживают. Dry-run возвращает
`data.plan.steps` без записей, но может делать проверочные чтения с портала.

```sh
b24cli task:update 123 --title "Новое название" --dry-run --json
b24cli task:add --title "Пример задачи" --creator 1 --responsible 2 --dry-run --json
```

Удаление задач/их ресурсов в automation требует `--force`; на TTY CLI запрашивает подтверждение.
Этот флаг не отключает проверку привязки сообщения, записи времени или пункта
к задаче. Удаление пункта чек-листа затрагивает его subtree; план показывает IDs.

## Результат и exit code

JSON имеет структуру `{"data":…, "meta":…, "error":…}`,
`meta.schemaVersion=1`.

| Данные | Расположение |
| --- | --- |
| Списки | `data.items` |
| Локальные команды подключений | `data.items` |
| Карточка | `data.task` |
| Поля метаданных | `data.items` |
| Права | `data.access` |
| Контекст времени | `data.time` |
| Одна мутация | `data.operation` |
| Несколько шагов | `data.items`, подтверждённые/частичные результаты |
| Dry-run | `data.plan.steps` |

Успешный полный ответ имеет `error=null`. Ошибки могут возвращать `data=null`;
неполный результат может содержать полезные данные и `error.code=partial-result`.
Для операции с несколькими шагами проверяй результат каждого шага.

| Exit code | Значение |
| --- | --- |
| `0` | Подтверждённый результат в заявленной области выборки/операции. |
| `1` | Ошибка подключения, прав или выполнения. |
| `2` | Ошибка аргументов или локального ввода. |
| `3` | Неполная выборка или частично выполненные шаги. |
| `4` | Отказ проверки CLI или недоказанная привязка ресурса. |
| `130` | Прерывание или отказ подтверждения. |

Сохраняй exit code отдельно от JSON, включая ответы с полезными данными при
exit `3`. При `meta.outcomeUnknown=true` ответ на запись потерян; транспорт
не повторяет её автоматически.

## Полнота и пагинация

- `task:list`/`task:find`: только `--id` задаёт серверный ID-фильтр;
  остальные фильтры и поиск title применяются локально при обходе.
  `--limit` режет вывод; `--all` снимает только этот лимит.
  `--max-scan` ограничивает обход, по умолчанию 10000.
  Проверяй `complete`, `scanned`, `matched`, `returned`, `limitApplied`, `reason`.
  Даже запрос с `--limit 1` может прочитать много страниц.
- `task:chat:list`: без `--all` возвращает окно сообщений; для обхода доступной
  истории используй `--all`, с бюджетом `--max-messages` (по умолчанию 10000).
  Проверяй `scope`, `complete`, `hasMore`, `reason`.
- `task:time:list`: обходит страницы в пределах бюджета 10000;
  явный `params.NAV_PARAMS.iNumPage` ограничивает scope выбранной страницей.
- `task:history:list`: при недоказанной навигации (`next`/`total`) выдаёт
  partial/exit `3`. История изменений не содержит переписку.
- `task:time:show`: поле `elapsedTime` — контекст задачи; оно не доказывает
  сумму всех записей времени. Отсутствующее поле может дать partial/exit `3`.
- Участники в `task:show`: недоступные users/поля сохраняют известные ID,
  возвращая null и partial/exit `3`. Запрашивай только нужные projections.

## Источники и недостаток возможностей

Источники команд и параметров — `b24cli list --raw` и
`b24cli help ИМЯ_КОМАНДЫ` установленного бинаря.
Работай в пределах интерфейса CLI. При неподдержанном запросе следуй
правилу остановки из [SKILL.md](../SKILL.md) и попроси пользователя создать
[issue в bitrix24-cli](https://github.com/mesilov/bitrix24-cli/issues).
