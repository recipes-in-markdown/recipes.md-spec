---
name: recipe-url-to-markdown
description: Convert recipe URLs or HTML files to structured Markdown with YAML frontmatter. In chat, paste a recipe URL or HTTP link. In Claude Code or agentic workflows, pass local HTML file paths or directories to batch-process multiple recipes. The skill extracts title, prep time, yield, servings, source, and tags (proteins, spices, cuisine, diet). Ingredient groupings and numbered instructions are preserved. Handles incomplete or malformed inputs gracefully with error callouts in the output. Use whenever you need to convert recipes from web sources to a consistent, queryable Markdown format following the Recipes.md specification.
compatibility: Requires Anthropic API access (uses Claude for parsing)
---

# Recipe URL to Markdown Converter

Convert recipe URLs or local HTML files to structured Recipes.md Markdown format.

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
recipe_url_to_markdown("recipes/beef_stew.html")

# Multiple files
recipe_url_to_markdown(["recipes/beef_stew.html", "recipes/chicken_tikka.html"])

# Directory (recurses for .html, .htm files)
recipe_url_to_markdown("./recipes/")

# Glob pattern
recipe_url_to_markdown("recipes/**/*.html")
```

Returns a list of .md file paths.

## Output format

Each recipe produces a .md file following the **Recipes.md specification** with:

- **YAML frontmatter**: title, prep-time, cook-time, yield, serves, source, author, tags
- **Optional description**: Context or notes before the recipe begins
- **::: ingredients** fenced div: Ingredient list with optional subsections
- **::: instructions** fenced div: Numbered steps with optional notes and pull quotes

### Example structure

```markdown
---
title:      Banana Bread
prep-time:  20 minutes
cook-time:  55–65 minutes
yield:      1 loaf (9×5 inch)
serves:     8–10
source:     https://example.com/banana-bread
author:     Jane Smith
tags:       [bread, baking, banana, vegetarian, quick-bread]
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

## Recipes.md specification

The output strictly follows the **Recipes.md Plain Text Recipe Format Specification** (v1.1.0-draft). This format:
- Is pure Markdown + YAML (CommonMark-compliant)
- Renders correctly in any Markdown viewer
- Converts to HTML, DOCX, PDF via pandoc
- Is agentic-friendly and version-control ready

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
recipe_url_to_markdown("~/recipes/")

# Process specific files
recipe_url_to_markdown([
  "~/recipes/breakfast/pancakes.html",
  "~/recipes/dinner/salmon.html"
])
```

## Notes

- **URLs**: Must be HTTP/HTTPS and publicly accessible
- **Local files**: Requires file system access (Claude Code / agentic only)
- **Output**: Always a .md file per recipe; no exceptions thrown on bad input
- **API key**: Uses `ANTHROPIC_API_KEY` environment variable
- **Model**: Defaults to `claude-sonnet-4-6`; can be overridden
