# Recipe URL to Markdown Skill

A Claude skill for converting recipes from URLs and HTML sources into structured Recipes.md format.

## Features

- Parses recipes intelligently from any HTML source (web pages, saved files)
- Extracts metadata: title, prep time, yield, servings, source URL
- Auto-tags by protein, spice, cuisine, and diet
- Preserves ingredient groupings and numbered instructions
- Gracefully handles incomplete, malformed, or inaccessible recipes
- Batch processing support for multiple files/directories
- Recipes.md compliant output

## Installation

Save the `.skill` file and install it in your Claude Code or Cowork environment.

## Version

v1.0 - Initial release with comprehensive error handling and tag inference.
