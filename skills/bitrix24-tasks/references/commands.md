# Карта команд CLI

Карта проверена по опубликованному релизу **b24cli 0.1.0**
12.10.2026: **35 прикладных команд — 30 задач и 5 подключений**.
Источник команд и флагов —
`b24cli list --format=json` и `b24cli help ИМЯ_КОМАНДЫ`.
`help`, `list` и `completion` — служебные команды CLI.

К шаблону команды добавляй `b24cli` в начале и `--json` в конце.
`TASK_ID`, `USER_ID`, `CHECKLIST_ID`, `ITEM_ID`, `ENTRY_ID`, `MESSAGE_ID`,
`FILE_ID` обозначают положительные числовые ID выбранных ресурсов.
Прочие значения в верхнем регистре тоже заменяются фактическими аргументами.

У всех команд записи/удаления **задач** есть `--dry-run`. У четырёх команд
удаления задач/их ресурсов есть `--force`, обязательный для automation.
Локальные команды `connection:*` не имеют этих флагов.

## Подключения CLI

`NAME` — alias сохранённого профиля. Каталог можно указать через
глобальную опцию `--config-dir PATH`. Эти команды работают с локальными
файлами конфигурации и не вызывают портал.

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `connection:add NAME` | Сохранить профиль; скрытый ввод webhook в TTY или `--webhook-stdin` для redirected stdin. Опции defaults: `--group-id`, `--creator-id`, `--responsible-id`, `--allows-time-tracking true\|false`; `--portal-domain` задаёт описательный домен. |
| `connection:list` | Показать сохранённые профили и безопасные metadata. |
| `connection:show NAME` | Показать metadata профиля. Без `NAME` показывает фактически выбранный источник подключения. |
| `connection:use NAME` | Сохранить профиль по умолчанию. Для разового выбора используй `--profile NAME` у команды задачи. |
| `connection:remove NAME` | Удалить локальный профиль и его credential file. Удаление текущего профиля очищает выбор по умолчанию; webhook на портале не отзывается. |

Для `connection:add --webhook-stdin -n` stdin содержит только полный webhook,
а не dotenv-файл. Не передавай секрет в аргументах. `connection:add` не
выбирает профиль по умолчанию; это делает `connection:use`.

```sh
b24cli --config-dir "/absolute/path/to/config" connection:list --json
b24cli --profile NAME task:list --max-scan 100 --limit 10 --json
```

## Задачи, поля и доступ

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `task:add --title "Название" --creator USER_ID --responsible USER_ID` | Создать задачу; `--group-id`, `--deadline`, `--description`/`--description-file`. Отдельный advanced режим: `--fields`/`--fields-file`. |
| `task:show TASK_ID` | Карточка; повторяемый `--select FIELD`. |
| `task:list` | Выборка; повторяемые `--id`, `--select`, `--order FIELD:asc`; `--responsible`, `--group-id`, `--status`, `--due-before`, `--where`/`--where-file`, `--limit`, `--all`, `--max-scan`. |
| `task:find --title "Подстрока"` | Поиск буквальной подстроки title без учёта регистра; `--group-id`, `--limit`, `--all`, `--max-scan`. |
| `task:update TASK_ID --title "Название"` | Явный патч; `--creator`, `--responsible`, `--group-id`, `--deadline`, `--description`/`--description-file`. Отдельный режим `--fields`/`--fields-file`. |
| `task:delete TASK_ID` | Удалить задачу; `--force`. |
| `task:fields:list` | Metadata всех полей; `--name FIELD` выбирает metadata одного поля. |
| `task:access:show TASK_ID` | Права пользователя подключения на задачу. |
| `task:assign TASK_ID --responsible USER_ID` | Изменить только исполнителя. |
| `task:deadline:set TASK_ID --at ISO8601` | Изменить только срок, с явным часовым поясом. |

У `task:add` title обязателен; creator/responsible могут поступить из
`default.task` выбранного профиля. Явные поля сильнее профильных defaults.
При подключении через `--env-file` или process credentials профильные
defaults не применяются. Подробнее — в [контракте транспорта](transport.md).

CLI проверяет допустимость полей. `task:fields:list --name FIELD`
показывает metadata одного поля; `task:show --select FIELD` выбирает поля
карточки. Например:

```sh
b24cli task:show 123 --select title --select auditors.id --json
b24cli task:show 123 --select accomplices --select auditors.email --json
```

Поддержанные projections участников: `id`, `name`, `email`, `gender`,
`externalAuthId`. Выбор роли без suffix означает `id/name`.

## Участники

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `task:participants:set TASK_ID --auditor USER_ID` | Повторяемые `--auditor`, `--accomplice`; `--clear-auditors`, `--clear-accomplices`. |

