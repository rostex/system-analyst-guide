# v4 — Confluence Storage Format

Файлы в этой директории используют [Confluence Storage Format](https://developer.atlassian.com/cloud/confluence/apis/rest/) — XML-формат, который Confluence понимает нативно. В отличие от HTML-импорта, этот подход **сохраняет всё форматирование**: цвета, панели, границы, таблицы.

## Как использовать

### Способ 1: REST API (рекомендуемый)

Создать страницу в Confluence через API:

```bash
curl -u user:token -X POST \
  "https://your-domain.atlassian.net/wiki/rest/api/content" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "page",
    "title": "Техническое мышление аналитика",
    "space": {"key": "SPACEKEY"},
    "body": {
      "storage": {
        "value": "'"$(cat index.xml | sed 's/"/\\"/g')"'",
        "representation": "storage"
      }
    }
  }'
```

Способ требует:
- API-токен Confluence (или логин/пароль)
- Ключ пространства (space key)

### Способ 2: Confluence CLI

```bash
confluence --action storePage \
  --space "SPACEKEY" \
  --title "Техническое мышление аналитика" \
  --file "index.xml"
```

### Способ 3: Import HTML Plugin (с ограничениями)

Можно попробовать скопировать содержимое `<body>` из XML как HTML, но плагин импорта может не распознать `ac:`-макросы. Этот способ не гарантирует результата.

## Как загрузить главы

Каждая глава — отдельная страница с родительской страницей (оглавлением):

1. Создать родительскую страницу из `index.xml`
2. Для каждой главы создать дочернюю страницу с `ancestors`: `[{id: parentId}]`
3. В оглавлении ссылки через `<ac:link><ri:page ri:content-title="..." /></ac:link>` подтянутся автоматически

## Преимущества Storage Format

| Что | Поддерживается |
|-----|:---:|
| Цветные панели (callout) | ✅ `ac:structured-macro ac:name="panel"` |
| Таблицы с цветными шапками | ✅ inline styles |
| Градиенты | ⚠️ заменить на `ac:macro` info |
| Зебра-строки | ✅ inline `background-color` |
| Ссылки на другие страницы | ✅ `ac:link` |
| Вложенные списки | ✅ |
| Моноширинный код | ✅ `<pre>` или `ac:structured-macro ac:name="code"` |
| Изображения | ✅ `ac:image` |
