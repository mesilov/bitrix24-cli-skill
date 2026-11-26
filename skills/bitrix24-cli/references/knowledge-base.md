# База знаний 2.0

Namespace `kb:*` читает и пишет статьи Базы знаний 2.0. Его нет в карте
0.2.0: команды добавлены в CLI для 0.3.0
([CLI #234](https://github.com/mesilov/bitrix24-cli/issues/234)). Справка
сверена 26.11.2026 по ветке CLI, source SHA
`1caf8cc375ada40bf3687eaa6018f3e0acb7213e`. Стабильная 0.3.0 ещё не опубликована.

## Перед использованием

Проверь, что установленный бинарь знает namespace:

```sh
b24cli list kb --raw
b24cli help kb:article:update
```

Если команд `kb:*` нет, это нехватка возможности CLI: действуй по workflow
из [SKILL.md](../SKILL.md). Старую базу знаний на лендингах CLI не поддерживает;
не читай и не пиши её другими средствами.

Портал должен быть на коммерческом тарифе, а вебхук — иметь scope `note`.
Отказ `permission-denied` или `scope-missing` — ошибка подключения, а не
отсутствие возможности CLI: сообщи её пользователю.

## Команды

К шаблону добавляй `b24cli` в начале и `--json` в конце. `COLLECTION_ID` —
ID базы знаний из `kb:list`, `ARTICLE_ID` — ID статьи из `kb:article:list`
или `documentId` из `kb:article:search`.

| Шаблон команды | Назначение и основные опции |
| --- | --- |
| `kb:list` | Все доступные базы знаний: `id`, `name`, `policyLevel`, `accessLevel`, `isArchived`. |
| `kb:article:list --collection COLLECTION_ID` | Дерево статей базы без архивных: `data.items[].children[]`. |
| `kb:article:search --query "ТЕКСТ"` | Поиск по заголовку и тексту во всех доступных базах; `--limit` от 1 до 200 (по умолчанию 50). Запрос — от 3 до 200 символов. |
| `kb:article:show ARTICLE_ID` | Статья в `data.article`, включая `markdown`, `updatedAt`, `isTrashed`. `--markdown` без `--json` выводит только содержимое. |
| `kb:article:add --collection COLLECTION_ID --title "Заголовок"` | Создать статью; `--parent ARTICLE_ID`, `--markdown-file PATH\|-`, `--allow-duplicate-title`, `--dry-run`. |
| `kb:article:update ARTICLE_ID --markdown-file PATH` | Заменить содержимое и/или `--title`; `--overwrite`, `--dry-run`. Нужен хотя бы один из `--title` и `--markdown-file`. |

## Найти и прочитать статью

1. Если пользователь назвал тему, начни с `kb:article:search --query`.
   `meta.hasMore = true` значит, что за пределами страницы есть ещё результаты:
   уточни запрос или увеличь `--limit`. `snippet` приходит как HTML с `<mark>`.
2. Для структуры конкретной базы используй `kb:list`, затем
   `kb:article:list --collection`. Exit 3 с `tree-truncated` — дерево неполное,
   ищи статью поиском.
3. Читай статью через `kb:article:show`. Статья из корзины тоже возвращается:
   проверь `isTrashed` и скажи об этом пользователю.

## Изменить статью

API не умеет частичную правку: `--markdown-file` **заменяет текст статьи
целиком**. Не отправляй фрагмент или только новый раздел — остальной текст
пропадёт. Порядок:

```sh
b24cli kb:article:show ARTICLE_ID --json > article.json
b24cli kb:article:show ARTICLE_ID --markdown > article.md
# изменить article.md, сохранив остальной текст
b24cli kb:article:update ARTICLE_ID --markdown-file article.md --dry-run --json
b24cli kb:article:update ARTICLE_ID --markdown-file article.md --json
```

- Сохрани `updatedAt` из первого чтения. Если перед записью он изменился,
  кто-то правил статью: перечитай её и согласуй изменения с пользователем.
  Одновременную правку API не блокирует, побеждает последняя запись.
- `--dry-run` показывает размер и SHA-256 нового содержимого, а не текст.
- `--overwrite` не передавай по умолчанию. Если запись отклонена с подсказкой
  про `--overwrite`, у статьи есть несохранённые правки в совместном редакторе.
  Повторяй с `--overwrite` только после явного согласия пользователя: чужие
  правки будут потеряны.
- Только заголовок меняй через `--title` без `--markdown-file`.

## Создать статью

`--title` обрезается по краям, длина 1–255 символов. Содержимое передаётся
файлом или stdin (`-`), до 1 048 576 байт, без изменений. Дочерняя статья
(`--parent`) должна быть в той же базе знаний.

Перед записью CLI проверяет дубль у того же родителя:

- exit 4 `article-exists` — статья с таким заголовком уже есть, ID в
  `error.details.candidates`. Предложи пользователю обновить её через
  `kb:article:update`, а не создавать вторую.
- exit 4 `search-incomplete` — дерево неполное, проверка невозможна.
- `--allow-duplicate-title` передавай только по явной просьбе пользователя.
  Архивные статьи проверка не видит.

## Результат записи

- `data.operation.readback = verified` — портал вернул ту же статью.
  Exit 3 с `readback: mismatch` — отличаются `title`, `collectionId` или
  `parentId`, список в `operation.mismatch`.
- `operation.markdownNormalized = true` — портал сохранил текст в другом виде;
  запись подтверждена, сохранённый текст — в `data.article.markdown`.
- `meta.outcomeUnknown = true` — исход неизвестен. Не повторяй запись:
  проверь `kb:article:list` после создания или `kb:article:show` после обновления.

Архивирование и удаление статей, создание и изменение самих баз знаний,
вложения и файлы в `kb:*` не входят.
