# Run the EDA notebook and manuscript

The [walkthrough](../notebooks/eda_walkthrough.ipynb) loads the committed,
synthetic [measurements CSV](../data/measurements.csv), reports incomplete cells,
and calls the EDA library for complete-case summaries and correlations. From the
EDA project directory, open the notebook with a Python kernel that has the project
dependencies. For example:

```sh
uv sync --locked --extra dev
MPLBACKEND=module://matplotlib_inline.backend_inline uv run --with jupyterlab jupyter lab notebooks/eda_walkthrough.ipynb
```

When editing the notebook or EDA library, run
`uv run python scripts/refresh_notebook_binding.py` to update the source binding
and review the diff with the corresponding behavioral changes.

## Register the checkout and run the tests

Install Python 3.15, uv, Pandoc 3.14, matching pandoc-crossref v0.3.26b, and
XeLaTeX with the [template's TeX packages](https://github.com/docxology/template/blob/87704feae0a4490ef7dad9c931d268e8c30273c2/README.md#L565-L574).
Set `EDA_CHECKOUT` to the absolute path of a clean checkout.

```sh
git clone https://github.com/docxology/template.git eda-template-runner
cd eda-template-runner
git checkout --detach 87704feae0a4490ef7dad9c931d268e8c30273c2
uv sync --locked --python 3.15
project=projects/working/template_eda_notebook
mkdir -p projects/working
git -C "$EDA_CHECKOUT" worktree add --detach "$PWD/$project" HEAD
uv run --no-sync --with pytest --with pytest-cov pytest "$project/tests" \
  --cov="$project/src" --cov-fail-under=90
```

The tests include the notebook-to-library binding and the headless analysis.
Run the following commands from the template checkout. Move the tracked
publication snapshot aside in the disposable worktree to start with fresh
artifacts.

```sh
mv "$project/output" "$(mktemp -d)/published-output"
uv run python "$project/scripts/eda_analysis.py"
uv run python "$project/scripts/generate_measurements_data.py"
uv run python scripts/pipeline/stage_03_render.py --project working/template_eda_notebook
uv run python scripts/pipeline/stage_04_validate.py --project working/template_eda_notebook
```

The analysis reads `data/measurements.csv` and writes three real plots, a
summary CSV, and a figure registry. The separate generator writes a synthetic
sibling at `output/data/measurements_generated.csv`. The
[configuration](manuscript/config.yaml) controls publication metadata and render
formats. Inspect the combined PDF in `$project/output/pdf/`, HTML at
`$project/output/web/index.html`, and the slide PDFs under
`$project/output/slides/`. Validation and provenance live in
`$project/output/reports/`.

After generating figures, run the offline link check from the staged project:

```sh
git diff --check
lychee --offline README.md STANDALONE.md AGENTS.md 'docs/**/*.md'
```
