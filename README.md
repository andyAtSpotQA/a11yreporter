# a11yreporter

A lightweight, zero-dependency single-page application for viewing and analysing [axe-core](https://github.com/dequelabs/axe-core) accessibility audit results. Load one or more JSON result files and get a clear, rule-centric breakdown of every violation, the affected elements, and how to fix them.

**Live demo:** [andyatspotqa.github.io/a11yreporter](https://andyatspotqa.github.io/a11yreporter/)

---

## Features

- **Executive summary** — at-a-glance stats broken down by impact level (critical, serious, moderate, minor) with a rolled-up violations table across all loaded files
- **Rule-centric detail view** — one card per violated rule showing the rule description, WCAG tags, fix guidance, axe documentation link, and a table of every affected element with its CSS selector and HTML snippet
- **Consolidated report** — merges results from multiple JSON files into a single deduplicated view; identical rule + element combinations are collapsed with an occurrence count and source file indicators
- **Per-file reports** — drill into results from an individual audit file
- **Drag-and-drop / file picker** — load axe-core JSON results by dropping files onto the page or using the file chooser; add more files at any time
- **Auto-load** — when served alongside a `json/` directory, known result files are fetched automatically on page load
- **Fully client-side** — no server, no build step, no dependencies; a single `index.html` file with embedded CSS and JS
- **Responsive** — works on desktop and mobile viewports

## Views

| Route | Description |
|---|---|
| `#summary` | Executive summary with stats, rolled-up violations table, and per-file cards |
| `#report` | Consolidated report — all files merged, deduplicated by rule + CSS selector |
| `#report/0`, `#report/1`, ... | Per-file report with rule-centric card layout |

## Getting Started

### 1. Generate axe-core JSON results

Run an axe-core audit against your pages and export the `violations` array as JSON. For example, using [@axe-core/cli](https://www.npmjs.com/package/@axe-core/cli):

```bash
npx @axe-core/cli https://example.com --save results.json
```

Or programmatically with axe-core in a Playwright/Puppeteer/Selenium test — serialise `results.violations` to a `.json` file.

### 2. Serve locally

Clone the repo and start any static file server from the project root:

```bash
git clone https://github.com/andyAtSpotQA/a11yreporter.git
cd a11yreporter
python3 -m http.server 8000
```

Then open [http://localhost:8000/docs/index.html](http://localhost:8000/docs/index.html).

The sample JSON files in `docs/json/` will load automatically. To use your own results, either:

- **Drop files** onto the page (or click "Choose JSON Files")
- **Place files** in `docs/json/` and add their filenames to the `KNOWN_FILES` array in `index.html` for auto-loading

### 3. Deploy to GitHub Pages

The app is designed to run from the `docs/` directory, which GitHub Pages can serve directly:

1. Push the repo to GitHub
2. Go to **Settings > Pages**
3. Set **Source** to `Deploy from a branch`, branch `main`, folder `/docs`
4. Your report will be live at `https://<username>.github.io/a11yreporter/`

To include your own audit data, commit your JSON files into `docs/json/` and update the `KNOWN_FILES` array.

## Project Structure

```
a11yreporter/
  docs/
    index.html          # The entire SPA (HTML + CSS + JS)
    json/
      <hash>.json       # axe-core violations JSON files
  README.md
```

## Input Format

Each JSON file should be an array of axe-core violation objects:

```json
[
  {
    "id": "color-contrast",
    "impact": "serious",
    "help": "Elements must meet minimum color contrast ratio thresholds",
    "description": "Ensures the contrast between foreground and background colors meets WCAG 2 AA minimum thresholds",
    "helpUrl": "https://dequeuniversity.com/rules/axe/4.10/color-contrast",
    "tags": ["wcag2aa", "wcag143"],
    "nodes": [
      {
        "target": ["#main > p.intro"],
        "html": "<p class=\"intro\">Some text</p>",
        "failureSummary": "Fix any of the following:\n  Element has insufficient color contrast of 3.45 (foreground color: #767676, background color: #ffffff, font size: 12pt, font weight: normal). Expected contrast ratio of 4.5:1"
      }
    ]
  }
]
```

This is the standard `results.violations` array returned by axe-core's `axe.run()`.

## Deduplication Logic

When viewing the consolidated report (`#report`), the app groups findings by **rule ID** and deduplicates elements by their **CSS selector** (`target`). If the same element fails the same rule across multiple source files, it appears once with:

- An **occurrence count** showing how many times it was found
- **Source file chips** indicating which audit files contained the finding

## License

MIT
