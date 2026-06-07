---
name: recipe-ocr
description: Extract recipes from scanned cookbook pages or photography and convert to Recipes.md format. In chat, upload image(s) of recipe pages. In Claude Code, pass local file paths or directories to batch-process multiple recipes (each image set represents one recipe). Intelligently merges multi-page recipes, auto-detects page order, and generates structured Markdown with YAML frontmatter, two-column ingredient/instruction layout, metadata, and proper formatting. Source citation is optional but recommended for proper attribution. Use whenever you need to digitize printed or handwritten recipes into a standard, machine-readable format.
compatibility: Requires Anthropic API access (uses Claude vision for OCR)
---

# Recipe OCR Extractor

Convert scanned cookbook pages or recipe photographs into structured Recipes.md Markdown files.

## Input modes

### Chat (image uploads)
Upload one or more recipe page images:
```
Extract this recipe from the scanned pages: [image 1] [image 2]
Source: Joy of Cooking by Irma S. Rombauer
```

Images are processed as a single recipe. Provide them in page order if the recipe spans multiple pages.

### Claude Code or agentic workflows (file paths)
Provide local image paths (JPG, PNG, GIF, WebP). Images are processed together as one recipe:

```python
# Single page
recipe_ocr("recipes/beef_stew.jpg", source="Serious Eats")

# Multi-page recipe (in order)
recipe_ocr(
  ["recipes/croissants_page1.jpg", "recipes/croissants_page2.jpg"],
  source="Julia Child - Mastering the Art of French Cooking"
)

# Directory of images (all treated as one recipe)
recipe_ocr("scans/", source="Grandmother's Cookbook")
```

Returns a list of `.md` file paths, one per recipe extracted.

## Output format

Each recipe produces a `.md` file following the **Recipes.md specification** with:

- **YAML frontmatter**: title, prep-time, cook-time, yield, serves, source, author, tags
- **Optional description**: Context or notes before the recipe begins
- **::: ingredients** fenced div: Ingredient list with optional subsections
- **::: instructions** fenced div: Numbered steps with optional notes and pull quotes

## Multi-page recipe handling

When multiple images are provided, the skill:
- Reads left-to-right, top-to-bottom across all pages
- Automatically detects page breaks and merges content logically
- Ensures instruction steps flow in cooking order, not page order
- Removes duplicate information that appears on multiple pages
- Excludes page numbers and running headers/footers from scans

## Filename generation

Recipe filenames are generated in kebab-case from the recipe title:
- "Chocolate Chip Cookies" → `chocolate-chip-cookies.md`
- "Crème Brûlée" → `creme-brulee.md`
- "Dal Makhani" → `dal-makhani.md`

## Metadata and tagging

The skill extracts:
- **Title, prep/cook time, yield, servings** from recipe text
- **Source citation** — required in the output for proper attribution. Provide as: "Book Title by Author" or a URL
- **Author** — extracted from source if available
- **Tags** — auto-inferred by ingredient, cuisine, diet, and cooking method following the Recipes.md taxonomy

## Recipes.md specification

The output strictly follows the **Recipes.md Plain Text Recipe Format Specification** (v1.1.0-draft). This format:
- Is pure Markdown + YAML (CommonMark-compliant)
- Renders correctly in any Markdown viewer
- Converts to HTML, DOCX, PDF via pandoc
- Is agentic-friendly and version-control ready

## Notes

- **Images**: JPG, PNG, GIF, WebP. Quality scans (200+ DPI) produce best results.
- **Multi-page**: Images must be provided in order. The skill auto-detects page order intelligently.
- **Source citation**: Optional but recommended. Format: "Book Title by Author" or URL. Without it, the `source` and `author` fields remain unpopulated.
- **Output**: Always a `.md` file per recipe; no errors thrown on incomplete or poorly scanned recipes.
- **API key**: Uses `ANTHROPIC_API_KEY` environment variable.
- **Model**: Claude 4 (vision-capable model).

## Examples

**Chat:**
```
Extract this recipe:
[photo of handwritten recipe card]
Source: Grandmother's collection
```

**Claude Code:**
```python
# Single image
recipe_ocr("~/cookbook_scans/tiramisu.jpg", source="Mastering the Art of Italian Cooking")

# Multi-page
recipe_ocr(
  ["page1.jpg", "page2.jpg"],
  source="https://www.seriouseats.com/homemade-pasta-recipe"
)

# Batch from directory
recipe_ocr("~/cookbook_pages/", source="Joy of Cooking")
```
