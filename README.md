# literature-review-ml

A [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) skill for conducting **systematic literature reviews tailored to machine learning and statistics research**.

Designed for survey-paper rigor — every paper passes through a documented PRISMA-adjacent flow, all citations are arXiv-ID / DOI verified, and reproducibility is explicitly assessed using a NeurIPS/JMLR-style checklist.

## What it does

Given a research topic, this skill guides Claude through 8 phases:

1. **Scoping** — Frame the question with **PMDB** (Problem / Method / Data / Baseline) instead of clinical PICO
2. **Multi-source search** — arXiv + Semantic Scholar + OpenAlex + CrossRef (all free, no API key)
3. **Screening + deduplication** — Title → abstract → full-text with PRISMA flow tracking
4. **Data extraction** — ML-specific schema capturing method, dataset, code link, compute, seeds
5. **Quality assessment** — 6-dimensional reproducibility score (0–18) per paper
6. **Thematic synthesis** — Organized by method family, not paper-by-paper; benchmark comparison table
7. **Citation verification** — Every arXiv ID and DOI is resolved against arXiv/CrossRef; mismatched metadata flagged
8. **PDF generation** — pandoc + xelatex with APA / Nature / IEEE / ACM / Vancouver styles

## Differences from generic literature-review skills

| Aspect | Generic biomed skill | **literature-review-ml** |
|---|---|---|
| Scoping frame | PICO (clinical) | **PMDB** (Problem/Method/Data/Baseline) |
| Primary search source | PubMed | **arXiv + Semantic Scholar** |
| Quality assessment | Cochrane RoB / AMSTAR 2 | **NeurIPS/JMLR reproducibility checklist** |
| Citation impact metric | Raw citation count | **`influentialCitationCount` (Semantic Scholar)** |
| Synthesis output | Thematic prose only | **Thematic prose + benchmark comparison table** |
| Citation verification | DOI resolves | **DOI + arXiv ID + metadata consistency check** |
| Dependencies | Paid APIs sometimes required | **All free, no API key required** |

## Installation

```bash
git clone https://github.com/chenlu-hung/literature-review ~/.claude/skills/literature-review-ml
```

Then verify:

```bash
ls ~/.claude/skills/literature-review-ml/SKILL.md
```

In Claude Code, the skill becomes available as `/literature-review-ml`. The `description` field in SKILL.md also lets Claude auto-invoke it when you ask for an ML/stats literature review.

## Dependencies

**Python** (standard library only, except `pyyaml` for the extraction template):
```bash
pip install pyyaml
```

**For PDF output**:
```bash
# macOS
brew install pandoc
brew install --cask mactex

# Linux
apt-get install pandoc texlive-xetex
```

Check with:
```bash
python ~/.claude/skills/literature-review-ml/scripts/generate_pdf.py --check-deps
```

## Quick usage

In Claude Code:
```
/literature-review-ml neural operators for parametric PDEs
```

Or invoke directly via natural language:
> Conduct a systematic literature review on operator learning methods for parametric PDEs, focusing on FNO and DeepONet variants since 2020.

Claude will then walk through all 8 phases, producing:
- `sources/` — JSON from each search source
- `screened/` — deduplicated candidates with screening decisions
- `extracted/` — per-paper YAML extracted using the ML-specific schema
- `review.md` + `review.pdf` — final output
- `verification_report.json` — citation verification results

A complete example workflow is included in `SKILL.md` under "Example: Operator Learning for PDEs Review".

## Standalone script usage

The Python scripts also work as standalone CLIs:

```bash
# Search arXiv
python scripts/search_arxiv.py "neural operator" \
    --categories cs.LG,stat.ML,math.NA \
    --year-start 2020 --max-results 100 \
    --output sources/arxiv.json

# Search Semantic Scholar (for citation counts)
python scripts/search_semantic_scholar.py "neural operator parametric PDE" \
    --min-citations 10 --max-results 50 \
    --output sources/s2.json

# Aggregate + deduplicate + rank
python scripts/aggregate.py sources/*.json \
    --deduplicate --rank influential_citations \
    --format markdown --output screened/candidates.md

# Verify citations in your draft
python scripts/verify_citations.py review.md \
    --output verification_report.json

# Generate PDF
python scripts/generate_pdf.py review.md \
    --citation-style nature \
    --output review.pdf
```

## Repository layout

```
literature-review-ml/
├── SKILL.md                          # main skill instructions (loaded by Claude)
├── assets/
│   ├── review_template.md            # full survey-paper skeleton
│   └── extraction_template.yaml      # per-paper extraction schema
├── references/
│   ├── ml_venues.md                  # ML / stats / SciML venue tiers
│   ├── search_strategies.md          # arXiv categories, S2 / OpenAlex tips
│   └── citation_styles.md            # APA / Nature / IEEE / ACM / Vancouver
└── scripts/
    ├── search_arxiv.py
    ├── search_semantic_scholar.py
    ├── search_openalex.py
    ├── search_crossref.py
    ├── aggregate.py
    ├── verify_citations.py
    └── generate_pdf.py
```

## Acknowledgments

This skill draws workflow inspiration from [K-Dense-AI/scientific-agent-skills](https://github.com/K-Dense-AI/scientific-agent-skills) (MIT-licensed). The PMDB frame, ML/stats venue tiers, reproducibility scoring rubric, and metadata-consistency citation verification are original to this skill.

## License

MIT — see [LICENSE](LICENSE).
