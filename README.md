# somoni-news

News feed for the Somoni apps, served by GitHub Pages at
`https://shohrukh-akhmatov.github.io/somoni-news/`.

News is published in **Russian (`ru`) and Tajik (`tg`)**. In the app the reader
switches between them with the Рус / Тоҷ control on the News tab.

`index.html` is a public preview of the feed (RU / TJ).

## Publishing

Posts are written with the **local admin panel**, which runs only on the
editor's laptop and isn't part of this repo or the site. It commits to a local
clone of this repo and publishes with a normal `git push`, so no token or key
is stored anywhere. GitHub Pages redeploys after each push, and the site and
apps show the change within about 1–2 minutes.

You can also edit the JSON by hand and push. The format is below.

## File format

You only need this section if you edit the JSON by hand. The admin panel writes
exactly this format.

```
index.json            list of posts, shown as cards in the News tab
articles/<id>.json    one full post, loaded when a card is tapped
images/               JPEG / PNG / WebP pictures (no SVG, no HEIC)
```

Rules:

- `id`: lowercase latin letters, digits, `-`, `_` (it's also the file name).
- Dates: full UTC timestamps, `2026-10-20T08:00:00Z`.
- Text: `{ "ru": "...", "tg": "..." }`. If one language is missing, the other
  is shown.
- Images: a path inside this repo (`images/x.jpg`) or a full `https://` URL.

### `index.json`

```json
{
  "version": 1,
  "updated_at": "2026-10-20T08:00:00Z",
  "articles": [
    {
      "id": "welcome",
      "published_at": "2026-10-20T08:00:00Z",
      "tag":     { "ru": "Обновление", "tg": "Навсозӣ" },
      "title":   { "ru": "...", "tg": "..." },
      "summary": { "ru": "...", "tg": "..." },
      "image": "images/welcome.png"
    }
  ]
}
```

`tag`, `summary` and `image` are optional.

### `articles/<id>.json`

```json
{
  "id": "welcome",
  "published_at": "2026-10-20T08:00:00Z",
  "title": { "ru": "...", "tg": "..." },
  "image": "images/welcome.png",
  "source_url": "https://example.com/original-post",
  "blocks": [
    { "type": "heading",   "text": { "ru": "...", "tg": "..." } },
    { "type": "paragraph", "text": { "ru": "...", "tg": "..." } },
    { "type": "image",     "url": "images/photo.jpg", "caption": { "ru": "...", "tg": "..." } },
    { "type": "quote",     "text": { "ru": "...", "tg": "..." } }
  ]
}
```

Paragraphs and quotes accept inline Markdown: `**bold**`, `*italic*`,
`[link](https://...)`. `source_url` is optional and must be https. Apps skip
block types they don't know, so new types can be added later without breaking
older app versions.
