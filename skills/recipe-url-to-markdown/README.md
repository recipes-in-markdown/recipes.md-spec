# Recipe URL to Markdown Skill

A Claude skill for converting recipes (from URLs or local HTML files) into structured Markdown format with YAML frontmatter.

## Features

- Parses recipes intelligently from any HTML source
- Extracts metadata: title, prep time, yield, servings, source URL
- Auto-tags by protein, spice, cuisine, and diet
- Preserves ingredient groupings and numbered instructions
- Gracefully handles incomplete, malformed, or inaccessible recipes
- Batch processing support for multiple files/directories

## Installation

Save the `.skill` file and install it in your Claude Code or Cowork environment.

## Version

v1.0 - Initial release with comprehensive error handling and tag inference.
