# Create a New Update

Create a new update post for my Jekyll site based on the user's input below. Follow these instructions precisely.

## User Input

$ARGUMENTS

## Instructions

### 1. Parse the Input

The user's input will contain:
- **Update text**: The body content for the update (required)
- **Photos**: Zero or more photos, provided as:
  - **File paths** on disk (e.g., `~/Photos/sunset.jpg`)
  - **Direct image URLs** (e.g., `https://example.com/photo.jpg`)
  - **A Google Photos shared album URL** (e.g., `https://photos.app.goo.gl/...` or `https://photos.google.com/share/...`)
  - **Explicit embed URLs** prefixed with `embed:` — these should be used as-is in markdown, not downloaded
- **Captions**: Optional captions for photos, provided inline near each photo reference

If a Google Photos album URL is provided:
1. Fetch the album page HTML using curl
2. Extract all `lh3.googleusercontent.com` image URLs from the page source using grep/regex
3. Filter for unique, full-resolution URLs (append `=w2048` to each URL if no size suffix present)
4. Confirm with the user how many images were found and which to include
5. Download each selected image

### 2. Create the Update File

- **Filename**: `_updates/YYYY-MM-DD-HHMM.md` using the current date and time (24-hour format)
  - Example: `_updates/2026-03-01-1435.md`
- **Frontmatter**:
  ```yaml
  ---
  date: YYYY-MM-DD HH:MM:00 -0600
  published: true
  ---
  ```
  - Use the current date/time
  - Timezone is `-0600` (Central Time)
- **Body**: The update text as markdown. Keep it exactly as the user wrote it, just clean up obvious typos if any.

### 3. Handle Photos

**For photos to download** (file paths, URLs, or Google Photos album images):
- Create a subdirectory: `assets/images/updates/YYYY-MM-DD-HHMM/`
- Copy or download each photo into that directory
- Name files using the `.optimize` convention for the GitHub Actions image pipeline:
  - `photo-1.optimize-1200w.jpg` (or `.png`, matching the original format)
  - `photo-2.optimize-1200w.jpg`, etc.
  - If the user provides meaningful names, use those instead of `photo-N`
- In the markdown body, reference them as:
  ```markdown
  ![Caption here](/assets/images/updates/YYYY-MM-DD-HHMM/photo-1.webp)
  ```
  Note: Reference the `.webp` version (the GitHub Action will convert `.optimize` files to `.webp` and update references automatically)

**For embed URLs** (prefixed with `embed:`):
- Use the URL directly in the markdown: `![Caption](https://the-url.com/image.jpg)`
- Do not download these

If photos have captions, use them as alt text. If no caption is provided, generate brief, descriptive alt text.

### 4. Create Branch, Commit, Push, and PR

1. Create a new branch: `update/YYYY-MM-DD-HHMM` from the current branch
2. Stage all new files (the update markdown + any downloaded images)
3. Commit with message: `Add update: YYYY-MM-DD-HHMM`
4. Push the branch: `git push -u origin update/YYYY-MM-DD-HHMM`
5. Create a PR to `master` using `gh pr create`:
   - Title: `New update: <first ~8 words of the update text>...`
   - Body: Preview of the update content and list of included photos

### Important Notes

- Do NOT ask unnecessary questions. Use sensible defaults and just create it.
- If anything is ambiguous, make a reasonable choice and note it in the PR description.
- The timezone is always `-0600` (Central Time).
- Always use `published: true` unless the user explicitly says "draft".
