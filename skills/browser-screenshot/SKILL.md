---
name: browser-screenshot
description: Capture full-page screenshots of web pages via the built-in browser tool, with lazy-load handling and visual QA. Use when asked to browse a URL and take a screenshot or describe a page visually.
metadata: {"clawdbot":{"emoji":"📸"}}
---

# browser-screenshot — Capture & Describe Web Pages

Take full-page screenshots of web pages with proper image loading, and describe what you see.

## When to use

- "Browse X and take a screenshot"
- "What does Y look like?"
- Visual QA of deployed sites
- Documenting page state at a point in time

## Workflow

### 1. Open the page

```
browser(action="open", url="<url>", target="host", loadState="networkidle")
```

- Always use `target="host"` — sandbox browser is `attachOnly` and won't start
- `loadState="networkidle"` waits for initial network activity to settle
- Save the returned `targetId` for subsequent calls
- **If first attempt times out:** retry once — Chrome starts on-demand, second attempt succeeds

### 2. Trigger lazy-loaded images

Scroll to bottom to fire lazy-load, wait for images, scroll back to top:

```
browser(action="act", target="host", targetId="<id>",
  request={"kind": "evaluate", "fn": "window.scrollTo(0, document.body.scrollHeight)"})

browser(action="act", target="host", targetId="<id>",
  request={"kind": "wait", "timeMs": 3000})

browser(action="act", target="host", targetId="<id>",
  request={"kind": "evaluate", "fn": "window.scrollTo(0, 0)"})

browser(action="act", target="host", targetId="<id>",
  request={"kind": "wait", "timeMs": 2000})
```

### 3. Take the screenshot

```
browser(action="screenshot", target="host", targetId="<id>", fullPage=true)
```

The result is a `MEDIA:` path — the image is automatically attached and visible.

Use `fullPage=false` for above-the-fold only (hero screenshots, quick checks).

### 4. Describe what you see

After the screenshot:
- Layout and sections (hero, nav, content blocks, footer)
- Key text (headlines, CTAs, subheadings)
- Images and visual elements present/missing
- Any issues (broken images, layout problems, missing content, misaligned elements)

## Error handling

| Error | Fix |
|-------|-----|
| First `open` times out | Retry once — Chrome cold start |
| `tab not found` | The tab closed — call `open` again to get a new `targetId` |
| Images still blank after scroll | Increase `timeMs` in the wait steps |

## Notes

- **Lazy loading:** Modern sites (Next.js, React) use lazy-load — images only render when scrolled into view. Always scroll before a full-page screenshot.
- **Full page vs viewport:** `fullPage=true` for documentation; `fullPage=false` for above-the-fold.
- **Screenshots land at:** `/home/openclaw/.openclaw/media/browser/` on the host (bind-mounted into sandbox at `/data/browser-screenshots/`)
- **Tab reuse:** Reuse `targetId` across calls in the same session for efficiency. Re-open if you get `tab not found`.
