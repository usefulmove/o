# Architecture

## Overview

A GitHub-hosted repo serving a public resume PDF via GitHub Pages. The resume is authored in LibreOffice Writer (`.odt`) and converted to PDF using LibreOffice headless. The PDF is the artifact served publicly.

## Repo Layout

```
/
  dedmonds_resume.odt          # Source of truth — edit this
  dedmonds_resume.pdf          # Generated artifact — served publicly
  dedmonds_resume_YYYYMMDD.odt # Dated backup of a prior version
  dedmonds_resume_YYYYMMDD.pdf # Dated backup of a prior version
  code/                        # Miscellaneous scripts
  *.html                       # Legacy data-viz files (not maintained)
  AGENTS.md                    # enso agent harness seed
  docs/                        # enso context management structure
```

## ODT → PDF Pipeline

```bash
libreoffice --headless --convert-to pdf dedmonds_resume.odt --outdir .
```

Requires LibreOffice installed. No display needed (`--headless` handles it).

## Public URL

The PDF is served via GitHub Pages from this repo. The URL is stable as long as the filename stays `dedmonds_resume.pdf` at the repo root.

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| ODT as source of truth | Full formatting control; not locked to Word |
| LibreOffice headless for PDF | No GUI required; reproducible from CLI |
| PDF at repo root | GitHub Pages serves it directly at a stable URL |
| Dated backup files | Preserve prior versions without relying on git to navigate binary diffs |

## Editing Workflow

1. Edit `dedmonds_resume.odt` — use the `docs/skills/odfpy-editing/` skill for programmatic edits
2. Regenerate PDF: `libreoffice --headless --convert-to pdf dedmonds_resume.odt --outdir .`
3. Verify PDF visually before committing
4. Commit both `.odt` and `.pdf`
