# .motion Protocol History

Updated: 2026-10-12. Protocol contracts, template versions, helpers, component libraries and deployment state are tracked separately.

| Contract | Status | Scope |
| --- | --- | --- |
| Content `0.1.0-draft.1` | Draft, existing compatibility contract | Author ZIP, parameters, assets, runtime and output |
| Delivery `1.0.0-rc.1` | Candidate | `personal-download-v1`, frozen release, personal credentials, signed provenance |
| Stable `1.0.0` | Not frozen | Must not be presented as the current released protocol |

## 2026-10-12 · Open format documentation

The primary page describes content structure, runtime profiles, parameter/material adaptation, component dependencies, integrity, licensing, compatible upgrades and interoperability. Online APIs are optional integrations, and MotionGit personal delivery has a separate platform appendix.

Delivery and content versions remain distinct. Existing files keep their declared versions. Internal author payloads are not a user-facing “public template” product. Authoring models/tools describe provenance and recommendations, not format admission rules.

## 2026-10-10 through 2026-10-11 · Personal delivery candidate

Six-file delivery preserves frozen inner bytes and embeds the downloader's current account Key. Ed25519/JCS receipts bind download provenance. Optional `protected-v1`, host execution, instances and fixed-version updates have separate contracts.

Larger bounded readers use helper baseline `1.0.0-rc.2` while the exchange version remains `1.0.0-rc.1`. Candidate capability records are not evidence that every production gate is open or every template has passed review.

## Existing 0.1 draft

`0.1.0-draft.1` uses exact version matching with manifests, parameter schemas, material roles, presets/variants, runtime declarations and file checks. Changing a version string alone is not migration.

## Evolution rules

Preserve published bytes and hashes. Version protocols, templates, helpers and components separately. Stop affected operations for unknown execution semantics. Future parameter migration needs an explicit contract and preview acceptance; discovering an update does not establish automatic migration support.

[Open format](README.md) · [Detailed content contract](LEGACY.en.md) · [Personal delivery profile](PERSONAL.en.md)
