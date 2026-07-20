# Image Embedding

Load only when the user provides their own photo or screenshot to place in the template. Never applies to reference-image assets — those get labeled placeholder boxes per the Guardrails, not embedded.

## Critical rules

1. Never use a relative path (`photo.jpg`) — breaks outside the exact folder the HTML lives in.
2. Never use `background: url(filepath)` — large images inline as background strings can crash the parser.
3. Always embed as a base64 `data:` URI in an `<img>` tag.
4. Always generate the HTML via Python (`Path.write_text()`), not shell heredocs — `$` and backticks in a base64 string get corrupted by shell interpolation.

## Steps

```bash
# Check the actual format — an extension may lie
file /path/to/image.png
```

```python
import base64
from pathlib import Path

img_path = Path("/path/to/image.png")
mime = "image/jpeg"  # match whatever `file` reported
b64 = base64.b64encode(img_path.read_bytes()).decode()
data_uri = f"data:{mime};base64,{b64}"

html = f"""
<div style="position:relative;width:100%;height:100%;">
  <img src="{data_uri}" style="position:absolute;inset:0;width:100%;height:100%;object-fit:cover;z-index:0;">
  <div style="position:absolute;inset:0;background:rgba(255,255,255,0.35);z-index:1;"></div>
  <!-- content goes here, z-index:2+ -->
</div>
"""
Path("/home/claude/template.html").write_text(html, encoding="utf-8")
```

If the image sits behind text, add the overlay shown above (`rgba(255,255,255,0.35)` on light backgrounds, `rgba(0,0,0,0.45)` on dark) so text stays readable regardless of what's dropped in.

## Common mistakes

| Mistake | What breaks | Fix |
|---|---|---|
| `<img src="photo.png">` | Broken image outside the original folder | base64 `data:` URI |
| `background: url('data:...')` inline | Parser crash on large strings | `<img>` tag with `object-fit: cover` |
| Shell `echo`/heredoc to write the HTML | `$`/backticks corrupt the base64 | Python `Path.write_text()` |
| Trusting the file extension | `.png` may actually be JPEG, wrong MIME breaks rendering | Run `file` first |