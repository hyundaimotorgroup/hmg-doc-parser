# HMG-Doc Parser

![Status](https://img.shields.io/badge/Status-Security_Review_in_Progress-yellow?style=flat-square)
![Release](https://img.shields.io/badge/Public_Release-November_2026-blue?style=flat-square)

> This project has not yet been released as open source. Final security review
> is in progress, with public release planned for the end of November 2026.
> Source code and documentation will be made available at release.

A PDF parser focused on Korean business documents. It extracts text, tables,
and images into Markdown for chatbot knowledge bases.

The project focuses on Korean document parsing and Markdown output. Broader
language coverage, general-purpose conversion, and exact visual reproduction
are outside its current scope.

## Processing approach

Each extraction method has limitations: OCR struggles to recover table
structure, VLMs can miss details in images with dense text or graphics, and
rule-based extraction from PDF internals is limited by variation in document
structure and layout. The pipelines combine these methods to address their
respective limitations.

- Use native PDF text extraction where available, OCR for scanned text, and
  VLMs for table and image interpretation. The combination depends on the profile.
- Router profiles use lightweight classifiers to select processing paths and
  VLM tiers by page and element difficulty. Element analysis sends only the
  required elements to VLMs, reducing unnecessary calls and cost.
- Independent branches and pages run in parallel to reduce parsing time.
  Adjust `resource_budget.scheduler_max_workers` and per-node `max_concurrency`
  in YAML to match available CPU/GPU resources and endpoint capacity.
- Save images with generated filenames and Markdown references. Tables support
  Markdown, HTML, JSON, and TOON output.
- An optional pipeline converts Markdown to Confluence HTML and uploads pages
  and attachments.

## Performance

Scores measure fact-level information preservation using `gpt-5.4-mini` as an
LLM judge. Each score is the proportion of reference facts judged to be correctly
preserved in the parsed output, accounting for missing or contradictory
information (0–1; higher is better).

| Evaluation set | Contents | Planned availability at release | ours<br>(configuration 1) | ours<br>(configuration 2) | mineru-2.5-pro<br>(advanced xhigh) | infinity-parser-v2-pro |
| --- | --- | --- | --- | --- | --- | --- |
| dp_evalset_general.pdf | Moderate difficulty.<br>Business reports, maintenance manuals, price lists, and scanned presentation slides | Internal benchmark (not publicly available) | 0.983 | 0.968 | 0.978 | 0.988 |
| dp_evalset_manufacturing.pdf | Extremely difficult.<br>Complex manufacturing documents, including manufacturing processes | Internal benchmark (not publicly available) | 0.84 | 0.783 | 0.747 | 0.759 |
| dp_evalset_simple.pdf | Easy. ebook | Planned for public release | 0.998 | 0.998 | - | - |
| dp_evalset_design.pdf | Difficult. Pseudonymized design specifications and review documents | Planned for public release | 0.96 | 0.798 | 0.841 | 0.821 |
| dp_evalset_external.pdf | Difficult.<br>A collection of challenging samples for traditional parsers | Planned for public release | 0.882 | 0.87 | 0.748 | 0.666 |

Configuration 1: `router: on`, `layout_parser: dolphin-v2`, `light_vlm: gpt-5.4-mini`, `medium_vlm: gemini-3.5-flash`, `heavy_vlm: gemini-3.1-pro-preview`.

Configuration 2: `router: on`, `layout_parser: dolphin-v1.5`, `light_vlm: gpt-5.4-mini`, `medium_vlm: gemini-3.5-flash`, `heavy_vlm: gemini-3.1-pro-preview`.

Selected evaluation sets, ground-truth facts, and the evaluation prompt and code
are planned for public release alongside the project. Internal benchmark datasets
will not be publicly released.


## Acknowledgements

We thank the maintainers and contributors of the following open-source projects
for their work:

- [TOON](https://github.com/toon-format/toon): the format used for optional TOON table output.
- [Dolphin](https://github.com/bytedance/Dolphin): document parsing models used for layout analysis.
- [PaddleOCR](https://github.com/PADDLEPADDLE/PADDLEOCR): OCR models used for text detection and recognition, including Korean text.
