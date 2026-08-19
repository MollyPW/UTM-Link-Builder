# UTM Link Builder

A free, no-code tool for building consistent UTM tracking links. Instead of typing UTM parameters by hand (a common source of messy, inconsistent attribution data), marketers choose from an approved, editable list of values — and can save recurring parameter sets as presets by campaign type.

**[Try it live →](#)** *(replace with your GitHub Pages link once enabled)*

---

## Why this exists

Freeform UTM tagging breaks attribution reporting: `Newsletter` vs `newsletter` vs `news-letter` all show up as separate sources in analytics tools. This tool adds guardrails — dropdowns instead of free text — while still being flexible enough to add, remove, or update the approved values as campaigns evolve.

## Features

- **Guided form** — choose source, medium, campaign, content, and term from dropdowns instead of typing them
- **Link location tracking** — the "content" field is built for distinguishing multiple links to the same URL (e.g. `header-cta` vs. `footer-link` in an email)
- **Editable value lists** — add or remove approved dropdown options anytime, no code required
- **Campaign-type presets** — save a set of values (e.g. "New landing page") and auto-fill them on future links
- **One-click copy** — generates the full tracking URL and copies it to your clipboard
- **Zero install** — a single HTML file, no build tools, no dependencies
- **Optional team sharing** — connect a free Supabase database so your whole team sees the same saved values (see setup guide)

## Getting started

1. Download `utm-builder.html` from this repo.
2. Open it in any browser — that's it, no installation.
3. (Optional) Host it publicly via GitHub Pages: **Settings → Pages → Deploy from branch**, then share the resulting link with your team.

## How to use it

1. **(Optional)** Pick a saved preset to auto-fill common fields for that campaign type.
2. Enter your destination URL and choose values for Source, Medium, and Campaign (required), plus Content and Term (optional).
3. Click **Generate link**, then **Copy link**.

To save your current field values as a new preset, click **Save these values as a preset** before or after generating a link.

To add or remove values from the dropdowns, open the **Manage approved values** section near the bottom of the page.

## Customizing & team setup

See [`SETUP-GUIDE.md`](./SETUP-GUIDE.md) for:
- Editing the starting dropdown values, presets, colors, and labels
- Connecting a free shared database (Supabase) so your whole team works from the same approved values and presets, instead of each person having their own separate local copy

## How data is stored

By default, this tool saves your options and presets to your browser's local storage — private to you, on that device. It does **not** phone home to any server. If you want shared storage across a team, follow the Supabase setup in `SETUP-GUIDE.md`.

## Tech

Plain HTML, CSS, and JavaScript. No frameworks, no build step, no npm install. Optional integration with [Supabase](https://supabase.com) for shared storage.

## License

MIT — free to use, modify, and share.
