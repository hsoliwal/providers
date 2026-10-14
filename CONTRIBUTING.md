# Contributing to the providers fork

[Back to the README](README.md)

## Choose the smallest change

- For navigation or presentation, start with [index.html](index.html),
  [_includes](_includes), [_layouts](_layouts), and [_config.yml](_config.yml).
- For generated catalog listings, start with
  [build-listing.py](scripts/build-listing.py).
- For provider detail documents, start with
  [build-providers.py](scripts/build-providers.py) and [_providers](_providers).
- For industry/country sections, start with
  [build-sections.py](scripts/build-sections.py).
- For rating-band labels and ranges, read
  [lib_bands.py](scripts/lib_bands.py); it loads the upstream rubric rather than
  defining a separate scoring system.

Generated files describe a snapshot. A listing is not a promise of current API
availability, pricing, authorization, or support from the named provider.
Use the provider's own linked documentation to confirm those details.

## Validate the existing snapshot

The checked-in data can be syntax-checked without regenerating the catalog.
From the repository root, with Ruby and its Psych YAML parser available:

```sh
ruby scripts/check-data.rb
```

The [validator](scripts/check-data.rb) parses JSON/YAML under `_data/` plus
`_config.yml` using Psych, matching the repository's documented Pages parsing
constraint. It does not validate provider endpoint availability, rating accuracy,
Liquid rendering, or a complete Jekyll deployment.

The [data validation workflow](.github/workflows/validate-data.yml) is
path-filtered. A Markdown-only PR may not trigger it; no workflow run is not
evidence of a passing build.

The README explains the per-clone [pre-commit hook setup](README.md#editing-_data--read-this-first).
Preserve the `dump_json()` helper for generated data: ordinary JSON validity
alone does not catch surrogate-escape failures in Pages' YAML parser.

## Regenerating the catalog

Python 3 and PyYAML are needed by the generation scripts. They resolve
`ROOT` **two directories above the site checkout**, not from the shell's
working directory. Their inputs are a wider workspace, for example:

```text
workspace/
  all/<provider>/apis.yml
  api-search/
    providers/_providers/<provider>.md
    signals/_data/scoring.yml
    network/_data/delisted.yml
    network/scripts/lib_tag_norm.py
  insights-work/_data/industries.yml
  api-evangelist/
    providers/                         # this checkout
    papers/_papers/                    # detail-page paper input
```

This layout is derived from the scripts' path constants. The external inputs
are not automatically fetched by cloning this fork. The section generator also
imports the external tag-normalization module; the band reader has a fallback
to `api-search/network/_data/scoring.yml`.

Before regeneration, confirm the source datasets, scoring rubric, and delisting
registry are present and current. A missing delisting registry can leave the
generator's exclusion guard inactive; do not publish such an incomplete rebuild.

After preparing those inputs, run only the generator relevant to your change
from the checkout root, then validate and inspect all resulting changes:

```sh
python3 scripts/build-listing.py
ruby scripts/check-data.rb
git diff --stat
git diff
```

The listing generator also rewrites page files and removes legacy root-level
letter HTML files. Do not run it merely to check a README edit.
The root [Dockerfile](Dockerfile) installs a ZIP utility; it is not a documented
Jekyll preview environment. This checkout has no root Gemfile/Gemfile.lock
pinning a reproducible local site-build dependency set.

## Review and provenance

Open changes for this fork against
[hsoliwal/providers](https://github.com/hsoliwal/providers/pulls), based on
`main`. Name the source data and generator used, include the affected page or
dataset, and distinguish syntax checks from rendering or deployment tests.

Preserve the attribution to
[API Evangelist](https://github.com/api-evangelist/providers) and the source
links on provider records. No repository-wide `LICENSE` file is present in
this checkout's root; do not infer a blanket reuse license from public access.
The [first-party provenance notice](M3-SYNEXIA-RIGHTS.md) does not relicense
third-party/provider material. Check the terms applying to the particular data,
images, code, or documents before redistributing them.

Do not commit credentials, private API responses, customer data, or private
operational records into this public catalog.
