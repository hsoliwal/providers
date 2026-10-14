# Validation and operational boundaries

[Documentation map](ARCHITECTURE_AND_COVERAGE.md) · [Generators](GENERATOR_CONTRACTS.md) · [Data contracts](DATA_AND_TEMPLATE_CONTRACTS.md)

This page distinguishes source-derived commands from executed results. No repository
script, generator, hook, Docker build, Jekyll build, test suite or M3 analysis was run
while preparing these guides. Local editorial inspection does not establish runtime
correctness or a passing deployment.

## Data validator contract

[`scripts/check-data.rb`](../scripts/check-data.rb) requires Ruby's `psych` library.
With no arguments it gathers JSON/YML/YAML recursively under `_data` and root
`_config.yml`. Explicit paths are expanded relative to the caller's working directory.
It filters targets to existing regular files, then calls `Psych.parse_file` per path.
Exceptions are collected; zero failures exits 0, otherwise diagnostics are printed
and the script exits 1. A nonexistent explicit path is silently filtered out, so
**zero parsed files can still be a successful exit**.

Its optional `diagnose(path)` looks for JSON surrogate escape spellings and supplies
a repair hint. That hint does not change the file. Psych parsing establishes syntax
acceptability for this parser; it is not JSON Schema validation, data completeness,
URL checking, privacy validation, Liquid execution, or a full Jekyll build.

Source-derived commands, not executed receipts:

```sh
# Existing snapshot only; no generation
ruby scripts/check-data.rb

# Confirm this exact file exists and check the reported count
ruby scripts/check-data.rb _data/providers-federal.json
```

The README requires `_data` writers to use `dump_json`. Current section/detail
writers contain direct `json.dump(..., ensure_ascii=False)` call sites, as documented
in [generator contracts](GENERATOR_CONTRACTS.md). Preserve that distinction between
contribution guidance and actual implementation; this Markdown-only change does not
alter or certify those writers.

## Staged-content hook and CI

The [pre-commit hook](../.githooks/pre-commit) changes to the repository root and
collects staged added/copied/modified/renamed paths under `_data` or `_config.yml`.
It copies staged JSON/YAML bytes with `git show :path` into temporary files and calls
the Ruby validator on those copies. Thus it checks staged content, not unstaged edits.
The temporary directory is removed on exit. If Ruby is unavailable it warns and
exits successfully without parsing; no relevant staged files also exits successfully.

The existing per-clone setup instruction is:

```sh
git config core.hooksPath .githooks
```

This changes local Git configuration. The hook is not automatically installed by
a clone, and connector-created commits do not demonstrate that a local hook ran.

The [Validate data files workflow](../.github/workflows/validate-data.yml) has one
Ubuntu job using checkout v4 and `ruby scripts/check-data.rb`. Push filters include
`_data/**`, `_config.yml`, the Ruby script, and the workflow itself. Pull-request
filters include only `_data/**` and `_config.yml`. It also supports manual dispatch.
A Markdown-only PR normally does not match these automatic filters. No workflow
run is not a pass, and a passing data-parse run is not a site-render result.

## Regeneration review sequence

1. Record the source revision and inspect the complete intended diff
2. Confirm external input checkout paths, the rubric and publication-exclusion
   registries; a warning that a guard is inactive is a reason to stop publication
3. Run only the required producer, recognizing its nontransactional writes and
   deletion/redirect side effects
4. Inspect changed paths and content, then parse the affected data with the Ruby guard
5. Check semantic counts, capped versus complete cohorts, unknown bands, missing
   metadata, retirement targets, and source provenance
6. Render the relevant Jekyll pages in an established site environment and test
   filtering, score panels, tool iframe open/close and redirect behavior
7. Check the actual PR head and applicable workflow results before review or release

This sequence is guidance for a separately authorized regeneration. The checkout
does not provide a root Gemfile/Gemfile.lock or Python dependency lock in the
enumerated tree. The root [Dockerfile](../Dockerfile) builds an Alpine image with
`zip`; it is not a Jekyll server or a reproducible catalog environment. Do not replace
missing prerequisites with invented build commands or silently regenerate from an
incomplete sibling workspace.

## Fortune maintenance utilities

[`tag-fortune1000.py`](../scripts/tag-fortune1000.py) is an operational script with
top-level execution, not an import-safe utility module. It constructs a lookup from
an embedded rank-ordered company list, normalizes names, checks slug and the first
2,048 descriptor characters, and tries exact then two-token-prefix matches. Tiers
are rank ≤100, ≤500, otherwise Fortune 1000. Existing matched Fortune tags cause a
skip; otherwise a regex-based top-level tags block is extended or created.

Default mode prints matches only. `--write` rewrites sibling `all/<slug>/apis.yml`
files. It does not validate that the embedded roster is the current Fortune ranking,
and its text edits are not a YAML round-trip or identity-proof mechanism. Review
ambiguous matches and the roster's provenance before applying them.

[`commit-fortune-tags.sh`](../scripts/commit-fortune-tags.sh) reads slugs from its first
argument, uses an author-specific absolute workspace path, and runs `git add`,
`git commit`, and `git push` in each matching sibling repository. It sleeps one second
per attempted push and prints pushed/failed/skipped counters. It is not a documentation
check or dry run; its push-success counter does not prove every preceding commit
succeeded. Do not run it for this PR or assume it is portable to another checkout.

## Optional M3 analysis wrapper

The separate [M3 wrapper README](../.m3/atomize-patternize/README.md) explains the
analysis lane. It does not replace the site build. Its two scripts are included in
the inventory because they are independently configured tooling, not evidence that
this catalog implements the broader M3 runtime or that analysis has passed.

`bootstrap.sh` scans ordinary files, excluding named build/vendor directories and
its own rollout/target paths. It records ordinary Java/POM path, byte size and SHA-256
rows, checks repeated size/hash reads, derives source roots and writes `inventory.tsv`
plus temporary evidence under `.m3/target/atomize-patternize`. Existing inventory
requires explicit `SYNEXIA_REBIND_INVENTORY=1`; it is not a read-only command.

`run.sh` requires that inventory, rebuilds the Java/POM manifest, checks content/count/
mode agreement, and writes an execution receipt. It exits `NO_JAVA_SOURCE_ROOTS` only
after those checks if no Java is found. Otherwise it requires
`SYNEXIA_RECIPE_VERSION`, uses the default recipe
`com.synexia.m3.EveryModuleAtomPatternMastery` unless overridden, and invokes
OpenRewrite `dryRunNoFork`. The plugin defaults to `5.23.1`; Maven wrapper is preferred
over `mvn`. It iterates POMs with `-N`, or creates a temporary Java-21 POM for source
roots when no POM exists. Counts above 4,096 POMs/roots fail explicitly.

The runner uses `mapfile`, unlike the Bash-3.2-targeted pre-commit hook. Its command
set also includes `sha256sum`; do not assume the hook's portability applies to the
analysis wrapper. The observed `.m3` subtree has no bound `inventory.tsv`, so the
normal runner's inventory prerequisite is currently absent in that inspected tree.
No wrapper was executed, no recipe artifact was resolved, and no Java-source or
M3 coverage result is claimed.

## Verification status for this documentation batch

The [coverage ledger](ARCHITECTURE_AND_COVERAGE.md) records inspected boundaries and
remaining gaps. Verification is limited to reading pinned GitHub source and local
editorial checks of the Markdown delta/navigation. Repository lint, compilation,
configured tests, generators, Jekyll rendering and runtime/browser checks are
**NOT RUN** in this connector-only documentation task. Third-party endpoints and
provider facts have not been revalidated.
