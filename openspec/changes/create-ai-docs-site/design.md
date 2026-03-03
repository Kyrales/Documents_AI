# Design: AI Docs Site

## 1. Архитектура (Architecture)
*   **Framework:** Astro 5.0 (или latest stable).
*   **Theme:** `@astrojs/starlight` (официальная тема документации).
*   **Сборка:** Static Site Generation (SSG). Результат — папка `dist/` с чистым HTML/CSS/JS.
*   **Хостинг:** GitHub Pages.

## 2. Реализация стиля (Neo-Brutalism)
Мы не будем писать всё с нуля, а кастомизируем Starlight.

### Технологии
*   **Tailwind CSS:** Подключаем через интеграцию `@astrojs/tailwind`.
*   **CSS Variables:** Переопределяем стандартные переменные Starlight в `src/styles/custom.css`.

### Дизайн-система
*   **Цвета:**
    *   Фон: Белый (`#ffffff`) или светло-бежевый (`#f0f0f0`).
    *   Текст: Черный (`#000000`).
    *   Акцент: Неоновый лайм (`#ccff00`) или Ярко-розовый (`#ff00ff`) — выберем один.
*   **Формы:**
    *   `border-radius: 0` (везде).
    *   `border: 2px solid black` (кнопки, карточки, сайдбар).
    *   `box-shadow: 4px 4px 0 0 black` (эффект "парящих" блоков).
*   **Типографика:**
    *   Заголовки: System Fonts (Arial/Helvetica) или Google Fonts (Space Grotesk).
    *   Код: Fira Code / JetBrains Mono.

## 3. Структура проекта
```text
/
├── astro.config.mjs      # Конфиг Astro + Starlight
├── tailwind.config.mjs   # Конфиг Tailwind (тема)
├── src/
│   ├── content/
│   │   ├── docs/         # Markdown-файлы
│   │   │   ├── index.mdx # Главная
│   │   │   └── ...
│   │   └── config.ts     # Схемы коллекций
│   └── styles/
│       └── custom.css    # Переопределение стилей
└── package.json
```

## 4. План миграции / Разработки
1.  Инициализация проекта: `npm create astro@latest -- --template starlight`.
2.  Установка Tailwind: `npx astro add tailwind`.
3.  Настройка `custom.css` для внедрения Brutalism.
4.  Проверка сборки и локальный запуск.
