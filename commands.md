---
name: commands
description: "Hashtag commands [name] can type in a prompt to trigger special behaviors (#disable memory, #enable memory, #report, #share, #backup)"
metadata: 
  node_type: memory
  type: reference
created: 2026-07-03
---

Commands [name] can type in any prompt. **Case-insensitive**; the token can appear anywhere in the message.

### #disable memory
Act memory-blind for the rest of the session: don't read any memory file, don't use
recalled facts, don't create/update memories. Answer as if no memory exists. For asking
on behalf of a friend, or any answer that shouldn't be shaped by [name]'s profile.
**Caveat:** the always-loaded index + `working-preferences.md` (~1,028 tokens) are injected
*before* the prompt is seen, so they're already in context on the turn this is invoked —
this suppresses *use*, not the initial load. True token-level non-loading needs a hook
tweak (not yet set up). Undo with **#enable memory**. [[working-preferences]]

### #enable memory
Resume normal memory behavior for the session (undoes #disable memory).

### #share `<format>` [without `<n,…>`] [with `<n,…>`]
Export the **current conversation** as a self-contained, shareable file: faithful full
reply text, all tables, every interactive diagram rebuilt as inline SVG, and any images
embedded as base64 — with [name]'s **private profile/memory references stripped out** (drop
the "I checked your profile…" asides and "no memory files changed"/"saved a memory note"
notes; keep the 🌟 Ok [name] openers, they're the point).

**Fully self-contained — no dependency on any other project's files.** Earlier versions of
this command pointed at a "reference implementation" living in `~/nanoblock-shelf/`
(an unrelated project folder); [name] flagged that as bad (2026-07-03) — the command should not
need to reach into other projects to run. The complete engine now lives **right here in this
memory file**. Every time `#share` runs: write a **brand-new, self-contained script** into the
session scratchpad (`/private/tmp/claude-<uid>/.../scratchpad/share_conversation.py` — path from
the "Scratchpad Directory" system context, never a project folder), embedding the template
below verbatim plus that conversation's own `TURNS` data. Run it from scratchpad, then discard
it — nothing is left behind in any project directory.

- **`<format>`**: `html` (default) or `pdf`. **`pdf` is the best share format** — it previews
  inline in WhatsApp / macOS Preview *and* is rendered as **one continuous page** (a small
  injected script sets `@page{size:<w> <h>}` to the full content height) so tables/diagrams
  never get split across a page break. Engine: headless Google Chrome
  (`--headless=new --no-pdf-header-footer --window-size=900,1200 --print-to-pdf`) with
  `*{print-color-adjust:exact}` so backgrounds/colors survive. `--paged` forces standard
  Letter pagination instead (rarely wanted). `html` stays continuous but needs a browser and
  won't preview inline in chat apps.
- **`without <n,…>`**: comma-separated **1-based turn numbers** (each turn = one user prompt
  in order) to drop, together with their replies *and* any diagram/image inside them.
  e.g. `#share html without 6,8`. Guard against out-of-range numbers.
- **`with <n,…>`**: the *opposite* of `without` — comma-separated **1-based turn numbers** to
  **keep**; every other turn is dropped. e.g. `#share pdf with 3,4` shares only turns 3 and 4
  (each with its reply), nothing else. Same out-of-range guard. **Mutually exclusive with
  `without`** — if both are given in one request, ask which was meant rather than guessing.
- **Always save into `~/Downloads`**, filename `<topic-slug>-share[.pdf|.html]` (a short
  slug for *this* conversation's subject, e.g. `banana-pi-router-share`, not a generic
  `conversation-share` — that generic name already collided with a prior export once).
  Announce the path; `without` exports get an `-excl-…` suffix and `with` exports get an
  `-only-…` suffix, so neither overwrites the full copy or each other.

**Engine template** (stdlib only — `argparse, base64, os, subprocess, sys`; adapt `TURNS`,
keep the rest as-is):

```python
#!/usr/bin/env python3
import argparse, base64, os, subprocess, sys

OUT_DIR = os.path.expanduser("~/Downloads")
CHROME  = "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
SLUG    = "REPLACE-ME"   # short topic slug for this conversation, e.g. "banana-pi-router"

def U(t): return f"<p>{t}</p>"   # wrap a user turn's text

# ---- fill in from the live conversation: one dict per turn ----
# assistant list items are ("text", html) | ("widget", raw_svg_str) | ("image", base64_png)
def build_turns():
    return [
        {"user": U("..."), "assistant": [("text", "<p>🌟 <strong>Ok [name]</strong> — ...</p>")]},
        # ...
    ]

CSS = """
  :root{--bg:#f6f5f1;--ink:#1a1a18;--muted:#6b6a64;--u:#e8f0fb;--ub:#cfe0f5;--c:#fbfbf9;--cb:#e7e6e0;--accent:#378add}
  *{box-sizing:border-box;-webkit-print-color-adjust:exact;print-color-adjust:exact}
  body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.6 -apple-system,"Segoe UI",Roboto,Helvetica,Arial,sans-serif}
  .wrap{max-width:900px;margin:0 auto;padding:32px 20px 64px}
  header{text-align:center;margin-bottom:24px}
  header h1{font-size:22px;margin:0 0 4px} header p{color:var(--muted);margin:0;font-size:14px}
  .msg{margin:16px 0;display:flex;flex-direction:column} .msg.user{align-items:flex-end}
  .who{font-size:12px;color:var(--muted);margin:0 6px 4px;font-weight:600}
  .bubble{max-width:94%;padding:12px 18px;border-radius:16px;border:1px solid var(--cb);background:var(--c);box-shadow:0 1px 2px rgba(0,0,0,.04)}
  .user .bubble{background:var(--u);border-color:var(--ub);max-width:88%}
  .bubble p{margin:0 0 10px} .bubble p:last-child{margin:0}
  .bubble h4{margin:16px 0 6px;font-size:15px}
  .bubble code{background:#00000010;padding:1px 5px;border-radius:5px;font-size:.9em}
  .bubble b,.bubble strong{color:var(--accent)} .bubble a{color:var(--accent)}
  ol,ul{margin:6px 0 10px 22px;padding:0} li{margin:4px 0}
  table{border-collapse:collapse;width:100%;margin:8px 0 12px;font-size:12.5px}
  th,td{border:1px solid var(--cb);padding:5px 8px;text-align:left;vertical-align:top} th{background:#0000000a}
  figure.diagram{margin:16px auto;padding:16px;background:#fff;border:1px solid var(--cb);border-radius:14px;box-shadow:0 1px 2px rgba(0,0,0,.04);break-inside:avoid}
  figure.diagram svg{width:100%;height:auto;display:block} figure.diagram img{width:100%;height:auto;display:block;border-radius:8px}
  footer{text-align:center;color:var(--muted);font-size:12px;margin-top:36px}
  @page{margin:14mm}
"""

def render_html(turns, exclude, title):
    parts = []
    for i, turn in enumerate(turns, 1):
        if i in exclude: continue
        parts.append(f'<div class="msg user"><div class="who">You</div><div class="bubble">{turn["user"]}</div></div>')
        for kind, payload in turn["assistant"]:
            if kind == "text":
                parts.append(f'<div class="msg claude"><div class="who">Claude</div><div class="bubble">{payload}</div></div>')
            elif kind == "widget":
                parts.append(f'<figure class="diagram">{payload}</figure>')
            elif kind == "image":
                parts.append(f'<figure class="diagram"><img alt="diagram" src="data:image/png;base64,{payload}" /></figure>')
    note = f'<footer>Faithful export of a Claude Code conversation · personal context removed{(" · excluding turns " + ",".join(map(str, sorted(exclude)))) if exclude else ""}</footer>'
    return ('<!doctype html><html lang="en"><head><meta charset="utf-8">'
            '<meta name="viewport" content="width=device-width, initial-scale=1">'
            f'<title>{title}</title><style>' + CSS + '</style></head><body><div class="wrap">'
            f'<header><h1>{title}</h1></header>' + "".join(parts) + note + '</div></body></html>')

PAGE_SCRIPT = ("<script>function setPage(){var r=document.body.getBoundingClientRect();"
               "var st=document.getElementById('pg')||document.createElement('style');st.id='pg';"
               "st.textContent='@page{size:'+Math.ceil(r.width)+'px '+Math.ceil(r.height)+'px;margin:0}';"
               "document.head.appendChild(st);}setPage();window.addEventListener('load',setPage);</script>")

def to_pdf(html_path, pdf_path, single_page=True):
    src_url = "file://" + html_path
    tmp = None
    if single_page:
        html = open(html_path).read().replace("</body>", PAGE_SCRIPT + "</body>")
        tmp = html_path + ".longtmp.html"
        with open(tmp, "w") as f: f.write(html)
        src_url = "file://" + tmp
    subprocess.run([CHROME, "--headless=new", "--disable-gpu", "--no-pdf-header-footer",
                    "--window-size=900,1200", f"--print-to-pdf={pdf_path}", src_url],
                   check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    if tmp: os.remove(tmp)

def main():
    ap = argparse.ArgumentParser()
    ap.add_argument("format", nargs="?", default="html")
    ap.add_argument("--without", default="")
    ap.add_argument("--with", dest="only", default="")   # 'with' is a Python keyword, dest avoids the clash
    ap.add_argument("--paged", action="store_true")
    a = ap.parse_args()
    fmt = a.format.lower()
    if fmt not in ("html", "pdf"): sys.exit("format must be 'html' or 'pdf'")
    drop = {int(x) for x in a.without.replace(" ", "").split(",") if x}
    keep_only = {int(x) for x in a.only.replace(" ", "").split(",") if x}
    if drop and keep_only: sys.exit("use --with or --without, not both")
    turns = build_turns()
    n = len(turns)
    bad = [x for x in (drop or keep_only) if x < 1 or x > n]
    if bad: sys.exit(f"no such turn(s): {bad} (conversation has {n} turns)")
    if keep_only:
        exclude = set(range(1, n + 1)) - keep_only
        suffix = "-only-" + "-".join(map(str, sorted(keep_only)))
    elif drop:
        exclude = drop
        suffix = "-excl-" + "-".join(map(str, sorted(drop)))
    else:
        exclude = set()
        suffix = ""
    html_path = os.path.join(OUT_DIR, f"{SLUG}-share{suffix}.html")
    with open(html_path, "w") as f:
        f.write(render_html(turns, exclude, title=SLUG.replace("-", " ").title()))
    kept = [x for x in range(1, n + 1) if x not in exclude]
    print(f"turns kept: {kept}" + (f"  (dropped {sorted(exclude)})" if exclude else ""))
    if fmt == "html":
        print("wrote", html_path)
    else:
        pdf_path = os.path.join(OUT_DIR, f"{SLUG}-share{suffix}.pdf")
        to_pdf(html_path, pdf_path, single_page=not a.paged)
        print("wrote", pdf_path)

if __name__ == "__main__":
    main()
```

### #backup
Snapshot the whole memory folder. Run the backup script:
`/Users/[name]/.claude/projects/-Users-[name]/memory-backups/backup.sh`
It copies `memory/` into a new timestamped folder under `memory-backups/`
(sibling of `memory/`, kept out of the recall scan path), keeps the newest 30
snapshots and auto-prunes older ones. Report back the snapshot path + file count.
Restore instructions live in `memory-backups/README.md`. _(added 2026-07-03)_

### #report
Produce a full memory report:
- **Mandatory per-turn overhead** (paid every turn): `MEMORY.md` index + `working-preferences.md`
  token estimates and their combined total. Note: Claude Code's own system prompt is also
  always loaded but isn't a readable file, so it can't be measured here.
- **On-demand files:** a table of every memory file with its ~token cost (loaded only when read).
- **Totals:** sum across all files + file count.
- **Trim candidates:** largest files flagged for review.
- Token counts are estimates (~chars ÷ 4, roughly ±15%), not an exact tokenizer count.

Related: [[working-preferences]].
