# CLAUDE.md

This file provides guidance to AI assistants (including Claude) working in this repository.

## Repository Overview

- **Repository**: vaska986/vaska986
- **Project**: Wheeler Brand Identity Generator
- **Stack**: Single-file HTML/CSS/JS application (SPA)
- **Language**: Serbian (UI, AI prompt, output); logo prompt in English
- **Author**: Ivana Dimitrijević

## What This App Does

A multi-step wizard that collects brand strategy inputs (name, archetype, audience, visual preferences via sliders) and calls the Claude API to generate a complete visual identity system based on Alina Wheeler's methodology. The result is rendered as an annotated brand mockup with designer sticker annotations — not a specification document.

## Architecture

Everything lives in `index.html` (~2200 lines, ~112KB):

```
Lines 1–6        HTML head, meta
Lines 7–660      CSS: wizard styles, loading states, responsive breakpoints
Lines 660–680    CSS: brand website layout (.brand-* classes)
Lines 680–778    CSS: mobile responsive (@media queries)
Lines 780–1165   HTML: 6-step wizard form (sections 1–6), brandContent div, author credit
Lines 1169–1192  JS: sanitizeHTML(), sanitizeFontName(), showNotification(), copyToClipboard()
Lines 1198–1320  JS: brandTranslator object (Serbian label mappings, incl. getShadow, getMaterial)
Lines 1322–1340  JS: updateSliderValue (6 sliders: tone, style, approach, design, dynamic, shadow)
Lines 1340–1430  JS: validateSection() for all 6 wizard steps
Lines 1445–1490  JS: generateBrand() — collects form data, shows loading state
Lines 1497–1690  JS: callAI() — prompt engineering, API call, JSON parsing with fallbacks
Lines 1690–1935  JS: renderBrandPage() — annotated brand mockup with stickers
Lines 1936–2160  JS: generateAndPrintPDF() — standalone HTML string with hardcoded CSS vars
Lines 2160–2200  JS: showError(), goToSection()
```

### Wizard Sections (Serbian UI)

1. **O vašem brendu** — name, category, superpower, vision
2. **Šta klijent dobija** — transformation result, post-interaction feeling
3. **Ko je vaš brend** — 12 archetypes (translated to Serbian: Dobrica, Borac, Buntovnik, Istraživač, Kreativac, Gazda, Mađioničar, Esteta, Oslonac, Znalac, Šaljivdžija, Svakidašnji)
4. **Ko su vaši ljudi** — 6 audience types (Gen Z, professionals, luxury, family, solopreneurs, employees), competitive advantage, tone slider
5. **Kakav je osećaj pod prstima** — material/texture selection (6 options)
6. **Fino podešavanje** — 6 sliders: style, approach, geometry, energy, shadow depth, tone

### Key Design Decisions

- **Single file** — must remain one file, intended to run as a Claude artifact
- **No backend** — API call goes directly from browser to `api.anthropic.com`
- **Annotated mockup** — output renders as a website simulation (nav, hero, services, CTA) with designer "sticker" annotations from `lezenda_dizajna`
- **60-30-10 color rule** — 60% white (#FFFFFF) for guide sections, sc.light only for Hero/CTA (site simulation), sc.identity only for accents/text/icons (never large block backgrounds)
- **Stickers in action color** — `lezenda_dizajna` annotations use `sc.action` background with white text to "pop" as designer notes
- **PDF = standalone HTML string** — no cloneNode; builds complete HTML with hardcoded CSS custom properties (`--primary`, `--support`, `--action`, `--light`, `--dark`), includes sticker annotations
- **Serbian throughout** — UI, AI prompt, AI output all in Serbian. Exception: `logoConceptPrompt` MUST be in English (for ChatGPT/Gemini image generation)
- **Author credit** — "Kreirala Ivana Dimitrijević" in wizard footer and brand page footer

### Claude Artifact Constraints

This file runs inside Claude's sandboxed iframe. Key restrictions:
- `window.print()` blocked → use Blob download instead
- `alert()` blocked → use DOM-based `showNotification()`
- External fetch may be blocked by CSP → `showError()` handles it
- Google Fonts may fail → `link.onerror` injects system font `@font-face` fallbacks
- File must stay under ~120KB to avoid artifact truncation

### AI JSON Response Schema

The `callAI()` prompt requests this JSON structure:
- `uniquenessStatement` — one-sentence brand positioning
- `brandEssence` — 5-7 word brand core
- `colorSystem` — identity, support, action, light, dark (HEX) + reasoning
- `usageMatrix` — where each color goes
- `typography` — headline font, body font, usage specs (Google Fonts)
- `uiSpecs` — borderRadius (CSS px), shadowStyle (CSS box-shadow), spacingUnits, description
- `guidelines` — dos[], donts[], always[]
- `logoConceptPrompt` — **in English** — 4-variation logo brief
- `lezenda_dizajna` — **in Serbian** — zasto_ovaj_naslov, psihologija_boja, uloga_dugmadi, glavna_zabrana

## Conventions

- **XSS safety**: All user inputs go through `sanitizeHTML()` before innerHTML. Font names go through `sanitizeFontName()`. Hex colors validated via regex.
- **Prompt injection**: Free-text inputs (brandName, superpower, vision) sanitized with `sanitizeInput()` — strips `{}[]`, limits to 500 chars.
- **CSS class naming**: Brand page uses `.brand-*` prefix. Wizard uses `.section`, `.card`, `.btn-*`.
- **No English in UI**: All user-facing text must be Serbian with proper characters (č, š, ž, đ). Only `logoConceptPrompt` is English.
- **Inline styles on brand page**: Brand colors, fonts, border-radius, box-shadow applied via inline `style` attributes since they're dynamic per generation. `br` (borderRadius) and `shadow` (shadowStyle) variables from AI response.
- **Conversational tone**: Wizard labels use natural, approachable Serbian — not formal/technical language.
- **White backgrounds for guide sections**: Story, Services, Typography, Values, Logo sections use `#FFFFFF`. Only Hero and CTA use `sc.light`.
- **Action color for stickers**: Designer annotation stickers use `sc.action` background, white text.

## Common Tasks

### Editing the wizard
Sections 1–6 are in the HTML between `id="section1"` and `id="section6"`. Validation logic is in `validateSection()`.

### Changing the AI prompt
The prompt template is in `callAI()`. It uses `fullContext` object which combines form data with `brandTranslator` mappings (getStyle, getApproach, getGeometry, getEnergy, getShadow, getTone, getAudience, getTransformation, getFeeling, getAdvantage, getIndustry, getMaterial). Expected JSON output schema is defined inline. Rules enforce English logo prompt and Serbian lezenda_dizajna.

### Modifying the brand page output
Edit `renderBrandPage()`. Key sections: nav, hero (with sticker), color flow, story (with sticker), services (with sticker), typography charter, CTA, values (with sticker), logo brief (with copy button), footer, sticky action bar. The `sticker()` helper renders `lezenda_dizajna` annotations. `copyToClipboard()` handles the logo prompt copy button.

### Fixing the PDF download
`generateAndPrintPDF()` builds a standalone HTML string from `window.brandSystem` data. No cloneNode — everything is template literals. CSS custom properties are hardcoded with HEX values. Stickers use `.sticker` CSS class. Backgrounds match the artifact: `#FFFFFF` for guide sections, `var(--light)` for Hero/CTA.

## Git Workflow

- **Remote**: origin (GitHub — vaska986/vaska986)
- Commit messages: concise, focused on "why"
- All text in codebase is Serbian — commit messages in English
