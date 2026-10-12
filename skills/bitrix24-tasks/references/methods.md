# Карта команд и REST-методов

Карта проверена по бинарю `dev-a361a1fecddef0f6b77eb40b26d14c9b67d4cb86`
12.10.2026: **30 команд задач, 25 уникальных имён REST-методов**
(27 сочетаний метод/версия). Доступность команд и флагов проверена через
`list --format=json` и `help`; маршруты взяты из
[CommandCatalog той же сборки](https://github.com/mesilov/bitrix24-cli/blob/a361a1fecddef0f6b77eb40b26d14c9b67d4cb86/src/Bootstrap/CommandCatalog.php).

К шаблону команды добавляй `b24cli` в начале и `--json` в конце.
`TASK_ID`, `USER_ID`, `CHECKLIST_ID`, `ITEM_ID`, `ENTRY_ID`, `MESSAGE_ID`,
`FILE_ID` обозначают положительные числовые ID выбранных ресурсов.
Прочие значения в верхнем регистре тоже заменяются фактическими аргументами.
Список маршрутов включает условные metadata/enrichment/binding reads;
фактические вызовы показывает `meta.apiCalls`.

У всех команд записи/удаления есть `--dry-run`. У четырёх команд удаления
есть `--force`, обязательный для неинтерактивного выполнения.

## Задачи, поля и доступ

| Шаблон команды | Назначение и основные опции | Возможные REST-маршруты |
| --- | --- | --- |
| `task:add --title "Название" --creator USER_ID --responsible USER_ID` | Создать задачу; `--group-id`, `--deadline`, `--description`/`--description-file`. Отдельный advanced режим: `--fields`/`--fields-file`. | `tasks.task.add` (3.0), `tasks.task.field.list` (3.0) |
| `task:show TASK_ID` | Карточка; повторяемый `--select FIELD`. | `tasks.task.get` (3.0), `tasks.task.get` (1.0), `user.get` (1.0) |
| `task:list` | Выборка; повторяемые `--id`, `--select`, `--order FIELD:asc`; `--responsible`, `--group-id`, `--status`, `--due-before`, `--where`/`--where-file`, `--limit`, `--all`, `--max-scan`. | `tasks.task.list` (3.0) |
| `task:find --title "Подстрока"` | Поиск буквальной подстроки title без учёта регистра; `--group-id`, `--limit`, `--all`, `--max-scan`. | `tasks.task.list` (3.0) |
| `task:update TASK_ID --title "Название"` | Явный патч; `--creator`, `--responsible`, `--group-id`, `--deadline`, `--description`/`--description-file`. Отдельный режим `--fields`/`--fields-file`. | `tasks.task.update` (3.0), `tasks.task.field.list` (3.0) |
| `task:delete TASK_ID` | Удалить задачу; `--force`. | `tasks.task.delete` (3.0), `tasks.task.get` (3.0) |
| `task:fields:list` | Metadata всех полей; `--name FIELD` выбирает metadata одного поля. | `tasks.task.field.list` (3.0), `tasks.task.field.get` (3.0) |
| `task:access:show TASK_ID` | Права пользователя подключения на задачу. | `tasks.task.access.get` (3.0) |
| `task:assign TASK_ID --responsible USER_ID` | Изменить только исполнителя. | `tasks.task.update` (3.0) |
| `task:deadline:set TASK_ID --at ISO8601` | Изменить только срок, с явным часовым поясом. | `tasks.task.update` (3.0) |

`tasks.task.field.list` в создании/обновлении вызывается для advanced JSON.
`task:fields:list --name FIELD` использует `tasks.task.field.get` вместо list.
Поля metadata портала могут выходить за allowlist CLI; проверку writable/select
выполняет транспорт.

В обычной карточке используется REST 3.0. Для выбранных `auditors`/`accomplices`
добавляется чтение ID через REST 1.0; для имени/email и других доступных
projections — `user.get`. Например:

```sh
b24cli task:show 123 --select title --select auditors.id --json
b24cli task:show 123 --select accomplices --select auditors.email --json
```

Поддержанные projections участников: `id`, `name`, `email`, `gender`,
`externalAuthId`. Выбор роли без suffix означает `id/name`.
`strict-rest3` отклоняет обогащённую карточку до запроса.

## Участники

| Шаблон команды | Назначение и основные опции | Возможные REST-маршруты |
| --- | --- | --- |
| `task:participants:set TASK_ID --auditor USER_ID` | Повторяемые `--auditor`, `--accomplice`; `--clear-auditors`, `--clear-accomplices`. | `tasks.task.update` (1.0), `tasks.task.get` (3.0) |

Каждая указанная роль заменяется целиком; пропущенная роль сохраняется.
`--clear-*` явно очищает соответствующую роль.

## Чат задачи

| Шаблон команды | Назначение и основные опции | Возможные REST-маршруты |
| --- | --- | --- |
| `task:chat:list TASK_ID` | Сообщения; `--before MESSAGE_ID`, `--after MESSAGE_ID`, `--limit`, `--all`, `--max-messages`. | `tasks.task.get` (3.0), `im.dialog.messages.get` (1.0) |
| `task:chat:send TASK_ID --text "Текст"` | Отправить сообщение; альтернатива `--text-file PATH`. | `tasks.task.chat.message.send` (3.0) |
| `task:chat:update TASK_ID --message MESSAGE_ID --text "Текст"` | Изменить сообщение; альтернатива `--text-file PATH`. | `tasks.task.get` (3.0), `im.message.update` (1.0), `im.dialog.messages.get` (1.0) |
| `task:chat:delete TASK_ID --message MESSAGE_ID` | Удалить сообщение; `--force`. | `tasks.task.get` (3.0), `im.message.delete` (1.0), `im.dialog.messages.get` (1.0) |

Отправка возвращает boolean ACK и `resourceId=null`.
ID существующего сообщения выбирается через `task:chat:list`.
Перед изменением/удалением CLI проверяет его принадлежность чату задачи.
Для IM-маршрутов нужны scope `im` и права пользователя.

## Файлы

| Шаблон команды | Назначение и основные опции | Возможные REST-маршруты |
| --- | --- | --- |
| `task:file:attach TASK_ID --file-id FILE_ID` | Прикрепить уже загруженный файл Диска; `--file-id` можно повторять. | `tasks.task.file.attach` (3.0) |

Загрузка локального файла на Диск этой командой не выполняется.
Каждый ID обрабатывается отдельным шагом; проверяй результат всех шагов.
ACK attach сам по себе не является независимой проверкой membership readback.

## Учёт времени

| Шаблон команды | Назначение и основные опции | Возможные REST-маршруты |
| --- | --- | --- |
| `task:time:show TASK_ID` | Контекст задачи, в том числе `elapsedTime`. | `tasks.task.get` (3.0) |
| `task:time:add TASK_ID --seconds SECONDS` | Добавить запись; `--text`. | `task.elapseditem.add` (1.0) |
| `task:time:list TASK_ID` | Записи; JSON `--params` или `--params-file PATH`. | `task.elapseditem.getlist` (1.0) |
| `task:time:update TASK_ID --entry ENTRY_ID --seconds SECONDS` | Изменить запись; `--text`. | `task.elapseditem.update` (1.0), `task.elapseditem.getlist` (1.0) |
| `task:time:delete TASK_ID --entry ENTRY_ID` | Удалить запись; `--force`. | `task.elapseditem.delete` (1.0), `task.elapseditem.getlist` (1.0) |

`SECONDS` — положительное целое число секунд. Для `--params` доступны
`order`, `filter`, `select`, `params.NAV_PARAMS`;
используй `help task:time:list` для синтаксиса. Update/delete проверяют запись
внутри задачи. Время из карточки и сумма записей имеют разный контракт.

## Чек-листы

| Шаблон команды | Назначение и основные опции | Возможные REST-маршруты |
| --- | --- | --- |
| `task:checklist:add TASK_ID --title "Название"` | Создать корневой чек-лист. | `task.checklistitem.add` (1.0) |
| `task:checklist:list TASK_ID` | Показать корневые чек-листы. | `task.checklistitem.getlist` (1.0) |
| `task:checklist:item:add TASK_ID --checklist CHECKLIST_ID --title "Пункт"` | Добавить пункт; `--parent ITEM_ID` для вложенности. | `task.checklistitem.getlist` (1.0), `task.checklistitem.add` (1.0) |
| `task:checklist:item:list TASK_ID --checklist CHECKLIST_ID` | Показать все вложенные пункты корня. | `task.checklistitem.getlist` (1.0) |
| `task:checklist:item:update TASK_ID --item ITEM_ID --title "Пункт"` | Изменить заголовок пункта. | `task.checklistitem.getlist` (1.0), `task.checklistitem.update` (1.0) |
| `task:checklist:item:complete TASK_ID --item ITEM_ID` | Выполнить пункт. | `task.checklistitem.getlist` (1.0), `task.checklistitem.complete` (1.0) |
| `task:checklist:item:renew TASK_ID --item ITEM_ID` | Вернуть пункт в невыполненное состояние. | `task.checklistitem.getlist` (1.0), `task.checklistitem.renew` (1.0) |
| `task:checklist:item:delete TASK_ID --item ITEM_ID` | Удалить пункт и всех потомков; `--force`. | `task.checklistitem.getlist` (1.0), `task.checklistitem.delete` (1.0) |

`CHECKLIST_ID` — ID корневого узла; `--parent` относится к тому же корню.
Без `--parent` новый пункт добавляется непосредственно в корень.
Это операции пунктов чек-листа, без изменения lifecycle задачи.

## История изменений

| Шаблон команды | Назначение и основные опции | Возможные REST-маршруты |
| --- | --- | --- |
| `task:history:list TASK_ID` | Изменения полей; JSON `--params` или `--params-file PATH`. | `tasks.task.history.list` (1.0) |

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

Карта покрывает весь namespace `task` проверенного бинаря. CLI не имеет
универсальной команды вызова REST-метода. Lifecycle задачи (start, complete,
renew и другие переходы), результаты задач, шаблоны, отдельное управление
tags/userFields и загрузка файлов не представлены отдельными командами этой
сборки. Не конструируй их из названий REST-методов.

Для другого бинаря сначала выполни:

```sh
b24cli --version
b24cli list task --raw
b24cli help task:update
```

Семантика результата, credentials, API policy и лимитов описана в
[контракте транспорта](transport.md).
