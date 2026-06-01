# Release Notes

## Release v1.0

### Summary
This release delivers the APHA Animal Health & Welfare Outbreak Dashboard portfolio project as a complete end-to-end analytics solution.

### Included artifacts
- `README.md` — project overview, data model, dashboard preview, and quick-start guidance.
- `documentation/data_model.md` — star schema, fact/dimension definitions, and DAX measure descriptions.
- `documentation/release_notes.md` — release summary and features.
- `documentation/github_pages_summary.md` — GitHub Pages guidance for publishing documentation or dashboards.
- `scripts/Data_Generator.ipynb` — reproducible Python notebook for synthetic outbreak data generation and validation.
- `data/animal_health_outbreaks.csv` — generated outbreak fact table.
- `data/farm_reference.csv` — generated farm reference dimension table.
- `dashboard/APHA Animal Health & Welfare Dashboard.pbix` — Power BI desktop report.
- `dashboard/APHA Animal Health & Welfare Dashboard.pdf` — exported dashboard snapshot.

### What’s included
- 10,000 simulated outbreak records across a 36-month period (May 2023 – June 2026)
- APHA-aligned region mapping and surveillance categories
- Validated fact table generation and referential farm data
- Power BI dashboard with executive overview, disease analysis, and operational response pages
- Documentation to support reuse, extension, and presentation

### Recommended next steps
1. Close Power BI Desktop if the `.pbix` file is in use.
2. Open `scripts/Data_Generator.ipynb` to regenerate or extend the synthetic dataset.
3. Publish this repository on GitHub and add a `Release` using the release notes below.
4. If desired, enable GitHub Pages for documentation hosting using the GitHub Pages summary guidance.

---

## Version history
- **v1.0** — Initial portfolio release with data generation, validation, dashboard assets, documentation, and GitHub-ready structure.
