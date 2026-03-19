# Favicon Studio — контекст проекту

## Що це
Браузерний HTML інструмент для batch-генерації унікальних SVG фавіконів для 600+ travel/booking доменів.
Файл: `favicon-generator-v2.html`
Іконки: `icons.svg` (614×1370px, Figma export, всі іконки 16×16px по координатах x,y)

## Іконки — завантаження з icons.svg
- `loadIconsSprite()` — fetch icons.svg → DOMParser → clipPath defs → `iconLookup['x_y'] = paths`
- Іконки витягуються по clip-path: знаходимо `<g clip-path="url(#clipN)">`, видаляємо fill/stroke, додаємо `translate(-x,-y)`
- `buildContent()` шукає `iconLookup[x_y]`, fallback → текст abbr
- `iconTransform`: regular `translate(8,8) scale(0.875)`, scallop `translate(10,10) scale(0.75)`
- **Без CDN** — немає залежності від unpkg/Lucide/Tabler
- Manifest: `iconCoords:[x,y]` замість `icon`/`lib`

## ICON_MAP з координатами icons.svg
```js
// {x,y} = top-left corner 16×16 іконки в спрайті
// ⚠ Координати відповідають clip-path індексам — перевір візуально
{cat:'spa',       icons:[{x:35,y:652},{x:61,y:652},{x:8,y:121}]}
{cat:'wellness',  icons:[{x:62,y:121},{x:90,y:121},{x:121,y:121}]}
{cat:'beach',     icons:[{x:8,y:722},{x:32,y:722},{x:80,y:722},{x:81,y:757},{x:108,y:757}]}
{cat:'ski',       icons:[{x:8,y:547},{x:30,y:547},{x:108,y:547}]}
{cat:'breakfast', icons:[{x:8,y:49},{x:36,y:49},{x:62,y:49}]}
{cat:'romantic',  icons:[{x:89,y:49},{x:118,y:49},{x:147,y:49}]}
{cat:'camping',   icons:[{x:176,y:49},{x:206,y:49},{x:236,y:49},{x:8,y:477},{x:38,y:477}]}
{cat:'glamping',  icons:[{x:57,y:477},{x:88,y:477},{x:133,y:477}]}
{cat:'luxury',    icons:[{x:8,y:442},{x:71,y:442},{x:96,y:442}]}
{cat:'boutique',  icons:[{x:119,y:442},{x:146,y:442},{x:8,y:229}]}
{cat:'pool',      icons:[{x:43,y:229},{x:69,y:229},{x:95,y:229},{x:31,y:14}]}
{cat:'waterpark', icons:[{x:109,y:14},{x:131,y:14},{x:8,y:862},{x:104,y:862}]}
{cat:'family',    icons:[{x:130,y:862},{x:230,y:862},{x:253,y:862}]}
{cat:'pet',       icons:[{x:274,y:862},{x:301,y:862},{x:321,y:862}]}
{cat:'hostel',    icons:[{x:345,y:862},{x:388,y:862},{x:409,y:862}]}
{cat:'apartment', icons:[{x:453,y:862},{x:480,y:862},{x:503,y:862}]}
{cat:'villa',     icons:[{x:529,y:862},{x:8,y:582},{x:35,y:582}]}
{cat:'cabin',     icons:[{x:62,y:582},{x:89,y:582},{x:170,y:582}]}
{cat:'resort',    icons:[{x:8,y:85},{x:65,y:86},{x:78,y:193},{x:149,y:193},{x:172,y:193}]}
{cat:'golf',      icons:[{x:228,y:193},{x:250,y:193},{x:8,y:299}]}
{cat:'airport',   icons:[{x:36,y:299},{x:83,y:299},{x:105,y:299}]}
{cat:'casino',    icons:[{x:129,y:302},{x:8,y:158},{x:33,y:158}]}
{cat:'capsule',   icons:[{x:58,y:158},{x:88,y:158},{x:110,y:156}]}
{cat:'hotel',     icons:[{x:8,y:264},{x:33,y:264},{x:58,y:264},{x:81,y:264}]}
```

## KEYWORD_MAP (вбудований в ICON_MAP, поле `kw`)
```
spa:       spa thermal terme therme balnear
wellness:  wellness relax yoga meditation retreat thalasso
beach:     beach strand mare plage seaside costa lido
ski:       ski snow montagne mountain berg alpin nieve
breakfast: breakfast bedandbreakfast beb bedebreakfast colazione petitdejeuner fruhstuck
romantic:  romantic amore love couples honeymoon amour romance
camping:   camping campground camp kemping campismo
glamping:  glamping
luxury:    luxury luxe lusso luxus premium grand
boutique:  boutique design lifestyle
pool:      pool piscina piscine zwembad
waterpark: waterpark aquapark wasserpark
family:    family kinder kids enfant bambini children familienhotel
pet:       pet dog hund animaux perros chien haustier
hostel:    hostel jugendherberge backpacker auberge
apartment: apartment aparthotel wohnung flat ferienwohnung
villa:     villa villas chateau manor palazzo
cabin:     cabin chalet bungalow hutte casita cottage
resort:    resort palm allinclusive inclusive
golf:      golf golfhotel golfresort
airport:   airport flughafen aeroport aeroporto
casino:    casino gambling spielbank
capsule:   capsule pod micro
hotel:     hotel inn lodge herberge motel
```

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
