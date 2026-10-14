# Provider catalog: data flow and contributor field guide

This guide explains the catalog implemented in this public fork. Start with the
[project README](../README.md) for provenance and upstream contacts. Reading the
catalog needs no API account, model runtime, or application server.

## The mental model

This is a **generated Jekyll catalog**, not a provider API gateway. The useful
distinction is between source descriptors, generated data, and presentation:

1. Provider repositories supply `apis.yml` descriptors.
2. Enriched provider Markdown supplies tags, scores, and agent-readiness metadata.
3. [build-listing.py](../scripts/build-listing.py) joins those inputs and writes
   alphabetical and rated listings, plus page stubs.
4. Jekyll reads the generated `_data` files and renders Liquid includes/layouts.

The generator writes the alphabetical output to `_data/providers.json` as
a map of letters to arrays. Rated outputs such as
`_data/providers-federal.json` have a `{total, bands}` shape. Treat these as
different contracts, not interchangeable lists.

## Repository map

| Surface | What to inspect or change |
| --- | --- |
| [scripts/build-listing.py](../scripts/build-listing.py) | Input paths, eligibility, joins, grouping, JSON output, generated page stubs |
| [scripts/lib_bands.py](../scripts/lib_bands.py) | Rubric-backed band IDs, labels, ranges, and local explanatory prose |
| [_data](../_data) | Generated catalogs and presentation configuration; establish each file's producer before editing |
| [_includes/home.html](../_includes/home.html) | Home cards read `provider-sections` data rather than hard-coded cards |
| [_includes](../_includes) and [_layouts](../_layouts) | Rendering and page structure |
| [_config.yml](../_config.yml) | Jekyll configuration, plugins, provider collection, exclusions |
| [scripts/check-data.rb](../scripts/check-data.rb) | Psych/YAML parsing guard for data and configuration |

The root [Dockerfile](../Dockerfile) installs `zip`; it is not a development
server or a complete Jekyll build environment.

## Before regenerating: understand the checkout layout

`build-listing.py` calculates its input root **two directories above the site
checkout**. For example, if the site lives at `work/api-evangelist/providers`,
the inputs are:

```text
work/
  all/<provider>/.git/
  all/<provider>/apis.yml
  api-search/providers/_providers/<provider>.md
  api-search/signals/_data/scoring.yml
  api-search/network/_data/delisted.yml
  api-evangelist/providers/                 # this checkout
```

This describes the paths the code expects, not files shipped by a standalone
clone. Do not regenerate a production catalog from an incomplete input tree.
In particular, a missing delisting registry prints a warning and disables that
guard. Missing enrichment or rubric inputs can produce incomplete metadata;
absence is not evidence that a provider has no APIs or has a score of zero.

Regeneration needs Python 3 plus PyYAML (`import yaml`) and the sibling input
repositories. The parsing guard needs Ruby with Psych. Install dependencies
using your normal managed environment; this repository does not supply a pinned
Python or Jekyll environment in the root files inspected for this guide.

## Command cheat sheet

Run these from this repository's root. Commands were checked against source;
they were **not executed** when preparing this documentation.

```bash
# Parse checked-in data without regenerating it.
ruby scripts/check-data.rb

# Check a specific existing data file.
ruby scripts/check-data.rb _data/providers.json

# Required per-clone hook setup described in the README; changes local Git config.
git config core.hooksPath .githooks

# Only after checking the sibling input layout and delisting registry:
python3 scripts/build-listing.py
ruby scripts/check-data.rb
git diff --stat
git diff --check
```

The generator is a write operation: it rewrites data, home/category/alphabetical
pages, and removes legacy root letter HTML files. Use a clean working branch and
review its entire diff. Passing the Ruby check establishes parseability, not
catalog completeness, accurate scoring, working links, or a successful Pages build.

## Worked source exploration: why is a provider missing?

Use a hypothetical slug `example-provider`; no real provider is implied.

1. Find `company_slugs()` in the generator. A directory must have both a `.git`
   directory and `apis.yml`, and must not be delisted. A descriptor without a
   `.git` directory is explicitly excluded and warned about.
