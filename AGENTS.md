# AGENTS.md

## Project overview

`novel-tools` is a Python 3.10+ library and CLI for parsing, validating, transforming, and rewriting novel files. The core data flow is:

`Reader(s) -> AggregateMatcher -> Validator(s) -> Transformer(s) -> Writer(s)`

`novel_tools.toolkit.analyze()` builds that workflow from JSON configuration and executes it through `Worker`. Pydantic models validate stage options.

## Repository map

- `ntcli.py`: CLI entry point (`cli` when installed, or `python -m ntcli`).
- `novel_tools/framework/`: shared `NovelData`, `Type`, abstract stage interfaces, and `Worker` orchestration.
- `novel_tools/readers/`: inputs that yield `NovelData` objects.
- `novel_tools/processors/matchers/`: title recognition and parsing.
- `novel_tools/processors/validators/`: index consistency checks.
- `novel_tools/processors/transformers/`: type assignment, title formatting, ordering, paths, and content replacement.
- `novel_tools/writers/`: structure and document outputs.
- `novel_tools/toolkit/`: workflow construction and reference-document generation.
- `novel_tools/utils/scan_stages.py`: dynamic stage discovery and JSON-to-object construction.
- `config/`: default workflow configurations and EPUB templates.
- `tests/`: unit tests plus file-based toolkit integration tests and their golden data.
- `docs/design.md`: architectural explanation; `docs/references.md`: built-in stage option reference.

## Setup and commands

Run commands from the repository root. Stage discovery uses paths relative to the current working directory.

```sh
uv sync --locked --all-extras --dev
uv run ruff check .
uv run pytest
```

Useful narrower checks:

```sh
uv run pytest tests/readers/test_text_reader.py
uv run pytest tests/processors/matchers/test_numbered_matcher.py -q
uv run pytest -m "not slow"
uv run ruff check path/to/changed_file.py
```

The CI-required checks are `uv run ruff check .` and `uv run pytest`. Ruff is configured for a 120-character line limit, and its formatter uses single quotes.

CLI smoke checks and examples:

```sh
uv run python -m ntcli -h
uv run python -m ntcli analyze -t struct -i path/to/novel.txt -o path/to/output
uv run python -m ntcli docgen -d /tmp/references.md
```

Do not run the analyze command against tracked fixtures unless overwriting them is intentional; writers create output in the selected directory.

## Architecture and extension conventions

- `Worker.execute()` exhausts readers in order. Every object passes through every processor in order and is then offered to every writer. Writers flush once, after all readers finish.
- `NovelData` is the intermediate representation. Its primary fields are `content`, `type`, and `index`; extensible metadata belongs in `others` and should be accessed with `has`, `get`, `set`, and `pop`.
- Matchers must leave unrelated/already classified data intact. A successful matcher sets temporary `matched=True`; `AggregateMatcher` short-circuits on it and removes the flag.
- Matcher order in JSON is significant. Validators and transformers likewise run in declaration order.
- Structure writers accumulate an in-memory book/volume/chapter hierarchy and write only after all input has been accepted.
- Configuration is organized by `readers`, `matchers`, `validators`, `transformers`, and `writers`. Each item requires a `class` value and may include fields from that class's `Options` model.
- Runtime path arguments (`in_dir`, `out_dir`, and, for file input, `text_filename`) override identically named JSON values when the workflow is created.

New discoverable stages must follow all of these rules:

1. Put the module in the package associated with its stage.
2. Name it with the recognized suffix: `_reader.py`, `_matcher.py`, `_validator.py`, `_transformer.py`, or `_writer.py`.
3. Expose a PascalCase class matching the module name (for example, `FooBarReader` in `foo_bar_reader.py`).
4. Expose an `Options` Pydantic model when the stage accepts configuration. Construct it in `__init__` so invalid JSON fails early.
5. Add focused tests in the matching `tests/` subtree. If the stage is configurable, ensure its class/options remain compatible with document generation.

Do not rename the internal `__aggregate_matcher__.py` or `__structure_writer__.py` modules casually; other modules import those exact paths.

## Code conventions

- Match the existing style: single-quoted strings, 120-character maximum lines, explicit type hints where useful, and `pathlib.Path` for filesystem work.
- Use relative imports within a package where the surrounding module does, and public `novel_tools...` imports across major package boundaries.
- Use Pydantic `BaseModel` plus `Field(description=...)` for user-facing stage options. Those descriptions feed generated documentation.
- Preserve streaming behavior in readers and processors; avoid loading an entire novel unless the component inherently needs a complete structure.
- Preserve input metadata when returning replacement `NovelData` objects, generally by merging `data.others` with new fields.
- Keep filesystem encoding behavior explicit when adding an encoding option. Existing defaults are UTF-8.
- Avoid broad cleanup or modernization unrelated to the requested change; some naming and formatting patterns are established public/configuration interfaces.

## Testing guidance

- Add or update tests with every behavioral change. Prefer a focused unit test for a reader, matcher, validator, transformer, or writer.
- Toolkit workflows are integration-tested because they compose many stages and perform real I/O. Mark comparable tests with `@mark.slow`.
- Reuse `NovelData` equality in processor tests and assert the complete expected object, including metadata.
- Custom constructor arguments in several processor tests are supplied through the registered `args` marker and read by fixtures.
- Reader fixtures live under `tests/readers/data/`. Toolkit golden inputs and outputs live under `tests/toolkit/data/<Novel N>/`.
- Writer and toolkit fixtures create temporary `tests/**/output` directories and remove them after the test. Do not place persistent fixtures in those output directories.
- Golden-file comparisons are exact, including blank lines and whitespace. Toolkit CSV comparisons normalize Windows path separators, so new path-bearing assertions should remain cross-platform.
- Keep the process working directory at the repository root; tests and dynamic stage discovery rely on it.
- `EpubWriter` currently has no automated tests. Changes there need proportionate manual inspection or a narrowly scoped new test.

## Configs, docs, and dependencies

- Treat files in `config/` as user-facing examples/defaults. When changing stage options or workflow semantics, check whether these JSON files and affected integration fixtures need updates.
- `docs/references.md` documents built-in classes, and its content is derived from class docstrings and Pydantic `Field` descriptions. Keep those sources accurate when changing public behavior.
- Update both English and Chinese README material when changing user-facing installation or workflow instructions, when practical.
- Dependencies and Python constraints are declared in `pyproject.toml`; `uv.lock` is the reproducible lockfile. If dependencies change, update both and verify with the locked sync command. Do not hand-edit `uv.lock`.

## Before handing off

1. Run focused tests for the changed area.
2. Run `uv run ruff check .`.
3. Run `uv run pytest` for changes that affect shared framework/data flow, configuration discovery, or public workflows.
4. Review `git diff` for accidental fixture outputs, generated files, or unrelated edits.
