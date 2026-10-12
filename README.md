# bitrix24-cli-skill

Общий Agent Skill для работы с задачами Битрикс24 через
[mesilov/bitrix24-cli](https://github.com/mesilov/bitrix24-cli).
Один пакет используется в Claude Code, Codex и OpenCode.

Скилл содержит описание транспорта и карту доступных методов, без
организационных правил, фиксированных порталов, ID, ролей или defaults задач.

## Состав

- [SKILL.md](skills/bitrix24-tasks/SKILL.md) — точка входа `bitrix24-tasks`.
- [Транспорт](skills/bitrix24-tasks/references/transport.md) — credentials,
  ввод/вывод, ошибки, API policy, dry-run и пагинация.
- [Карта методов](skills/bitrix24-tasks/references/methods.md) — все 30 команд
  задач проверенной dev-сборки и их 25 REST-методов.
- [Проверка сборки](docs/verification.md) — происхождение бинаря и границы проверки.

Бинарь устанавливается отдельно; в пакет скилла не включаются credentials
или platform-specific executable. Для подготовки использован dev-артефакт
[run 38169692267](https://github.com/mesilov/bitrix24-cli/actions/runs/38169692267),
версия `dev-a361a1fecddef0f6b77eb40b26d14c9b67d4cb86`.

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

Если скилл уже установлен, обнови содержимое существующего каталога.
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
Проверь `b24cli --version` и `b24cli list task --raw`.

Для credentials в файле `.env.local` укажи абсолютный путь:

```sh
B24CLI_ENV_FILE="/absolute/path/to/.env.local" \
  b24cli task:list --max-scan 100 --limit 10 --json
```

Не считай расположение `.env.local` рядом со скиллом автоматической настройкой
standalone-бинаря. Секрет читает транспорт; включать его в скилл не нужно.
Все особенности установленной версии уточняются через `b24cli help КОМАНДА`.

## Лицензия

[MIT](LICENSE).
