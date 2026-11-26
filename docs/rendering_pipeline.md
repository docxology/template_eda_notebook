# Rendering pipeline

The [quickstart](quickstart.md) registers a clean checkout and runs the pinned
template. The editable paper is [`docs/manuscript/`](manuscript/README.md).
Its numbered Markdown sections, preamble, and bibliography are inputs to
Stage 03. Diagram examples in authoring guides are outside the numbered paper.

## Analysis inputs and outputs

`scripts/eda_analysis.py` calls the tested EDA library on
`data/measurements.csv`. It writes three PNGs and their
`output/figures/figure_registry.json`, then writes the summary statistics CSV.
The registry binds the referenced figures to their descriptions and generated
filenames. `scripts/generate_measurements_data.py` writes a separate synthetic
CSV under `output/data/`. Rendering uses the original fixture's results.

## Render and validate

Stage 03 uses [the manuscript config](manuscript/config.yaml) to enable PDF,
HTML, and slides. It resolves the figure references against the generated PNGs
and processes citations from [the bibliography](manuscript/references.bib).
Stage 04 checks emitted formats, figures, evidence bindings, and the output
inventory and records render provenance under `output/reports/`.

Check the combined outputs for the complete-case analysis, summary and
correlation discussion, all three figures, and the reference list. The
[output inventory](output_inventory.md) lists the files from each producer.
If the renderer reports a missing figure, regenerate analysis in the same
staged checkout. For unresolved citations or unavailable converters, review
[troubleshooting](troubleshooting.md).