Каждая указанная роль заменяется целиком; пропущенная роль сохраняется.
`--clear-*` явно очищает соответствующую роль.

## Чат задачи

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `task:chat:list TASK_ID` | Сообщения; `--before MESSAGE_ID`, `--after MESSAGE_ID`, `--limit`, `--all`, `--max-messages`. |
| `task:chat:send TASK_ID --text "Текст"` | Отправить сообщение; альтернатива `--text-file PATH`. |
| `task:chat:update TASK_ID --message MESSAGE_ID --text "Текст"` | Изменить сообщение; альтернатива `--text-file PATH`. |
| `task:chat:delete TASK_ID --message MESSAGE_ID` | Удалить сообщение; `--force`. |

Отправка возвращает boolean ACK и `resourceId=null`.
ID существующего сообщения выбирается через `task:chat:list`.
Перед изменением/удалением CLI проверяет его принадлежность чату задачи.

## Файлы

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `task:file:attach TASK_ID --file-id FILE_ID` | Прикрепить уже загруженный файл Диска; `--file-id` можно повторять. |

Загрузка локального файла на Диск этой командой не выполняется.
Каждый ID обрабатывается отдельным шагом; проверяй результат всех шагов.
Подтверждение прикрепления и независимая проверка наличия файла имеют
разный контракт; учитывай данные, возвращённые CLI.

## Учёт времени

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `task:time:show TASK_ID` | Контекст задачи, в том числе `elapsedTime`. |
| `task:time:add TASK_ID --seconds SECONDS` | Добавить запись; `--text`. |
| `task:time:list TASK_ID` | Записи; JSON `--params` или `--params-file PATH`. |
| `task:time:update TASK_ID --entry ENTRY_ID --seconds SECONDS` | Изменить запись; `--text`. |
| `task:time:delete TASK_ID --entry ENTRY_ID` | Удалить запись; `--force`. |

`SECONDS` — положительное целое число секунд. Для `--params` доступны
`order`, `filter`, `select`, `params.NAV_PARAMS`;
используй `help task:time:list` для синтаксиса. Update/delete проверяют запись
внутри задачи. Время из карточки и сумма записей имеют разный контракт.

## Чек-листы

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `task:checklist:add TASK_ID --title "Название"` | Создать корневой чек-лист. |
| `task:checklist:list TASK_ID` | Показать корневые чек-листы. |
| `task:checklist:item:add TASK_ID --checklist CHECKLIST_ID --title "Пункт"` | Добавить пункт; `--parent ITEM_ID` для вложенности. |
| `task:checklist:item:list TASK_ID --checklist CHECKLIST_ID` | Показать все вложенные пункты корня. |
| `task:checklist:item:update TASK_ID --item ITEM_ID --title "Пункт"` | Изменить заголовок пункта. |
| `task:checklist:item:complete TASK_ID --item ITEM_ID` | Выполнить пункт. |
| `task:checklist:item:renew TASK_ID --item ITEM_ID` | Вернуть пункт в невыполненное состояние. |
| `task:checklist:item:delete TASK_ID --item ITEM_ID` | Удалить пункт и всех потомков; `--force`. |

`CHECKLIST_ID` — ID корневого узла; `--parent` относится к тому же корню.
Без `--parent` новый пункт добавляется непосредственно в корень.
Эти команды меняют пункты чек-листа; завершение всей задачи ими не выполняется.

## История изменений

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `task:history:list TASK_ID` | Изменения полей; JSON `--params` или `--params-file PATH`. |

Параметры истории ограничены `filter.FIELD` (строка) и
`order.createdDate` (`ASC` или `DESC`). Другие ключи, включая `select`,
отклоняются локально с exit `2`. Например, структура `--params`:

```json
{"filter":{"FIELD":"FIELD_NAME"},"order":{"createdDate":"DESC"}}
```

Замени `FIELD_NAME` кодом поля истории, по которому нужен фильтр;
`filter` и `order` можно передавать по отдельности.
При неполной/недоказанной пагинации CLI возвращает exit `3`.

## Границы карты

Карта покрывает namespaces `task` и `connection` релиза 0.1.0. Команды
завершения всей задачи, загрузки локального файла на Диск, работы с
результатами задач и шаблонами в этой сборке отсутствуют.
Если установленный CLI не покрывает требуемую операцию, **остановись**,
назови недостающую возможность и попроси пользователя создать
[issue в bitrix24-cli](https://github.com/mesilov/bitrix24-cli/issues).
Не выполняй её через другой транспорт и не создавай issue автоматически.

Для другого бинаря сначала выполни:

```sh
b24cli --version
b24cli list task --raw
b24cli help task:update
```

Семантика результата, credentials и лимитов описана в
[контракте транспорта](transport.md).
