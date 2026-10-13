# Проблемные сценарии и пакетный разбор

Используй этот workflow только при наличии `doctor:problem-trajectory` в
`b24cli list --raw` и соответствующих actions/flags в help установленного CLI.
Базовая карта скилла относится к 0.1.0; наличие doctor определяется справкой,
а не предположением по версии. Скилл не требует Codex-specific инструментов,
доступа к истории чатов или отдельного background job.

## При запуске скилла

Один раз за запуск, если doctor доступен:

```sh
b24cli doctor:problem-trajectory --action status --json
```

Передай известный `--config-dir`, если он выбран в этом сеансе. Каталог,
telemetry defaults, даты и счётчики определяет CLI. Не читай config.json или
`.problem-trajectories` напрямую, не считай дни shell/Python-скриптом.
Status не требует webhook и не создаёт config. При ошибке сообщи её; обычная
поддержанная задача может продолжаться, если doctor не нужен для её выполнения.

Если `data.analysisDue=false`, не предлагай разбор. Если true, один раз
предложи обработать накопленные карточки. После показа предложения отметь его:

```sh
b24cli doctor:problem-trajectory --action record-offer --json
```

Отказ/отсутствие ответа не является согласием. Не анализируй и не отправляй
пакет; CLI сохраняет отдельную Offered date и не будет предлагать снова семь
суток. Пользователь может явно запросить разбор раньше. Без запуска скилла
фоновая обработка не происходит.

## Когда не хватает возможности

Сначала проверь нужные команды/флаги в help. Несколько штатных вызовов и
обычный JSON parsing допустимы. Ошибка прав/credentials или неполный ответ
не доказывают отсутствие команды. Если необходима собственная реализация
недостающей функции, останови этот путь; не исполняй обвязку ради регистрации.

Подготовь обезличенный report: schemaVersion=1, sanitizedScenario,
availableCommands (только имена команд/флагов), missingCapability,
wrapperAlgorithm (описания шагов), expectedOutcome, фактическая cliVersion,
skillVersion=null если версия пакета неизвестна.

Убери ФИО, контакты, portal domains, пути, персональные IDs, названия/тексты
реальных задач, переписку, credentials и raw argv/env/output. Используй
обобщённую цель и синтетические примеры. Не помещай исполняемый скрипт или
токен в текст. Одних regex-проверок CLI недостаточно для анонимности текста.

```json
{
  "schemaVersion": 1,
  "sanitizedScenario": "Получить сгруппированную сводку задач",
  "availableCommands": ["task:list", "--json"],
  "missingCapability": "Нативная агрегация списка",
  "wrapperAlgorithm": ["Прочитать список", "Сгруппировать по заданному критерию"],
  "expectedOutcome": "Полная сводка через публичную команду",
  "cliVersion": "0.1.0",
  "skillVersion": null
}
```

Пример показывает формат, не доказывает дефект текущего CLI. Версию в своём
report бери из `--version`. Передай подготовленный JSON через временный файл
или redirected stdin; это только сериализация публичного входа, без собственной
реализации хранения/алгоритма. Например:

```sh
b24cli doctor:problem-trajectory --report-file report.json --idempotency-key '<UNIQUE_EPISODE_KEY>' --json
```

До выполнения замени `<UNIQUE_EPISODE_KEY>` новым уникальным opaque ключом
этого эпизода (1..64 символа `[A-Za-z0-9_-]`). Агент выбирает ключ сам и
сохраняет его вместе с подготовленным JSON для retry; placeholder не является
готовым значением. Для другого эпизода нужен другой ключ, даже при похожем
сценарии. После неопределённого результата повторяй тот же key/JSON,
не создавай новую карточку на каждый retry.
Не используй в key персональные IDs, пути или контакты.
Проверь exit code и `data.cardId/state`. При сохранении сообщи пользователю
недостающую возможность и сохранённый ID; предложи immediate issue либо
следующий пакетный разбор, не требуя ручного issue для каждого случая.
Collection-disabled/storage-full/ошибка очистки означают, что запись не сохранена;
не утверждай обратное. Без doctor сообщи ограничение и предложи обычный issue,
сохраняя запрет обходов. Не устанавливай или обновляй бинарь без поручения.

## Разбор после согласия

```sh
b24cli doctor:problem-trajectory --action list --state pending --json
```

При `nextOffset` продолжи чтение через `--offset`; сохрани конкретные cardIds
snapshot. Порции до 100; большие накопления разбирай несколькими snapshot.
Сравни цели, missing capability и описания алгоритмов. Группируй повторения,
проверяй установленный CLI/help и existing issues по уже имеющимся проверенным
сведениям/ссылкам пользователя или публичной функции CLI, если она доступна.
Если проверить дубликаты не удалось, ставь deduplication=unchecked и не отправляй
кандидат до пакетной проверки; не добавляй собственный HTTP/SDK/MCP обход.

Владелец: cli, skill/docs, environment или unclear. Один report — hypothesis;
не называй число карточек количеством пользователей или процентом всех запусков.
Синтетическое воспроизведение отделяй от наблюдённого поведения. Выбирай
candidate, linked-existing (с canonical issue link) или dismissed.

Каждый group содержит groupId, cardIds, owner, title, expected, actual,
syntheticReproduction, acceptanceCriteria, confidence, deduplication, decision,
issueLinks. Корень batch: schemaVersion=1, opaque batchId, groups. Используй
разные batchId для разных решений; повторяется только тот же payload.

```sh
b24cli doctor:problem-trajectory --action record-analysis --batch-file analysis.json --json
```

Выдай один digest с count of reports, владельцами и предложениями. В publication
CLI добавляет версии CLI/скилла из выбранных карточек; unknown означает
неизвестную версию скилла, её нельзя угадывать. Покажи
`data.groups[].publication` — это текст, который будет использован при отправке.
CLI сохраняет анализ и Processing date; новые карточки вне snapshot остаются
pending. Само завершение анализа не является согласием на передачу.

## Согласованная репетиция feedback

В MVP #96 пользователь разрешил **пока мокать отправку**. Используй уже данное
согласие на конкретный проверенный пакет; если его нет, получи одно пакетное
согласие на выбранный publication text и mock rehearsal. Не включай mock
молча вместо реальной отправки. Selection JSON: только
`{"groupRefs":["saved groupRef"]}`; реальные значения бери из CLI результата.

```sh
b24cli doctor:problem-trajectory --action submit --batch-file selection.json \
  --confirm-publication --mock-feedback --json
```

**mocked=true/delivered=false** и mock-submitted означают локальную репетицию.
Нет отправленного обращения, внешнего moderation или GitHub issue. Без
`--mock-feedback` endpoint сейчас недоступен; сообщи это и сохрани пакет.
Не отправляй его через curl, SDK, MCP или прямой GitHub API как обход CLI.
Обычный issue можно предложить пользователю, как в правиле missing capability.

Для неизвестного результата используй публичное reconcile в том же режиме:

```sh
b24cli doctor:problem-trajectory --action reconcile --batch-file selection.json \
  --mock-feedback --json
```

При exit 3 проверь результат каждой группы и meta.outcomeUnknown. Не повторяй
неизвестный submit автоматически. Receipt и issue/release — разные стадии.
Mock карточки не удаляются как доставленные; явный dismiss можно сохранить
новым batch после решения пользователя.

При storage-recovery-required доступно `--action recover --json`; не удаляй
служебный journal и не пытайся чинить конфигурацию файлами самостоятельно.
Настройки меняются только `--action configure --settings-file` (strict boolean
collectProblemTrajectories и positive integer maxProblemTrajectoryFiles).
