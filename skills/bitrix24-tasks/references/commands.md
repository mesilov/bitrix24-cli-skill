# Карта команд CLI

Карта **b24cli 0.2.0** сверена 14.10.2026 по `dev`, source SHA
`1ed4422a8d7a33138134d6a8763594820c75c256`:
**56 публичных команд — 43 задач, 4 пользователей, 5 подключений,
1 doctor и 3 служебные**. Источник: фактические `list --format=json`,
`--help` и обе формы командной справки (`COMMAND --help`, `help COMMAND`).
Скрытая `_complete` не входит в публичную карту. Каталог совпал с бинарём
0.2.0-rc-1; отличие расчёта длительностей описано в
[рабочих сценариях](task-workflows.md). Стабильная 0.2.0 ещё не опубликована.

К шаблону команды добавляй `b24cli` в начале и `--json` в конце.
`TASK_ID`, `USER_ID`, `CHECKLIST_ID`, `ITEM_ID`, `ENTRY_ID`, `MESSAGE_ID`,
`FILE_ID`, `RESULT_ID`, `FOLDER_ID`, `DEPARTMENT_ID` обозначают положительные
числовые ID выбранных ресурсов.
Прочие значения в верхнем регистре тоже заменяются фактическими аргументами.

У всех команд записи/удаления **задач** есть `--dry-run`. У пяти команд
удаления задач/их ресурсов есть `--force`, обязательный для automation.
Локальные команды `connection:*` и doctor не имеют этих флагов.

## Общие команды

Эти команды не требуют подключения; `--json` envelope к ним не относится.

| Шаблон команды | Назначение |
| --- | --- |
| `help COMMAND` | Справка команды; эквивалент `COMMAND --help`. |
| `list` | Каталог; `--raw`, `--format txt\|xml\|json\|md`, необязательный namespace (`list task`). |
| `completion SHELL` | Скрипт автодополнения; допустимые SHELL уточняй в help. |

Корневые `b24cli --help`/`-h` показывают обзор, `b24cli --about` — сведения
о версии, лицензии и репозиториях. `list --help` — справка именно команды list.

## Подключения CLI

`NAME` — alias сохранённого профиля. Каталог можно указать через
глобальную опцию `--config-dir PATH`. Эти команды работают с локальными
файлами конфигурации и не вызывают портал.

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `connection:add NAME` | Создать новый профиль; скрытый webhook в TTY или `--webhook-stdin`. Defaults: `--group-id`, `--creator-id`, `--responsible-id`, `--allows-time-tracking true\|false`, `--observer-id`, `--manager-id`, `--auto-add-observer true\|false`; `--portal-domain` — описательный домен. Существующий NAME не обновляется. |
| `connection:list` | Показать сохранённые профили и безопасные metadata. |
| `connection:show NAME` | Показать metadata профиля. Без `NAME` показывает фактически выбранный источник подключения. |
| `connection:use NAME` | Сохранить профиль по умолчанию. Для разового выбора используй `--profile NAME` у команды задачи. |
| `connection:remove NAME` | Удалить локальный профиль и его credential file. Удаление текущего профиля очищает выбор по умолчанию; webhook на портале не отзывается. |

Для `connection:add --webhook-stdin -n` stdin содержит только полный webhook,
а не dotenv-файл. Не передавай секрет в аргументах. `connection:add` не
выбирает профиль по умолчанию; это делает `connection:use`.
`--auto-add-observer true` требует `--observer-id`. Изменение существующего
профиля требует явного remove/add со всеми нужными defaults и повторного
ввода webhook; сначала прочитай `connection:show NAME` и проверь наличие
исходных credentials. Для текущего default после пересоздания нужен
`connection:use NAME`.

```sh
b24cli --config-dir "/absolute/path/to/config" connection:list --json
b24cli --profile NAME task:list --max-scan 100 --limit 10 --json
```

