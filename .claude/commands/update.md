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
- **Google Photos shared album URL** (`photos.app.goo.gl/...` or `photos.google.com/share/...`): Fetch the album page, extract `lh3.googleusercontent.com` image URLs, and download all of them into the repo (do not ask which to include). If fetching fails, tell the user to open the album and share individual photo URLs instead.
- **Local file paths** (e.g., `~/Photos/sunset.jpg`): Copy into the repo's image directory.

### 2. Create the Update File

- **Get the current time**: Run `TZ=America/Chicago date "+%Y-%m-%d-%H%M"` to obtain the current date/time in Chicago time. Use this value for both the filename and the frontmatter `date` field. Also run `TZ=America/Chicago date "+%z"` to get the current UTC offset (e.g., `-0500` during CDT or `-0600` during CST).
- **Filename**: `_updates/YYYY-MM-DD-HHMM.md` using the current date and time (24-hour format)
  - Example: `_updates/2026-03-01-1435.md`
- **Frontmatter**:
  ```yaml
  ---
  date: YYYY-MM-DD HH:MM:00 <offset>
  published: true
  image: /assets/images/updates/YYYY-MM-DD-HHMM/first-image.webp
  ---
  ```
  - Use the current date/time
  - Use the UTC offset obtained from the `date "+%z"` command
  - Set `image:` to the path of the first image in the update. If the update has no images, omit the `image:` key entirely (it will fall back to `og-default.webp`).
- **Body**: The update text as markdown. Keep it exactly as the user wrote it, just clean up obvious typos if any.

### 3. Handle Images

All images — regardless of source — are downloaded into the repo and referenced as `.webp`. The GitHub Actions workflow (`.github/workflows/optimize-images.yml`) converts `.optimize-1200w` files to optimized `.webp` on push.

**Destination directory**: `assets/images/updates/YYYY-MM-DD-HHMM/`

**Naming**: Use a slugified version of the original filename when meaningful (e.g., `morning-light-on-river.optimize-1200w.jpg`). Fall back to `photo-1`, `photo-2`, etc. when the URL filename is opaque (hashes, UUIDs, query-string-only URLs).

**Markdown reference**: Always reference the `.webp` version:
```markdown
![Caption](/assets/images/updates/YYYY-MM-DD-HHMM/morning-light-on-river.webp)
```

**Captions**: Use the user-provided caption as alt text if given. Otherwise, leave the alt text empty (e.g., `![](/path/to/image.webp)`).

**Acquisition by source type**:

- **Single image URL**: Download with `curl -L -o` into the destination directory with `.optimize-1200w.{ext}` naming.
- **Google Photos album** (`photos.app.goo.gl/...` or `photos.google.com/share/...`):
  - Fetch the album page HTML using curl
  - Extract all `lh3.googleusercontent.com` image URLs
  - Filter for unique URLs; append `=w2048` if no size suffix is present
  - Download all images with `.optimize-1200w.jpg` naming
- **Local file path**: `cp` into the destination directory with `.optimize-1200w.{ext}` naming.

### 4. Commit and Push to Master

1. Run `git pull` to ensure the local branch is up to date with the remote
2. Stage all new files (the update markdown + any downloaded images)
3. Commit with message: `Add update: YYYY-MM-DD-HHMM`
4. Push to `origin master`

### Important Notes

- Do NOT ask unnecessary questions. Use sensible defaults and just create it.
- If anything is ambiguous, make a reasonable choice.
- Minimize the number of Bash tool calls to reduce permission prompts. Chain related shell commands together with `&&` (e.g., `mkdir -p dir && curl ... && curl ...`). Combine the git pull, add, commit, and push into a single Bash call.
- All timestamps must be derived using `TZ=America/Chicago date` — never rely on the system default timezone. The UTC offset must be obtained dynamically via `date "+%z"`.
- Always use `published: true` unless the user explicitly says "draft".
