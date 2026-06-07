# Recipes.md Specification & Skills

This repository contains the **Recipes.md Plain Text Recipe Format Specification** and a suite of Claude skills for recipe management.

**Start here:** [Read the full Recipes.md specification →](./Recipes.md)

## Claude Code Skills

Example skills to parse online recipes and recipe photos to `Recipes.md` format:

- [recipe-url-to-markdown](./skills/recipe-url-to-markdown/) — Convert web recipes to Markdown
- [recipe-ocr](./skills/recipe-ocr/) — Digitize scanned cookbook pages

To install these skills in Claude Code, you need to move the `.skill` files into the `.claude/skills` folder, either in your home directory (global install) or your project directory. 

To install these skills in Claude Cowork:

1. Open Cowork and click Customize (left sidebar)
1. Click the "+" button to add skills
1. Select Skills tab (if not already active)
1. Choose Install from file or Browse local folder
1. Select:
   - recipe-url-to-markdown.skill
   - recipe-ocr.skill
1. Click Install
1. Skills appear in your skill menu

## License

See [LICENSE](./LICENSE)
