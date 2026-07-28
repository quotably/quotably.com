# Quotably.com Landing Redesign: Statement Poster

Date: 2026-07-28
Status: Approved direction; pending final spec review

## Purpose

Replace the current heritage-letterpress brand plate with a dark "statement poster" that says what Quotably does. The page is a placeholder with intent: nobody is funneled here yet, but it should represent where the business is going and read sharp to anyone who looks it up.

Positioning it expresses: the way software gets built is changing fast; Quotably helps teams become AI-native by building team-wide judgment, not by burning tokens.

## What was wrong with the current page

- The email CTA anchor and its label are empty in the HTML, so the page has no visible action and the lower half of the viewport is dead space.
- The page never says what Quotably does; the Grace Hopper quote occupies the value-prop slot.
- The aged-paper aesthetic (cream, hairlines, "Est. 2025") contradicts a message about rethinking how software gets made.

## Content (final, locked with Ben)

- Header left: Q glyph (assets/q-glyph.png, rendered white)
- Header right, mono small caps: "Custom Software Consulting"
- Headline (H1, two lines): "The bottleneck isn't writing code anymore."
- Subline: "It's everyone on the team able to make the right calls at the new pace. **We help teams build that judgment at scale.**" (second sentence visually emphasized in bone white; US spelling "judgment")
- Contact: mailto link "ben@quotably.com" plus an explicit "Copy" button with a "Copied" confirmation state
- Footer left: quote in italic serif: "The most damaging phrase in the language is 'We've always done it this way.'" with attribution line "REAR ADM. GRACE HOPPER, 1987" in mono small caps
- Footer right: wordmark (assets/wordmark.png, rendered white) as the signature
- Title tag: "Quotably · Custom software consulting" (no em-dash)
- Meta description: "Quotably helps software teams build the judgment to ship quality software at the new pace. Custom software consulting by Ben Tucker."
- Favicon: q-glyph.png

Removed: "Est. 2025", the eyebrow with flanking hairlines, the line-dot-line divider, the invisible shift-click-to-copy interaction, the giant centered wordmark hero.

## Visual system

- Theme: dark, locked at page level (brand choice; ignores prefers-color-scheme).
- Surface: radial gradient, #12293F at top-left through #0F2438 to #0A1A2B (the wordmark's own ink navy family). No flat fill.
- Text: bone #F4EFE6 (headline, emphasized sentence), slate #92A7BC (subline), muted #7D92AA (quote, labels; chosen to clear WCAG AA 4.5:1 against the surface, unlike the earlier #54687D draft which measured ~2.8:1).
- Accent: amber #E8B04B, used only for the email link. Single accent, page-wide.
- Type: Geist 500/600/700 for headline, subline, email; Geist Mono 400 for the two small labels and quote attribution; system serif stack (Iowan Old Style, Palatino, Georgia) italic for the quote line. Fonts self-hosted as woff2 in assets/fonts/ with font-display: swap; no Google Fonts link in production.
- Logos: both PNGs are flat navy on transparent; rendered white via CSS filter (brightness(0) invert(1)) at slight sub-100% opacity.

## Layout

- Grid: header / main / footer rows, min-height 100dvh, no scroll at desktop.
- Statement block: left-aligned, vertically centered, max-width 880px, horizontal padding clamp(24px, 6vw, 88px).
- Headline scale: clamp(38px, 6.2vw, 76px), letter-spacing -0.025em, line-height 1.06.
- Footer: quote bottom-left (max 44ch), wordmark bottom-right at clamp(120px, 14vw, 190px).
- Mobile (<= 640px): footer stacks left-aligned with wordmark above quote; everything else scales via clamps. No content is dropped.

## Motion

- One staggered entrance on load: header, headline, subline, contact, footer (translateY 14px + fade, 0.9s, delays 0.1s to 0.8s).
- Hover: email underline strengthens; Copy button brightens.
- prefers-reduced-motion: entrance collapses to instant.
- Nothing loops; no scroll-driven effects.

## Implementation

- Single static index.html, vanilla HTML/CSS/JS, no build step (matches existing GitHub Pages deploy workflow).
- Add assets/fonts/ with Geist and Geist Mono woff2 subsets (weights listed above), @font-face declarations, font-display: swap.
- Clipboard copy via navigator.clipboard.writeText with graceful no-op if unavailable; button text swaps to "Copied" for ~1.6s.
- Keep semantic structure: header, main with single h1, footer; alt text on both logo images; the quote marked up as a blockquote-equivalent with visible attribution.

## Verification

- Contrast: spot-check amber (~8:1), slate (~6.4:1), muted (~4.9:1) against #0F2438; all clear AA.
- Visual: load locally in a real browser at desktop, ~768px, and ~375px widths; confirm no scrollbar at desktop, footer stacks on mobile, entrance animation plays once and respects reduced motion.
- Functional: mailto link opens mail client; Copy button writes to clipboard and shows confirmation.
- Reference mockup: .superpowers/brainstorm/22643-1785272620/content/final-mockup.html (approved by Ben 2026-07-28; production should match it apart from self-hosted fonts and the AA-fixed muted color).

## Out of scope

- No additional sections, pages, or nav; the page remains a single statement screen.
- No analytics, no forms.
- Logo redesign: the wordmark and glyph PNGs are used as-is (recolored via CSS only).
