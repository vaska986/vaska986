# CLAUDE.md

This file provides guidance to AI assistants (including Claude) working in this repository.

## Repository Overview

- **Repository**: vaska986/vaska986
- **Project**: Wheeler Brand Identity Generator
- **Stack**: Single-file HTML/CSS/JS application (SPA)
- **Language**: Serbian (UI, AI prompt, output)
- **Author**: Ivana Dimitrijević

## What This App Does

A multi-step wizard that collects brand strategy inputs (name, archetype, audience, visual preferences via sliders) and calls the Claude API to generate a complete visual identity system based on Alina Wheeler's methodology. The result is rendered as an immersive brand landing page — not a specification document.

## Architecture

Everything lives in `index.html` (~1920 lines, ~86KB):

```
Lines 1–6        HTML head, meta
Lines 7–660      CSS: wizard styles, loading states, responsive breakpoints
Lines 660–680    CSS: brand website layout (.brand-* classes)
Lines 680–778    CSS: mobile responsive (@media queries)
Lines 780–1155   HTML: 6-step wizard form (sections 1–6), brandContent div, author credit
Lines 1159–1187  JS: sanitizeHTML(), sanitizeFontName(), showNotification()
Lines 1188–1310  JS: brandTranslator object (Serbian label mappings, incl. getShadow)
Lines 1478–1650  JS: callAI() with prompt engineering
Lines 1655–1817  JS: renderBrandPage() — builds immersive brand landing page
Lines 1819–1905  JS: generateAndPrintPDF() — clones page, downloads HTML
Lines 1906–1920  JS: showError(), goToSection()
```

### Wizard Sections (Serbian UI)

1. **O vašem brendu** — name, category, superpower, vision
2. **Šta klijent dobija** — transformation result, post-interaction feeling
3. **Ko je vaš brend** — 12 archetypes (translated to Serbian: Dobrica, Borac, Buntovnik, etc.)
4. **Ko su vaši ljudi** — audience, competitive advantage, tone slider
5. **Kakav je osećaj pod prstima** — material/texture selection (6 options)
6. **Fino podešavanje** — 5 sliders: style, approach, geometry, energy, shadow depth

### Key Design Decisions

- **Single file** — must remain one file, intended to run as a Claude artifact
- **No backend** — API call goes directly from browser to `api.anthropic.com`
- **Brand page = website, not spec** — output renders as a real branded landing page (nav, hero, color flow, CTA form, footer), not labeled swatches
- **PDF = Blob download** — `window.print()` blocked in artifact sandbox, so we clone rendered DOM, collect CSS, build standalone HTML, download via Blob
- **Serbian throughout** — UI, AI prompt, AI output, logo prompt all in Serbian (proper č, š, ž characters)

### Claude Artifact Constraints

This file runs inside Claude's sandboxed iframe. Key restrictions:
- `window.print()` blocked → use Blob download instead
- `alert()` blocked → use DOM-based `showNotification()`
- External fetch may be blocked by CSP → `showError()` handles it
- Google Fonts may fail → `link.onerror` injects system font `@font-face` fallbacks
- File must stay under ~100KB to avoid artifact truncation

## Conventions

- **XSS safety**: All user inputs go through `sanitizeHTML()` before innerHTML. Font names go through `sanitizeFontName()`. Hex colors validated via regex.
- **Prompt injection**: Free-text inputs (brandName, superpower, vision) sanitized with `sanitizeInput()` — strips `{}[]`, limits to 500 chars.
- **CSS class naming**: Brand page uses `.brand-*` prefix. Wizard uses `.section`, `.card`, `.btn-*`.
- **No English in UI**: All user-facing text must be Serbian with proper characters (č, š, ž, đ). Logo prompt is also Serbian (for ChatGPT/Gemini).
- **Inline styles on brand page**: Brand colors, fonts, border-radius applied via inline `style` attributes since they're dynamic per generation.
- **Conversational tone**: Wizard labels use natural, approachable Serbian — not formal/technical language.

## Common Tasks

### Editing the wizard
Sections 1–6 are in the HTML between `id="section1"` and `id="section6"`. Validation logic is in `validateSection()`.

### Changing the AI prompt
The prompt template is in `callAI()`. It uses `fullContext` object which combines form data with `brandTranslator` mappings (getStyle, getApproach, getGeometry, getEnergy, getShadow, getTone, getAudience, getTransformation, getFeeling, getAdvantage, getIndustry). Expected JSON output schema is defined inline.

### Modifying the brand page output
Edit `renderBrandPage()`. All sections (nav, hero, color flow, story, typography, CTA, values, logo, footer) are built as template literals. Inline styles use `sc.identity`, `sc.support`, `sc.action`, `sc.light`, `sc.dark` for colors.

### Fixing the PDF download
`generateAndPrintPDF()` clones `#brandContent`, strips the sticky action bar, collects `.brand-*` CSS rules from `document.styleSheets`, and wraps everything in a standalone HTML document.

## Git Workflow

- **Remote**: origin (GitHub — vaska986/vaska986)
- Commit messages: concise, focused on "why"
- All text in codebase is Serbian — commit messages in English
