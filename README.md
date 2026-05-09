# pyarchinit-api-docs

Auto-generated API reference for [PyArchInit](https://github.com/pyarchinit/pyarchinit),
the QGIS plugin for archaeological data management.

This repository hosts only the **rendered documentation** — the Python
source it documents lives in the main `pyarchinit/pyarchinit` repo.

## Live site

🌐 **https://pyarchinit-api-docs.readthedocs.io/** (once published)

## Local preview

```bash
git clone https://github.com/pyarchinit/pyarchinit-api-docs.git
cd pyarchinit-api-docs

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

mkdocs serve  # http://127.0.0.1:8000/
```

## Build a static site

```bash
mkdocs build
# Output in ./site/
```

## How the docs are regenerated

The Markdown sources under `docs/` are produced from the PyArchInit source
tree by AST walkers — see the scripts in `/tmp/` referenced from
[`docs/CHANGELOG.md`](docs/CHANGELOG.md). On a new milestone:

1. Run the regenerator over the plugin's `modules/`, `tabs/`, `gui/` trees.
2. Update `API_INDEX.md` and the per-area pages (`gui.md`, `tabs.md`,
   `database.md`, `s3dgraphy.md`, `stratigraph.md`, `storage.md`,
   `utility.md`, `misc.md`).
3. Bump the stats banner numbers in `docs/index.md`.
4. Commit + push — Read the Docs auto-builds the live site.

## Theme

[mkdocs-material](https://squidfunk.github.io/mkdocs-material/) with a
custom palette in `docs/assets/extra.css`. Light + dark modes auto-toggle
on system preference and can be overridden via the header switch.

## License

GPL v2 (matches the upstream PyArchInit plugin).
