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

## Сопровождение репозитория

Локальный [скилл мейнтейнера](.agents/skills/bitrix24-cli-skill-maintainer/SKILL.md)
описывает работу над этим репозиторием. Он требует содержательную запись
в [CHANGELOG.md](CHANGELOG.md) со ссылкой на issue в каждом PR с изменениями
и проверку diff перед открытием PR и merge. Этот скилл хранится отдельно
от пользовательского пакета `skills/bitrix24-tasks`.

## Лицензия

[MIT](LICENSE).
