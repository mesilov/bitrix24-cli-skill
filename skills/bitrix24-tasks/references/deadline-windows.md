# Дедлайны и просрочка

Используй эту карту только если `b24cli help task:list` или `b24cli help task:find`
подтверждает новые flags. Базовая версия 0.1.0 их не предоставляет;
версию установленного бинаря проверяй через `--version`, возможности — по help.
Если параметра нет, следуй разделу «Если возможностей CLI недостаточно» в SKILL.md.
Эта карта сопровождает [CLI issue #95](https://github.com/mesilov/bitrix24-cli/issues/95).

| Запрос пользователя | Аргументы CLI |
| --- | --- |
| Дедлайн сегодня | `--deadline-window today` |
| Дедлайн завтра | `--deadline-window tomorrow` |
| Дедлайн вчера | `--deadline-window yesterday` |
| Дедлайн позавчера | `--deadline-window day-before-yesterday` |
| Дедлайн на этой неделе | `--deadline-window this-week` |
| Дедлайн в этом месяце | `--deadline-window this-month` |
| Дедлайн в этом квартале | `--deadline-window this-quarter` |
| Просроченные задачи | `--overdue` |
| Должны были закрыть вчера / позавчера | `--overdue` + `--deadline-window yesterday` / `day-before-yesterday` |
| Должны были закрыть на этой неделе / в этом месяце / квартале | `--overdue` + `--deadline-window this-week` / `this-month` / `this-quarter` |

```sh
b24cli task:list --group-id 42 --deadline-window today --timezone Asia/Bishkek --json
b24cli task:list --group-id 42 --overdue --deadline-window this-week --timezone Asia/Bishkek --json
b24cli task:find --title "отчёт" --group-id 42 --overdue --deadline-window this-month --timezone Asia/Bishkek --json
```

Подставляй известный ID проекта и IANA timezone пользователя; не принимай 42 или
Asia/Bishkek из примера за defaults. Если timezone неизвестен и влияет на смысл
«сегодня», уточни его; при отсутствии flag CLI использует TZ или timezone PHP
окружения и сообщает выбранный пояс в `meta.deadlineFilter`.

Окна календарные: неделя начинается в понедельник, месяц и квартал охватываются
целиком. Они не означают следующие 7/30/90 дней. Skill не вычисляет даты сам.
CLI фиксирует один now, рассчитывает `[from, to)` и отправляет даты на сервер.
`--overdue` означает непустой deadline<now, исключает completed/declined и включает
deferred/supposedly_completed. Без `--overdue` окно не ограничивает статус.
«Должны были закрыть» здесь относится к дедлайну; запрос о фактическом закрытии
требует отдельной возможности CLI, не заменяй его фильтром deadline.

Новый временной режим использует объявленный CLI-маршрут REST 1.0;
`--api-policy strict-rest3` отклоняет его с exit4. Для ответа используй только
public CLI: не переключайся на HTTP, SDK или MCP. Не смешивай новые temporal flags
с `--where`/`--where-file` или `--due-before`; задавай одно окно.

Проект, роль, название и статус пересекаются через AND. `--limit` ограничивает
вывод, `--max-scan` — обход; `--all` не снимает бюджет. При exit3/complete=false
не заявляй полноту проекта. Разбирай `meta.deadlineFilter`: window, timezone,
now, fromInclusive/toExclusive, route и место применения фильтров.
