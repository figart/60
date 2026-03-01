# Create a New Update

Create a new update post for my Jekyll site based on the user's input below. Follow these instructions precisely.

## User Input

$ARGUMENTS

## Instructions

### 1. Parse the Input

The user's input will contain:
- **Update text**: The body content for the update (required)
- **Image URLs**: Zero or more image URLs pasted inline (optional). These are downloaded into the repo and optimized.
- **Captions**: Optional captions for images, provided near each URL (e.g., `https://url.com/photo.jpg "Morning light on the river"`)
- **Google Photos shared album URL** (`photos.app.goo.gl/...` or `photos.google.com/share/...`): Fetch the album page, extract `lh3.googleusercontent.com` image URLs, confirm with the user which to include, then download them into the repo. If fetching fails, tell the user to open the album and share individual photo URLs instead.
- **Local file paths** (e.g., `~/Photos/sunset.jpg`): Copy into the repo's image directory.

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

### 3. Handle Images

All images — regardless of source — are downloaded into the repo and referenced as `.webp`. The GitHub Actions workflow (`.github/workflows/optimize-images.yml`) converts `.optimize-1200w` files to optimized `.webp` on push.

**Destination directory**: `assets/images/updates/YYYY-MM-DD-HHMM/`

**Naming**: Use a slugified version of the original filename when meaningful (e.g., `morning-light-on-river.optimize-1200w.jpg`). Fall back to `photo-1`, `photo-2`, etc. when the URL filename is opaque (hashes, UUIDs, query-string-only URLs).

**Markdown reference**: Always reference the `.webp` version:
```markdown
![Caption](/assets/images/updates/YYYY-MM-DD-HHMM/morning-light-on-river.webp)
```

**Captions**: Use the user-provided caption as alt text if given. Otherwise, generate brief descriptive alt text.

**Acquisition by source type**:

- **Single image URL**: Download with `curl -L -o` into the destination directory with `.optimize-1200w.{ext}` naming.
- **Google Photos album** (`photos.app.goo.gl/...` or `photos.google.com/share/...`):
  - Fetch the album page HTML using curl
  - Extract all `lh3.googleusercontent.com` image URLs
  - Filter for unique URLs; append `=w2048` if no size suffix is present
  - Confirm with the user how many images were found and which to include
  - Download selected images with `.optimize-1200w.jpg` naming
- **Local file path**: `cp` into the destination directory with `.optimize-1200w.{ext}` naming.

### 4. Create Branch, Commit, Push, and PR

1. Create a new branch: `update/YYYY-MM-DD-HHMM` from the current branch
2. Stage all new files (the update markdown + any downloaded images)
3. Commit with message: `Add update: YYYY-MM-DD-HHMM`
4. Push the branch: `git push -u origin update/YYYY-MM-DD-HHMM`
5. Create a PR to `master` using `gh pr create`:
   - Title: `New update: <first ~8 words of the update text>...`
   - Body: Preview of the update content and list of included images

### Important Notes

- Do NOT ask unnecessary questions. Use sensible defaults and just create it.
- If anything is ambiguous, make a reasonable choice and note it in the PR description.
- The timezone is always `-0600` (Central Time).
- Always use `published: true` unless the user explicitly says "draft".
