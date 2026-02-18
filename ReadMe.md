# Circular 📄🔁

Static “latest weekly circular” site that **auto-updates** when new PDFs are added.

---

## What this repo does

Circular is a lightweight static site designed to host weekly circular PDFs with **zero backend**:

* You add a new weekly PDF to the repo (using a strict filename format).
* A GitHub Actions workflow scans the repo and generates/updates a manifest: **`circulars.json`**.
* The site uses that manifest to determine which circular is “current” and displays it.

**Result:** drop in new PDFs → push to `main` → site updates itself → visitors see the correct weekly circular.

---

## How it works (end-to-end)

### 1) Content: weekly PDFs

Weekly PDFs live in the repo and follow this naming convention:

```
GCYYYY0MMDD_Merged.pdf
```

Example:

```
GC202600206_Merged.pdf
```

→ circular dated **2026-02-06**

> The date embedded in the filename is the source of truth.

---

### 2) Automation: manifest generation (`circulars.json`)

A GitHub Actions workflow runs on pushes to `main` (and can be run manually). It:

1. Finds matching PDF files
2. Extracts dates from filenames
3. Produces a manifest file:

```json
{
  "circulars": [
    { "date": "2026-01-23", "file": "GC202600123_Merged.pdf" },
    { "date": "2026-02-06", "file": "GC202600206_Merged.pdf" }
  ],
  "meta": {
    "timezone": "America/New_York",
    "pattern": "GCYYYY0MMDD_Merged.pdf",
    "generatedAtUtc": "..."
  }
}
```

4. Commits `circulars.json` back to the repo **only if it changed**

The manifest is always current without manual edits.

---

### 3) Runtime: selecting the “current” circular (`index.html`)

At page load, the site:

1. Fetches `circulars.json` with caching disabled (`cache: 'no-store'`)
2. Computes the “effective Friday” in **America/New_York**

Rules:

* The circular “rolls over” at **12:00 AM Friday ET**
* Selection is **not based on the viewer’s local timezone**

3. Picks the newest circular where:

```
date <= effectiveFriday
```

This mirrors common retail circular behavior: the Friday drop remains active all week.

---

### 4) Device behavior (mobile vs desktop)

PDF embedding is inconsistent on mobile browsers, so the UI is split:

* **Desktop / larger screens:** embeds the selected PDF in an `<iframe>`
* **Mobile-like devices:** shows a single **Open Circular** button that navigates directly to the PDF

This improves reliability on iOS and Android browsers.

---

## Why this is useful (portfolio angle)

This repo demonstrates:

* **Static-site + automation architecture** (GitHub Pages + CI-generated content index)
* **Deterministic timezone business logic** (ET-based weekly rollover)
* **Cross-platform UX pragmatism**
* **Operational awareness** (cache control, analytics hook, debug logging)
* **Clean separation of concerns**

  * PDFs = content
  * `circulars.json` = index / manifest
  * `index.html` = presentation + selection logic
  * GitHub Actions = automation

---

## Usage

### Add a new weekly circular

1. Add the PDF with the correct filename pattern:

```
GCYYYY0MMDD_Merged.pdf
```

2. Commit and push to `main`
3. GitHub Actions regenerates `circulars.json`
4. The site serves the new circular according to the Friday ET rollover rule

---

## Notes / assumptions

* The filename date is treated as authoritative.
* If a Friday circular is missing, the site falls back to the most recent prior entry.
* Rollover timezone is fixed to **America/New_York**, regardless of viewer location.

---

## Why this exists

This site was built as a short-term solution.

Our original circular site is hosted on WordPress. At the time, I had no experience with the platform and no working knowledge of its theme structure, templating model, or plugin ecosystem. Rather than delay delivery, I implemented this static alternative to ensure weekly circulars could continue to be published reliably.

The goal was operational continuity — not platform replacement.

Since then, I have gained sufficient familiarity with WordPress to replicate the same deterministic rollover logic and publishing workflow within the primary site. That implementation is now live.

As a result, this repository is entering a dormant state. It remains available as:

* A documented example of the static + CI automation pattern
* A reference implementation of the ET-based rollover logic
* A fallback option if the primary platform ever requires it again

---


