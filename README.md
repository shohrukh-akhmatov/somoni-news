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

## Notifications

When a new post id appears in the **live** `index.json`, a Supabase job sends one
push notification to the iOS app: the headline as the title and the summary as the
text, in the reader's news language. Tapping it opens the post. Because the job reads
the published site, the notification never arrives before the post is readable.

The job runs every 30 minutes from 09:08 to 22:08 Dushanbe time. Expect the
notification within about 30 minutes of the push to GitHub; a post published at night
goes out at 09:08.

- Each post is announced **once**. Editing it later never notifies again.
- `"notify": false` on a post in `index.json` publishes it silently (the admin's
  checkbox).
- Posts dated more than 3 days in the past are published silently.

## When the app checks

While the News tab is open, the app asks for `index.json` once a minute with
`If-Modified-Since`. An unchanged file answers `304` with no body; a new post shows
up as a new card at the top within about a minute of GitHub Pages serving it.
Nothing is checked while the tab is closed or the app is in the background.

## What the phone keeps

The app stores only the newest 30 entries of `index.json` (headlines and summaries).
The body and pictures of an opened post stay until the end of that day and are
removed the next time the app is opened. All pictures are cleared at least once a
week.

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

`tag`, `summary` and `image` are optional. `notify` is optional too: leave it out
(or `true`) to send the notification, `false` to skip it. Apps ignore it.

Two more optional fields, for cards like your Telegram channel or discussion group:

```json
{
  "id": "telegram-channel",
  "published_at": "2026-10-21T09:00:00Z",
  "pinned": true,
  "link": "https://t.me/your_channel",
  "title":   { "ru": "Наш Telegram-канал", "tg": "Канали Telegram-и мо" },
  "summary": { "ru": "Курсы и новости первыми", "tg": "Қурбҳо ва хабарҳо аввалин" },
  "notify": false
}
```

- `pinned: true` keeps the card at the top of the list (apps and site). Several
  pinned cards are ordered newest first by `published_at`.
- `link` (https only) makes the card open that address instead of an article. A
  link card needs no `articles/<id>.json` and is never announced by push.

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
