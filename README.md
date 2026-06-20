# recall-ops

Maintenance commands for a recall memory store: touch, garbage-collect, inspect stats, walk lineage, and edit entries — the housekeeping the store needs once it's holding real memories.

## Why it exists

The v0.1 recall store can record and retrieve memories, but it has no way to maintain itself. Bumping a recall count, retiring entries that have aged out, seeing what's actually in the store, tracing which memory superseded which — every one of those was hand-rolled at the shell, which is how stores drift and rot.

`recall-ops` makes that housekeeping a set of real commands with stable output and exit codes. It works directly against the existing v0.1 store — the SQLite index and the `.md` files — without changing the schema, so it sits alongside the deployed binary rather than replacing it. The commands are the ones a store needs in practice, not in theory: the set you reach for when the corpus is large enough that you can no longer eyeball it.

## Install

```sh
git clone --depth 1 https://github.com/j0yen/recall-ops.git
cd recall-ops
./install.sh
```

`install.sh` runs `cargo install --path . --locked` and drops the `recall-ops` binary in `~/.cargo/bin/`. Requires `cargo` / `rustc` 1.85+ and `git`. The commands shell out to `sqlite3`, so that must be on `$PATH` at run time. To build without installing:

```sh
cargo build --release   # → target/release/recall-ops
```

## Commands

Every command takes `--root <dir>` (default `~/.claude/recall`) and reads the index at `<root>/index/recall.sqlite`. Text output by default; `--format json` for stable, machine-readable output.

```sh
# bump recall_count and last_recalled_at for one or more ids
recall-ops touch <id> [<id>...]

# list entries past a threshold; --apply to actually delete .md + row
recall-ops gc --older-than 30d [--never-recalled] [--apply]

# totals by kind and subject, oldest entry, distinct embedders
recall-ops stats

# walk supersedes chains both ways: ancestors and descendants
recall-ops lineage <id>

# edit an entry in place
recall-ops update <id> [--body <text>] [--confidence <0..1>]
```

Durations for `gc` are `<N>d`, `<N>h`, or `<N>m`. `gc` is a dry run unless you pass `--apply`, and it refuses to run without `--older-than` so you can't sweep the store by accident.

## Quickstart

See what's there, then retire the entries that have aged out — preview first, apply second:

```sh
$ recall-ops stats
total: 142
  kind=fact: 98
  kind=decision: 44
  subject=mqo: 61
oldest_created: 2026-01-04T18:22:10Z

$ recall-ops gc --older-than 90d --never-recalled
dry-run: 3 candidates
  a1b2 (last_seen=2026-02-11T...)
  ...

$ recall-ops gc --older-than 90d --never-recalled --apply
deleted 3 memories
```

Exit codes are part of the contract: `0` success, `1` expected failure (unknown id, no candidates), `2` invocation error (bad `--root`, missing `sqlite3`, malformed args).

## Where it fits

Part of the **recall** family — the agentic-memory stack. `recall-ops` is the maintenance layer; `recall-doctor` audits store integrity, and the v0.1 recall binary handles record and retrieve. These ship standalone today; folding them into a single v0.2 recall binary is later work.

## Status

V0.1-companion, deliberately scoped to the five maintenance ops above. It reads and writes the existing store byte-compatibly via `sqlite3` shell-out rather than embedding a driver. Each acceptance criterion has a matching test under `tests/acceptance_ac<n>.rs`.

## Provenance

Scaffolded from a PRD through the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline. Originally a subdirectory of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this is a standalone snapshot for easier consumption.

## License

Licensed under either of Apache-2.0 ([LICENSE-APACHE](LICENSE-APACHE)) or MIT ([LICENSE-MIT](LICENSE-MIT)), at your option.
