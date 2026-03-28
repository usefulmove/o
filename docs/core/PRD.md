# Resume Repo PRD

## Problem

Maintain an accurate, well-formatted resume that is publicly accessible via a GitHub Pages URL and can be regenerated as a PDF on demand.

## Goals

- Resume content stays current and accurate — especially for fast-moving projects like enso.
- PDF renders with correct formatting (fonts, sizes, layout) matching the ODT source.
- Public URL remains stable and points to the latest PDF.

## Scope

**In scope:**
- `dedmonds_resume.odt` — the source of truth for resume content and formatting
- `dedmonds_resume.pdf` — generated from the ODT; the file served publicly
- ODT editing workflows (content updates, terminology changes)
- ODT → PDF generation via LibreOffice headless

**Out of scope:**
- Legacy HTML files (`confederate-symbols.html`, `congress.html`, `covid-19.html`, etc.) — no longer maintained
- GitHub Pages configuration — already set up, no changes needed

## Success Criteria

- Resume content reflects current projects, roles, and language.
- PDF generated from ODT renders identically — no font-size regressions, no layout breaks.
- Public link serves the latest PDF.
