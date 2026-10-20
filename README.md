# somoni-news

News feed for the Somoni apps, served by GitHub Pages at
`https://shohrukh-akhmatov.github.io/somoni-news/`.

News is published in **Russian (`ru`) and Tajik (`tg`)**. In the app the reader
switches between them with the Рус / Тоҷ control on the News tab.

| Page | What it is |
|---|---|
| `/` (`index.html`) | Public preview of the feed (RU / TJ) |
| `/admin.html` | Admin panel: create, edit and delete posts |

## Admin panel

`admin.html` publishes by committing to this repo through the GitHub API.
GitHub Pages then redeploys, and the site and apps show the change within about
1–2 minutes. There's no server and nothing to deploy.

### One-time setup: create a token

1. GitHub → **Settings → Developer settings → Personal access tokens →
   Fine-grained tokens → Generate new token**.
2. **Repository access:** *Only select repositories* → `somoni-news`.
3. **Permissions → Repository permissions → Contents: Read and write**. Leave
   everything else as is.
4. Set an expiry date, generate the token, and copy it.
5. Open `https://shohrukh-akhmatov.github.io/somoni-news/admin.html`, paste the
   token, and click **Подключиться**.

The token is stored only in that browser (or for the tab session only, if
"Запомнить" is unchecked) and is sent only to `api.github.com`. The admin page
itself is public, but without a token it can't change anything. If a token
leaks, revoke it on GitHub and create a new one.

### What the panel does

- Lists posts from `index.json`, newest first.
- Editor with Russian and Tajik fields side by side: title, card summary, tag,
  cover image, and a body built from blocks (paragraph, subheading, quote,
  image with caption) that you can reorder.
- Resizes images in the browser to 1600 px wide and saves them as JPEG under
  500 KB.
- Writes the image files, `articles/<id>.json` and `index.json` in **one
  commit**. If someone else pushed at the same moment, it retries on top of
  their change.
- Deleting a post removes its article file, its images and its entry in the
  feed.

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