2. Read `display_name_and_description()`. It takes top-level descriptor fields,
   with a titleized-slug fallback for the name. Alphabetical placement uses that
   display name, so a slug beginning with `e` need not appear under E.
3. Follow `read_provider_metadata()` and `main()`. A corresponding enriched
   Markdown file selects an APIs.io provider URL. Without one, the entry points
   to the upstream provider's GitHub repository instead.
4. Compare category rules. Federal, university, Apache, and CNCF membership
   depend on enrichment tags; Fortune classification also examines descriptors.
   Being alphabetically listed does not imply membership in every category.
5. Follow `band_grouped()` into `lib_bands.band_ladder()`. Unscored entries use
   the not-yet-rated group. Missing rubric ranges are left empty rather than
   invented. Check inputs before changing a threshold or hand-editing output.

For a wrong home-card count, start instead at `home.html`: `count.from` selects
`total`, `size`, or `sum`, and market cards sum their referenced totals. Match
that choice to the actual data shape before altering a template.

## Safe contribution boundaries

- Keep provider provenance, upstream attribution, and existing notices intact.
- Never reintroduce delisted providers to make a count look complete.
- All `_data` JSON writers must use the generator's `dump_json()`, which repairs
  surrogate pairs and writes real UTF-8 with `ensure_ascii=False`. Ordinary JSON
  validity alone does not establish Jekyll/Psych compatibility.
- Fix descriptors or enrichment at their source when possible. Hand edits to
  generated files can disappear at the next regeneration.
- Keep documentation-only changes separate from catalog regeneration and site
  deployment. This guide adds no API backend or M3 runtime integration.

## End-to-end example: inspect the generated federal listing

This read-only exercise starts with checked-in output, so it does not need the
sibling input repositories and cannot accidentally regenerate a partial catalog.
From the repository root, with Python 3:

```bash
python3 - <<'PY'
import json
from pathlib import Path

catalog = json.loads(Path('_data/providers-federal.json').read_text(encoding='utf-8'))
groups = catalog['bands']
print('declared total:', catalog['total'])
print('grouped total:', sum(group['count'] for group in groups))
for group in groups:
    print(group['band'], group['count'], 'rows:', len(group['providers']))
    for provider in group['providers'][:2]:
        print(' ', provider['slug'], provider['name'], provider['url'])
PY
```

This command is an unexecuted inspection example derived from the generator's
output contract. It should report groups, counts, and sample links, not a fixed
provider count. Compare each group's declared count with its array length and
the sum of grouped counts with the overall total. A mismatch is a reason to
inspect producer inputs and grouping logic, not to patch counts by hand.

Follow the same entry back through `main()`: the enriched metadata's federal
tag admits it to the category; `make_rated()` attaches available score details;
`band_grouped()` orders and groups it; the generated federal page chooses
`data_key: providers-federal` and `company-listing-rated.html`. This separates
membership, score, serialization, and rendering into independently inspectable
steps. A correctly displayed row does not establish accurate source metadata.

### Failure diagnosis that avoids misleading green checks

- `KeyError: 'bands'`: confirm that you opened a rated file. The alphabetical
  `providers.json` file has letter keys instead of the rated contract.
- Missing file: inspect the checkout/ref and actual `_data` inventory; do not run
  regeneration until its external inputs are present.
- A zero-file Ruby check: `check-data.rb` filters out nonexistent paths before
  parsing and can succeed with zero targets. Check its reported file count;
  a typo in an explicit path is not a successful validation of that file.
- Empty band ranges: inspect `lib_bands.py`'s canonical/mirror rubric candidates.
  Its deliberate empty-range fallback prevents invented thresholds.
- Parse checks pass but counts look wrong: parsing cannot detect missing source
  repositories, excluded descriptors, stale enrichment, or category-policy errors.

## Deeper contract reference

The [architecture and coverage map](ARCHITECTURE_AND_COVERAGE.md) links the
[generator contracts](GENERATOR_CONTRACTS.md), [data and template contracts](DATA_AND_TEMPLATE_CONTRACTS.md),
and [validation/operations guide](VALIDATION_AND_OPERATIONS.md). It records inspected
source revisions and unresolved coverage instead of treating a directory index as
complete package or runtime documentation.
