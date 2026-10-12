# Проверка bitrix24-tasks с CLI 0.1.0

Дата: 12.10.2026. Среда исполнения: macOS arm64.

## Происхождение транспорта

| Параметр | Значение |
| --- | --- |
| Релиз | [0.1.0](https://github.com/mesilov/bitrix24-cli/releases/tag/0.1.0), опубликован 12.10.2026, не draft и не prerelease. |
| Source SHA | `a49a1882b0eac4033ec67c08ae01eacf00e21567` |
| Архив | [b24cli-macos-arm64.tar.gz](https://github.com/mesilov/bitrix24-cli/releases/download/0.1.0/b24cli-macos-arm64.tar.gz) |
| Checksums | [SHA256SUMS](https://github.com/mesilov/bitrix24-cli/releases/download/0.1.0/SHA256SUMS) из того же релиза. |
| SHA-256 архива | `d216957d7973a8f8a74d93c55a6984097afd77c395691d382bbadc9798aba91b` |
| SHA-256 executable | `910940d99446c2284900e3c699f093f2490b1ceac63b07ea33b6fca4f971a96f` |

Архив совпал с SHA256SUMS и digest опубликованного asset. Source SHA и версия
подтверждены build-info.json; executable показывает `b24cli 0.1.0`.
Проверка `codesign --verify --strict` прошла.
Linux и Windows archives представлены в релизе, но локально не запускались.

## Карта интерфейса CLI

- В карту включены 35/35 прикладных команд: 30 `task:*`, 5 `connection:*`.
  Служебные `help`, `list`, `completion` описаны отдельно.
- `help` всех 35 команд завершился с exit 0.
- Указанные в карте флаги сверены с `list --format=json` релизного бинаря.
- Скилл описывает команды, параметры и результаты CLI. Внутренняя карта
  вызовов отсутствует; при недостающей возможности действует остановка
  с просьбой создать issue в репозитории CLI.

## Проверки выполнения без портала

8 basic dry-run сценариев прошли с exit 0, непустым планом и нулём запросов:
`task:add`, `task:update`, `task:assign`, `task:deadline:set`, `task:chat:send`,
`task:time:add`, `task:checklist:add`, `task:file:attach`.
Два локальных сценария history JSON подтвердили допустимую структуру
`filter.FIELD`/`order.createdDate` и отклонение `select` с exit 2.

11 проверок выполнены с тестовым профилем и вымышленным webhook в отдельном
временном config-dir, без доступа к пользовательской конфигурации:

- Все пять `connection:*` команд прошли; add не меняет defaultProfile,
  use устанавливает его, remove удаляет профиль/credential и очищает default.
- `task:add` получил creator/responsible/group/time-tracking из выбранного
  профиля; явный responsible переопределил default.
- `task:update` не получил defaults создания задачи.
- `--env-file` не наследует profile defaults.
- Одновременные `--profile` и `--env-file` отклонены с exit 2.
- Явный `--env-file` выбран при конфликтующих credentials процесса.

Результаты локальных команд подключений находятся в `data.items`.
Сетевые вызовы не выполнялись; реальный `.env.local` не читался.
Эти проверки подтверждают интерфейс и локальное поведение CLI, но не
работоспособность всех команд на реальном портале.

## Поведение скилла

Отдельный агент проверил четыре сценария по справке релиза: явный env-file
при чужом default, использование profile defaults с явным исполнителем,
добавление профиля через stdin без смены постоянного default и запрос
неподдержанного завершения задачи. Поддержанные запросы выражены командами
CLI; неподдержанный запрос приводит к остановке и просьбе создать issue.
Выявленное уточнение области действия dry-run внесено в инструкции.

## Пакет и совместимость агентов

Пакет использует общий формат Agent Skills: SKILL.md с YAML frontmatter,
Markdown и относительные ссылки. Метаданные Codex находятся в agents/openai.yaml.
Бинарь и credentials не входят в ZIP скилла.

Пути установки сверены с официальными инструкциями:
[Claude Code](https://code.claude.com/docs/en/skills),
[Codex](https://developers.openai.com/codex/skills/),
[OpenCode](https://opencode.ai/docs/skills/).
Запуск внутри всех трёх клиентов отдельно не выполнялся.
