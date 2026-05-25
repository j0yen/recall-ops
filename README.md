# recall-ops

> PRD-agentic-memory §A items 4/5/7/8/9 (touch/update/gc/stats/lineage) — operational maintenance commands missing from recall v0.1 that /self-review hand-rolls today.

## Install

### One-liner

```sh
curl -fsSL https://raw.githubusercontent.com/j0yen/recall-ops/main/install.sh | bash
```

### Manual

```sh
git clone --depth 1 https://github.com/j0yen/recall-ops.git
cd recall-ops
./install.sh
```

Installs the `recall-ops` binary via `cargo install --path . --locked`. Requires `cargo` / `rustc 1.85+` and `git`. Built binary lands in `~/.cargo/bin/`.

## Why

PRD-agentic-memory §A items 4/5/7/8/9 (touch/update/gc/stats/lineage) — operational maintenance commands missing from recall v0.1 that /self-review hand-rolls today. Like recall-doctor (slice 1 of v0.2), this ships standalone alongside the deployed v0.1 binary; full absorption into a v0.2 recall binary is later work. Each subcommand reads and (where applicable) writes the index + .md store using sqlite3 shell-out, preserving the existing schema byte-compatibly.

## Build

```sh
cargo build --release
```

Produces `target/release/recall-ops`. Symlink into `~/.local/bin/` if you want it on `$PATH`.

## Usage

```sh
recall-ops --help
```

## Audience

the author and /self-review consuming `recall-ops touch <id>...` to bump recall_count, `recall-ops gc --older-than 30d --dry-run|--apply` for retention, `recall-ops stats` for overview, `recall-ops lineage <id>` to walk supersedes chains, `recall-ops update <id> --body` for edits. Output: text default, JSON on --format json.

## Acceptance criteria

This project was scaffolded from a PRD via the `autobuilder` pipeline. The MUST-level acceptance criteria are:

- **AC1**: `recall-ops touch <id>` bumps `recall_count` by 1 and sets `last_recalled_at` to now (RFC3339) in `memories_meta` for the given id. Exits 0 on success, 1 if the id is unknown.
- **AC2**: `recall-ops touch <id1> <id2>...` accepts multiple ids; `--format json` returns `{touched: N, results: [{id, new_count, last_recalled_at}, ...]}`. Default text format prints one line per id.
- **AC3**: `recall-ops gc --older-than <duration> [--never-recalled]` lists candidate memory ids matching the predicate (created_at + last_recalled_at older than now-duration; if --never-recalled, also requires recall_count=0). Without --apply, pri...
- **AC4**: `recall-ops gc ... --apply` removes the .md file at memories_meta.path AND deletes the row from memories_meta for each candidate. Reports `{deleted: N, ids: [...]}` in JSON mode. Refuses to run without --older-than to prevent accidental ...
- **AC5**: `recall-ops stats [--format text|json]` reports `{total, by_subject: {<subject>: N}, by_kind: {<kind>: N}, oldest_created: <RFC3339 or null>, distinct_embedder_ids: [...]}` from memories_meta. Deterministic JSON output.
- **AC6**: `recall-ops lineage <id> [--format text|json]` walks `supersedes_json` (a JSON array of ids stored on each row) bidirectionally: ancestors (this row supersedes …) and descendants (this row is superseded by …). Returns chain entries with ...
- **AC7**: `recall-ops update <id> --confidence <0..1>` updates the row's confidence value in-place. `--body <text>` rewrites the .md body (preserving frontmatter). Exit 1 on unknown id, 0 on success.
- **AC8**: All subcommands accept `--root <dir>` (default `~/.claude/recall`). Missing/unreadable root → exit 2 with stderr diagnostic. Missing sqlite3 binary → exit 2.
- **AC9**: Exit codes: 0 success, 1 expected failure (unknown id, no candidates, dry-run with no matches), 2 invocation error. Stable across runs on identical inputs.

Each AC has a matching integration test under `tests/acceptance_ac<n>.rs`.

## Provenance

Built via the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline (PRD intake -> intent-card -> scaffold -> iterate-and-prove). Originally consolidated as a subdir of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this standalone repo is a fresh-init snapshot for easier consumption and distribution.

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.
