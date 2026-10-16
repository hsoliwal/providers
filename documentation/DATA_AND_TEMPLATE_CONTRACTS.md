# Data and template contracts

[Documentation map](ARCHITECTURE_AND_COVERAGE.md) · [Generators](GENERATOR_CONTRACTS.md) · [Validation](VALIDATION_AND_OPERATIONS.md)

These contracts come from inspected producers and selected consumers at
`6ac8607839bf768ad40cd0117e47831070fbd9fd`. They are not a schema-validation result
for every checked-in record. In particular, the large `_providers` subtree could
not be enumerated in this pass, and complete provider layouts and interactive-tool
templates remain an explicit inspection gap.

## Data families

| Family | Shape and consumers | Source of truth / important distinction |
| --- | --- | --- |
| `_data/providers.json` | Map of lowercase ASCII-letter/`0-9` keys to entry arrays; `company-listing.html` | `build-listing.py`; not `{total, bands}` |
| Listing-builder category and band files | `{total, bands}`; each nonempty group has `band`, `label`, `range`, `blurb`, `count`, `providers`, optional `open` | No emitted-row cap in this producer |
| Section-builder rated files | Same band concept plus top-level `shown`, `limit` and per-band `shown` | `count` can exceed array length intentionally because rows are capped |
| `_data/sections-industries.json` | Card array: slug/name/description/icon/count/roster-only flag | Industry selection combines taxonomy and tag clusters |
| `_data/sections-countries.json` | Card array with slug/name/flag/count/by-tag/by-HQ counts | Tags and explicit HQ statements are unioned; providers may occur in multiple countries |
| `_data/providers-vcs.json` | `{total, portfolio_companies, on_network, vcs}` | Portfolio strength/rating, not provider API scores |
| `_providers/<slug>.md` | YAML front matter with provider layout and empty body; retirement files use redirect layout | `build-providers.py`; data fields retained by explicit `KEEP` list |
| `_data/papers.json` | Array of slug/kind/url/title/tagline/area/price/cover objects | Sibling paper Markdown; sorted/deduplicated by slug |
| `_data/tag-industries.yml` | Published tag-cluster definitions plus optional include/exclude lists | `TAG_INDUSTRIES` and override tables in `build-sections.py` |
| `_data/scoring.yml` | Mirrored scoring/rubric data | External rubric; listing and section copy paths differ, see generator guide |
| `_data/resource_groups.yml` | Mirrored resource taxonomy | Sibling provider build when present |
| `_data/provider-sections.yml` | Groups containing home-card configuration | Read directly by `home.html`; do not replace it with a generated provider array |

The tree also contains experiences, policies, properties, rules, strategies,
navigation, partner, ticker, tag-slug, and other datasets. Their exact ownership,
schema, and external generation provenance are not established by their filenames.
They remain included but partial in the [coverage ledger](ARCHITECTURE_AND_COVERAGE.md).
Likewise, `api-rules.yml`, `operational-rules.yml`, `review.yml`, `apis.yml` and
`apis.json` are tracked metadata; their presence does not expose a running API.

## Row contract and count invariants

Alphabetical rows always carry `name`, `slug`, `url`, and `type`; description and
screenshot are conditional. Section rows begin with `name` and `slug`; they do not
require the alphabetical `url`/`type` fields. Rated enrichment can add `score`,
`band`, `band_label`, `facets`, `scored_at`, `schema_version`, `regulatory`,
`agent_score`, `agent_band`, and `agent_dims`. Missing score is unknown, not zero.

For uncapped output, compare `total`, group counts and provider-array lengths.
For capped output, compare `shown` with the sum of emitted array lengths and
per-group `shown`; compare full group counts separately with `total`. An unknown
band can cause a mismatch because only recognized ladder groups are emitted.
These are useful diagnostic expectations, not checks already executed on the
snapshot. A parseable file can still be stale, incomplete, or internally inconsistent.

Illustrative shape only, not a real provider record:

```json
{
  "total": 12,
  "shown": 1,
  "limit": 1,
  "bands": [{
    "band": "strong", "label": "Strong", "range": "", "blurb": "",
    "count": 12, "shown": 1, "open": true,
    "providers": [{"slug": "example-provider", "name": "Example", "score": 60, "band": "strong", "rank": 1}]
  }]
}
```

The example demonstrates truncation only; it does not specify current scoring
thresholds. Read the rubric rather than deriving thresholds from this row.

## Jekyll project boundary

[`_config.yml`](../_config.yml) declares the `providers` collection with output
enabled and `/providers/:path/` permalinks, and assigns provider layout by default.
It uses Kramdown/Rouge and lists feed, SEO, sitemap, gist and avatar plugins. It
includes `.well-known` and excludes scripts, node_modules, Gemfile and lockfile.
The configured upstream site URL is not evidence of this fork's deployment.

Root `index.html`, alphabetical pages, category pages and industry/country pages
select includes through front matter. Their directories are route/data groupings,
not separate distribution packages. The source tree contains three layouts:
`default.html`, `provider.html`, and `redirect.html`. Producer/configuration contracts
identify their roles, but their complete bodies were not available to this pass;
full layout rendering and redirect behavior remain unverified.

