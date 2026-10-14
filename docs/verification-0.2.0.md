# Проверка карты bitrix24-tasks для 0.2.0

Дата: 14.10.2026. Задача: [skill #6](https://github.com/mesilov/bitrix24-cli-skill/issues/6),
связь с [CLI #100](https://github.com/mesilov/bitrix24-cli/issues/100).
Основной источник — текущий **dev**, по поручению пользователя.

## Источник dev

- Source SHA: `1ed4422a8d7a33138134d6a8763594820c75c256`.
- [Карта приложения](https://github.com/mesilov/bitrix24-cli/blob/1ed4422a8d7a33138134d6a8763594820c75c256/docs/cli-command-map.md).
- OpenSpec `verify-cli-help-and-docs-0-2-0`:
  [архив](https://github.com/mesilov/bitrix24-cli/tree/1ed4422a8d7a33138134d6a8763594820c75c256/openspec/changes/archive/2026-10-14-verify-cli-help-and-docs-0-2-0).
  Его исторический каталог описывает ранний snapshot на 46 команд;
  новая карта построена по фактическому текущему CLI на 56, без копирования планов.
- Dev launcher без build-info.json сообщает `b24cli 0.1.0`. Поэтому provenance
  текущей карты определяется source SHA и реально исполненным list/help,
  а не ошибочным заявлением о стабильном релизе 0.2.0.
- Source CLI исполнялся в Docker (PHP 8.5, зависимости из composer.lock),
  checkout и vendor подключены read-only, network=none, HOME/config-dir
  временные, реальные dotenv/credentials не читались.

## Каталог и справка

- `b24cli list --format=json`: 56 публичных команд, 43 task / 4 users /
  5 connection / 1 doctor / 3 общих. Скрытая Symfony `_complete` исключена.
- В таблицах commands.md перечислены ровно 56/56 команд, без дублей.
- Все flags в строках таблиц сверены с публичными definitions list JSON.
- Обе формы root help (`--help`/`-h`), `--about`, `list --help` и обе формы
  help каждой публичной команды прошли: **112 командных help-вызовов**.
- Все fenced JSON примеры разбираются. Shell CLI примеры разобраны с учётом
  продолжений строк и их flags сверены с definitions соответствующих команд.
- Quick_validate skill-creator и ссылки устанавливаемого пакета проверены.
- [Машиночитаемый результат](verification-0.2.0.json) содержит точные counts,
  command names/options и SHA-256 фактической help каждой команды.

## Standalone RC

Дополнительная проверка выполнена на macOS arm64 вне checkout с пустым
окружением подключения; основной контракт карты остаётся dev.

| Параметр | Значение |
| --- | --- |
| Версия | `0.2.0-rc-1` |
| Source SHA | `9c268e5bbb6b3c3cea85cfbb3726cbd8b5efbb99` |
| [Run](https://github.com/mesilov/bitrix24-cli/actions/runs/38537920077) | completed / success, source совпадает с metadata |
| [Artifact set](https://github.com/mesilov/bitrix24-cli/actions/runs/38537920077/artifacts/11881912444) | b24cli-release-assets, не истёк на дату проверки |
| macOS archive SHA-256 | `f402ffaaf74f92412593354b57e0ac5799a7a2785e559019d1bcd073f47f08c2` |
| Executable SHA-256 | `ee0b9e56d814ce117a546addd5e439352277c2183209e266da14249eb71975ef` |

Архив совпал с SHA256SUMS; build-info подтвердил версию/source, strict codesign
прошёл. Root help/about/list и 112 public command-help вызовов прошли.
Имена команд и definitions flags совпадают с dev. Единственный изменённый
публичный catalog record — task:status:durations: RC считает календарные
секунды, dev после [#136](https://github.com/mesilov/bitrix24-cli/issues/136)
исключает субботу/воскресенье и добавляет seconds/hours/days. Эти результаты
разделены в task-workflows.md; RC не выдан за проверку нового расчёта.

## Выполнение публичных примеров

**42 сценария** source CLI выполнены с network=none и `meta.apiCalls=[]`:

- 12 успешных dry-run: basic создание/обновление, назначение, срок, чат,
  запись времени, чек-лист, Disk ID, локальный файл, теги/очистка, native результат.
- Корректный ввод lifecycle/result update/delete/list, user search/org,
  role/temporal filters, task:my, custom JSON и projections доходит до ожидаемой
  ошибки отсутствующего подключения (exit 1), не до usage error.
- Три неверных комбинации дают exit 2; три несовместимых strict-rest3 плана
  дают exit 4 до подключения.
- Все пять connection команд выполнены с синтетическим webhook в временном
  config-dir. Observer/manager defaults входят в создание, существующий alias
  не обновляется (configuration-error / exit 1), use/remove меняют только
  локальный выбор. Doctor status выполняется без подключения.

Это проверка синтаксиса, локальной валидации, plan, профилей и policy boundaries.
Exit 1 на корректном сетевом запросе не доказывает успех серверной операции.
Live portal acceptance в этом репозитории не выполнялась.

## Поведение скилла

До обновления отдельный агент проверил пять реалистичных запросов: native
result/completion, локальный файл, мои открытые завтра, поиск сотрудника и
произвольный auditor ID, длительности статусов. Старый пакет не давал полного
проверенного маршрута для всех пяти; также выявлены устаревший issue fallback
в карте и недостаточная оговорка temporal mode в разделе пагинации.

После обновления отдельный агент подтвердил выбор поддержанных команд и
flags, различие completed/supposedly_completed, отсутствие folder search,
четыре штатные выборки для открытых статусов завтра, остановку на произвольном
ID наблюдателя и разные semantics RC/dev для длительностей. Согласованы
source SHA всех текущих документов. Capability-gate, запрет прямых REST/SDK/
curl/MCP обходов и существующий doctor consent/mock workflow сохранены.

## Границы доставки

Последний стабильный опубликованный CLI на дату проверки — 0.1.0; новая
карта ориентирована на dev для 0.2.0, команды которой пользователь считает
зафиксированными. Проверка финальных стабильных артефактов остаётся в
[release #109](https://github.com/mesilov/bitrix24-cli/issues/109).
Историческая [проверка 0.1.0](verification.md) сохранена отдельно.
Linux/Windows standalone и native UI Claude Code/Codex/OpenCode здесь
повторно не запускались. RC run имеет отдельные native receipts приложения;
они не приписываются этой проверке скилла. Установленный пользовательский
пакет не заменялся. Внешний feedback и issue автоматически не создавались.
