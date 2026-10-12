# Проверка bitrix24-tasks

Дата: 12.10.2026. Среда исполнения бинаря: macOS arm64.

## Происхождение транспорта

| Параметр | Значение |
| --- | --- |
| Репозиторий | [mesilov/bitrix24-cli](https://github.com/mesilov/bitrix24-cli) |
| Workflow | Package standalone binaries |
| Run | [38169692267](https://github.com/mesilov/bitrix24-cli/actions/runs/38169692267) |
| Branch / event | `dev` / `pull_request`; run завершился `success`. |
| Source SHA и версия бинаря | `a361a1fecddef0f6b77eb40b26d14c9b67d4cb86`, `dev-a361a1fecddef0f6b77eb40b26d14c9b67d4cb86` |
| Artifact | [b24cli-release-assets, 11713828678](https://github.com/mesilov/bitrix24-cli/actions/runs/38169692267/artifacts/11713828678) |
| Проверенный executable | `b24cli` из `b24cli-macos-arm64.tar.gz` |
| SHA-256 macOS архива | `d70cc732e852d1ad9d9ffe9c2ff239c8973f12a545dc5f538d270e9a640bd532` |
| SHA-256 executable | `ef43e96c30861054b36d07f51bed18218b93f69e1536da0aed61614f467e018e` |

SHA подтверждён и через `build-info.json`, и через `b24cli --version`.
Все три platform archives совпали с `SHA256SUMS` из того же artifact.
`codesign --verify --strict --verbose=2` для macOS executable завершился успешно.
Linux и Windows binaries не запускались.

Текущая dev-ветка на момент подготовки была новее этого binary artifact
(`d145b226fd048423c2ce2af93b28efd733d4ccec`). В ней появились `connection:*`,
`--env-file` и другие параметры подключения. Карта и основной способ
подключения описывают реально скачанный бинарь; разница отмечена в transport reference.

## Локальная проверка

- 30/30 команд `task:*` из `list --format=json` представлены в карте;
  лишних и дублирующих команд нет.
- `help` каждой из 30 команд завершился с exit `0`.
- Основные флаги таблиц присутствуют в command definitions бинаря.
- 8 basic dry-run сценариев завершились с exit `0`, `meta.dryRun=true`,
  непустым планом и пустым `meta.apiCalls`: add, update, assign, deadline:set,
  chat:send, time:add, checklist:add, file:attach.
- Штатный `quick_validate.py` проверил frontmatter пакета; относительные
  ссылки и metadata также проверены локально.
- Две локальные проверки history JSON подтвердили разрешённую структуру
  (`filter.FIELD`, `order.createdDate`) и отклонение `select` с exit `2`.
  Проверка выполнялась локально, запросов к порталу не было.
- Отдельный агент выполнил четыре сценария чтения reference skill:
  подключение/поиск, замена одной роли участников, ограничения lifecycle/upload,
  установка пакета в три агента. Выявленное уточнение параметров истории
  проверено локально и внесено в карту.

Dry-run проверялся без credentials, из отдельного каталога.
Сценарии с binding/metadata reads не выдаются за offline-проверку.

## Проверка границы CLI

После уточнения требований карта содержит только публичные команды CLI
и их параметры. Покрытие 30/30 команд повторно сверено с каталогом бинаря;
внутренняя карта вызовов удалена, локальные ссылки и skill validator прошли.

Прежняя инструкция в сценарии завершения задачи сообщала ограничение и
предлагала другой инструмент. После добавления правила остановки отдельный
агент проверил три сценария:

- Завершение задачи: агент остановился и попросил пользователя создать issue
  в `mesilov/bitrix24-cli`; обход ограничения и запись не выполнялись.
- Создание из шаблона: агент остановился с той же просьбой; обычная задача
  вместо задачи из шаблона не создавалась.
- Поддерживаемая выборка задач: агент подготовил корректную команду CLI
  с credentials-file selector, фильтром и бюджетом обхода.

Issue автоматически не создавались. Проверка использовала только пакет
скилла и offline help, без чтения credentials и обращения к порталу.

## Чтение с портала

Для проверки использован внешний `.env.local`, выбранный через
`B24CLI_ENV_FILE`. Значения credentials и данные задач не включены в репозиторий.

| Проверка | Наблюдаемый результат |
| --- | --- |
| `task:list --max-scan 1 --limit 1 --json` | Одна задача, exit `3`, `complete=false`, `reason=scan-budget-exhausted`; ожидаемая граница обхода. |
| `task:show TASK_ID --select id --json` | Карточка с выбранным ID, exit `0`, `error=null`. |
| `task:fields:list --name title --json` | Metadata выбранного поля, exit `0`, `error=null`. |

Записи и удаления на портале не выполнялись. Эта проверка подтверждает
подключение и три чтения на одном портале; она не доказывает работоспособность
всех команд на любом портале.

## Совместимость агентов

Пакет использует общий формат Agent Skills: `SKILL.md` с YAML frontmatter,
обычный Markdown и относительные ссылки на references. Он не использует
Claude-specific dynamic context, Codex-specific tools или OpenCode plugins.
`agents/openai.yaml` содержит только UI metadata.

Пути установки сверены с официальными инструкциями:
[Claude Code](https://code.claude.com/docs/en/skills),
[Codex](https://developers.openai.com/codex/skills/) и
[OpenCode](https://opencode.ai/docs/skills/).
Проверка чтения инструкции отдельным агентом не заменяет запуск в этих трёх
клиентах; такой запуск не выполнялся.
