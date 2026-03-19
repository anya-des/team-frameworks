# Favicon Studio — контекст проекту

## Що це
Браузерний HTML інструмент для batch-генерації унікальних SVG фавіконів для 600+ travel/booking доменів.
Файл: `favicon-generator-v2.html`
Іконки: `icons.svg` (614×1370px, Figma export, всі іконки 16×16px по координатах x,y)

## Архітектура — Plan B
- Основний output: `manifest.json` → бекенд читає і роздає через `GET /favicon?domain=example.com`
- SHA-256 детермінована генерація (один домен = завжди один фавікон)
- Унікальність через combo_key Set з попереднього manifest
- Іконки витягуються з icons.svg по координатах, нормалізуються до viewBox="0 0 16 16"
- Рендер: `<g transform="translate(4,4)">` всередині SVG 24×24

## Генерація
- Палітра: `(h[0]^h[6]^h[12]^h[18] + offset) % 5`
- Колір: `(h[1]^h[7]^h[13]^h[19] + offset) % 16`
- Градієнт: `(h[2]^h[8] + offset) % 6`
- Форма: `(h[3]^h[9] + offset) % 8`
- Іконка: `(h[5]^h[11]^h[17] + offset) % icons.length`

## Критичні баги — ЗАБОРОНЕНО
- `id="G"` для всіх градієнтів → всі одного кольору → використовувати `G${globalIdx}`
- `<svg x y>` для іконок → зсув → тільки `<g transform>`
- `h[0] % N` без XOR → схожі домени = одна палітра → XOR далеко рознесених байтів
- `dominant-baseline="auto"` → текст зміщений → використовувати `"central"`
- Новий manifest без старих доменів → втрата історії → завжди merge

## Manifest структура
```json
{
  "generated": "ISO date",
  "total": 312,
  "domains": {
    "domain.com": {
      "abbr": "BB",
      "category": "breakfast",
      "icon": "Croissant",
      "palette": "warm",
      "shape": "squircle",
      "gradient": "diagonal",
      "combo_key": "warm-squircle-diagonal-Croissant",
      "svg": "<svg...>...</svg>"
    }
  }
}
```

## UI структура
- Ліва панель 280px: завантажити manifest, textarea для доменів, прогрес, кнопка генерації, фільтри палітри/форми, регенерація одного домену, статистика
- Права область: grid preview S/M/L, групування по категоріях
- Кнопка "Зберегти manifest.json" — PRIMARY
- Кнопка "Скачати ZIP як backup" — secondary

## GitHub
- Repo: anyaserpen-dot/team-frameworks
- Branch: main
- Завжди робити commit і push після змін

## Команди
- Запуск: `claude` в папці `~/Desktop/team-frameworks`
- Push: `git add . && git commit -m "опис" && git push`
