# Рабочие сценарии 0.2.0

Контракт текущего `dev` на 14.10.2026:
`1ed4422a8d7a33138134d6a8763594820c75c256`. Используй публичные команды;
справка установленной сборки определяет доступные flags. Для каждого вызова
сохраняй exit code и `data/meta/error`, в том числе при partial/exit 3.

## Завершение и native результат

Для перехода используй lifecycle команду, не raw status в task:update.
CLI проверяет текущее состояние, право конкретного действия и требование
результата. План может показать контроль и завершение подзадач сервером.
Native результат — отдельная сущность; сообщение в чате или файл его не заменяет.

```sh
b24cli --profile NAME task:complete 123 --dry-run --json
b24cli --profile NAME task:result:list 123 --all --json
b24cli --profile NAME task:result:add 123 --text-file "/absolute/path/to/result.txt" --dry-run --json
```

Если требуется результат и подходящего нет, согласуй его содержание, затем
выполни `task:result:add` без dry-run и проверь подтверждение/ID. Завершай
`task:complete` только в рамках поручения пользователя. Result list/update/delete
работают с ID native результата; update/delete проверяют принадлежность задаче.
Создание результата само по себе не завершает задачу. CLI его автоматически не добавляет.

После перехода `data.operation` содержит `taskId/action/confirmed`,
`data.task` — `id/status/needsControl/requireResult`. Фактический
`supposedly_completed` означает ожидание контроля; `completed` — завершение.
Не выводи статус только из `needsControl` или названия команды.
`meta.complete` описывает полноту выполнения и чтения, а не статус задачи.
Подтверждённая запись с неудачным readback даёт partial: используй task:show.
При `outcomeUnknown=true` не повторяй переход автоматически.
Lifecycle недоступен при `--api-policy strict-rest3`; результаты доступны.

## Локальный файл или существующий Disk ID

```sh
b24cli --profile NAME task:file:attach 123 \
  --file "/absolute/path/to/report.pdf" --folder-id 456 \
  --name "Отчёт.pdf" --dry-run --json
```

Заменяй TASK_ID, folder ID и путь данными запроса. CLI не выбирает папку
автоматически; её поиск в этом каталоге отсутствует. Режим `--file` загружает
один обычный файл, затем привязывает его к задаче; нужны права disk/tasks.
Не смешивай с повторяемыми `--file-id`. Stdin (`-`), URL и специальные файлы
не поддерживаются. Имя — UTF-8, до 255 символов, без разделителей пути и
управляющих символов; default — basename. Совпадение имени не перезаписывает файл.

Лимит default — 500 MB (500000000 байт). `maxUploadBytes` в runtime JSON
`--config` — положительное целое число байт; может уменьшать и увеличивать лимит.
Dry-run проверяет локальный ввод и показывает план без API, credentials и
содержимого файла. Локальный режим недоступен при strict-rest3.

После partial с подтверждённой загрузкой используй возвращённый Disk ID
с `--file-id` для повторения только привязки, проверив задачу/права.
Не загружай файл повторно. При outcomeUnknown сначала проверь сведения о
папке/привязке доступными публичными командами; если CLI не позволяет доказать
состояние, сообщи неопределённость и остановись. Подтверждение attach и
независимое доказательство наличия файла — разные результаты.

## Теги

`--tag` у task:add/task:update повторяется и задаёт полный новый набор.
Без опции теги сохраняются; `task:update --clear-tags` явно очищает набор
и не смешивается с `--tag`. Теги пишутся отдельным шагом, недоступным
при strict-rest3. После partial создания сохрани TASK_ID, проверь
`task:show TASK_ID --select tags.id --select tags.name --json` и повторяй
только нужный task:update, не task:add.

## Пользовательские поля

`--user-fields` либо `--user-fields-file PATH|-` задаёт непустой JSON-объект
с точными `UF_*` кодами выбранного портала. Поддержаны строки, числа,
booleans и плоские списки; `null` и вложенные объекты отклоняются.
Системные `UF_CRM_TASK`, `UF_TASK_WEBDAV_FILES`, `UF_MAIL_MESSAGE` запрещены.
Пустая строка/список явно очищает значение; пропущенные поля сохраняются.
Схему читает CLI; обязательность и множественность окончательно проверяет портал.

Предпочитай basic flags с отдельным JSON custom полей, например:

```sh
b24cli --profile NAME task:update 123 \
  --user-fields-file "/absolute/path/to/user-fields.json" --dry-run --json
b24cli --profile NAME task:show 123 --select userFields.key --select userFields.value --json
```

Файл может содержать `{"UF_QUERY":"6"}` только если такой код есть на портале;
это пример, не default. Custom dry-run читает схему и требует подключения.
При custom создании advanced `--fields` использует точные ключи `TITLE`,
`CREATED_BY`, `RESPONSIBLE_ID`, `GROUP_ID`, `DESCRIPTION`, `DEADLINE`,
`ALLOW_TIME_TRACKING` (`Y`/`N`). Без custom input advanced режим использует
camelCase (`title`, `creatorId`, `responsibleId`, `allowsTimeTracking` с boolean).
Не смешивай advanced JSON с basic flags; CLI не переименовывает ключи сам.

Составной task:update меняет ordinary поля, затем custom, затем явно заданные
теги. Partial сохраняет TASK_ID и подтверждённые шаги; проверь существующую
задачу перед повторением только неуспешных изменений. Автоматического rollback
или retry записи нет. Custom режим недоступен при strict-rest3.

## Длительности статусов

```sh
b24cli --profile NAME task:status:durations 123 --json
```

`--as-of` задаёт ISO8601 с часовым поясом, не в будущем; default — момент
до чтения данных. В **dev для 0.2.0** суббота и воскресенье исключены по
серверному offset `created`. Остальные часы, включая завершённый статус,
учитываются полностью; день = 24 часа. Это время состояния, не трудозатраты
и не производственный календарь праздников.

`data.totals` по каждому статусу содержит `seconds` (целое), `hours`/`days`
(до двух знаков, независимое округление). Интервалы имеют те же единицы;
повторные открытия создают отдельные интервалы. Проверяй
`meta.semantics=weekdays-only` и `meta.calendar`, сохраняя исходные timestamps.
При неоднозначном начале/порядке, race, неизвестных статусах или неполной
истории CLI выдаёт `complete=false`, суммы null и exit 3. Null означает
«неизвестно», не ноль. `elapsedBetweenEventsSeconds` — разница меток,
не доказанное время состояния. Старые события вынесены в excludedHistoryEvidence.
Нельзя заменять этот результат `elapsedTime` или суммой записей task:time:list.
Команда недоступна при strict-rest3.

**Замороженный бинарь 0.2.0-rc-1** предшествует этому изменению: там
`meta.semantics=calendar-seconds`, суммы — календарные секунды, включая выходные,
без нового объекта seconds/hours/days. Не приписывай ему контракт dev;
используй версию, help и фактическую semantics в JSON.
