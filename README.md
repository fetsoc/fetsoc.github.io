# rssfeeds

A lightweight **RSS/Atom feed factory** powered by **GitHub Actions** + **GitHub Pages**.

Use this repo to generate and host RSS feeds for sources that:

- don’t provide RSS/Atom (but *do* provide a sitemap),
- provide RSS/Atom but you want to **normalize** and **re-host** them in one place,
- are GitHub repos/folders (commits/releases) where Atom feeds already exist.

The output feeds are published as static XML under `docs/feeds/` and can be consumed by any RSS reader, automation pipeline, SIEM/SOAR tooling, or webhook bridge.

---

## Quick start (GitHub UI)

1. **Create / update** `feeds.yml` in the repo root.
2. Commit changes.
3. Run the workflow: **Actions → Build RSS feeds → Run workflow** (or wait for the schedule).
4. After GitHub Pages is enabled, access feeds at:

```
https://<github-username>.github.io/<repo>/feeds/<feed-id>.xml
```

---

## Repository layout

```
.
├─ docs/
│  ├─ feeds/                 # generated RSS output (*.xml)
│  └─ FEEDS_YML.md           # documentation for feeds.yml (optional)
├─ generators/
│  └─ build.py               # feed builder (reads feeds.yml)
├─ .github/
│  └─ workflows/
│     └─ build-feeds.yml     # GitHub Actions workflow
├─ feeds.yml                 # feed definitions
└─ requirements.txt          # Python dependencies
```

---

## Enable GitHub Pages

1. Repo **Settings → Pages**
2. **Source**: “Deploy from a branch”
3. **Branch**: `main`
4. **Folder**: `/docs`
5. Save

After it publishes, your feeds become public URLs under `/feeds/`.

---

## Configure feeds (`feeds.yml`)

`feeds.yml` is the only file you typically edit.

Example minimal config:

```yaml
site_base: "https://<github-username>.github.io/<repo>"
output_dir: "docs/feeds"

feeds:
  - id: "hunt-blog"
    type: "sitemap_blog"
    title: "Hunt.io Blog"
    home_url: "https://hunt.io/blog"
    sitemap_url: "https://hunt.io/sitemap.xml"
    include_prefix: "https://hunt.io/blog/"
    exclude_exact:
      - "https://hunt.io/blog"
      - "https://hunt.io/blog/"
    max_items: 50
    enrich_candidates: 80

  - id: "slimkql-azure-folder-commits"
    type: "passthrough_feed"
    title: "SlimKQL — Azure folder commits"
    home_url: "https://github.com/SlimKQL/Hunting-Queries-Detection-Rules/tree/main/Azure"
    source_feed_url: "https://github.com/SlimKQL/Hunting-Queries-Detection-Rules/commits/main/Azure.atom"
    max_items: 50
```

> **YAML tip:** Use spaces (not tabs) and keep indentation consistent.

For full schema + examples for each template type, see `docs/FEEDS_YML.md`.

---

## Supported template types

The generator supports multiple “template types” (feed builders). The exact list depends on what’s implemented in `generators/build.py`, but typically includes:

- `sitemap_blog` — Build a blog feed from a site’s sitemap (best for JS-rendered blog indexes)
- `sitemap_blog_tag` — Same as above but only include posts matching tag tokens
- `passthrough_feed` — Re-host/normalize an existing RSS/Atom feed
- `html_list` — Scrape a server-rendered list page using a CSS selector
- `json_api` — Build a feed from a JSON endpoint using JMESPath expressions
- `github_releases` — Build a feed from GitHub Releases API

---

## GitHub Sources (easy wins)

GitHub exposes Atom feeds for common repo activity. You can use these with `passthrough_feed`.

Examples:

- **Repo releases**: `https://github.com/<owner>/<repo>/releases.atom`
- **Repo commits (branch)**: `https://github.com/<owner>/<repo>/commits/<branch>.atom`
- **Folder/file commits**: `https://github.com/<owner>/<repo>/commits/<branch>/<path>.atom`

---

## GitHub Actions workflow

The workflow:

- installs dependencies,
- runs `python generators/build.py`,
- commits updated `docs/feeds/*.xml` back to the repo.

If you want to change schedule frequency, edit the cron expression in:

`.github/workflows/build-feeds.yml`

---

## Troubleshooting

### Workflow fails with YAML parse errors
- Most common cause is indentation or tabs in `feeds.yml`.

### A feed returns 0 items
- Ensure your URL/prefix is correct.
- For `sitemap_blog_tag`, confirm the `tag_tokens` actually appear in the post HTML.

### One feed failure breaks everything
- Recommended: add “skip-on-error” behavior in `generators/build.py` so the run continues even if one feed fails.

---

## Contributing

PRs are welcome.

- Add new template types in `generators/build.py`.
- Document them in `docs/FEEDS_YML.md`.
- Keep configs reproducible and avoid disabling TLS verification globally.

---

## License

Use whatever license fits your org/team (or inherit the repo’s current license if present).
