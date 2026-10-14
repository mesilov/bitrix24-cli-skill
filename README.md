# bitrix24-cli-skill

Общий Agent Skill для работы с задачами Битрикс24 через
[mesilov/bitrix24-cli](https://github.com/mesilov/bitrix24-cli).
Один пакет используется в Claude Code, Codex и OpenCode.

Скилл содержит описание транспорта и карту доступных команд CLI, без
организационных правил, фиксированных порталов, ID, ролей или defaults задач.
Если CLI не покрывает запрос, скилл останавливает неподдерживаемый путь и при
наличии doctor сохраняет обезличенную карточку публичной командой CLI. При
запуске он проверяет weekly предложение пакетного разбора; без doctor предлагает
обычный [issue в bitrix24-cli](https://github.com/mesilov/bitrix24-cli/issues).

## Состав

- [SKILL.md](skills/bitrix24-tasks/SKILL.md) — точка входа `bitrix24-tasks`.
- [Транспорт](skills/bitrix24-tasks/references/transport.md) — credentials,
  ввод/вывод, ошибки, dry-run и пагинация.
- [Карта команд CLI](skills/bitrix24-tasks/references/commands.md) — все 56 публичных
  команд 0.2.0 по `dev`: 43 задач, 4 пользователей, 5 подключений, 1 doctor
  и 3 служебные, с их параметрами.
- [Рабочие сценарии](skills/bitrix24-tasks/references/task-workflows.md) — статусы
  и native результаты, локальные вложения, теги, custom поля, длительности статусов.
- [Временные фильтры](skills/bitrix24-tasks/references/deadline-windows.md) —
  перевод календарных дедлайнов и просрочки в public CLI (при наличии flags в help).
- [Проблемные сценарии](skills/bitrix24-tasks/references/problem-trajectories.md) —
  capability-gated doctor, локальные карточки, weekly разбор и явный feedback mock
  из [#96](https://github.com/mesilov/bitrix24-cli/issues/96); используется при наличии
  команд/actions в help установленного CLI.
- [Проверка 0.2.0](docs/verification-0.2.0.md) — source SHA, фактическая справка
  dev, RC-бинарь, примеры и границы проверки.
- [Историческая проверка 0.1.0](docs/verification.md) — предыдущая стабильная сборка.
- [Проверка doctor workflow](docs/problem-trajectories-verification.md) —
  исходники CLI, package validation и network=none smoke публичных примеров.

Бинарь устанавливается отдельно; в пакет скилла не включаются credentials
или platform-specific executable. Карта сверена 14.10.2026 по
[dev `1ed4422`](https://github.com/mesilov/bitrix24-cli/tree/1ed4422a8d7a33138134d6a8763594820c75c256)
с `list`, root help/about и справкой всех публичных команд. Проверен также
standalone macOS arm64 `0.2.0-rc-1`; в dev изменён расчёт длительностей статусов
после этого RC. Стабильный 0.2.0 пока не опубликован; 0.1.0 не покрывает новую карту.

Связь обновления: [issue скилла #6](https://github.com/mesilov/bitrix24-cli-skill/issues/6),
[issue приложения #100](https://github.com/mesilov/bitrix24-cli/issues/100),
OpenSpec `verify-cli-help-and-docs-0-2-0`
([архив](https://github.com/mesilov/bitrix24-cli/tree/dev/openspec/changes/archive/2026-10-14-verify-cli-help-and-docs-0-2-0)).
Повторная сверка финального релизного артефакта —
[release issue #109](https://github.com/mesilov/bitrix24-cli/issues/109).

## Установка скилла

Скопируй **весь каталог** `skills/bitrix24-tasks`, включая `references` и
`agents`, в каталог навыков выбранного агента. Содержимое пакета одинаковое;
`agents/openai.yaml` добавляет только метаданные интерфейса Codex.

| Агент | Для пользователя | Для проекта | Вызов |
| --- | --- | --- | --- |
| Claude Code | `~/.claude/skills/bitrix24-tasks/` | `.claude/skills/bitrix24-tasks/` | `/bitrix24-tasks` или подходящий запрос |
| Codex | `~/.agents/skills/bitrix24-tasks/` | `.agents/skills/bitrix24-tasks/` | `$bitrix24-tasks` или подходящий запрос |
| OpenCode | `~/.config/opencode/skills/bitrix24-tasks/` | `.opencode/skills/bitrix24-tasks/` | Запрос на использование `bitrix24-tasks`, загружается через native `skill` tool |

Например, из корня этого репозитория на Linux/macOS:

```sh
# Claude Code
mkdir -p "$HOME/.claude/skills"
cp -R skills/bitrix24-tasks "$HOME/.claude/skills/"

# Codex
mkdir -p "$HOME/.agents/skills"
cp -R skills/bitrix24-tasks "$HOME/.agents/skills/"

# OpenCode
mkdir -p "$HOME/.config/opencode/skills"
cp -R skills/bitrix24-tasks "$HOME/.config/opencode/skills/"
```

Если скилл уже установлен, замени его каталог целиком. Копирование поверх
старой папки оставляет удалённые из новой версии справочники; они не должны
сохраняться в установленном пакете.
В Windows скопируй тот же каталог в соответствующий user/project path.
Проектная установка выполняется в целевом проекте, где запускается агент.

Формат и пути сверены 12.10.2026 с официальной документацией
[Claude Code](https://code.claude.com/docs/en/skills),
[Codex](https://developers.openai.com/codex/skills/) и
[OpenCode](https://opencode.ai/docs/skills/).
Запуск скилла внутри всех трёх клиентов отдельно не выполнялся.

## Транспорт и подключение

Установи `b24cli` для своей платформы по
[инструкции транспорта](skills/bitrix24-tasks/references/transport.md).
Проверь `b24cli --version`, `b24cli --about` и `b24cli list --raw`.

Для credentials в файле `.env.local` укажи абсолютный путь:

```sh
b24cli --env-file "/absolute/path/to/.env.local" \
  task:list --max-scan 100 --limit 10 --json
```

Не считай расположение `.env.local` рядом со скиллом автоматической настройкой
standalone-бинаря. Секрет читает транспорт; включать его в скилл не нужно.
Все особенности установленной версии уточняются через `b24cli КОМАНДА --help`.
Для сохранённого подключения используй `--profile NAME`. Одновременно
передавать `--profile` и `--env-file` нельзя. Команды `connection:*` управляют
локальными профилями CLI; их параметры описаны в карте команд.

## Лицензия

[MIT](LICENSE).
