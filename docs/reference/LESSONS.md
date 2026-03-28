# Lessons

Hard-won lessons from working in this repo. Update in place — git tracks history.

---

## odfpy: never clear paragraph child nodes when editing text

**Date:** 2026-03-27

Clearing a paragraph's child nodes with `para.removeChild()` and replacing with `para.addText(new_text)` strips all `<text:span>` elements that carry character styles — font-size, font-name, color, etc. The paragraph style is preserved but the text renders at the wrong size/font because the character-level styling is gone.

**The correct approach:** Walk the paragraph's child nodes recursively to collect all leaf `Text` nodes. Concatenate their `.data` values, perform the string replacement on the combined string, write the result back into the first leaf node's `.data`, and zero out the rest. This preserves all span/element structure.

See `docs/skills/odfpy-editing/SKILL.md` for working code.

---

## odfpy: fixing a font-size regression after character spans are lost

**Date:** 2026-03-27

If character-style spans are already gone (e.g. from a prior bad edit), the recovery path is to patch the paragraph's automatic style directly. Find the `<style:text-properties>` child of the target auto-style in `doc.automaticstyles`, then set `font-size`, `font-name`, and related attributes to match a known-good sibling paragraph's style.

Diagnostic steps:
1. `libreoffice --headless --convert-to txt` to verify text content is correct
2. Inspect `doc.automaticstyles` to find the broken style name (e.g. `P13`)
3. Compare its `<text:properties>` against a visually correct neighbor (e.g. `P11`)
4. Patch the missing attributes directly on the element

See `docs/skills/odfpy-editing/SKILL.md` for working code.