## Задачи, поля и доступ

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `task:add --title "Название" --creator USER_ID --responsible USER_ID` | Создать задачу; `--group-id`, `--deadline`, `--description`/`--description-file`, повторяемый `--tag`, `--user-fields`/`--user-fields-file`. Отдельный advanced режим: `--fields`/`--fields-file`. |
| `task:show TASK_ID` | Карточка; повторяемый `--select FIELD`. |
| `task:list` | Выборка; повторяемые `--id`, `--select`, `--order FIELD:asc`; `--responsible`, `--group-id`, `--status`, `--role`, `--due-before`, `--where`/`--where-file`, `--deadline-window`, `--overdue`, `--timezone`, `--limit`, `--all`, `--max-scan`. |
| `task:find --title "Подстрока"` | Буквальная подстрока title без регистра Unicode; повторяемые `--id`/`--order`, `--responsible`, `--group-id`, `--status`, `--role`, `--due-before`, `--where`/`--where-file`, `--deadline-window`, `--overdue`, `--timezone`, `--limit`, `--all`, `--max-scan`. |
| `task:my --user USER_ID` | Задачи явного исполнителя; `--open`, `--due-before`, `--max-scan`. Это не автоматическое определение пользователя подключения. |
| `task:update TASK_ID --title "Название"` | Явный патч; `--creator`, `--responsible`, `--group-id`, `--deadline`, `--description`/`--description-file`, повторяемый `--tag` либо `--clear-tags`, `--user-fields`/`--user-fields-file`. Отдельный режим `--fields`/`--fields-file`. |
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

В 0.2.0 доступны также projections `tags.id`/`tags.name` и
`userFields.key`/`userFields.value`. Запись — в [рабочих сценариях](task-workflows.md).

`--status` принимает одно из `pending`, `in_progress`, `supposedly_completed`,
`completed`, `deferred`, `declined`. `--role` принимает `responsible`,
`creator`, `accomplice`, `auditor` и относится к пользователю **выбранного
подключения**, а не произвольному USER_ID. Для другого исполнителя есть
`--responsible USER_ID`; фильтра по ID другого наблюдателя нет.
Критерии пересекаются через AND. Роль сама по себе не ограничивает статус.
`--where` и `--where-file` взаимоисключающие; не смешивай их с basic criteria
`--group-id`, `--responsible`, `--status`, `--due-before` или временными flags.
Временные flags также нельзя смешивать с `--due-before`.

Порядок по умолчанию `id:desc`; `--order` меняет обход и вывод.
`task:my --open` включает pending/in_progress/supposedly_completed/deferred,
выводит до 50 строк без `--all`/`--limit`, бюджет 10000 меняется `--max-scan`.
`--due-before` — строго раньше границы, задачи без срока исключаются.
Для календарных окон используй [временные фильтры](deadline-windows.md).

## Пользователи

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `users:current` | Пользователь выбранного подключения; ID не выводится из webhook. |
| `users:find --email EMAIL` | Активные сотрудники: точный email либо `--name "ФИО"` (все токены без регистра, порядок не важен); критерии взаимоисключающие. `--limit`, `--all`, `--max-scan`. |
| `users:colleagues` | Активные коллеги текущего отдела, без себя и дочерних отделов; `--department-id DEPARTMENT_ID`, `--limit`, `--all`, `--max-scan`. |
| `users:manager` | Руководитель по оргструктуре; `--department-id DEPARTMENT_ID`, `--source org\|config`. Config явно выбирает `default.users.managerId` профиля. |

При нескольких текущих отделах у colleagues и manager/org явно выбери
`--department-id`. По оргструктуре CLI поднимается к первому другому
руководителю, если текущий пользователь сам руководитель или отдел без него.
Поиск возвращает кандидатов, не назначает человека автоматически.
JSON сохраняет `ID`, `NAME`, `SECOND_NAME`, `LAST_NAME`, `EMAIL`, `UF_DEPARTMENT`.
Current/manager возвращают `data.user` (у manager может быть null),
find/colleagues — `data.items`; полнота — только в пределах прав подключения.
У find/colleagues default limit 50 и max-scan 1000; `--all` снимает только limit.

```sh
b24cli --profile NAME users:current --json
b24cli --profile NAME users:find --email "person@example.test" --all --json
b24cli --profile NAME task:list --role auditor --all --json
```

Последняя команда выбирает задачи наблюдателя подключения; она не фильтрует
по ID из предыдущего поиска. Для «мои открытые на завтра» используй четыре
штатные выборки `task:list --role responsible --deadline-window tomorrow
--timezone TIMEZONE --status STATUS --all --json`, по одной на каждый открытый
статус. Проверяй полноту каждой. Не передавай `--deadline-window` в task:my
или `--open` в task:list: этих flags у соответствующих команд нет.

