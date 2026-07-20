# Claude Code Mode

Load only when this skill is running inside Claude Code, where real sub-agents and a headless browser are available.

## Build + audit loop

1. A **build sub-agent** produces the HTML following the main SKILL.md's Steps 1-3.75.
2. A separate **audit sub-agent**, with no memory of the build agent's decisions, reviews the output fresh against the Step 3 self-check list. If the project has `PRODUCT.md`/`DESIGN.md` context files for the `impeccable` skill, also check against its shared design laws (proportion, hierarchy, spacing rhythm, contrast) — skip if absent.
3. Loop: audit finds issues → build agent fixes → re-audit → repeat until clean or a 3-round cap is hit.
4. Present the final file with a one-line summary of what the loop caught and fixed.

## Automated exact-pixel export

Skip the manual True Size + screenshot flow — export directly via Playwright.

- Render at a small viewport matching the canvas's aspect ratio (e.g. 432×540 for a 1080×1350 canvas — exactly 1/2.5 scale), and set `device_scale_factor` to bring it back to full resolution (2.5 here). Setting the viewport directly to the full 1080×1350 causes layout reflow; rendering small and scaling up on capture doesn't.
- Wait for web fonts to finish loading (~2-3s) before capturing, or headlines render in a fallback font.
- Use `page.screenshot(clip=...)` scoped to just the canvas element so the edit panel never leaks into the export.

```python
import asyncio
from pathlib import Path
from playwright.async_api import async_playwright

INPUT_HTML = Path("/path/to/template.html")
OUTPUT_PNG = Path("/path/to/output/template.png")

VIEW_W, VIEW_H = 432, 540   # 1080x1350 at 1/2.5 scale
SCALE = 1080 / VIEW_W       # = 2.5

async def export():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page(
            viewport={"width": VIEW_W, "height": VIEW_H},
            device_scale_factor=SCALE,
        )
        await page.set_content(INPUT_HTML.read_text(encoding="utf-8"), wait_until="networkidle")
        await page.wait_for_timeout(3000)  # let Google Fonts load

        await page.evaluate("document.querySelector('#panel').style.display = 'none'")
        canvas = await page.query_selector(".canvas")
        await canvas.screenshot(path=str(OUTPUT_PNG))

asyncio.run(export())
```

## Common export mistakes

| Mistake | What breaks | Fix |
|---|---|---|
| Viewport set to 1080×1350 directly | Layout reflows, fonts/spacing shift | Small viewport + `device_scale_factor` |
| Shell script generating the HTML | `$`/backticks corrupt any embedded base64 | Python `Path.write_text()` |
| Not waiting for fonts | Headlines render in fallback system font | `wait_for_timeout(3000)` after load |
| Screenshotting full page | Captures the edit panel too | Scope `screenshot()` to `.canvas` element only |