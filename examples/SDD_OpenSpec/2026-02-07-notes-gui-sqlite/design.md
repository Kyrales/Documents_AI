# Design: Desktop Notes (Python GUI + SQLite)

## Архитектура
- **Стиль:** локальное desktop-приложение (монолит) с четким разделением слоев
- **Слои:**
  - **UI (GUI):** формы и события (Tkinter)
  - **Application Service:** сценарии (create/update/delete/search/export/import)
  - **Repository (Data Access):** операции с SQLite
- **Цели дизайна:**
  - минимальная связность UI и хранения
  - тестируемость логики без GUI

## Модель данных

### Сущности
- **Note**
  - `id` (целое, PK)
  - `title` (строка, обязательна)
  - `body` (строка)
  - `is_pinned` (bool)
  - `created_at` (ISO-строка)
  - `updated_at` (ISO-строка)
- **Tag**
  - `id` (целое, PK)
  - `name` (строка, уникальна в нормализованном виде)
- **NoteTag** (m:n)
  - `note_id`, `tag_id` (составной уникальный ключ)

### Схема SQLite (v1)
```sql
PRAGMA foreign_keys = ON;

CREATE TABLE IF NOT EXISTS schema_version (
  version INTEGER NOT NULL
);

CREATE TABLE IF NOT EXISTS notes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  body TEXT NOT NULL DEFAULT '',
  is_pinned INTEGER NOT NULL DEFAULT 0,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_notes_pinned_updated
  ON notes(is_pinned DESC, updated_at DESC);

CREATE TABLE IF NOT EXISTS tags (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  name_norm TEXT NOT NULL UNIQUE
);

CREATE TABLE IF NOT EXISTS note_tags (
  note_id INTEGER NOT NULL REFERENCES notes(id) ON DELETE CASCADE,
  tag_id INTEGER NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (note_id, tag_id)
);

CREATE INDEX IF NOT EXISTS idx_note_tags_tag_id ON note_tags(tag_id);
```

### Поиск
- **Базовый режим:** `LIKE` по `lower(title)` и `lower(body)` для простоты.
- **Опционально:** FTS5 таблица `notes_fts` для ускорения поиска на 5k+ записей.
- **Поведение:** если FTS5 недоступен, используется базовый режим.

## Интерфейсы и API (внутренние)

### Application Service
- `create_note(title, body, tags, is_pinned) -> Note`
- `update_note(note_id, title, body, tags, is_pinned) -> Note`
- `delete_note(note_id) -> None`
- `search_notes(query, tag=None) -> list[NoteSummary]`
- `export_json(path) -> ExportStats`
- `import_json(path) -> ImportStats`

### Repository
- `get_note(note_id) -> Note | None`
- `list_notes(tag=None, query=None) -> list[NoteSummary]`
- `insert_note(...) -> note_id`
- `update_note(...) -> None`
- `delete_note(note_id) -> None`
- `upsert_tags(names) -> list[tag_id]`
- `set_note_tags(note_id, tag_ids) -> None`

## Зависимости (Dependencies)
- **Python:** 3.11+
- **GUI:** Tkinter (stdlib)
- **DB:** sqlite3 (stdlib)
- **Формат экспорта:** json (stdlib)
- **Логирование:** logging (stdlib)

## Наблюдаемость (Observability)
- **Логирование:**
  - уровень INFO по умолчанию, DEBUG по флагу запуска
  - события: запуск, открытие БД, миграции, импорт/экспорт, ошибки сохранения
  - маскирование: не логировать body заметок; допустим только длина текста
- **Метрики (локальные):**
  - длительность операций `search`, `save`, `import`, `export` (в логах)
- **Диагностика ошибок:**
  - сообщения пользователю: кратко и понятно
  - технические детали — в лог

## Безопасность
- **Аутентификация/Авторизация:** не требуется (однопользовательский локальный режим)
- **Данные:**
  - SQL выполняется только параметризованно
  - путь к файлу БД выбирается пользователем или задается настройкой
  - не логировать содержимое заметок, чтобы не утекали данные в логи
- **Уязвимости:**
  - защита от SQL-инъекций параметризацией
  - импорт JSON валидируется по схеме (минимальный набор обязательных полей)

## Потоки данных
- **Создание/сохранение:**
  - UI собирает поля → Service валидирует → Repository пишет в SQLite → Service возвращает модель → UI обновляет список
- **Поиск:**
  - UI передает `query/tag` → Repository выполняет SQL/FTS → UI перерисовывает список
- **Экспорт/импорт:**
  - Service читает из Repository → сериализация JSON → запись на диск
  - импорт: чтение JSON → дедупликация (по `title` + `created_at` в учебном режиме) → вставка/привязка тегов

## Риски и меры
- **Смешение UI и логики:** выделить Service слой; UI вызывает только сценарии.
- **Коррупция/блокировка SQLite:** обрабатывать исключения, показывать режим восстановления (создать новую БД/выбрать другую).
- **FTS5 не в сборке:** предусмотреть fallback на `LIKE`, предупредить в логах.
- **План отката (Rollback):**
  - перед миграцией создавать копию файла БД рядом: `db.sqlite.bak`
  - при ошибке миграции — восстановление из `.bak`
  - миграции применяются по `schema_version` строго последовательно
- **Feature Flags:**
  - `use_fts` (auto/on/off) для выбора поиска

## План миграции
- `schema_version` хранит текущую версию.
- При запуске: сравнить версию в БД с версией приложения, применить миграции `vN -> vN+1` транзакционно.
