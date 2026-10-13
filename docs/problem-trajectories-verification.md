# Проверка расширения doctor

Дата: 2026-10-13. Связь: https://github.com/mesilov/bitrix24-cli/issues/96,
OpenSpec `analyze-agent-trajectories` в репозитории CLI. Базовая историческая
проверка релиза 0.1.0 в verification.md сохраняется; doctor проверялся по
исходникам feature/96-agent-trajectory-feedback, не по опубликованному релизу.

- Skill creator quick_validate прошёл; локальные Markdown links пакета разрешаются.
- В SKILL.md присутствует inventory/help gate, один startup status и ссылка на
  workflow. Без doctor действует обычный missing-capability/issue fallback.
- Reviewed decision cases: normal multi-call/JSON parsing не trigger; missing
  capability сохраняется через CLI; disabled/full/error не выдаются за saved;
  false analysisDue не отвлекает пользователя; decline не означает согласия;
  недоступная проверка duplicates остаётся unchecked; нет прямого FS/HTTP обхода.
- JSON report из самого references/problem-trajectories.md и snapshot из CLI docs
  выполнены реальным public CLI в Docker с network=none и временным config-dir.
  Проверено 6 этапов: read-only status, report file, weekly offer/suppression,
  publication preview, default endpoint-unavailable, explicit mock/reconcile.
- Full CLI unit/source/contract checks сохранены в его OpenSpec verification.md.
  Mock показывает mocked=true/delivered=false; real feedback и GitHub issue не
  создавались. `--confirm-publication` относится к выбранному просмотренному пакету.

Это проверка структуры, инструкций и исполнимости публичных примеров. Native UI
Codex/Claude/OpenCode, актуализация установленного пользовательского пакета,
Windows/macOS standalone doctor и live endpoint не выполнялись. Расписание,
чаты/rollout files и прямой Bitrix24 API отсутствуют в расширении скилла.
