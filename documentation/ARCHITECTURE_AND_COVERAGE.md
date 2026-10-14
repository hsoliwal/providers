# Architecture and documentation coverage

[README](../README.md) · [Contributor field guide](CONTRIBUTOR_GUIDE.md)

This is an additive, source-backed map of the provider-catalog fork at
[`6ac8607839bf768ad40cd0117e47831070fbd9fd`](https://github.com/hsoliwal/providers/tree/6ac8607839bf768ad40cd0117e47831070fbd9fd),
inspected on 14 October 2026. It preserves the existing contributor guides,
upstream identity and notices. It does not claim a complete M3 execution run or
full semantic documentation of every provider record and vendored API.

## Start with the boundary you are changing

- [Generator contracts](GENERATOR_CONTRACTS.md): all six first-party Python
  modules/scripts, inputs, eligibility, score precedence, mutations, output shapes,
  errors/fallbacks, and important function contracts
- [Data and template contracts](DATA_AND_TEMPLATE_CONTRACTS.md): alphabetical versus
  rated versus VC data, provider front matter, counts/caps, selected Liquid consumers
  and remaining template/package gaps
- [Validation and operations](VALIDATION_AND_OPERATIONS.md): Ruby parser, staged-file
  hook, path-filtered CI, cross-repository Fortune utilities and optional M3 wrapper
- [Existing contribution instructions](../CONTRIBUTING.md): setup, provenance and
  regeneration precautions

## Actual project and package boundaries

The repository is [hsoliwal/providers](https://github.com/hsoliwal/providers), forked
from [api-evangelist/providers](https://github.com/api-evangelist/providers). It is
primarily a Jekyll catalog with Python data-generation tools. It is not a provider
API gateway. The M3 direction in the README is an exploration purpose, not evidence
that the catalog implements an M3 computational runtime.

| Project/configured unit | Boundary and evidence | Documentation status |
| --- | --- | --- |
| Provider catalog/site | `_config.yml`, page front matter, `_data`, `_providers`, `_includes`, `_layouts`, scripts and assets | Partial: pipeline/selected contracts documented; complete provider inventory/rendering unavailable |
| Optional analysis wrapper | `.m3/atomize-patternize/{bootstrap.sh,run.sh,README.md}`; own inventory/receipt/configuration lifecycle | Source contract documented; execution and bound inventory absent |
| ZIP container utility | Root `Dockerfile`, Alpine base and `apk add zip` | Source contract documented; not a Jekyll development environment; not built |

There is no first-party package-manager/workspace manifest or `scripts/__init__.py`
in the enumerated paths. The six `.py` files are language modules/scripts; `scripts`
is not documented as an installable distribution. Category, industry, country and
alphabetical directories represent routes, not one Python/npm package per directory.

The inspected vendored manifest identifies `monaco-editor` version `0.52.0`, from
Microsoft, with an MIT license file. Its declared ESM entry points are absent from
this vendored tree; minified runtime files and a declaration bundle are present.
See the [package boundary](DATA_AND_TEMPLATE_CONTRACTS.md#vendored-monaco-editor-package)
before assuming this is a complete npm installation. Other bundled JavaScript/CSS
is included in asset accounting without inventing package identities from filenames.

## Inventory extent and honest denominators

The pinned root contains 133 entries: 115 directories and 18 files. Nontruncated
subtree responses were obtained for 114 root directories. Both recursive and
nonrecursive reads of `_providers` failed with a transport error. The union of the
successful responses contains 1,215 known tree entries, including 855 blobs, plus
the unresolved contents below that one known directory. These are **partial-tree
counts**, not total repository file counts.

The available tree identifies 8 scripts, 32 include files, 3 layouts, and 202 `_data`
files. No root AGENTS.md/CLAUDE.md, root Gemfile/lockfile, Python package manifest,
or test directory was found in those enumerated paths. This is not proof that no
such file exists below the unread subtree. Relevant existing contribution documents
and the optional-wrapper README were read before authoring.

| Level | Discovered / included | Excluded | Unresolved or blocked | Reviewed-complete | Current treatment |
| --- | --- | --- | --- | --- | --- |
| Repository | 1 / 1 | 0 | Complete descendant inventory blocked | 0 | Partial, with source revision and limitations explicit |
| Configured projects/units | 3 / 3 | 0 | Site provider/template/render coverage incomplete | 0 | Site partial; other two source-described, execution unverified |
| First-party Python modules | 6 / 6 | 0 | Runtime/tests not run; taxonomy prose and every API leaf not independently validated | 0 | All six have substantive contract coverage, still partial under full completion gate |
| Auxiliary executable modules | 5 / 5 | 0 | Runtime checks not run | 0 | Ruby validator, Fortune shell publisher, pre-commit hook, M3 bootstrap and runner source-described; partial |
| Liquid includes/layouts | 35 / 35 | 0 | Complete bodies unavailable for 13; remaining fragments not all semantically reviewed | 0 | Selected producer/consumer contracts documented; no all-template claim |
| Vendored package | 1 / 1 | 0 | Complete editor API/integration/runtime review unresolved | 0 | Identity, version, manifest entry points, tree mismatch and license source documented; partial |

“Reviewed-complete” requires the full documentation/verification gate; zero here
does not mean no useful documentation was written. It prevents source inspection
and a PR from being reported as complete runtime/API coverage. File accounting and
semantic documentation coverage are separate. Generated/provider/vendor content
has not been silently excluded to improve a percentage.

## First-party module ledger

| Module | Responsibility and important contracts | Guide |
| --- | --- | --- |
| `scripts/build-listing.py` | Descriptor/enrichment join; `.git` eligibility; categories; UTF-8 repair; uncapped rated output; listing page writes/deletions | [Listing builder](GENERATOR_CONTRACTS.md#alphabetical-and-category-builder) |
| `scripts/build-providers.py` | Front-matter reuse/fallback; artifacts; papers; publication guards; blocked-page deletion; retirement redirects | [Detail builder](GENERATOR_CONTRACTS.md#provider-detail-builder) |
| `scripts/build-sections.py` | Taxonomy/tag/roster membership; score-gap fallback; capped rows; independent VC contract; post-pass | [Section builder](GENERATOR_CONTRACTS.md#section-builder) |
| `scripts/build-section-tools.py` | Opted-in page scan; qualifying-cohort threshold; graded credit; tools front-matter/stubs | [Tool post-pass](GENERATOR_CONTRACTS.md#shared-bands-and-market-tool-post-pass) |
| `scripts/lib_bands.py` | Rubric candidate loading, cache, known-ID filter, labels/ranges/fallbacks | [Shared bands](GENERATOR_CONTRACTS.md#shared-bands-and-market-tool-post-pass) |
| `scripts/tag-fortune1000.py` | Top-level operational scan; embedded matching roster; dry-run versus cross-repository write | [Fortune maintenance](VALIDATION_AND_OPERATIONS.md#fortune-maintenance-utilities) |

Ruby `check-data.rb`, Bash `commit-fortune-tags.sh`, the pre-commit hook, and both
M3 shell entry points are separately explained in [validation and operations](VALIDATION_AND_OPERATIONS.md).
They are not additional Python packages.

## Presentation and data ledger

| Surface | Accounting and current semantic coverage |
| --- | --- |
| `_data` | 202 paths enumerated; key producer-owned families described; all records and remaining schemas not audited |
| `_providers` | Known generated collection boundary from config/producer; enumeration blocked; record count and per-record coverage unknown |
| Root/category/letter routes | Enumerated in returned subtrees; generated family contracts described; individual rendered pages untested |
| `industries`, `countries`, `alphabetical` | Subtrees enumerated; shared generators/shape documented; individual membership facts not independently verified |
| `_includes/home.html`, `company-listing.html`, `company-listing-rated.html`, `company-listing-item.html` | Bodies fetched; principal consumer and interaction contracts reviewed; no browser test |
| Other fetched includes | Activity, AI banner, plain/simple listings, detail header, footer, goals, head, score embed, paper promo/card, performance, provenance banner, artifact box, metadata, random paper, service banner, large sidenav; individual semantic coverage remains partial |
| Unfetched includes | `provider-checklist-offer.html`, `section-cards.html`, `sidenav-sm.html`, `source-widget.html`, `tag-selector.html`, `tool-cohort-json.html`, `tool-opportunities.html`, `tool-priorities.html`, `topnav.html`, `vc-listing.html`; inventoried only |
| `_layouts` | All 3 paths inventoried; producer/config roles established; full bodies not fetched |
| `assets` | Complete subtree listing returned; Monaco manifest/license inspected; vendor/bundle/image/source-map roles accounted; complete package APIs and other asset provenance incomplete |
| `.well-known`, robots, sitemap, CNAME, root descriptors/rules/review metadata | Paths accounted; detailed publishing/search/schema behavior not validated |
| `.github`, `.githooks` | Workflow and staged-content validator source reviewed; no hook execution or CI result inferred |
| `.m3` | Full wrapper source read; no bound inventory/analysis receipt in inspected subtree; no execution |

## Important limitations and next inspection targets

1. Finish `_providers` inventory and inspect representative ordinary, minimal, coverage,
   and redirect documents. Do not infer the count from historical generator comments
2. Read all three layouts and the ten currently unfetched includes, then reconcile
   actual browser interactions, source loading and field requirements
3. Review Monaco API/integration usage and the remaining asset dependencies without
   replacing upstream notices; no blanket repository license was identified at root
4. Review remaining data schemas and producer provenance, plus inline comments that
   still describe flat lists/no-links despite newer implementation behavior
5. Run declared parsing, semantic-data and site-render checks in a complete authorized
   environment. Record failures and unexecuted stages separately

These are open documentation/verification gaps, not permission to change source,
dependencies, workflows, publication guards, assets or licenses. This batch adds
Markdown guidance only and preserves all existing documentation text.
