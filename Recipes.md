---
title: Recipes.md — Plain Text Recipe Format Specification
version: 1.1.0-draft
date: 2026-06-06
status: Draft
tags: [recipes, cooking, markdown, plaintext, pandoc ]
---

# Recipes.md — Plain Text Recipe Format Specification
**Reference implementation:** [pandoc](https://pandoc.org/)

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Philosophy](#2-design-philosophy)
3. [TL;DR — A Complete Example](#3-tldr--a-complete-example)
4. [CommonMark Compliance and Pandoc Extensions](#4-commonmark-compliance-and-pandoc-extensions)
5. [File Conventions](#5-file-conventions)
6. [YAML Front Matter](#6-yaml-front-matter)
7. [Body Content Structure](#7-body-content-structure)
8. [Fenced Div Blocks](#8-fenced-div-blocks)
9. [Ingredient Format](#9-ingredient-format)
10. [Instruction Format](#10-instruction-format)
11. [Optional Elements](#11-optional-elements)
12. [Tagging Taxonomy](#12-tagging-taxonomy)
13. [Pandoc Implementation Guidance](#13-pandoc-implementation-guidance)
14. [Developer Integration](#14-developer-integration)
15. [Format Comparison](#15-format-comparison)
16. [Reference](#16-reference)

---

## 1. Introduction

**Recipes.md** is a plain-text recipe markup format that produces publication-ready output via [pandoc](https://pandoc.org/), the universal document converter. It is built on the [CommonMark specification](https://spec.commonmark.org/) and uses pandoc's `fenced_divs` extension to encode recipe-specific layout structure without departing from readable, portable plain text.

Recipes.md is **not** a replacement for [Schema.org JSON-LD](https://schema.org/Recipe) for web publishing, nor for [Cooklang](https://cooklang.org/) for tight kitchen tooling. It occupies the space between those: a human-writable, human-readable format that also compiles cleanly into designed output formats via pandoc. One `.md` file yields any format pandoc supports, with HTML, DOCX, and PDF being the most common outputs.

Pandoc is the reference implementation for converting Recipes.md files to designed output. All format features and behaviors are specified against pandoc's documented behavior. Other Markdown tools and renderers will handle Recipes.md files correctly, with one limitation: the `fenced_divs` syntax for multi-column layout degrades to sequential content blocks in renderers that do not support the extension.

> [!NOTE]
> `Recipes.md` is the result of a project at Omaha Startup Weekend held on June 5-7, 2026 at the Catalyst Coworking space in Omaha, Nebraska. 

---

## 2. Design Philosophy

### Plain Text First

A Recipes.md file must be fully readable without any rendering pipeline. The format is writable by anyone who can write a recipe in plain text — no special syntax to learn for basic use, no application required to open or edit the file. The plain text source is the canonical form; rendered output is derived from it.

### One Source, Many Outputs

Pandoc supports [over 50 output formats](https://pandoc.org/index.html). A single Recipes.md file converts to HTML, DOCX, PDF, ePub, LaTeX, RTF, and more without modification. HTML, DOCX, and PDF are the most practical outputs for recipe use. The format makes no assumptions about target format; it is the templates and reference documents that define the final look, not the source file.

Other Markdown conversion tools — GitHub rendering, Obsidian preview, VS Code Markdown preview — will render Recipes.md files with minimal degradation. The only feature lost without pandoc is the `fenced_divs` two-column layout; content and metadata remain fully readable.

### Tool Agnostic

Recipes.md files can be created, edited, and managed in any plain-text editor. vim, [Obsidian](https://obsidian.md/), [VS Code](https://code.visualstudio.com/), BBEdit, Notepad, iA Writer — all work without plugins or configuration. Plain-text files can be stored anywhere (local disk, external drive, NAS), synced to any cloud service (iCloud, Dropbox, Google Drive, OneDrive), and searched by the operating system's built-in search or by any text editor's find function. No proprietary format, no locked database, no subscription required to open your own recipes.

### Agentic Friendly

Recipes.md is designed for use with AI agents and LLMs as part of a workflow. The format is trivially parseable: YAML front matter contains all structured metadata, and the body content follows predictable structural conventions. An agent can create a new Recipes.md file from a web page, a photo of a recipe card, or a user's verbal description. An agent can extract ingredients for a shopping list, identify substitutions, or scale a recipe by reading the plain text directly. The format's strict conventions make agentic parsing reliable; its plain-text nature means no API or special tooling is required to read or write it.

### Minimal Extension of Markdown

Structure is added only where CommonMark alone cannot represent the concept. YAML front matter handles metadata. Fenced divs (optional) handle layout zones. Everything else is standard Markdown. A Recipes.md file without fenced divs is valid plain Markdown and renders correctly in any Markdown renderer.

### Liberal in What It Accepts

Tools processing Recipes.md files should be forgiving about optional fields and minor formatting variations. This spec defines normative behavior for *authors* and *template developers*. It is not a strict validation schema. A recipe with a missing `source` field or an unconventional filename is still a valid Recipes.md file.

---

## 3. TL;DR — A Complete Example

The following is a complete, valid Recipes.md file demonstrating all major features:

```markdown
---
title:      Banana Bread
prep-time:  20 minutes
cook-time:  55–65 minutes
yield:      1 loaf (9×5 inch)
serves:     8–10
source:     https://example.com/banana-bread
author:     Jane Smith
tags:       [bread, baking, vegetarian, banana, quick-bread]
---

Use bananas that are fully black — the blacker the banana, the sweeter
and more flavourful the bread. The batter comes together in one bowl
and takes less than five minutes to mix.

::: ingredients
## Ingredients

- 3 very ripe bananas (about 1 1/2 cups mashed)
- 1/3 cup unsalted butter, melted
- 3/4 cup granulated sugar
- 1 large egg, beaten
- 1 tsp vanilla extract
- 1 tsp baking soda
- Pinch of salt
- 1 1/2 cups all-purpose flour
:::

::: instructions
## Instructions

1. Preheat the oven to 350°F (175°C). Grease a 9×5-inch loaf pan.
2. Mash the bananas in a large bowl until smooth. Stir in the melted butter.
3. Mix in the sugar, egg, and vanilla extract.
4. Add the baking soda and salt. Stir to combine.
5. Fold in the flour until just incorporated. Do not overmix.
6. Pour the batter into the prepared pan. Bake for 55–65 minutes,
   until a toothpick inserted in the centre comes out clean.
7. Cool in the pan for 10 minutes, then turn out onto a wire rack.

The loaf keeps at room temperature, wrapped, for 3 days. Freezes well
for up to 3 months.

> "Oh dear Lord, these are incredibly good. So good they disappeared
> in an hour." — online reviewer
:::
```

**Pandoc conversion to HTML:**
```bash
pandoc banana-bread.md -f markdown -t html5 -s --template recipecard.html5 -o banana-bread.html
```

**Pandoc conversion to DOCX:**
```bash
pandoc banana-bread.md -f markdown -t docx --reference-doc recipe_template.docx -o banana-bread.docx
```

---

## 4. CommonMark Compliance and Pandoc Extensions

### 4.1 Base Specification

Recipes.md is based on **[CommonMark](https://spec.commonmark.org/)**. All CommonMark block and inline elements are valid in Recipes.md files. The following CommonMark elements are used extensively:

| Element | Use in Recipes.md |
|---|---|
| ATX headings (`##`, `###`) | Section labels within fenced divs |
| Unordered lists | Ingredient lists |
| Ordered lists | Instruction steps |
| Blockquotes (`> text`) | Pull quotes, anecdotes |
| Paragraphs | Descriptions, notes |
| Thematic breaks (`---`) | Delimits front matter |
| Emphasis (`*text*`) | Technique notes within instructions |

**List markers:** CommonMark permits multiple list item indicators for unordered lists (`-`, `*`, `+`) and ordered lists (`1.`, `1)`, or any numeral followed by `.` or `)`). Recipes.md files may use any CommonMark-compliant list marker. Consistency within a file is recommended but not required. See [CommonMark §5.3 (lists)](https://spec.commonmark.org/0.31.2/#lists) for the full specification.

### 4.2 Pandoc Extensions

#### Required: `yaml_metadata_block`

YAML front matter is required for all Recipes.md files and depends on pandoc's `yaml_metadata_block` extension. This extension is enabled by default for `markdown` input format and must be explicitly enabled for `commonmark` input.

#### Optional but Recommended: `fenced_divs`

The `fenced_divs` extension enables the `::: ingredients` and `::: instructions` layout containers that produce the two-column recipe card layout in HTML output. It is also used for callout blocks.

`fenced_divs` is enabled by default for `markdown` input. Without it, all recipe content can still be written using standard CommonMark headings and lists; only the two-column layout and callout styling are unavailable.

**Why it's recommended:** The structural separation of ingredients and instructions that fenced divs provide is what enables pandoc templates to apply targeted CSS classes for two-column layout. Without it, you get well-structured sequential content — perfectly readable, just not the designed recipe card format.

#### Callout Blocks

With `fenced_divs` enabled, the following callout classes are available for visual emphasis:

```markdown
::: { .note } :::::::::::::::
A note about technique or ingredient substitution.
:::

::: { .tip } ::::::::::::::::
A time-saving shortcut.
:::

::: { .warning } ::::::::::::
Something that commonly goes wrong.
:::

::: { .info } :::::::::::::::
Background information or context.
:::
```

Common callout classes: `.note`, `.tip`, `.info`, `.warning`, `.alert`, `.success`.

Recipes.md is structured enough that these callouts are rarely needed — notes and tips fit naturally as plain paragraphs within the `instructions` div. They are available for cases where stronger visual differentiation is useful, or for output formats (like a cooking newsletter or cookbook) where the full callout vocabulary makes sense.

**Important for template authors:** Callout classes require explicit CSS styling in HTML templates and equivalent paragraph styles in DOCX reference documents. Pandoc passes the class name through to the output element but does not apply any default visual treatment. See [Section 13](#13-pandoc-implementation-guidance).

#### Enabling Extensions Explicitly

For `markdown` input (default, extensions already active):
```bash
pandoc input.md --from markdown --to html5 -s -o output.html
```

For `commonmark` input (extensions must be explicitly added):
```bash
pandoc input.md \
  --from commonmark+yaml_metadata_block+fenced_divs \
  --to html5 -s -o output.html
```

---

## 5. File Conventions

### 5.1 File Format

- **Encoding:** UTF-8, no BOM
- **Extension:** `.md` (recommended) or `.markdown`
- **Line endings:** LF (Unix-style) preferred; CRLF is tolerated by pandoc

### 5.2 Filename Conventions

Choose a naming scheme based on how you want your recipe collection sorted and searched. The scheme should be consistent across your collection, not mixed. Options:

| Scheme | Example | Best for |
|---|---|---|
| Date prefix | `2024.03.15_banana-bread.md` | Chronological sorting, tracking when recipes were added |
| Source prefix | `nytimes_banana-bread.md` | Provenance-first collections |
| Protein prefix | `chicken_mole-negro.md` | Meat-first browsing |
| Cuisine prefix | `mexican_mole-negro.md` | Cuisine-first browsing |
| Plain name | `banana-bread.md` | Small collections, alphabetical sorting |

All schemes use kebab-case for the recipe name portion. The prefix is separated by `_` when used. Regardless of scheme, the filename should be specific enough to identify the recipe unambiguously without opening the file.

---

## 6. YAML Front Matter

### 6.1 Structure

The front matter is a [YAML](https://yaml.org/spec/1.2.2/) block delimited by `---` at the top of the file. It must be the first element in the document.

```yaml
---
title:      Banana Bread
prep-time:  20 minutes
cook-time:  55–65 minutes
yield:      1 loaf (9×5 inch)
serves:     8–10
source:     https://example.com/banana-bread
author:     Jane Smith
tags:       [bread, baking, vegetarian, banana, quick-bread]
---
```

### 6.2 Field Reference

All fields except `title` are optional. Omit a field entirely if it has no value — do not use empty strings or null placeholders.

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | String | Yes | Recipe name. Used as the document title in templates. |
| `prep-time` | String | No | Active preparation time before cooking begins (e.g., `"20 minutes"`). |
| `cook-time` | String | No | Oven, stovetop, or other cooking time (e.g., `"55–65 minutes"`). Separate from `prep-time` for clarity and Schema.org compatibility. |
| `yield` | String | No | Quantity produced (e.g., `"2 loaves"`, `"24 cookies"`, `"1 litre"`). Use for baking and batch recipes. |
| `serves` | String or Integer | No | Number of servings or portions. Use when `yield` is not the right unit. Include both if both apply. |
| `source` | String | No | Original URL, cookbook title, or citation. |
| `author` | String | No | Recipe author or originator. |
| `tags` | List | No | Hashtag-style tags for indexing and filtering. See [Section 12](#12-tagging-taxonomy). |

### 6.3 Time Format

`prep-time` and `cook-time` are free-form strings. Use natural language:

```yaml
prep-time:  20 minutes
cook-time:  1 hour 10 minutes
cook-time:  55–65 minutes
prep-time:  Overnight + 20 minutes active
```

Do not use ISO 8601 duration format (`PT1H20M`). That encoding belongs in Schema.org JSON-LD output, not human-readable source. If generating JSON-LD from a Recipes.md file, convert the time strings programmatically (see [Section 14.4](#144-schema-org-json-ld-generation)).

### 6.4 Custom Fields

Any valid [YAML](https://yaml.org/spec/1.2.2/) field name can be added to the front matter. Custom fields are available to pandoc templates and Lua filters. A field only appears in pandoc output if it is referenced in the output template, using `$fieldname$` or `${fieldname}` syntax:

```yaml
---
title:      Mole Negro
difficulty: advanced
season:     autumn
occasion:   special
---
```

In a custom HTML template:
```html
$if(difficulty)$<p class="difficulty">$difficulty$</p>$endif$
```

Custom field names must not be interpretable as YAML numbers or boolean values (`yes`, `true`, `15` are invalid as field names).

### 6.5 Multi-value Fields

Use either YAML block list or inline array format:

```yaml
# Block list
tags:
  - chicken
  - indian
  - garam-masala

# Inline array (equivalent)
tags: [chicken, indian, garam-masala]
```

Both are valid [YAML](https://yaml.org/spec/1.2.2/). The inline array is more compact and works well for tags; the block list is clearer for longer values.

---

## 7. Body Content Structure

### 7.1 Description

An optional description paragraph may appear between the front matter closing `---` and the first fenced div. This is the place for information the cook should read *before* starting the recipe: technique notes, timing considerations, ingredient sourcing advice, or context that affects how the recipe is approached.

```markdown
---
title: Croissants
prep-time: 30 minutes active
cook-time: 20 minutes
---

This recipe requires a laminated dough with three sets of folds and an
overnight rest. Read through the entire recipe before starting. The
butter block must be cold and pliable — if your kitchen is warm, chill
the dough for 30 minutes between each fold.

::: ingredients
```

The description is distinct from notes and tips that appear *after* the instruction steps. If the source material has no meaningful description, omit this section.

### 7.2 Heading Conventions

Within the fenced divs, headings follow this hierarchy:

| Level | Markdown | Use |
|---|---|---|
| `##` (h2) | `## Ingredients` | Section label — first element in each fenced div |
| `###` (h3) | `### For the filling` | Ingredient group or instruction stage subheading |

**Do not include an `# h1` title in the document body.** The recipe title lives in the YAML front matter field `title`. The output template is responsible for pulling `$title$` into the rendered output as an `<h1>` or equivalent. Putting the title in the body creates a duplicate when rendered through pandoc.

---

## 8. Fenced Div Blocks

### 8.1 Ingredients Div

```markdown
::: ingredients
## Ingredients

- 2 cups all-purpose flour
- 1 tsp baking soda
- 1/2 tsp salt

### Optional Topping

- 2 tbsp turbinado sugar
:::
```

**Class name:** `ingredients`

Maps to `<div class="ingredients">` or `<section class="ingredients">` in HTML output.

**Rules:**
- One `ingredients` div per document (for standard recipes)
- Must appear before the `instructions` div
- May contain `###`-level subheadings to group ingredients
- `## Ingredients` should be the first element inside the div

### 8.2 Instructions Div

```markdown
::: instructions
## Instructions

1. Preheat the oven to 375°F. Grease a 9×5-inch loaf pan.
2. Whisk the dry ingredients together in a large bowl.
3. In a separate bowl, mash the bananas.

The loaf keeps at room temperature for 3 days. Freezes well.

> "Best banana bread I've ever made." — reliable source
:::
```

**Class name:** `instructions`

**Rules:**
- One `instructions` div per document
- Must follow the `ingredients` div
- Instructions must be numbered (`1.`, `2.`, `3.`)
- Notes and tips follow the numbered list as plain paragraphs
- Pull quotes use blockquote syntax (`>`) at the end of the div

### 8.3 Nested Divs and Grouped Recipes

For recipes with discrete components (e.g., pastry dough plus filling plus glaze), use `###`-level subheadings within the divs rather than multiple ingredient/instruction div pairs:

```markdown
::: ingredients
## Ingredients

### For the dough

- 3 cups bread flour
- 2 tsp instant yeast
- 1 tsp salt

### For the filling

- 1 cup brown sugar
- 2 tbsp cinnamon
- 4 tbsp unsalted butter, softened
:::

::: instructions
## Instructions

### Make the dough

1. Combine the flour, yeast, and salt.
2. Add 1 cup warm water. Knead for 10 minutes.

### Prepare the filling

1. Mix the sugar and cinnamon. Set aside.
:::
```

The HTML float-based two-column layout assumes a single `ingredients`/`instructions` pair. Grouped content lives inside that pair via subheadings.

### 8.4 Callout Blocks

Callout divs for visual emphasis (when `fenced_divs` is enabled). The trailing colons after the attribute block are cosmetic and optional — they aid visual scanning of the source:

```markdown
::: { .note } :::::::::::::::
This dough can be refrigerated overnight after the first rise.
:::

::: { .warning } ::::::::::::
Do not substitute bread flour with all-purpose here — the gluten
structure is critical for the texture.
:::
```

Callout blocks can appear in the body description, within the `instructions` div after the numbered steps, or between the two divs. They should not appear inside the `ingredients` div.

Callout classes require CSS styling in the HTML template. See [Section 13.3](#133-callout-block-css).

---

## 9. Ingredient Format

### 9.1 Canonical Form

Each ingredient is an unordered list item in the form:

```
- [quantity] [unit] [ingredient name] [optional preparation note]
```

Examples:

```markdown
- 2 cups all-purpose flour, sifted
- 1 1/2 tsp baking powder
- 3 large eggs, room temperature
- 450g (1 lb) unsalted butter, cut into cubes
- Salt and freshly ground black pepper
- 1 tbsp lemon zest (optional)
```

**Do not use bold formatting on quantities.** Write them as plain text.

### 9.2 Quantity Formatting

- Fractions: `1/2`, `1/4`, `3/4` — use ASCII, not Unicode fraction characters
- Mixed numbers: `1 1/2` (space between whole and fractional parts)
- Ranges: `2–3 tablespoons` (en dash `–` or hyphen `-`)
- "To taste" items: `- Salt, to taste`
- Optional items: `- 1 tbsp lemon zest (optional)`

### 9.3 Unit Abbreviations

Use consistent abbreviations within a recipe:

| Unit | Abbreviation |
|---|---|
| teaspoon | tsp |
| tablespoon | tbsp |
| cup | cup |
| ounce | oz |
| pound | lb |
| gram | g |
| kilogram | kg |
| millilitre | ml |
| litre | l or litre |

Do not abbreviate where the abbreviation would be ambiguous in context.

---

## 10. Instruction Format

### 10.1 Numbered Steps

Instructions are an ordered list. Each step should be a complete, actionable sentence or short paragraph:

```markdown
1. Preheat the oven to 350°F (175°C). Line a 9×13-inch baking pan
   with parchment paper.
2. Whisk the flour, baking powder, and salt together in a medium bowl.
   Set aside.
3. In a large bowl, beat the butter and sugar until light and fluffy,
   about 3 minutes.
```

### 10.2 Step Granularity

Each step should represent one discrete action or a closely related set of actions. A practical heuristic: if you would pause between two actions to let something cook or cool, they belong in separate steps.

### 10.3 Temperature and Measurement Formatting

- Temperatures: `350°F (175°C)` — Fahrenheit first, Celsius in parentheses, or choose one consistently
- Dimensions: `9×13-inch` (multiplication symbol `×` preferred over the letter `x`)
- Times: write out — `10 minutes`, `1 hour 30 minutes`

---

## 11. Optional Elements

### 11.1 Notes and Tips

Notes following the numbered steps are plain paragraphs inside the `instructions` div. No special markup is required:

```markdown
::: instructions
## Instructions

1. Step one.
2. Step two.

Make-ahead note: the dough can be refrigerated overnight after step 2.
Bring to room temperature before continuing.
:::
```

### 11.2 Pull Quotes and Anecdotes

Blockquote syntax renders as an italic pull quote. Place at the end of the `instructions` div, after any notes:

```markdown
> "The best beef stew I've ever made. Restaurant quality in your own
> home." — family dinner verdict
```

In the HTML template, blockquotes are styled in italic muted text. In DOCX output, blockquotes map to the `Block Text` Word style.

### 11.3 Images

Standard Markdown image syntax is valid anywhere in the body:

```markdown
![Finished loaf on a wire rack](images/banana-bread.jpg)
```

Images between the front matter and the first fenced div render full-width above the two-column section in the HTML template. Images within the `instructions` div render inside the right column.

---

## 12. Tagging Taxonomy

### 12.1 Format

Tags are a list of plain strings in the YAML front matter. Either format is valid:

```yaml
# Inline array
tags: [chicken, indian, garam-masala, gluten-free]

# Block list
tags:
  - chicken
  - indian
  - garam-masala
  - gluten-free
```

Tags are plain strings — no `#` prefix in the source file. The `#` is a YAML comment character and would cause parsing errors without quoting. Tools like Obsidian display tags with a `#` prefix in their UI and use `#tagname` as a search operator, but the stored value in YAML front matter is the bare word.

### 12.2 Recommended Tag Categories

**Protein:** `chicken` `beef` `pork` `lamb` `fish` `shrimp` `seafood` `tofu` `eggs` `beans` `lentils`

**Key ingredient** (when central to the dish): `chocolate` `lemon` `garlic` `tomato` `mushroom`

**Notable spice/flavour** (when defining): `saffron` `cardamom` `garam-masala` `turmeric` `paprika` `miso` `tahini`

**Cuisine:** `mexican` `indian` `italian` `thai` `french` `japanese` `chinese` `greek` `spanish` `korean` `middle-eastern`

**Diet/restriction:** `vegetarian` `vegan` `gluten-free` `dairy-free` `nut-free` `low-carb`

**Course/category:** `breakfast` `lunch` `dinner` `dessert` `snack` `side` `salad` `soup` `bread` `baking` `drinks` `preserves`

**Method** (optional): `grilled` `roasted` `slow-cooker` `instant-pot` `no-bake` `fermented` `smoked`

**Source** (optional): `nytimes-cooking` `serious-eats` `family-recipe`

### 12.3 Tag Conventions

- Lowercase only
- Hyphens for multi-word tags: `gluten-free`, not `glutenfree`
- Do not repeat the same concept in different forms
- Maximum 10 tags per recipe

---

## 13. Pandoc Implementation Guidance

### 13.1 Conversion Commands

#### HTML Output

```bash
pandoc input.md \
  --from markdown \
  --to html5 \
  --standalone \
  --template recipecard.html5 \
  -o output.html
```

With explicit extension flags for non-default input formats:

```bash
pandoc input.md \
  --from commonmark+yaml_metadata_block+fenced_divs \
  --to html5 \
  --standalone \
  --template recipecard.html5 \
  -o output.html
```

#### DOCX Output

DOCX output uses a reference document, not a template file:

```bash
pandoc input.md \
  --from markdown \
  --to docx \
  --reference-doc recipe_template.docx \
  -o output.docx
```

#### PDF Output

PDF output via LaTeX requires a LaTeX installation. For most users, the HTML-to-PDF path (browser print or [WeasyPrint](https://weasyprint.org/)) is more practical:

```bash
# Via LaTeX
pandoc input.md \
  --from markdown \
  --to pdf \
  --pdf-engine=xelatex \
  -o output.pdf

# Via HTML intermediate (recommended)
pandoc input.md \
  --from markdown \
  --to html5 \
  --standalone \
  --template recipecard.html5 \
  -o output.html
# Then print to PDF from browser or pass to WeasyPrint
```

#### Batch Conversion

```bash
for f in recipes/*.md; do
  pandoc "$f" \
    --from markdown \
    --to html5 \
    --standalone \
    --template recipecard.html5 \
    -o "output/$(basename "$f" .md).html"
done
```

#### Pandoc Defaults File

For repeated use, create a defaults file (`recipe-html.yaml`):

```yaml
from: markdown
to: html5
standalone: true
template: recipecard.html5
```

Then:

```bash
pandoc --defaults recipe-html.yaml input.md -o output.html
```

#### Metadata Override

```bash
pandoc input.md \
  --metadata title="My Custom Title" \
  --metadata prep-time="45 minutes" \
  --to html5 --standalone \
  --template recipecard.html5 \
  -o output.html
```

---

### 13.2 HTML Template Guide

#### Template Variables

Pandoc HTML templates use `$variable$` syntax. Recipes.md-specific template variables:

| Template variable | Source field | Notes |
|---|---|---|
| `$title$` | `title` | Standard pandoc variable |
| `$for(prep-time)$...$endfor$` | `prep-time` | Loop required — custom YAML fields become list-type variables in pandoc |
| `$for(cook-time)$...$endfor$` | `cook-time` | Same pattern |
| `$for(yield)$...$endfor$` | `yield` | Same pattern |
| `$if(serves)$...$endif$` | `serves` | Conditional block |
| `$body$` | Document body | Renders all fenced divs as `<div class="...">` elements |

#### Header Block

The recipe header renders above the two-column body:

```html
$if(title)$<header>
<h1 class="title">$title$</h1>
<section class="metadata">
  $for(prep-time)$<p class="meta">
    <span class="label">Prep: </span>$prep-time$
  </p>$endfor$
  $for(cook-time)$<p class="meta">
    <span class="label">Cook: </span>$cook-time$
  </p>$endfor$
  $for(yield)$<p class="meta right">
    <span class="label">Yield: </span>$yield$
  </p>$endfor$
  $if(serves)$<p class="meta right">
    <span class="label">Serves: </span>$serves$
  </p>$endif$
</section>
</header>
$endif$
$body$
```

`prep-time` and `cook-time` use loops rather than `$if()$` conditionals because custom YAML fields become list-type variables in pandoc, even when they have a single value.

#### Two-Column CSS Layout

Pandoc converts `::: ingredients` to `<div class="ingredients">` and `::: instructions` to `<div class="instructions">`. Target these class names directly in CSS:

```css
section.ingredients,
div.ingredients {
  width: 30%;
  float: left;
  margin-right: 4%;
}

section.instructions,
div.instructions {
  width: 65%;
  float: left;
}

/* Clearfix for the parent container */
header:after {
  clear: both;
  content: '';
  display: table;
}
```

This produces a 30/65 split (5% total gutter). Adjust ratios for different page widths or paper sizes.

#### Ingredient List Styling

The reference templates style ingredient list items as open checkbox markers, targeting specifically the `.ingredients` context so the styling does not bleed into other list elements:

```css
.markdown-body .ingredients ul li {
  list-style: none;
  position: relative;
}

.markdown-body .ingredients ul li:before {
  content: '\20DE'; /* Unicode combining enclosing square — open box marker */
  position: absolute;
  left: -1.5rem;
  top: 0.125rem;
  color: #AAA;
  font-size: 1.25em;
  line-height: 0.75em;
}
```

Default bullets (`-` rendered as `<li>`) work fine and require no custom CSS. The open-box marker is an aesthetic choice in the reference templates, not a requirement.

#### Callout Block CSS

Callout divs (`{.note}`, `{.warning}`, etc.) require CSS to render visually distinct from body text:

```css
.note, .tip, .info, .warning, .alert {
  padding: 11px;
  margin-bottom: 24px;
  border-style: solid;
  border-width: 1px;
  border-radius: 4px;
}

.note   { color: #2f363d; background-color: #f6f8fa; border-color: #d5d8da; }
.tip    { color: #22662c; background-color: #e2f9e5; border-color: #bad3be; }
.info   { color: #246;    background-color: #e2eef9; border-color: #bac6d3; }
.warning { color: #4c4a42; background-color: #fff9ea; border-color: #dfd8c2; }
.alert  { color: #911;    background-color: #fcdede; border-color: #d2b2b2; }
```

#### Print Styles

```css
@media print {
  body, .markdown-body { background-color: #fff; }
  .markdown-body h1, .markdown-body h2, strong {
    font-weight: bold;
    page-break-after: avoid;
  }
  table, figure, li { page-break-inside: avoid; }
  .markdown-body img { max-width: 80%; }
}
```

#### Creating a New HTML Template

Start from pandoc's default:

```bash
pandoc -D html5 > my-recipe-template.html5
```

Then:

1. Add the recipe header block before `$body$`
2. Add CSS for `.ingredients` / `.instructions` two-column layout
3. Add CSS for ingredient list styling
4. Add CSS for callout classes if used
5. Add print media styles
6. Save with a `.html5` extension (conventional, not required)

---

### 13.3 DOCX Reference Document Guide

#### How Reference Documents Work

For DOCX output, pandoc uses a **reference document** whose *styles* are adopted and whose *content is ignored*. It functions as a style library, not a layout file. Specify with `--reference-doc`:

```bash
pandoc input.md --reference-doc recipe_template.docx -o output.docx
```

#### Creating a Reference Document

Start from pandoc's default:

```bash
pandoc --print-default-data-file reference.docx > reference.docx
```

Open in Word. Modify styles without adding content. All pandoc output elements map to named Word styles.

#### Style Mapping

| Markdown element | Word style name |
|---|---|
| `# Heading 1` | `Heading 1` |
| `## Heading 2` | `Heading 2` |
| `### Heading 3` | `Heading 3` |
| Body paragraphs | `Body Text` |
| Unordered list items | `List Paragraph` (bulleted) |
| Ordered list items | `List Paragraph` (numbered) |
| Blockquotes | `Block Text` |
| Code | `Verbatim Char` / `Code Block` |

#### Custom Recipe Metadata in DOCX

YAML front matter fields can surface in the DOCX header using Word's Document Property field codes. Insert via **Insert → Quick Parts → Field → Document Property** in Word. Pandoc populates these during conversion from YAML metadata.

For custom fields (`prep-time`, `cook-time`, `yield`, `serves`), use a Lua filter to inject them as styled paragraphs at the top of the output:

```lua
-- recipe-header.lua
function Pandoc(doc)
  local meta = doc.meta
  local rows = {}

  local function add_field(label, key)
    if meta[key] then
      table.insert(rows, pandoc.Para({
        pandoc.Strong({ pandoc.Str(label .. ": ") }),
        pandoc.Str(pandoc.utils.stringify(meta[key]))
      }))
    end
  end

  add_field("Prep", "prep-time")
  add_field("Cook", "cook-time")
  add_field("Yield", "yield")
  add_field("Serves", "serves")

  for i, row in ipairs(rows) do
    table.insert(doc.blocks, i, row)
  end
  return doc
end
```

Use with:

```bash
pandoc input.md \
  --lua-filter recipe-header.lua \
  --reference-doc recipe_template.docx \
  -o output.docx
```

#### Two-Column DOCX Layout

Native two-column DOCX layout requires Word's section-based column support, which pandoc does not generate from fenced divs. Options:

1. **Accept linear layout.** Recommended for most use cases. Ingredients appear first, instructions below. Clean and readable.
2. **Post-process with python-docx.** A script can reflow the output into a two-column table after pandoc generates it.
3. **Use a Word macro.** A VBA macro in the reference document can reformat on open.

Linear layout (option 1) is strongly recommended.

---

## 14. Developer Integration

### 14.1 Parsing Recipes.md Files

Use a pandoc-compatible Markdown parser with YAML front matter support.

**Python:**
```python
import frontmatter  # pip install python-frontmatter

with open("recipe.md") as f:
    post = frontmatter.load(f)

title = post.metadata.get("title")
tags  = post.metadata.get("tags", [])
body  = post.content
```

**JavaScript:**
```javascript
import matter from 'gray-matter';  // npm install gray-matter

const { data: metadata, content: body } = matter(fileContents);
```

### 14.2 Extracting Fenced Div Content

**Regex approach (adequate for well-formed files):**
```python
import re

def extract_section(body, class_name):
    pattern = rf':::\s+{re.escape(class_name)}\n(.*?)\n:::'
    match = re.search(pattern, body, re.DOTALL)
    return match.group(1).strip() if match else None

ingredients_md = extract_section(body, 'ingredients')
instructions_md = extract_section(body, 'instructions')
```

**For production use:** Convert to pandoc's JSON AST and traverse typed nodes:

```bash
pandoc input.md --to json | python process_ast.py
```

The JSON AST gives you structured access to all elements without regex fragility.

### 14.3 Tag Extraction

```python
tags = post.metadata.get('tags', [])
# tags = ['chicken', 'indian', 'garam-masala']
```

### 14.4 Schema.org JSON-LD Generation

To generate [Schema.org Recipe JSON-LD](https://schema.org/Recipe) from a Recipes.md file for SEO:

```python
def to_jsonld(metadata, ingredients_list, instructions_list):
    return {
        "@context": "https://schema.org",
        "@type": "Recipe",
        "name": metadata.get("title"),
        "prepTime": to_iso8601_duration(metadata.get("prep-time")),
        "cookTime": to_iso8601_duration(metadata.get("cook-time")),
        "recipeYield": metadata.get("yield") or str(metadata.get("serves", "")),
        "author": {
            "@type": "Person",
            "name": metadata.get("author")
        },
        "recipeIngredient": ingredients_list,
        "recipeInstructions": [
            {"@type": "HowToStep", "text": step}
            for step in instructions_list
        ]
    }
```

Schema.org's `prepTime` and `cookTime` expect ISO 8601 duration (`PT30M`, `PT1H10M`). A conversion step from the human-readable Recipes.md string is required.

### 14.5 Pandoc JSON AST

The most robust approach for programmatic processing:

```bash
pandoc input.md --from markdown --to json -o ast.json
```

The AST exposes Div blocks with class names, headers, and list items as typed nodes. This avoids writing a Markdown parser and handles edge cases (nested divs, continuation lines, etc.) correctly.

---

## 15. Format Comparison

### 15.1 Formats Surveyed

| Format | Type | Machine-readable | Human-writable | Designed output |
|---|---|---|---|---|
| MealMaster | Plain text (column-based) | Partial | Yes | No |
| RecipeML | XML | Yes | Poor | No |
| Schema.org JSON-LD | JSON | Yes | Poor | No |
| Ad hoc Markdown | Markdown | No | Yes | No |
| [Cooklang](https://cooklang.org/) (`.cook`) | Custom plain text | Yes | Yes | Via CLI tools |
| **Recipes.md** | Markdown + YAML | Partial | Yes | Via pandoc |

### 15.2 Cooklang

[Cooklang](https://cooklang.org/) is a markup language where ingredients, cookware, and timers are annotated directly in the instruction prose: `Place @bacon strips{1%kg} on a baking sheet and glaze with @syrup{1/2%tbsp}`. The format uses `@` for ingredients, `#` for cookware, and `~` for timers, with quantities in curly braces.

**Cooklang vs. Recipes.md:**

| Criterion | Cooklang | Recipes.md |
|---|---|---|
| Ingredient-in-step linking | Yes, by design | No |
| Shopping list generation | Native | Requires external processing |
| Recipe scaling | Supported | Not built in |
| Multi-format designed output | Via CLI tools | Via pandoc (50+ formats) |
| Layout/typography control | Limited | Strong (pandoc templates) |
| Learning curve | Low (3 symbols) | Low (standard Markdown) |
| Ecosystem | Growing | Pandoc ecosystem |
| SEO rich snippets | No | No (requires export step) |

Cooklang's decisive advantage is ingredient-in-step linking, which enables shopping list generation and scaling from a single parse pass. Recipes.md's decisive advantage is the pandoc pipeline and the classic ingredient-list / numbered-steps structure that matches how most published recipes are written and how most cooks prefer to work.

### 15.3 Schema.org JSON-LD

[Schema.org JSON-LD](https://schema.org/Recipe) is the web standard for recipe structured data and enables Google's recipe rich results in search. It is machine-first; no one writes recipes in JSON-LD by hand. Recipes.md can serve as a human-writable source format that generates JSON-LD as a build step (see [Section 14.4](#144-schema-org-json-ld-generation)).

### 15.4 Ad Hoc Markdown

The most common format for personal recipe collections is plain Markdown with no formal structure. It renders anywhere, requires no tooling, and is fully readable. Its weaknesses: no consistent metadata schema, no layout semantics, no tagging convention.

Recipes.md adds schema and layout semantics to plain Markdown while remaining fully compatible with it. A Recipes.md file without fenced divs is valid plain Markdown.

### 15.5 MealMaster

MealMaster was the dominant recipe exchange format in the BBS era (1990s), using fixed-column plain text for machine parsing over dial-up. It is obsolete for new use but a large archive exists.

### 15.6 Recipes.md: Pros and Cons

**Pros:**

- Readable without tooling — any text editor, any device, any cloud service
- Pandoc pipeline gives production-quality output in 50+ formats from one source
- CommonMark base is compatible with every Markdown renderer; fenced divs degrade gracefully
- YAML front matter provides consistent, parseable metadata
- No new syntax — write a recipe the way you normally would
- Agentic friendly — LLMs can create, parse, and transform Recipes.md files without special tooling
- Version control friendly — plain text diffs cleanly in Git

**Cons:**

- Pandoc required for designed output — the format is portable, the output pipeline is not
- No ingredient-in-step linking — shopping list generation and scaling require external tools
- DOCX two-column layout requires a Lua filter or post-processing
- No machine-readable unit parsing — scaling requires a quantity string parser
- Tag vocabulary is informal — no controlled vocabulary enforcement
- Not a web SEO standard — Schema.org JSON-LD requires a separate export step

---

## 16. Reference

### 16.1 Minimal Valid Example

```markdown
---
title: Simple Pasta
source: family recipe
---

::: ingredients
## Ingredients

- 400g spaghetti
- 2 cloves garlic
- 4 tbsp olive oil
- Salt and black pepper
:::

::: instructions
## Instructions

1. Cook spaghetti in well-salted boiling water until al dente.
2. While the pasta cooks, gently fry the garlic in olive oil until golden.
3. Drain the pasta, reserving 1/2 cup of cooking water.
4. Toss with the garlic oil, adding pasta water as needed.
:::
```

### 16.2 Pandoc Command Quick Reference

```bash
# HTML output
pandoc input.md -f markdown -t html5 -s --template recipecard.html5 -o output.html

# DOCX output
pandoc input.md -f markdown -t docx --reference-doc recipe_template.docx -o output.docx

# PDF via LaTeX
pandoc input.md -f markdown -t pdf --pdf-engine=xelatex -o output.pdf

# View default HTML5 template
pandoc -D html5

# Get default reference.docx
pandoc --print-default-data-file reference.docx > reference.docx

# Convert with metadata override
pandoc input.md --metadata title="My Recipe" -t html5 -s -o output.html

# Batch convert (bash)
for f in *.md; do
  pandoc "$f" -f markdown -t html5 -s --template recipecard.html5 \
    -o "${f%.md}.html"
done
```

### 16.3 Specification Links

- [CommonMark Specification 0.31.2](https://spec.commonmark.org/0.31.2/)
- [pandoc User's Guide](https://pandoc.org/MANUAL.html)
- [pandoc fenced_divs documentation](https://pandoc.org/MANUAL.html#extension-fenced_divs)
- [pandoc Templates](https://pandoc.org/MANUAL.html#templates)
- [YAML Specification 1.2.2](https://yaml.org/spec/1.2.2/)
- [Schema.org Recipe](https://schema.org/Recipe)
- [Cooklang Specification](https://cooklang.org/docs/spec/)
- [WeasyPrint — HTML to PDF](https://weasyprint.org/)
