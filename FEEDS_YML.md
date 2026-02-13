# feeds.yml — Configuration Reference

This document defines **all supported feed templates** for the `rssfeeds` repository and explains how to configure them safely and correctly in `feeds.yml`.

---

## File Structure

Example top-level layout:

```yaml
site_base: "https://fetsoc.github.io/rssfeeds"
output_dir: "docs/feeds"

feeds:
  - id: "example-feed"
    type: "template_type"
    ...
```

---

## Top-Level Keys

| Key | Required | Description |
|---|---|---|
| `site_base` | ✅ | Base GitHub Pages URL (used for `<link rel="self">`) |
| `output_dir` | ✅ | Directory where generated RSS files are written |
| `feeds` | ✅ | List of feed definitions |

---

## Feed Object — Common Fields

Every feed entry supports the following common fields:

```yaml
- id: "unique-feed-id"
  type: "template_type"
  title: "Human-readable feed title"
  home_url: "https://example.com"
  max_items: 50
```

### Common Fields

| Field | Required | Description |
|---|---|---|
| `id` | ✅ | Unique feed ID (used as filename: `<id>.xml`) |
| `type` | ✅ | Template engine used to build the feed |
| `title` | ✅ | RSS `<title>` |
| `home_url` | ✅ | RSS `<link rel="alternate">` |
| `max_items` | ⭕ | Maximum number of items (default: 50) |

---

# Template Types

---

## 1️⃣ sitemap_blog

**Use when:**
A blog exists, but the index page is JavaScript-rendered or incomplete.

**How it works:**
- Reads `sitemap.xml`
- Filters URLs by blog prefix
- Fetches each post to extract title, date, and description

### Required Fields

```yaml
type: "sitemap_blog"
sitemap_url: "https://example.com/sitemap.xml"
include_prefix: "https://example.com/blog/"
```

### Optional Fields

| Field | Description |
|---|---|
| `exclude_exact` | URLs to exclude |
| `enrich_candidates` | Number of posts to enrich before sorting |

### Example

```yaml
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
```

---

## 2️⃣ sitemap_blog_tag

**Use when:**
You want **only a specific tag or category** from a blog (e.g. “Labs Research”).

**How it works:**
- Reads sitemap
- Fetches blog posts
- Includes only posts whose HTML contains tag tokens

### Required Fields

```yaml
type: "sitemap_blog_tag"
sitemap_url: "https://example.com/sitemap.xml"
include_prefix: "https://example.com/blog/"
tag_tokens:
  - "labs research"
```

### Optional Fields

| Field | Description |
|---|---|
| `tag_match_mode` | "any" (default) or "all" |
| `scan_limit` | Max sitemap URLs inspected |
| `enrich_limit` | Max posts enriched after tag match |

### Example

```yaml
- id: "nozomi-labs-research"
  type: "sitemap_blog_tag"
  title: "Nozomi Networks — Labs Research"
  home_url: "https://www.nozominetworks.com/blog?tag=labs-research"
  sitemap_url: "https://www.nozominetworks.com/sitemap.xml"
  include_prefix: "https://www.nozominetworks.com/blog/"
  tag_tokens:
    - "labs research"
    - "labs-research"
  tag_match_mode: "any"
  max_items: 50
  scan_limit: 400
  enrich_limit: 200
```

---

## 3️⃣ passthrough_feed

**Use when:**
The source already provides RSS or Atom (GitHub, advisories, vendor blogs).

**How it works:**
- Downloads an existing feed
- Normalizes and re-hosts it

### Required Fields

```yaml
type: "passthrough_feed"
source_feed_url: "https://example.com/feed.xml"
```

### Example — GitHub folder commits

```yaml
- id: "slimkql-azure-folder-commits"
  type: "passthrough_feed"
  title: "SlimKQL — Azure folder commits"
  home_url: "https://github.com/SlimKQL/Hunting-Queries-Detection-Rules/tree/main/Azure"
  source_feed_url: "https://github.com/SlimKQL/Hunting-Queries-Detection-Rules/commits/main/Azure.atom"
  max_items: 50
```

---

## 4️⃣ html_list

**Use when:**
A blog index is server-rendered HTML with stable selectors.

**How it works:**
- Scrapes list page
- Extracts links using a CSS selector
- Enriches each post

### Required Fields

```yaml
type: "html_list"
list_url: "https://example.com/blog"
item_link_selector: "h2.entry-title a"
```

### Example

```yaml
- id: "example-html-blog"
  type: "html_list"
  title: "Example HTML Blog"
  home_url: "https://example.com/blog"
  list_url: "https://example.com/blog"
  item_link_selector: "article h2 a"
  max_items: 30
```

---

## 5️⃣ json_api

**Use when:**
A site exposes content via JSON (WordPress REST, custom APIs).

**How it works:**
- Fetches JSON
- Extracts fields using JMESPath expressions

### Required Fields

```yaml
type: "json_api"
api_url: "https://example.com/api/posts"
items_expr: "posts"
title_expr: "title"
url_expr: "url"
```

### Optional Fields

```yaml
date_expr: "published_at"
```

### Example

```yaml
- id: "example-json-blog"
  type: "json_api"
  title: "Example JSON Blog"
  home_url: "https://example.com/blog"
  api_url: "https://example.com/api/posts"
  items_expr: "posts"
  title_expr: "title"
  url_expr: "url"
  date_expr: "published_at"
  max_items: 50
```

---

## 6️⃣ github_releases

**Use when:**
Tracking GitHub project releases.

**How it works:**
- Uses GitHub Releases API
- Emits releases as RSS items

### Required Fields

```yaml
type: "github_releases"
owner: "org_or_user"
repo: "repository"
```

### Example

```yaml
- id: "sentinel-releases"
  type: "github_releases"
  title: "Microsoft Sentinel — Releases"
  home_url: "https://github.com/Azure/Azure-Sentinel/releases"
  owner: "Azure"
  repo: "Azure-Sentinel"
  max_items: 30
```

---

# YAML Rules (Important)

✅ Use **spaces only**, never tabs
✅ `feeds:` must appear before any `- id:`
✅ List items must be indented **two spaces** under `feeds:`

### ✅ Correct

```yaml
feeds:
  - id: "good"
    type: "passthrough_feed"
```

### ❌ Incorrect (will break parsing)

```yaml
feeds:
- id: "bad"
```

or

```yaml
site_base: "..."
- id: "bad"
```

---

## Recommended Workflow

1. Add a new feed block to `feeds.yml`
2. Commit changes
3. Run GitHub Actions
4. Verify output at:

`https://fetsoc.github.io/rssfeeds/feeds/<id>.xml`

---

## Notes

- One failing feed should **not** block others (skip-on-error recommended)
- Prefer GitHub Atom feeds when available (fast, stable, no scraping)
- Use `sitemap_blog_tag` when tag views are JavaScript-rendered
