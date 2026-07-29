# Quotably Landing Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the letterpress-style landing page with the approved dark "statement poster" design from `docs/superpowers/specs/2026-07-28-landing-redesign-design.md`.

**Architecture:** Single static `index.html` (vanilla HTML/CSS/JS, no build step) served by the existing GitHub Pages workflow. Self-hosted variable woff2 fonts in `assets/fonts/`. Existing logo PNGs recolored to white via CSS filter.

**Tech Stack:** Plain HTML/CSS/JS, Geist + Geist Mono (self-hosted woff2), Python http.server for local verification.

## Global Constraints

- Page copy is locked; use it byte-for-byte as written in Task 2 (US spelling "judgment").
- Zero em-dash (—) or en-dash (–) characters anywhere in visible page text, title, or meta tags.
- One accent color only: `#E8B04B`, used only for the email link.
- Dark theme locked at page level; no `prefers-color-scheme` variation.
- No external network requests from the page (no Google Fonts links, no CDNs).
- `min-height: 100dvh`, never `h-screen`-style `100vh`-only sizing.
- Entrance animation must collapse to instant under `prefers-reduced-motion: reduce`.
- Reference mockup (approved): `.superpowers/brainstorm/22643-1785272620/content/final-mockup.html`. Production matches it except: self-hosted fonts, muted text color `#7D92AA` instead of `#54687D`.

---

### Task 1: Self-hosted fonts

**Files:**
- Create: `assets/fonts/geist-latin.woff2`
- Create: `assets/fonts/geist-mono-latin.woff2`

**Interfaces:**
- Produces: two variable-weight woff2 files at exactly these paths; Task 2's `@font-face` rules reference `assets/fonts/geist-latin.woff2` and `assets/fonts/geist-mono-latin.woff2` with `font-weight: 100 900`.

- [ ] **Step 1: Download the variable woff2 files from Google Fonts**

Google Fonts serves woff2 only to woff2-capable user agents, so pass a modern Chrome UA. From the repo root:

```bash
mkdir -p assets/fonts
UA="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
GEIST_URL=$(curl -s -A "$UA" "https://fonts.googleapis.com/css2?family=Geist:wght@100..900&display=swap" | grep -o "https://fonts.gstatic.com/[^)]*latin[^)]*\.woff2" | tail -1)
MONO_URL=$(curl -s -A "$UA" "https://fonts.googleapis.com/css2?family=Geist+Mono:wght@100..900&display=swap" | grep -o "https://fonts.gstatic.com/[^)]*latin[^)]*\.woff2" | tail -1)
curl -s -o assets/fonts/geist-latin.woff2 "$GEIST_URL"
curl -s -o assets/fonts/geist-mono-latin.woff2 "$MONO_URL"
```

Note: the css2 response lists multiple subsets; each `/* latin */` comment precedes its URL. The grep above keeps URLs whose path contains `latin`; `tail -1` picks the plain latin subset (listed after latin-ext). If the grep returns nothing, dump the css2 response and pick the URL under the `/* latin */` comment manually.

- [ ] **Step 2: Verify the files are real woff2 (magic bytes `wOF2`), non-trivial size**

```bash
for f in assets/fonts/geist-latin.woff2 assets/fonts/geist-mono-latin.woff2; do
  head -c 4 "$f" | grep -q "wOF2" && echo "$f OK ($(wc -c < "$f") bytes)" || echo "$f BAD"
done
```

Expected: both lines end in `OK` with sizes roughly 15,000-80,000 bytes. If either prints `BAD`, the download grabbed CSS or an error page; redo Step 1.

- [ ] **Step 3: Commit**

```bash
git add assets/fonts
git commit -m "Add self-hosted Geist and Geist Mono variable fonts"
```

---

### Task 2: Rewrite index.html as the statement poster

**Files:**
- Modify: `index.html` (full replacement of current content)

**Interfaces:**
- Consumes: `assets/fonts/geist-latin.woff2`, `assets/fonts/geist-mono-latin.woff2` (Task 1); `assets/wordmark.png`, `assets/q-glyph.png` (already in repo).
- Produces: the final page. Task 3 verifies it; element ids used by JS and tests: `#email`, `#copyBtn`.