## Inspected listing/home consumers

### Home cards

[`home.html`](../_includes/home.html) iterates `site.data['provider-sections']`.
Group fields control headings, column classes and compact styling. Cards can
reference a single count source or a `markets` array. `count.from` selects:

- `total`: take `site.data[key].total`
- `size`: take the referenced collection's size
- `sum`: add the lengths of values in a map, as for alphabetical groups
- `markets`: sum each market's referenced total and render a link per market

These counts summarize the referenced datasets. Summing overlapping markets is
not a deduplicated count of organizations. Card links and text come from config;
editing the template is not necessary to add an ordinary card.

### Alphabetical view

[`company-listing.html`](../_includes/company-listing.html) lowercases `page.letter`
and reads `site.data.providers[key]`. It emits local Profile, external APIs.io,
and upstream GitHub buttons built from the slug, rather than the row's stored URL.
Descriptions are truncated to 160 characters. Screenshot images are lazy-loaded
and remove themselves on error; an image failure does not remove the row.

Its browser `filterItems()` performs a case-insensitive substring filter on
rendered name/slug attributes and toggles a no-results message. The search is only
over the current page's rows, not a backend or the full provider collection.

### Rated view and row component

[`company-listing-rated.html`](../_includes/company-listing-rated.html) reads
`site.data[page.data_key]` plus `site.data.scoring`. It renders band `<details>` groups,
cap explanations, and the [row include](../_includes/company-listing-item.html).
`page.separate_tiers` moves matching **emitted rows** into separate tier groups; it
does not recover rows omitted by the producer's global rank cap. Separate-tier
counts are derived from those emitted rows.

The view adds a local text filter, expands matching groups while searching, and
restores recorded open states when clearing. Score clicks toggle each row's
breakdown. If `page.tools` is present, it renders modal buttons and lazy-loaded
iframe tool pages. Close buttons, backdrop click and Escape hide the dialog and
restore focus; source inspection is not an accessibility or browser-test pass.
The tool pages themselves are separate templates whose implementation is still a
coverage gap, even though the stub-generation contract is documented.

## Provider-facing include boundaries

The fetched include set contains the artifact box, provider metadata, paper cards,
random-paper sidebar, score embed, provenance banner, head/footer/navigation, and
supporting promotional components. They are view fragments, not standalone public
APIs. Refer to their source and the producer's `KEEP`/`ARTIFACT_KEYS` lists when
changing field names. This pass does not claim every fragment's parameter/error
contract is reviewed; [the ledger](ARCHITECTURE_AND_COVERAGE.md) lists exact missing
reads rather than treating successful tree enumeration as semantic coverage.

## External assets and provenance

`assets` contains styles, images, JavaScript bundles, source maps and a vendored
Monaco editor tree. Do not infer redistribution terms for all these assets from
public repository access or from the fork's first-party notice. Existing assets,
notices and upstream identity remain unchanged by these guides.

### Vendored Monaco editor package

The pinned [package manifest](https://github.com/hsoliwal/providers/blob/6ac8607839bf768ad40cd0117e47831070fbd9fd/assets/js/monaco-editor/package.json)
identifies `monaco-editor` version `0.52.0`, a browser code editor authored by Microsoft
Corporation, with upstream repository `microsoft/monaco-editor`. This is a vendored
third-party package, not a root first-party npm workspace. The manifest declares:

- `typings`: `./esm/vs/editor/editor.api.d.ts`
- `module`: `./esm/vs/editor/editor.main.js`
- No `main` or `exports` field in this inspected manifest
- Development dependencies for upstream builds/tests, including `monaco-editor-core`
  version `0.52.0`, TypeScript, language services, bundlers, Mocha and Playwright

The enumerated vendored subtree does **not** contain those two `esm` entry points or
the upstream `src`, `build` and `test` directories referenced by manifest scripts.
It does contain `min/vs/loader.js`, `min/vs/editor/editor.main.js` and CSS, basic-language
modules, language workers/modes, localized messages and `monaco.d.ts`. Treat it as a
selected browser asset snapshot. The manifest alone does not make `npm test`, an ESM
import, or an upstream build runnable inside this directory. Do not install or rebuild
it merely to validate these documentation changes.

The adjacent [LICENSE](https://github.com/hsoliwal/providers/blob/6ac8607839bf768ad40cd0117e47831070fbd9fd/assets/js/monaco-editor/LICENSE)
is the MIT license with Microsoft copyright; it includes a notice-preservation
condition and warranty disclaimer. Preserve that actual file and applicable notices
when using or redistributing this component. That license is evidence for this
component, not a repository-wide license grant for provider records, images or all
other bundles. This page records the notice; it does not change license terms.

The full editor API declarations, runtime implementation, workers, actual site
initialization and browser behavior have not been reviewed/tested in this pass.
Those remain package-level coverage gaps rather than an assertion of compatibility
or a working editor integration.
