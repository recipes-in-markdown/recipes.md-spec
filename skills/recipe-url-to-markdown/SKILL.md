---
name: recipe url-to-markdown
description: Convert recipe URLs or HTML files to structured Markdown with YAML frontmatter. In chat, paste a recipe URL or HTTP link. In Claude Code or agentic workflows, pass local HTML file paths or directories to batch-process multiple recipes. The skill extracts title, prep time, yield, servings, source, and tags (proteins, spices, cuisine, diet). Ingredient groupings and numbered instructions are preserved. Handles incomplete or malformed inputs gracefully with error callouts in the output. Use whenever you need to convert recipes to a consistent, queryable Markdown format.
compatibility: Requires Anthropic API access (uses Claude for parsing)
---

# Recipe to Markdown Converter

Convert recipe HTML to structured Markdown with extracted metadata, ingredient groups, and instructions.

## Input modes

### Chat (single URL)
Provide a recipe URL:
```
Convert this recipe: https://www.seriouseats.com/best-beef-stew-recipe
```

The skill will fetch the HTML, parse it with Claude, and return a downloadable .md file.

### Claude Code or agentic workflows (batch)
Provide file paths, glob patterns, or directories:

```python
# Single file
recipe_to_markdown("recipes/beef_stew.html")

# Multiple files
recipe_to_markdown(["recipes/beef_stew.html", "recipes/chicken_tikka.html"])

# Directory (recurses for .html, .htm files)
recipe_to_markdown("./recipes/")

# Glob pattern
recipe_to_markdown("recipes/**/*.html")
```

Returns a list of .md file paths.

## Output format

Each recipe produces a .md file with:

```markdown
---
title: Recipe Name
prep-time: 30 minutes
yield: 2 loaves
serves: 4
source: https://example.com/recipe
tags: #chicken, #garlic, #italian, #gluten-free
---

Optional brief description.

::: ingredients
## Ingredients

* 2 cups flour
* 1 tbsp salt

### Grouped Section (if in original)

* 1 tsp cumin
:::

::: instructions
## Instructions

1. First step
2. Second step

> Chef's tip if present
:::
```

Fields without values are omitted. Only include fields that have parsed content.

## Tag inference

The skill auto-tags based on key ingredients and cuisine:

- **Proteins**: #chicken, #beef, #pork, #shrimp, #fish, #seafood, #tofu, etc.
- **Notable spices**: #paprika, #cumin, #saffron, #turmeric, #garam-masala, etc.
- **Cuisine type**: #mexican, #indian, #italian, #thai, #french, etc.
- **Diet**: #vegetarian, #vegan, #gluten-free (if applicable)

You can add additional tags manually after conversion.

## Error handling

The skill includes error callouts for problematic inputs:

**Incomplete or malformed recipe:**
```markdown
> **⚠️ Parsing issue**: Missing instruction steps. Extracted what was available.
```

**Not actually a recipe:**
```markdown
> **⚠️ Not a recipe**: This HTML does not appear to contain recipe content (no ingredient list detected).
```

**Inaccessible URL (paywall/auth):**
```markdown
> **⚠️ Not accessible**: This URL appears to contain a recipe, but it is not accessible (possibly behind a paywall, requires login, or blocks automated access). Try copying the recipe content and uploading it as an HTML file instead.
```

The skill detects auth-gated sites (NYT Cooking, America's Test Kitchen, etc.) and suggests manual extraction as a workaround. No errors are thrown—always Markdown output.

## Implementation

The skill uses Claude to intelligently parse any recipe format via the Anthropic API. It:

1. Fetches the HTML (if URL) or reads the local file
2. Sends to Claude with a detailed recipe parsing system prompt
3. Cleans up response markup
4. Writes to `.md` file(s)
5. Returns file path(s)

Ingredient groupings and instruction steps are preserved if present in the original. Very long HTML is truncated to avoid token limits.

## Examples

**Chat:**
```
Convert this to Markdown: https://www.example.com/pasta-carbonara
```

**Claude Code:**
```python
# Batch convert a folder
recipe_to_markdown("~/recipes/")

# Process specific files
recipe_to_markdown([
  "~/recipes/breakfast/pancakes.html",
  "~/recipes/dinner/salmon.html"
])
```

## Notes

- URLs: Must be HTTP/HTTPS and publicly accessible
- Local files: Requires file system access (Claude Code / agentic only)
- Output: Always a .md file per recipe; no exceptions thrown on bad input
- API key: Uses `ANTHROPIC_API_KEY` environment variable
- Model: Defaults to `claude-sonnet-4-6`; can be overridden
