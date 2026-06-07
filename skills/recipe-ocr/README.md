# Recipe OCR Skill

Digitize printed and handwritten recipes from scanned cookbook pages or recipe photography into structured Recipes.md format.

## Features

- **Multi-page support** — Intelligently merges recipes spanning multiple pages
- **Auto page-order detection** — Handles out-of-order scans
- **Recipes.md compliant** — Structured YAML metadata, ingredient/instruction sections, proper formatting
- **Smart extraction** — Preserves original measurements, techniques, and author notes
- **Batch processing** — Convert entire recipe collections in Claude Code
- **Metadata** — Auto-extracts title, prep/cook time, yield, servings, and generates contextual tags
- **Filename generation** — Creates clean kebab-case filenames from recipe titles

## Usage

**Chat:**
Upload scanned recipe pages or photos. Provide source citation for attribution.

**Claude Code:**
```python
recipe_ocr("path/to/recipe.jpg", source="Cookbook Title")
recipe_ocr(["page1.jpg", "page2.jpg"], source="Author Name")
recipe_ocr("recipes/", source="Family Collection")
```

## Output Format

Recipes.md Plain Text Recipe Format (v1.1.0-draft):
- YAML frontmatter with metadata
- Two-column layout (ingredients / instructions) via fenced divs
- Pandoc-ready for conversion to HTML, DOCX, PDF
- Human-readable in any text editor
- Version-control friendly

## Installation

Save the `.skill` file and install in your Claude Code or Cowork environment.

## Version

v1.0 - Initial release with multi-page recipe support and Recipes.md compliance.
