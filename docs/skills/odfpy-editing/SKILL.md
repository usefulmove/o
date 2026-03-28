# Skill: odfpy-editing

## When to use

Use this skill when making programmatic text edits to `dedmonds_resume.odt` using odfpy.

Run with `uv run script.py` (inline dependencies, no venv needed).

---

## Safe text replacement in an existing .odt

**Never** clear a paragraph's child nodes and use `addText()`. This strips `<text:span>` elements that carry character styles, causing font-size/font-name regressions in the rendered PDF.

**Always** operate on leaf `Text` nodes in place.

```python
# /// script
# dependencies = ["odfpy"]
# ///

from odf.opendocument import load
from odf import teletype
from odf.namespaces import TEXTNS
from odf.element import Text

doc = load("dedmonds_resume.odt")

def collect_paragraphs(node, results):
    if hasattr(node, 'qname'):
        if node.qname in [(TEXTNS, 'p'), (TEXTNS, 'h')]:
            results.append(node)
    if hasattr(node, 'childNodes'):
        for child in node.childNodes:
            collect_paragraphs(child, results)

def get_leaf_text_nodes(node, leaves):
    """Collect all Text leaf nodes in document order."""
    if isinstance(node, Text):
        leaves.append(node)
        return
    if hasattr(node, 'childNodes'):
        for child in node.childNodes:
            get_leaf_text_nodes(child, leaves)

def replace_in_paragraph(para, old_text, new_text):
    """
    Replace old_text with new_text by operating only on leaf Text nodes.
    Preserves all span/element structure — no formatting regressions.
    Returns True if replacement was made.
    """
    leaves = []
    get_leaf_text_nodes(para, leaves)
    combined = "".join(leaf.data for leaf in leaves)
    if old_text not in combined:
        return False
    new_combined = combined.replace(old_text, new_text)
    if leaves:
        leaves[0].data = new_combined
        for leaf in leaves[1:]:
            leaf.data = ""
    return True

paragraphs = []
collect_paragraphs(doc.text, paragraphs)

replacements = [
    ("old text here", "new text here"),
]

for para in paragraphs:
    full = teletype.extractText(para)
    for old_text, new_text in replacements:
        if old_text in full:
            ok = replace_in_paragraph(para, old_text, new_text)
            if ok:
                print(f"Replaced: {old_text[:60]}...")
            break

doc.save("dedmonds_resume.odt")
print("Saved.")
```

---

## Fixing a font-size regression (character spans already lost)

If a prior bad edit already stripped character spans, patch the paragraph's automatic style directly.

```python
# /// script
# dependencies = ["odfpy"]
# ///

from odf.opendocument import load

doc = load("dedmonds_resume.odt")

# Namespace URIs (used as dict keys in odfpy attributes)
FO    = "urn:oasis:names:tc:opendocument:xmlns:xsl-fo-compatible:1.0"
STYLE = "urn:oasis:names:tc:opendocument:xmlns:style:1.0"

# Names of the broken automatic paragraph styles (find via inspection below)
TARGET_STYLES = {"P8", "P13", "P21"}

# Values to match a known-good sibling paragraph (e.g. P11 in this resume)
FONT_ATTRS = {
    (FO,    'font-size'):          '8pt',
    (STYLE, 'font-name'):          'Libre Franklin Thin',
    (FO,    'language'):           'en',
    (FO,    'country'):            'US',
    (STYLE, 'font-size-asian'):    '8pt',
    (STYLE, 'font-name-complex'):  'Times New Roman1',
    (STYLE, 'font-size-complex'):  '8pt',
}

for style in doc.automaticstyles.childNodes:
    name = style.getAttribute("name") if hasattr(style, 'getAttribute') else None
    if name not in TARGET_STYLES:
        continue
    for child in style.childNodes:
        if hasattr(child, 'qname') and child.qname == (STYLE, 'text-properties'):
            for attr, val in FONT_ATTRS.items():
                child.attributes[attr] = val
            print(f"Patched {name}")
            break

doc.save("dedmonds_resume.odt")
print("Saved.")
```

### How to find the broken style names

```python
# Inspect all automatic styles and their text-properties
for style in doc.automaticstyles.childNodes:
    name = style.getAttribute("name") if hasattr(style, 'getAttribute') else None
    for child in style.childNodes:
        if hasattr(child, 'qname') and child.qname == (STYLE, 'text-properties'):
            props = {k[1]: v for k, v in child.attributes.items()}
            print(f"{name}: {props}")
```

Compare output against a known-good paragraph style. Missing `font-size` / `font-name` is the culprit.

---

## Regenerate PDF after edits

```bash
libreoffice --headless --convert-to pdf dedmonds_resume.odt --outdir .
```

## Verify text content

```bash
libreoffice --headless --convert-to txt dedmonds_resume.odt --outdir /tmp/
cat /tmp/dedmonds_resume.txt | grep -i "enso"
```