- [ ] **Step 1: Replace the entire contents of index.html with:**

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Quotably · Custom software consulting</title>
<meta name="description" content="Quotably helps software teams build the judgment to ship quality software at the new pace. Custom software consulting by Ben Tucker.">
<link rel="icon" type="image/png" href="assets/q-glyph.png">
<style>
  @font-face {
    font-family: 'Geist';
    src: url('assets/fonts/geist-latin.woff2') format('woff2');
    font-weight: 100 900;
    font-display: swap;
  }
  @font-face {
    font-family: 'Geist Mono';
    src: url('assets/fonts/geist-mono-latin.woff2') format('woff2');
    font-weight: 100 900;
    font-display: swap;
  }

  :root {
    --ink: #0F2438;
    --ink-deep: #0A1A2B;
    --ink-lift: #12293F;
    --bone: #F4EFE6;
    --slate: #92A7BC;
    --muted: #7D92AA;
    --amber: #E8B04B;
  }

  * { box-sizing: border-box; }

  html, body { margin: 0; padding: 0; }

  body {
    min-height: 100vh;
    min-height: 100dvh;
    background: radial-gradient(120% 90% at 20% 0%, var(--ink-lift) 0%, var(--ink) 45%, var(--ink-deep) 100%);
    color: var(--bone);
    font-family: 'Geist', -apple-system, 'Helvetica Neue', sans-serif;
    -webkit-font-smoothing: antialiased;
    display: grid;
    grid-template-rows: auto 1fr auto;
    overflow-x: hidden;
  }

  header, main, footer {
    padding-left: clamp(24px, 6vw, 88px);
    padding-right: clamp(24px, 6vw, 88px);
  }

  header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-top: 32px;
    padding-bottom: 32px;
  }
  header img {
    height: 30px;
    width: auto;
    filter: brightness(0) invert(1);
    opacity: 0.92;
  }
  .tag {
    font-family: 'Geist Mono', ui-monospace, Menlo, monospace;
    font-size: 11px;
    letter-spacing: 0.22em;
    text-transform: uppercase;
    color: var(--muted);
  }

  main { display: flex; align-items: center; }

  .statement { max-width: 880px; }

  h1 {
    font-size: clamp(38px, 6.2vw, 76px);
    font-weight: 700;
    letter-spacing: -0.025em;
    line-height: 1.06;
    margin: 0;
    color: var(--bone);
  }

  .sub {
    margin: 28px 0 0;
    font-size: clamp(16px, 1.6vw, 20px);
    font-weight: 500;
    line-height: 1.6;
    color: var(--slate);
    max-width: 56ch;
  }
  .sub strong { color: var(--bone); font-weight: 600; }

  .contact {
    margin-top: 40px;
    display: inline-flex;
    align-items: baseline;
    gap: 18px;
  }
  .email {
    font-size: clamp(17px, 1.7vw, 22px);
    font-weight: 600;
    color: var(--amber);
    text-decoration: none;
    border-bottom: 1px solid rgba(232, 176, 75, 0.35);
    padding-bottom: 3px;
    transition: border-color 0.25s;
  }
  .email:hover,
  .email:focus-visible { border-color: var(--amber); }

  .copy-btn {
    font-family: 'Geist Mono', ui-monospace, Menlo, monospace;
    font-size: 11px;
    letter-spacing: 0.16em;
    text-transform: uppercase;
    color: var(--muted);
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    transition: color 0.2s;
  }
  .copy-btn:hover { color: var(--slate); }

  footer {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    gap: 32px;
    padding-top: 40px;
    padding-bottom: 40px;
  }
  .quote {
    margin: 0;
  }
  .quote p {
    font-family: 'Iowan Old Style', Palatino, Georgia, serif;
    font-style: italic;
    font-size: 14px;
    line-height: 1.6;
    color: var(--muted);
    max-width: 44ch;
    margin: 0;
  }
  .quote figcaption {
    margin-top: 6px;
    font-family: 'Geist Mono', ui-monospace, Menlo, monospace;
    font-size: 10px;
    letter-spacing: 0.18em;
    text-transform: uppercase;
    color: var(--muted);
  }
  footer > img {
    width: clamp(120px, 14vw, 190px);
    height: auto;
    filter: brightness(0) invert(1);
    opacity: 0.9;
  }

  .rise {
    opacity: 0;
    transform: translateY(14px);
    animation: rise 0.9s cubic-bezier(0.2, 0.7, 0.2, 1) forwards;
  }
  .d1 { animation-delay: 0.1s; }
  .d2 { animation-delay: 0.3s; }
  .d3 { animation-delay: 0.5s; }
  .d4 { animation-delay: 0.8s; }
  @keyframes rise { to { opacity: 1; transform: none; } }

  @media (prefers-reduced-motion: reduce) {
    .rise { animation-duration: 0.001s; animation-delay: 0s; }
  }

  @media (max-width: 640px) {
    footer { flex-direction: column; align-items: flex-start; }
    footer > img { order: -1; }
  }
