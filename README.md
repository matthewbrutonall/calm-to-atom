# CALM to AtoM Converter

![Catalogue records crossing systems while their hierarchy stays connected](assets/calm-to-atom.png)

If your archive has catalogue records in Axiell CALM and wants to move them to AtoM (Access to Memory) or Heratio, the export needs to be rearranged for the new system. This tool prepares that rearranged file.

It reads a **CSV export** from CALM and creates a new **CSV file** with columns for AtoM. A CSV is a table you can open in spreadsheet software. The tool works with exported files; it does not connect to your live catalogue or import records into AtoM.

For example, imagine a collection called *Test Collection* with a series called *First Series* and an item called *First Item*. Their CALM reference numbers are `GB 123 TEST`, `GB 123 TEST/1`, and `GB 123 TEST/1/PP-1`. The converter puts them into AtoM-style rows and links the item to its series and the series to its collection. That relationship is called the **archival hierarchy**. This example is included in [`examples/sample_calm.csv`](examples/sample_calm.csv).

## Who might use it?

An archivist, records manager, or migration consultant preparing CALM descriptions for an AtoM or Heratio import. You need a CALM CSV export and a way to review the resulting CSV before importing it.

## What it does

- Moves common details, including titles, dates, descriptions, creators, and access conditions, into AtoM columns. For example, CALM `Description` becomes AtoM `scopeAndContent`.
- Uses reference numbers to suggest parent links when the parent record is present in the same export. For example, `GB 123 TEST/1/PP-1` links to `GB 123 TEST/1`.
- Converts familiar CALM levels to AtoM terms. For example, `Piece` becomes `item`.
- Can also prepare a separate CALM people or organisations export as an AtoM authority-record CSV. An authority record describes a creator, such as a person or organisation, separately from the records they created.

The tool offers templates for AtoM `2.10` (the default), `2.9`, `2.8`, `2.6`, `2.3`, and `2.1`, plus `heratio`. Your archive's CALM fields may differ from the defaults; you can supply a custom mapping file.

## Before you import the CSV

Review a sample against the CALM export. Check titles, dates, creators, access conditions, reference numbers, and parent links. The hierarchy is inferred from reference numbers, so unusual numbering or missing parent records needs human review. Also check any local CALM fields that may need a custom mapping.

The optional `--audit` flag reports some issues, including a missing title **and** identifier or a parent link absent from the converted rows. It still writes the output file; it is not an import approval or a guarantee that every record is ready.

Keep the original CALM export. The converter does not change it, but **it replaces an existing file at the output path**. Choose a new output filename. It creates a CSV for review; you carry out the import separately.

## Set it up with your AI agent

Copy and paste this into an agent that can work on your computer:

```text
Install the CALM to AtoM converter from https://github.com/matthewbrutonall/calm-to-atom on my computer. Check that Python 3.8 or later is available, create a virtual environment, install the normal command-line package from the repository, and verify that `calm2atom --help` works. Tell me how to run it. Do not process any collection data yet.
```

## Run it yourself

You need Python 3.8 or later. On Linux or macOS, install the command in a virtual environment:

```bash
git clone https://github.com/matthewbrutonall/calm-to-atom.git
cd calm-to-atom
python3 -m venv .venv
. .venv/bin/activate
python -m pip install -e .
calm2atom --help
```

Try the included example first. This writes a new CSV called `my_atom_import.csv` in the repository folder:

```bash
calm2atom examples/sample_calm.csv my_atom_import.csv --atom-version 2.10
```

For your own export, replace both filenames:

```bash
calm2atom my_calm_export.csv my_atom_import.csv
```

`2.10` is the default output template. Use `--atom-version heratio` for the Heratio template, or select another supported version to match your destination. Run `calm2atom --help` for all options.

## More options

Your CALM export may contain fields with local names. Use a JSON mapping file to say which CALM field belongs in which AtoM column; [`example_mapping.json`](example_mapping.json) shows the format:

```bash
calm2atom my_calm_export.csv my_atom_import.csv --mapping example_mapping.json
```

By default, an AtoM row keeps its full CALM reference number as its identifier. `--identifier-mode leaf` uses only the last slash-separated part, so `GB 123 TEST/1/PP-1` becomes `PP-1` in the identifier column. The full reference number remains available to the tool for linking records. Check which identifier style your import needs.

For a separate CALM people or organisations export, use `--type isaar` to prepare an authority-record CSV:

```bash
calm2atom calm_people.csv atom_authorities.csv --type isaar
```

Other options include `--chunk-size` to split a large output into numbered CSV files, `--audit` to log limited checks, and `--verbose` for detailed logs. With chunking, existing numbered output files can also be replaced.

## For developers

The conversion code is in `core.py` and `authorities.py`; `cli.py` supplies the command. `web_app.py` is an optional FastAPI wrapper skeleton. To install the optional test tools and run the tests:

```bash
python -m pip install -e ".[dev]"
pytest -q
```

Issues and pull requests are welcome, especially examples of CALM field mappings from different archives.

Released under the [MIT License](LICENSE). © 2026 Archive Hosting UK.
