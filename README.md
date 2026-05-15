> YouTube: [youtube.com/@drunkleen](https://youtube.com/@drunkleen)

# Downloader Action

> **[Persian / فارسی](README-FA.md)**

A GitHub Actions-powered file downloader. Fork this repository and use GitHub's infrastructure to download, store, and access files directly from your repository — no server required.

---

## Table of Contents

- [How It Works](#how-it-works)
- [Getting Started](#getting-started)
- [Workflows](#workflows)
  - [Full Downloader](#full-downloader)
  - [Web Browser](#web-browser)
  - [Delete Downloads](#delete-downloads)
  - [History Cleaner](#history-cleaner)
- [File Storage Structure](#file-storage-structure)
- [Accessing Your Files](#accessing-your-files)
- [Tips & Notes](#tips--notes)

---

## How It Works

All workflows run on GitHub Actions via manual trigger (`workflow_dispatch`). When you run a download workflow:

1. GitHub spins up an Ubuntu runner
2. The file is downloaded using `aria2c` (with `curl` fallback)
3. The file is committed back into your repository under `downloads/`
4. A `README.md` with metadata and direct download links is generated automatically

No external storage, no tokens, no subscriptions — just a forked GitHub repository.

---

## Getting Started

1. **Fork** [drunkleen/downloader-action](https://github.com/drunkleen/downloader-action) to your GitHub account
2. Go to the **Actions** tab of your forked repo
3. Enable workflows if prompted
4. Pick a workflow from the left sidebar and click **Run workflow**
5. Fill in the inputs and run

> `downloads/` is gitignored locally but workflows use `git add -f` to force-commit files.

---

## Workflows

### Full Downloader

**File:** `.github/workflows/main.yml`

Downloads files using `aria2c` for fast multi-connection transfers, with automatic `curl` fallback. All files are packaged into a zip archive. No file size limit.

**Inputs:**

| Input      | Description                    | Required |
| -----------|--------------------------------|----------|
| `urls`     | Download URLs, space-separated | Yes      |
| `password` | Zip archive password           | No       |

**Features:**

- `aria2c` with 2 connections and 20MB min split; falls back to `curl` on failure
- Up to 3 retry attempts with exponential backoff
- All downloaded files packaged into one split-zip archive before committing
- Archives over 95MB are automatically split into 95MB parts (GitHub file size compatible)
- Optional password protection for zip archives
- Uploads over 200MB use incremental batch push (3 files/batch, up to 5 push retries each) to avoid timeouts
- Archive `README.md` generated with metadata and direct download links
- Root `README.md` updated with list of all downloads

**Examples:**

Download a single file:
```
URLs: https://example.com/file.iso
```

Download multiple files as a password-protected zip:
```
URLs: https://example.com/a.mp4 https://example.com/b.mp4
Password: mysecret
```

---

### Web Browser

**File:** `.github/workflows/browser.yml`

Visits a URL with a headless Chromium browser (Puppeteer), takes a full-page screenshot, saves the HTML, and downloads all media assets found on the page.

**Inputs:**

| Input | Description | Required |
|-------|-------------|----------|
| `url` | The URL to visit | Yes |

**What gets saved:**

| Path                                        | Description                                    |
| ---------------------------------------------| ------------------------------------------------|
| `pages/<domain>/<timestamp>/page.html`      | Raw HTML                                       |
| `pages/<domain>/<timestamp>/screenshot.png` | Full-page screenshot                           |
| `pages/<domain>/<timestamp>/media/`         | Downloaded images, audio, video, documents     |
| `pages/<domain>/<timestamp>/index.md`       | Summary: screenshot, media list, links         |
| `pages/<domain>/<timestamp>.zip`            | Zip archive of the full session                |
| `browse.md`                                 | Running log of all visited sites with favicons |
| `pages/README.md`                           | Index of all captured pages                    |

**Favicon resolution order:**
1. `/favicon.ico` at the root domain
2. `<link rel="icon">` tag in the page HTML
3. Google's favicon service as fallback

**Media types downloaded:** `png jpg jpeg gif svg webp bmp ico mp3 wav ogg flac aac mp4 webm avi mov mkv pdf doc docx xls xlsx ppt pptx zip rar tar gz 7z`

---

### Delete Downloads

**File:** `.github/workflows/cleaner.yml`

Wipes all downloaded content and resets the repository to a clean state.

**What gets deleted:**
- `downloads/` — reset to empty placeholder
- `pages/` — all captured browser sessions removed
- `browse.md` — browsing log removed
- Generated sections in `README.md` removed

> **Warning:** Permanently deletes all files listed above. Data remains in git history until you run the History Cleaner.

**Inputs:**

| Input | Description |
|-------|-------------|
| `warning` | Read-only reminder — no effect on execution |

---

### History Cleaner

**File:** `.github/workflows/history_cleaner.yml`

Rewrites git history to strip all blobs larger than 20MB. Run this after deleting downloads to actually free up GitHub storage space.

> **Warning:** Destructive and irreversible. Rewrites all commits and force-pushes all branches and tags. All local clones will be out of sync and must be re-cloned.

**Steps it performs:**

1. Reports current `.git` size and lists all blobs >20MB
2. Runs `git filter-repo --strip-blobs-bigger-than 20M --force`
3. Force-pushes rewritten history to all branches and tags
4. Reports new repository size

**Recommended sequence:**
1. Run **Delete Downloads** first (removes files from working tree)
2. Run **History Cleaner** (purges large blobs from git history)

---

## File Storage Structure

```
downloads/
  README.md                         ← index of all downloads
  <archive-name>/
    <archive-name>.zip              ← archive (.z01, .z02... for splits)
    README.md                       ← part list + extraction instructions

pages/
  README.md                         ← index of all captured pages
  <domain>/
    <timestamp>/
      page.html
      screenshot.png
      index.md
      media/
    <timestamp>.zip

browse.md                           ← running log of visited sites
```

---

## Accessing Your Files

After a workflow completes, files are committed to your repo. To download:

1. Navigate to `downloads/<folder-name>/` in your repo
2. Open `README.md` — it has direct raw download links
3. Click to download

Or use the raw URL directly:
```
https://github.com/drunkleen/downloader-action/raw/<branch>/downloads/<folder>/<filename>
```

---

## Tips & Notes

- **Multiple URLs:** Space-separated in the input field
- **Filename sanitizing:** Characters `<>:"|?*` are replaced with `_` automatically
- **Duplicate folder names:** A random suffix is appended automatically to avoid conflicts
- **`[skip ci]`:** All automated commits include this to prevent recursive workflow triggers
- **Storage limit:** GitHub repos have a soft 5GB limit — run the History Cleaner periodically when downloading large files
- **Timeouts:** Full Downloader runs up to 120 minutes
