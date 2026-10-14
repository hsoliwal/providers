# Generator and module contracts

[Documentation map](ARCHITECTURE_AND_COVERAGE.md) · [Data and templates](DATA_AND_TEMPLATE_CONTRACTS.md) · [Validation](VALIDATION_AND_OPERATIONS.md)

This guide describes source at `6ac8607839bf768ad40cd0117e47831070fbd9fd`.
The Python files are standalone scripts/modules, not an installed Python distribution.
Their presence does not establish a successful catalog build. No generator was run for
this documentation change.

## Workspace and lifecycle

The four builders derive `SITE` from their own file location. The listing, detail,
and section builders derive `ROOT` two directories above `SITE`; changing the shell's
working directory does not redirect those inputs. See the existing
[workspace example](../CONTRIBUTING.md#regenerating-the-catalog).

| Module | Main inputs | Writes and effects |
| --- | --- | --- |
| [build-listing.py](../scripts/build-listing.py) | `all/*/apis.yml`, enriched provider Markdown, delisting registry, scoring rubric, screenshots | Alphabetical/category/band JSON, scoring mirror, home/listing HTML; deletes legacy root letter HTML |
| [build-providers.py](../scripts/build-providers.py) | Descriptors, enriched Markdown, papers, resource taxonomy, delisted/prospect/retired registries | Provider front matter, paper JSON, taxonomy mirror, removals for blocked slugs, retirement redirects |
| [build-sections.py](../scripts/build-sections.py) | Live enriched pages, descriptors, industry taxonomy, curated rosters, score files, registries, external tag normalizer | Industry/country/sector/VC JSON and pages, taxonomy/rubric mirrors; invokes tool builder |
| [build-section-tools.py](../scripts/build-section-tools.py) | Local scoring mirror, opted-in `index.html` files, their banded JSON datasets | Replaces/appends `tools:` front matter and writes priorities/opportunities HTML stubs |
| [lib_bands.py](../scripts/lib_bands.py) | Canonical scoring rubric or network mirror | Process-local cached band metadata; no file writes |
| [tag-fortune1000.py](../scripts/tag-fortune1000.py) | Embedded rank list, sibling descriptors | Dry-run report by default; `--write` modifies sibling repositories' descriptors |

Builders write directly to their destinations, without a staging transaction or a
rollback mechanism. A later failure can leave earlier outputs changed. None of these
scripts provides a lock or concurrent-build protocol. Run regeneration as a deliberate,
single-writer operation with complete inputs and review the full resulting diff.

## Alphabetical and category builder

### Eligibility and identity

[`company_slugs()`](https://github.com/hsoliwal/providers/blob/6ac8607839bf768ad40cd0117e47831070fbd9fd/scripts/build-listing.py#L153-L208)
requires an immediate child directory under `ALL`, an actual `.git` directory, and
`apis.yml`; it removes delisted slugs and returns case-insensitively sorted names.
A Git worktree whose `.git` is a file does not satisfy that directory test. Missing
`ALL` raises a filesystem error. A missing delisting registry prints a warning and
returns an empty exclusion set; that is not safe evidence that no exclusions exist.

`display_name_and_description(slug)` returns a pair of strings. The name falls back
to a titleized slug; description whitespace is collapsed. `_top_level_scalar` reads
one top-level YAML block, rejects mapping/list values, and returns `None` when absent
or unparsable. `resolve_conflicts` picks one side of conflict-marker text for reading:
it chooses the second side only when the first positively names another `aid` and
the second does not. It does not repair source files or provide a general merge engine.

`read_provider_metadata()` scans enriched `.md` files with regular expressions for
the two-space `band:` line and unindented `tags:` list. A readable Markdown file can
establish membership in its returned map even when those fields do not match. This
is a format-sensitive reader, not validation of complete front matter.

`main()` chooses the stored `url` and `type` from enrichment presence: enriched
entries point to an APIs.io provider page; others point to the upstream GitHub
repository. The alphabetical template independently constructs its own Profile,
APIs.io, and GitHub buttons, so stored `url` is not the only navigation contract.
`group_for(name)` uses the first trimmed character, accepting ASCII a–z and putting
all other initials into `0-9`. Entries are sorted by lowercase display name.

### Categories, score enrichment, and serialization

Fortune membership is a descriptor-text regex plus explicit exclusions; federal,
European-government, Apache, CNCF, university, and band membership use enriched
metadata. University membership uses the `University` tag, not the broader
`Higher Education` tag. European-government membership requires both a government
tag and a European-location tag. These are catalog selection rules, not live
verification of a provider's organization or location.

`read_score_details(slug)` caches parsed `score` and `agent_readiness` blocks,
including misses, for the life of the process. `make_rated(entry)` copies a row and
adds available composite/band/facets/scoring date/schema/regulatory and agent fields.
[`band_grouped(entries)`](https://github.com/hsoliwal/providers/blob/6ac8607839bf768ad40cd0117e47831070fbd9fd/scripts/build-listing.py#L445-L498)
then sorts descending by composite, breaks ties by lowercase name, assigns global
1-based ranks, and emits nonempty bands in rubric order. The first two emitted
groups have `open: true`. Missing band fields join `unrated`; an unrecognized
nonempty band is not automatically mapped to `unrated` and can be omitted from the
groups while remaining in `total`. Compare accounting as well as syntax.

`dump_json(obj, fh, **kwargs)` delegates to `json.dump` after `desurrogate` and forces
`ensure_ascii=False`. The caller owns the open file handle. `desurrogate` recursively
handles strings, lists, and dictionary **values**, not dictionary keys; it folds
valid UTF-16 pairs and removes surrogate code points if decoding fails. Unsupported
object types still follow JSON's usual serialization behavior/errors. This is a
targeted output repair, not general input sanitization or schema validation.

`mirror_scoring` copies the canonical signals rubric if present; otherwise it leaves
any existing mirror untouched. `write_pages` rewrites home/category/alphabetical/band
stubs and deletes existing root `a.html` through `z.html`. This is why importing a
helper or syntax-checking a file is different from running the script's main block.

## Provider detail builder

[`build-providers.py`](https://github.com/hsoliwal/providers/blob/6ac8607839bf768ad40cd0117e47831070fbd9fd/scripts/build-providers.py)
scans `all/*/apis.yml`, without the alphabetical builder's `.git` directory gate.
`--only` filters comma-separated exact slugs; `--limit` slices the sorted list.
These flags restrict the provider-writing loop, **not all side effects**: papers and
resource taxonomy are processed first, blocked pages are pruned before selection,
and retirement redirects are processed afterward.

### Reader and output API

- `read_frontmatter(path)` returns a mapping or `None`. It requires an initial
  `---\n`, tries successive closing delimiter lines, and accepts the first candidate
  that parses as a mapping. Missing/unreadable files and unsuccessful YAML candidates
  fall back rather than proving that metadata is absent
- `trim(fm)` copies nonempty values for the explicit `KEEP` list. It retains identity,
  score/access fields and artifact collections while dropping heavy `source_yaml`,
  `api_specs`, subway-map, and source-pointer fields
- `minimal_from_all(slug)` reads the descriptor when enrichment is unavailable. It
  copies identity/tags/dates, counts a list-shaped `apis`, and maps dictionary-shaped
  `accessModel` to `access_model`; unreadable/unparsable input yields a name/slug stub
- `artifact_summary(fm)` counts only list-valued collections in `ARTIFACT_KEYS` and
  returns `(phrases, total)`. `artifact_total` is a count of these artifacts, not an
  API count and not an endpoint availability measure
- `dump(slug, data, papers_len, paper_index=None)` mutates `data` with layout/navigation,
  network, artifact total, coverage and optional paper index, then writes Unicode YAML
  front matter and an empty body. `paper_index` is accepted but unused. When papers
  exist, `random_paper = sum(ord(c) for c in slug) % papers_len`; it is stable only
  for the same slug and catalogue length/order, not a permanent paper assignment

`load_papers()` scans the sibling paper directory, rejects `published: false` and
entries without a slug, resolves relative covers to the paper host, keeps the first
entry for a repeated slug, and sorts the result by slug. Missing paper inputs yield
an empty catalogue. The implementation writes `papers.json` with `json.dump` and
`ensure_ascii=False`; unlike `dump_json`, it does not call `desurrogate`.

### Removal and publication boundaries

`delisted_slugs()` reads the network registry. `prospect_slugs()` imports the external
`lib_prospects.active_prospect_slugs` helper. Missing registry/import failures warn
and yield an empty set. Confirm both guards are active before publication; this
documentation does not certify their external inputs or authorize publishing
private evaluations.

The union of these sets is excluded from the loop and existing corresponding pages
are removed. `read_coverage(slug)` only exports `none`, `gated`, or `unreadable` states
with both `reason` and `detail`; it excludes the operational `blocked` state and only
retains evidence entries carrying a URL. `dump` replaces or removes any enrichment
coverage with this checked descriptor-derived result.

[`write_tombstones()`](https://github.com/hsoliwal/providers/blob/6ac8607839bf768ad40cd0117e47831070fbd9fd/scripts/build-providers.py#L402-L478)
uses `retired.yml` mappings, never guesses a survivor, and writes a `redirect` layout
stub with `sitemap: false` only if the survivor page exists. It can create a missing
retired page as well as replace one. Missing/unreadable registry is a skip, and missing
survivors are reported. Because this pass follows pruning, inspect registry overlap
rather than assuming that all earlier deletions remain deleted. Unrelated stale
provider pages are not comprehensively removed by a general garbage-collection pass.

## Section builder

The large [`build-sections.py`](../scripts/build-sections.py) combines several
cohorts in one main pass; its filename/docstring is narrower than its implemented
outputs. `main()` also builds banking, payments, health, insurance, energy, real-estate,
travel, telecom, headless, API-tooling, marketing/revenue, logistics, secondary-market,
and VC views. The inline taxonomy/roster definitions are selection data and editorial
descriptions, not additional Python packages.

### Source membership and score precedence

`lib_tag_norm` is imported from the external network scripts directory at module load;
there is no local fallback. `catalog_slugs()` requires enriched provider Markdown,
excludes redirect layouts found in the first 400 characters, and subtracts the
conventions registry. Missing provider directory is fatal. Missing conventions and
retirement registries instead warn and return empty results. `meta_of`, inside main,
also excludes delisted slugs and requires a readable descriptor.

Industry membership unions the jobs taxonomy with canonicalized enriched-page tag
clusters; retired roster slugs resolve to survivors, and per-industry include/exclude
sets adjust membership. Taxonomy-only industries carry `roster_only` metadata.
Country membership unions exact descriptor tag aliases with explicit country-alias
HQ/based-in phrases in descriptions. It does not infer a country from a city name.
The code can match more than one country; those cards do not partition the catalogue.

`read_scores()` first uses enriched score blocks with composite and band, then fills
missing entries from the lexically newest `all/<slug>/kin/score-*.yml` file. Existing
mirror values win. `read_score_details()` caches details and fills missing score and
agent **blocks** independently; it does not merge every missing field within an
otherwise present block. The listing builder does not implement this same Kin-file
fallback, so the two outputs are not guaranteed identical from stale enrichment.

### Bounded rated output

[`band_grouped`](https://github.com/hsoliwal/providers/blob/6ac8607839bf768ad40cd0117e47831070fbd9fd/scripts/build-sections.py#L1954-L1982)
sorts and adds `rank` **in place**. Unlike the listing builder, it caps emitted rows
by global rank with `LISTING_LIMIT`, parsed as an integer environment variable at
import time and defaulting to `1000`. A band's `count` remains its full membership;
`shown` and `providers` describe rendered rows. Top-level `total`, `shown`, and `limit`
retain the distinction. `0` is a zero-row cap, not an unlimited sentinel. Invalid
integer text raises; negative values are not validated into a useful mode.

`build_roster_section_group(data_dir, meta_of, scores, sections, tier_labels, counts)`
reads each named JSON roster, filters through the supplied metadata function, attaches
tier labels, writes banded data/pages, and updates/returns the supplied counts mapping.
`listing_page` selects rated/plain includes and emits separation/tool opt-in metadata.
Selected standards/regulator tier labels are presented separately by the template;
that presentation does not change the generator's rank/cap calculation.

`build_secondary_market` reads venue membership from a sibling YAML roster and emits
ordinary rated entries with a venue-count tier badge. `build_vcs` reads a distinct
VC roster, excludes delisted firms, and sorts by descending portfolio `strength`, then
name. Its `{total, portfolio_companies, on_network, vcs}` result is not the rated-band
contract and does not assign an API Kin Score to the firm itself.

The module reassigns `SCORING_YML` to the network mirror later in the file; that is
the effective copy source in `main`, despite a generated header naming signals.
Most section JSON writers use `json.dump(..., ensure_ascii=False)` directly, rather
than the listing module's repair helper. Preserve the existing README's stronger
writer guidance, but do not assume these current call sites already enforce it.

The last main step runs `build-section-tools.py` with `sys.executable` and `check=True`.
A failing post-pass propagates after section files have already been written.

## Shared bands and market-tool post-pass

`lib_bands._load()` tries signals scoring then the network mirror, skips I/O/YAML
errors, and accepts mapping entries with IDs under `bands`. `_bands()` fills a
process-global list; an empty result is retried on later calls. `band_ids()` retains
rubric IDs present in the local prose dictionary, falling back to its six keys if
none survive. A new rubric ID alone therefore does not guarantee a new page.
`band_meta()` supplies labels/descriptions; `band_ladder()` supplies labels/ranges/
blurbs plus an unrated group. Unreadable ranges become empty, never guessed thresholds.

The [tool builder](../scripts/build-section-tools.py) walks site `index.html` files,
skipping directory names beginning `_site`, `.`, or `node_modules`. Both `data_key`
and truthy `market_tools` are required. It loads local rubric YAML and the referenced
JSON, retaining rows with non-`None` score and nonempty agent dimensions. Fewer than
`MIN_COHORT = 8` qualifying rows, a missing file, or absence of `bands` skips wiring.
Malformed JSON/rubric structure can raise rather than skip.

`credit(dim, hit, prov_credit)` returns full credit for `True`, zero for `False`,
`None`, or empty string, then tries the dimension table, the provenance table, and
finally full credit. Nonempty graded strings must therefore be understood through
the tables; truthiness is not the intended calculation. `cohort_stats` sums points
for dimensions held by nobody at positive credit and returns `(n, total, unclaimed,
number_of_checks)`. Its cohort consists of emitted qualifying rows, potentially a
capped subset, not every provider in the market.

`splice_tools` replaces/appends a `tools:` block using text/indentation matching.
Its front-matter parser splits on the first two `---` delimiters, unlike the detail
builder's candidate-YAML parser. `write` compares old text before writing and returns
whether it changed. The summary's `listings wired` counter measures changed index
files, not all visited eligible pages or changed stub files. Skipping a now-small
cohort does not delete old tool stubs or strip an old tools block.

See [validation and operations](VALIDATION_AND_OPERATIONS.md) before executing any
builder or the separate cross-repository Fortune utilities.
