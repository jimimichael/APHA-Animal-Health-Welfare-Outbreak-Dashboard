# GitHub Pages Summary

## Purpose
This document explains how to publish project documentation or summary content using GitHub Pages.

## Recommended approach
1. In the GitHub repository, go to `Settings` > `Pages`.
2. Select the source branch: `main`.
3. Choose the folder: `/docs`.
4. Save the settings and wait for GitHub to publish the site.

## Suggested GitHub Pages content
To create a lightweight GitHub Pages site for this project, add content to a `docs/` folder such as:
- `docs/index.md` — project introduction and dashboard summary
- `docs/data_model.md` — star schema and data definitions
- `docs/release_notes.md` — release summary and version details

## Example `docs/index.md` structure
```markdown
# APHA Animal Health & Welfare Outbreak Dashboard

Welcome to the APHA outbreak analytics portfolio project.

## Project highlights
- Synthetic outbreak data generation and validation
- Power BI dashboard for operational reporting
- Star schema data model with fact and dimension tables
- Release notes and documentation for portfolio presentation
```

## Notes
- GitHub Pages works best for static markdown-driven documentation.
- If you want to publish a richer site, consider adding a `docs/index.md` with screenshots and links to the Power BI PDF.
- The current repository structure already supports a clean docs-based site once a `docs/` folder is added.
