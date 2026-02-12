# CLAUDE.md

This file provides guidance to AI assistants (including Claude) working in this repository.

## Repository Overview

- **Repository**: vaska986/vaska986
- **Project**: Wheeler Brand Identity Generator
- **Stack**: Single-file HTML/CSS/JS application (SPA)
- **Language**: Serbian (UI, AI prompt, output)
- **Author**: Ivana Dimitrijević ([ivanadimitrijevic.rs](https://www.ivanadimitrijevic.rs))

## What This App Does

A 6-step wizard that collects brand strategy inputs (name, archetype, audience, material, visual preferences via sliders) and calls the Claude API to generate a complete visual identity system based on Alina Wheeler's methodology (Shape → Color process). The result is rendered as an immersive brand landing page — not a specification document.

## Architecture

Everything lives in `index.html` (~1960 lines, ~88KB):

```
Lines 1–6          HTML head, meta
Lines 7–653        CSS: wizard styles, loading states, animations
Lines 654–678      CSS: brand website layout (.brand-* classes)
Lines 681–778      CSS: mobile responsive (@media 768px, 480px)
Lines 780–979      HTML: Sections 1–3 (O brendu, Transformacija, Arhetip)
Lines 980–1106     HTML: Sections 4–5 (Publika, Materijal)
Lines 1107–1176    HTML: Section 6 (Parametri dizajna — 5 sliders + generate button)
Lines 1178–1206    JS: sanitizeHTML(), sanitizeFontName(), showNotification()
Lines 1207–1330    JS: brandTranslator (Serbian label mappings, slider descriptions)
Lines 1332–1453    JS: updateSliderValue(), validateSection(), goToSection()
Lines 1455–1496    JS: generateBrand() — orchestrates flow, loading screen
Lines 1497–1692    JS: callAI() — prompt engineering, API call, JSON parsing
Lines 1693–1856    JS: renderBrandPage() — builds immersive brand page
Lines 1857–1941    JS: generateAndPrintPDF() — clones page, Blob download
Lines 1942–1961    JS: showError(), goToSection(1) init
```

### Key Design Decisions

- **Single file** — must remain one file, intended to run as a Claude artifact
- **No backend** — API call goes directly from browser to `api.anthropic.com`
- **Brand page = website, not spec** — output renders as a real branded landing page (nav, hero, color flow, story, typography, CTA form, values, logo brief, footer)
- **PDF = Blob download** — `window.print()` blocked in artifact sandbox, so we clone rendered DOM, collect `.brand-*` CSS rules, build standalone HTML, download via Blob
- **Natural Serbian throughout** — UI uses conversational tone ("Kakav je osećaj pod prstima?", "Ko su vaši ljudi?"), not formal/academic language. Logo prompt also in Serbian (for ChatGPT/Gemini).
- **Wheeler process** — AI prompt enforces Shape → Color order (geometry first, then color derived from geometry + material)

### Claude Artifact Constraints

This file runs inside Claude's sandboxed iframe. Key restrictions:
- `window.print()` blocked → Blob download of standalone HTML instead
- `alert()` blocked → DOM-based `showNotification()` toasts
- External fetch may be blocked by CSP → `showError()` handles it
- Google Fonts may fail → `link.onerror` injects system font `@font-face` fallbacks
- Blob download may fail → fallback chain: Blob → clipboard → window.open
- **File must stay under ~100KB** to avoid artifact truncation (currently ~88KB)

## Wizard Sections

| # | ID | Title | What it collects |
|---|-----|-------|-----------------|
| 1 | section1 | O vašem brendu | brandName, category, superpower, vision |
| 2 | section2 | Šta klijent dobija | transformation, feeling |
| 3 | section3 | Karakter brenda | archetype (12 Jungian archetypes) |
| 4 | section4 | Ko su vaši ljudi? | audience, advantage, toneSlider |
| 5 | section5 | Kakav je osećaj pod prstima? | material |
| 6 | section6 | Fino podešavanje | styleSlider, approachSlider, designSlider, dynamicSlider, shadowSlider |

### Archetypes (Serbian names)

Dobrica, Čovek iz naroda, Borac, Buntovnik, Istraživač, Kreativac, Gazda, Mađioničar, Esteta, Oslonac, Znalac, Šaljivdžija

## AI Prompt Structure

The `callAI()` function sends a conversational prompt in Serbian. Key fields in the expected JSON response:

- `uniquenessStatement` — one sentence positioning
- `brandEssence` — 5-7 word distillation
- `colorSystem` — 5 hex colors (identity, support, action, light, dark) + reasoning
- `kako_koristiti` — usage instructions for each color + typography hierarchy
- `typography` — Google Font names + detailed size/weight specs
- `uiSpecs` — borderRadius, shadowStyle, spacingUnits, description
- `guidelines` — dos[], donts[], always[]
- `logoConceptPrompt` — Serbian prompt for ChatGPT/Gemini logo generation

## Conventions

- **XSS safety**: All user inputs through `sanitizeHTML()` before innerHTML. Font names through `sanitizeFontName()`. Hex colors validated via regex.
- **Prompt injection**: Free-text inputs sanitized with `sanitizeInput()` — strips `{}[]`, limits to 500 chars.
- **CSS class naming**: Brand page uses `.brand-*` prefix. Wizard uses `.section`, `.card`, `.btn-*`.
- **No English in UI**: All user-facing text must be natural, conversational Serbian. Avoid formal/academic terms and anglicisms.
- **Inline styles on brand page**: Brand colors, fonts, border-radius applied via inline `style` attributes since they're dynamic per generation.
- **Copyright**: "Kreirala Ivana Dimitrijević" appears in wizard footer and brand page footer.

## Common Tasks

### Editing the wizard
Sections 1–6 are in the HTML between `id="section1"` and `id="section6"`. Validation logic is in `validateSection()`. All navigation buttons say "Dalje"/"Nazad".

### Changing the AI prompt
The prompt template is in `callAI()`. It uses `fullContext` object which combines form data with `brandTranslator` mappings. Expected JSON schema is defined inline. The prompt uses conversational tone, not formal language.

### Modifying the brand page output
Edit `renderBrandPage()`. Sections: nav, hero, color flow, story ("Priča iza boja"), typography, CTA signup, values, logo brief, footer. Inline styles use `sc.identity`, `sc.support`, `sc.action`, `sc.light`, `sc.dark`.

### Fixing the PDF download
`generateAndPrintPDF()` clones `#brandContent`, strips the sticky action bar, collects `.brand-*` CSS rules from `document.styleSheets`, and wraps in a standalone HTML document. Triple fallback: Blob → clipboard → window.open.

### Adding a new slider
1. Add HTML in Section 6 (slider-row div)
2. Add descriptions array in `updateSliderValue()`
3. Add `get*` function in `brandTranslator`
4. Collect value in `generateBrand()` (`formData.*Slider`)
5. Add to `fullContext` in `callAI()`
6. Reference in the prompt template

## Git Workflow

- **Remote**: origin (GitHub — vaska986/vaska986)
- Commit messages: concise, focused on "why", in English
- All text in codebase is Serbian
