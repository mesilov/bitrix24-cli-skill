# Changelog

## [Unreleased]

### Добавлено

- Репозиторный `bitrix24-cli-skill-maintainer`: сопровождение пакета через
  публичный CLI, проверка поставки и обязательная содержательная запись
  в changelog в каждом PR с изменениями. Diff changelog проверяется перед
  открытием PR и непосредственно перед merge
  ([#8](https://github.com/mesilov/bitrix24-cli-skill/issues/8)).
- Справочник [Базы знаний 2.0](skills/bitrix24-cli/references/knowledge-base.md)
  для команд `kb:list`, `kb:article:list`, `kb:article:search`,
  `kb:article:show`, `kb:article:add` и `kb:article:update`. Скилл включает их
  только если установленный `b24cli` показывает namespace `kb`, и описывает
  правку статьи как «прочитать → изменить локально → записать полный текст»:
  обновление заменяет содержимое целиком, `--overwrite` передаётся только
  с согласия пользователя. Описание скилла, метаданные Codex и README упоминают
  Базу знаний. Справка сверена с веткой CLI
  [#234](https://github.com/mesilov/bitrix24-cli/issues/234); команды не входят
  в опубликованный CLI 0.2.0
  ([#16](https://github.com/mesilov/bitrix24-cli-skill/issues/16)).

### Изменено

- Пользовательский скилл переименован из `bitrix24-tasks` в `bitrix24-cli`.
  Обновлены каталог пакета, имя в Codex, команды вызова и инструкции перехода
  для существующих установок
  ([#14](https://github.com/mesilov/bitrix24-cli-skill/issues/14)).

### Документировано

- README упрощён: удалён раздел «Состав» с перечнем файлов, сведениями
  о сверке бинарей и ссылками на связанные задачи; инструкции по установке
  и подключению сохранены
  ([#11](https://github.com/mesilov/bitrix24-cli-skill/issues/11)).

- Зафиксировано покрытие относительных календарных периодов дедлайна для
  `task:list`/`task:find` через `--deadline-window`, `--overdue` и `--timezone`.
  Карта уже находится в `main`; интерфейс подтверждён на CLI `0.2.0-rc-1`,
  что не означает публикацию стабильного CLI 0.2.0. Дополнительные периоды,
  назначение относительных сроков и фильтры создания/завершения выделены
  в [CLI #141](https://github.com/mesilov/bitrix24-cli/issues/141) на 0.3.0
  ([#4](https://github.com/mesilov/bitrix24-cli-skill/issues/4)).
