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
- [Карта команд CLI](skills/bitrix24-tasks/references/commands.md) — все 30 команд
  задач и 5 команд подключений релиза `0.1.0`, с их параметрами.
- [Временные фильтры](skills/bitrix24-tasks/references/deadline-windows.md) —
  перевод календарных дедлайнов и просрочки в public CLI (при наличии flags в help).
- [Проблемные сценарии](skills/bitrix24-tasks/references/problem-trajectories.md) —
  capability-gated doctor, локальные карточки, weekly разбор и явный feedback mock
  из [#96](https://github.com/mesilov/bitrix24-cli/issues/96). Это дополнение к карте
  релиза 0.1.0; оно используется только при наличии команд/actions в help.
- [Проверка сборки](docs/verification.md) — происхождение бинаря и границы проверки.
- [Проверка doctor workflow](docs/problem-trajectories-verification.md) —
  исходники CLI, package validation и network=none smoke публичных примеров.

Бинарь устанавливается отдельно; в пакет скилла не включаются credentials
или platform-specific executable. Скилл проверен с опубликованным
[релизом CLI 0.1.0](https://github.com/mesilov/bitrix24-cli/releases/tag/0.1.0).

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
Проверь `b24cli --version` и `b24cli list --raw`.

Для credentials в файле `.env.local` укажи абсолютный путь:

```sh
b24cli --env-file "/absolute/path/to/.env.local" \
  task:list --max-scan 100 --limit 10 --json
```

Не считай расположение `.env.local` рядом со скиллом автоматической настройкой
standalone-бинаря. Секрет читает транспорт; включать его в скилл не нужно.
Все особенности установленной версии уточняются через `b24cli help КОМАНДА`.
Для сохранённого подключения используй `--profile NAME`. Одновременно
передавать `--profile` и `--env-file` нельзя. Команды `connection:*` управляют
локальными профилями CLI; их параметры описаны в карте команд.

## Сопровождение репозитория

Локальный [скилл мейнтейнера](.agents/skills/bitrix24-cli-skill-maintainer/SKILL.md)
описывает работу над этим репозиторием. Он требует содержательную запись
в [CHANGELOG.md](CHANGELOG.md) со ссылкой на issue в каждом PR с изменениями
и проверку diff перед открытием PR и merge. Этот скилл хранится отдельно
от пользовательского пакета `skills/bitrix24-tasks`.

## Лицензия

[MIT](LICENSE).
