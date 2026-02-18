# Circular 📄🔁

Static “latest weekly circular” site that **auto-updates** when new PDFs are added.

## What this repo does

Circular is a lightweight static site designed to host weekly circular PDFs with **zero backend**:

* You add a new weekly PDF to the repo (using a strict filename format).
* A GitHub Actions workflow scans the repo and generates/updates a manifest: **`circulars.json`**.
* The site uses that manifest to determine which circular is “current” and displays it.

Result: **drop in new PDFs → push to `main` → site updates itself → visitors see the correct weekly circular** ✅

---

## How it works (end-to-end)

### 1) Content: weekly PDFs

Weekly PDFs live in the repo and follow this naming convention:

`GCYYYY0MMDD_Merged.pdf`

Example: `GC202600206_Merged.pdf` → circular dated **2026-02-06**.

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

So the manifest is always current without manual edits. 🧰

---

### 3) Runtime: selecting the “current” circular (`index.html`)

At page load, the site:

1. Fetches `circulars.json` with caching disabled (`cache: 'no-store'`) to reduce stale reads.
2. Computes the “effective Friday” in **America/New_York** using this rule:

* The circular “rolls over” at **12:00 AM Friday ET**
* Selection is **not based on the viewer’s local timezone**

3. Picks the newest circular where `date <= effectiveFriday`.

This mimics common retail circular behavior: **the Friday drop stays current all week** 🗓️

---

### 4) Device behavior (mobile vs desktop)

PDF embedding is inconsistent on mobile browsers, so the UI is split:

* **Desktop / larger screens:** embeds the selected PDF in an `<iframe>`
* **Mobile-like devices:** shows a single **Open Circular** button that navigates directly to the PDF

This improves reliability on iOS/Android (PDF iframe rendering can be flaky). 📱

---

## Why this is useful (portfolio angle) 💼

This is a small repo, but it demonstrates practical engineering choices:

* **Static-site + automation architecture** (GitHub Pages + CI-generated content index)
* **Deterministic timezone business logic** (ET-based weekly rollover, DST-safe approach)
* **Cross-platform UX pragmatism** (mobile avoids iframe embedding pitfalls)
* **Operational thinking** (cache control, basic analytics hook, debug logging)
* **Clean separation of concerns**

  * PDFs = content
  * `circulars.json` = index/manifest
  * `index.html` = presentation/runtime selection
  * GitHub Actions = automation

---

## Usage

### Add a new weekly circular

1. Add the PDF with the correct filename pattern:

   * `GCYYYY0MMDD_Merged.pdf`
2. Commit + push to `main`
3. GitHub Actions will regenerate `circulars.json`
4. The site will begin serving the new “current” circular at the next Friday ET rollover rule

---

## Notes / assumptions

* The filename date is treated as authoritative.
* If a “current week” Friday PDF is missing, the site will fall back to the most recent prior entry in the manifest.
* The timezone for rollover logic is fixed to **America/New_York** regardless of viewer locale.



---

If you paste your actual workflow filename (or the real `circulars.json` schema), I can tune the README to match exactly (keeping it human-written, not prompt-output-ish).