## Статусы и результаты задачи

| Шаблон команды | Назначение |
| --- | --- |
| `task:start TASK_ID` | Начать работу. |
| `task:pause TASK_ID` | Приостановить работу. |
| `task:defer TASK_ID` | Отложить задачу. |
| `task:renew TASK_ID` | Возобновить задачу. |
| `task:complete TASK_ID` | Завершить либо передать на контроль. |
| `task:approve TASK_ID` | Принять результат задачи. |
| `task:disapprove TASK_ID` | Вернуть на доработку. |
| `task:result:add TASK_ID --text "Результат"` | Добавить native результат; альтернатива `--text-file PATH`. |
| `task:result:list TASK_ID` | Список результатов; `--limit`, `--all`, `--max-scan`. |
| `task:result:update TASK_ID --result RESULT_ID --text "Результат"` | Изменить результат; альтернатива `--text-file PATH`. |
| `task:result:delete TASK_ID --result RESULT_ID` | Удалить результат; `--force`. |

Все записи поддерживают `--dry-run`; он может читать задачу, права и результаты.
`requireResult=true` требует native результат: сообщение/файл его не заменяет.
`meta.complete=true` не означает завершённый status. Фактический status и
подтверждение действия проверяй по [рабочим сценариям](task-workflows.md).

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
| `task:file:attach TASK_ID --file-id FILE_ID` | Прикрепить существующий файл Диска; повторяемый `--file-id`. Альтернативный режим: `--file PATH --folder-id FOLDER_ID`, необязательный `--name NAME`. |

Локальный режим загружает один обычный файл и затем прикрепляет его.
Режимы взаимоисключающие. Не придумывай folder ID; цель выбирает пользователь.
Лимит размера, plan и partial recovery — в [рабочих сценариях](task-workflows.md).
Каждый существующий ID обрабатывается отдельным шагом; проверяй все шаги.
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
| `task:status:durations TASK_ID` | Время в статусах; `--as-of ISO8601` с часовым поясом, не в будущем. Контракт dev и отличие RC — в [рабочих сценариях](task-workflows.md). |

Параметры истории ограничены `filter.FIELD` (строка) и
`order.createdDate` (`ASC` или `DESC`). Другие ключи, включая `select`,
отклоняются локально с exit `2`. Например, структура `--params`:

```json
{"filter":{"FIELD":"FIELD_NAME"},"order":{"createdDate":"DESC"}}
```

Замени `FIELD_NAME` кодом поля истории, по которому нужен фильтр;
`filter` и `order` можно передавать по отдельности.
При неполной/недоказанной пагинации CLI возвращает exit `3`.

## Локальные проблемные сценарии

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `doctor:problem-trajectory --action status` | Локальные обезличенные карточки и пакетный workflow; `--report-file`, `--idempotency-key`, `--batch-file`, `--settings-file`, `--state`, `--offset`, `--limit`, `--confirm-publication`, `--mock-feedback`. |

Actions: `status`, `record` (default), `list`, `record-offer`, `record-analysis`,
`submit`, `reconcile`, `configure`, `recover`. Входы, результаты, согласие
и ограничение mock — в [problem-trajectories.md](problem-trajectories.md).

## Границы карты

Карта покрывает весь публичный каталог dev для 0.2.0. Создание из шаблона,
native результат из сообщения (`task:result:from-message`), произвольный
ID наблюдателя в фильтре и выбор/поиск папки Диска в этом каталоге отсутствуют.
Если установленный CLI не покрывает требуемую операцию, **остановись**,
назови недостающую возможность и следуй workflow из [SKILL.md](../SKILL.md):
при доступном doctor сохрани обезличенную карточку и предложи issue или
пакетный разбор; без doctor предложи [issue в bitrix24-cli](https://github.com/mesilov/bitrix24-cli/issues).
Не выполняй её через другой транспорт и не создавай issue автоматически.

Для другого бинаря сначала выполни:

```sh
b24cli --version
b24cli list task --raw
b24cli help task:update
```

Семантика результата, credentials и лимитов описана в
[контракте транспорта](transport.md).
