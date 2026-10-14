## M3 direction in this fork

M3 is based on the idea that shared immutable structure and indexed metadata can
provide a basis for reusable computation. The programme starts with M3 String,
regex, and precompute in M3JDK21, then collections and SWT/Eclipse integration.

This provider-catalog fork is an exploration surface for organizing reusable API knowledge and provenance. Its role is supporting tooling around M3, with the catalog's own data and build contracts remaining authoritative.

**Explore the idea:** [M3 technical design paper](https://github.com/hsoliwal/M3jdk21/blob/master/m3/papers/M3_SHARED_STRUCTURE_AND_REUSABLE_COMPUTATION.md) ·
[Programme goals and roadmap](https://github.com/hsoliwal/M3jdk21#project-goals-and-plan).
The paper explains the proposed architecture, cost tradeoffs, and evaluation plan.
Readers are invited to examine the work and contribute representative workloads.

This section describes the direction of the hsoliwal fork. Upstream documentation
follows below; upstream authorship, licenses, and project identity remain intact.

---

# API Evangelist Providers (api-evangelist-providers)
An alphabetical listing of every provider tracked across the API Evangelist network.

## Working with this fork

This is [hsoliwal/providers](https://github.com/hsoliwal/providers), a fork of
[api-evangelist/providers](https://github.com/api-evangelist/providers).
The catalog describes public API providers; it does not host or grant access to
those providers' APIs. Upstream site URLs and maintainer attribution below are
retained and do not imply that this fork deploys to those domains.

- [Contributor guide](CONTRIBUTING.md): source layout, generation prerequisites,
  data validation, and provenance
- [Site configuration](_config.yml), [templates](_layouts), and
  [shared includes](_includes): static-site entrypoints
- [Generated catalog data](_data) and [provider documents](_providers):
  inspect the committed snapshot without running generators


**URL:** [Visit APIs.json URL](https://github.com/api-evangelist/providers/blob/main/apis.yml)

## Scope

- **Type:** Providers
- **Position:** Consuming
- **Access:** 3rd-Party

## Tags

- Providers

## Timestamps

- **Created:** 2024-10-14
- **Modified:** 2026-06-03

## Listing

The site is a static Jekyll catalog with alphabetical, category, and rating-band
listings. The current tree also includes provider detail documents in
[`_providers/`](_providers), rendered using the provider collection in
[`_config.yml`](_config.yml) and [provider layout](_layouts/provider.html).

[`scripts/build-listing.py`](scripts/build-listing.py) writes the alphabetical
map to `_data/providers.json`, category/band datasets to
`_data/providers-*.json`, and generated listing pages. Its source URLs point to:

- an **APIs.io provider page** when enriched provider metadata is available, or
- the provider's **API Evangelist GitHub repository** otherwise

[`scripts/build-providers.py`](scripts/build-providers.py) separately generates
the local provider detail documents and paper catalog.
[`scripts/build-sections.py`](scripts/build-sections.py) builds industry,
country, and Australian-bank sections.

The generators depend on a larger workspace containing provider repositories,
enriched metadata, scoring rules, and delisting information. These inputs are
not all included in a standalone clone. Read the
[generation prerequisites](CONTRIBUTING.md#regenerating-the-catalog) before running:

```sh
python3 scripts/build-listing.py
```

Generation rewrites data and pages. This is a static site, not an API backend;
the committed [apis.json](apis.json) and [apis.yml](apis.yml) files are catalog
metadata, not live provider endpoints.

## Editing `_data/` — read this first

Jekyll does **not** parse `_data/*.json` with a JSON parser. It hands them to
SafeYAML → Psych, and YAML is stricter than JSON: it accepts `\uXXXX` only for the
Basic Multilingual Plane and rejects the surrogate-pair escapes JSON uses for
anything above U+FFFF. A file can be perfectly valid JSON and still kill the entire
Pages build before one page renders — which is what a single emoji in one provider
description, written by a bare `json.dump`, did on 2026-08-11.

So: **anything that rewrites a file under `_data/` must go through `dump_json()` in
`scripts/build-listing.py`**, which desurrogates and writes `ensure_ascii=False`.
Never a bare `json.dump` (its default `ensure_ascii=True` is the trap).

Two guards enforce this:

```bash
ruby scripts/check-data.rb        # parse every data file the way Pages will
git config core.hooksPath .githooks   # run that check automatically pre-commit
```

The second line is **required once per clone** — `core.hooksPath` lives in
`.git/config` and is not versioned. `.github/workflows/validate-data.yml` is the
backstop if it was never run; it reports in ~1 minute instead of the ~25 the Pages
build takes to tell you.

## Properties

- [GitHubRepository](https://github.com/api-evangelist/providers)
- [Documentation](https://developer.apievangelist.com/documentation/)

## Common Properties

- [GitHubOrganization](https://github.com/api-evangelist/)

## Maintainers

**FN:** Kin Lane

**Email:** info@apievangelist.com

## Contributor field guide

See the [catalog data-flow and contributor guide](documentation/CONTRIBUTOR_GUIDE.md) for the
input checkout layout, source walkthrough, validation commands, and safe regeneration boundaries.