</style>
</head>
<body>

<header class="rise d1">
  <img src="assets/q-glyph.png" alt="Quotably Q monogram">
  <span class="tag">Custom Software Consulting</span>
</header>

<main>
  <div class="statement">
    <h1 class="rise d1">The bottleneck isn't<br>writing code anymore.</h1>
    <p class="sub rise d2">It's everyone on the team able to make the right calls at the new pace. <strong>We help teams build that judgment at scale.</strong></p>
    <div class="contact rise d3">
      <a class="email" id="email" href="mailto:ben@quotably.com">ben@quotably.com</a>
      <button class="copy-btn" id="copyBtn" type="button">Copy</button>
    </div>
  </div>
</main>

<footer class="rise d4">
  <figure class="quote">
    <p>&ldquo;The most damaging phrase in the language is &lsquo;We&rsquo;ve always done it this way.&rsquo;&rdquo;</p>
    <figcaption>Rear Adm. Grace Hopper, 1987</figcaption>
  </figure>
  <img src="assets/wordmark.png" alt="Quotably">
</footer>

<script>
  (function () {
    var btn = document.getElementById('copyBtn');
    if (!navigator.clipboard) { btn.hidden = true; return; }
    btn.addEventListener('click', function () {
      navigator.clipboard.writeText('ben@quotably.com').then(function () {
        btn.textContent = 'Copied';
        setTimeout(function () { btn.textContent = 'Copy'; }, 1600);
      });
    });
  })();
</script>

</body>
</html>
```

- [ ] **Step 2: Static checks**

```bash
# No em/en dashes anywhere in the page:
grep -cE "—|–" index.html
# Expected: 0 (grep exits 1 with "0" output)

# Locked copy present verbatim:
grep -c "The bottleneck isn't" index.html          # Expected: 1
grep -c "judgment at scale" index.html             # Expected: 1
grep -c "ben@quotably.com" index.html              # Expected: 3 (href, link text, clipboard)

# No external requests:
grep -cE "googleapis|gstatic|cdn\." index.html     # Expected: 0
```

- [ ] **Step 3: Serve locally and confirm assets resolve**

```bash
python3 -m http.server 8123 >/dev/null 2>&1 &
sleep 1
for p in "" assets/q-glyph.png assets/wordmark.png assets/fonts/geist-latin.woff2 assets/fonts/geist-mono-latin.woff2; do
  curl -s -o /dev/null -w "%{http_code} /$p\n" "http://localhost:8123/$p"
done
kill %1
```

Expected: five lines, all `200`.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Redesign landing page as dark statement poster"
```

---

### Task 3: Visual and functional verification

**Files:**
- No file changes expected; fix regressions in `index.html` if found.

**Interfaces:**
- Consumes: the served page from Task 2 (`#email`, `#copyBtn`).

- [ ] **Step 1: Serve the site**

```bash
python3 -m http.server 8123 >/dev/null 2>&1 &
```

- [ ] **Step 2: Browser checks with Playwright MCP tools**

Open `http://localhost:8123/` with the Playwright browser tools and verify, taking a screenshot at each size:

1. 1440x900: headline renders in Geist (not fallback), two lines; no vertical or horizontal scrollbar (`document.documentElement.scrollHeight <= window.innerHeight` and same for width); quote bottom-left; white wordmark bottom-right; email in amber.
2. 768x1024: layout intact, no overflow.
3. 375x812: footer stacked with wordmark above quote, everything left-aligned, no horizontal scroll.
4. Click `#copyBtn`: its text becomes `Copied`, then returns to `Copy` after ~1.6s. (If the headless clipboard write rejects, grant clipboard permissions or verify the click handler fires without throwing.)
5. Emulate `prefers-reduced-motion: reduce`, reload: content visible immediately (no 0.8s stagger).

Expected: all five pass. Screenshots saved to the scratchpad directory for the final report.

- [ ] **Step 3: Stop the server**

```bash
kill %1 2>/dev/null || pkill -f "http.server 8123"
```

- [ ] **Step 4: Commit (only if fixes were needed)**

```bash
git add index.html
git commit -m "Fix visual regressions found in verification"
```
