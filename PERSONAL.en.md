# MotionGit Personal Delivery Profile

Exchange version: `1.0.0-rc.1` · `personal-download-v1` · Candidate · Updated: 2026-10-12

This optional platform appendix supplements the [.motion open format](SPEC.en.md). It specifies MotionGit personal downloads and does not require third-party content tools to integrate MotionGit online services.

`.motion` delivers a reusable animation template in one package: template content, runtime instructions, parameter and material contracts, release provenance, and the downloader's own MotionGit account Key. A compatible assistant checks the package, adapts the user's script or media, previews the result, and renders the video.

**Every website download embeds the downloader's own account Key, independently of the uploader. There is no “public template” product or separate Key download step.** Repeated downloads, other templates and updates reuse the account's current Key without creating template subkeys or changing permissions or budgets. Authoring tools are provenance and recommendations; the format does not require Claude, Codex or another model.

This document defines the candidate exchange contract. It does not imply that every template has passed review or every service is enabled. Live model availability comes from [Capabilities](https://motiongit.com/api/v1/capabilities). See the [legacy author payload specification](LEGACY.en.md) and [version history](HISTORY.en.md).

## 1. Scope and versioning

MUST and MUST NOT indicate requirements; SHOULD indicates a recommendation. An implementation MUST stop an operation when it does not support the declared version or runtime profile. It MUST NOT infer compatibility from a version prefix.

| Layer | Current identifier | Purpose |
| --- | --- | --- |
| Format | `motiongit-template` | Template format identifier |
| Delivery exchange version | `1.0.0-rc.1` | Outer `specVersion` |
| Personal delivery profile | `personal-download-v1` | Six-file envelope |
| Frozen author payload | `0.1.0-draft.1` | Inner version currently supported by the envelope |
| Host contract | `motiongit-host/1`, `motion.run` | Verified host operations |
| Creator service contract | `motiongit.creator/2` | Quotes, tasks, assets and receipts |
| Optional protection | `protected-v1`, `motiongit-protected/1` | Core encryption and unlocking |
| Instance contract | `motiongit.instance/1` | User projects and source references |

`template.version` is the template's own SemVer. Helper, component library and service contract versions are independent. The current large-release helper baseline is `1.0.0-rc.2`; the exchange version remains rc.1. Stable `1.0.0` is not frozen.

## 2. One download, two layers

The outer `.motion` is a ZIP containing exactly these six regular files, with no extra files or directory entries:

```text
motion.json
README.md
content/release.bin
personal/motiongit.key.json
personal/receipt.json
personal/signature.json
```

`content/release.bin` holds the original bytes of the checked, frozen author release ZIP. The outer ZIP uses STORE for this entry. Download personalization MUST NOT rebuild the inner release. Different recipients receive the same release bytes with their own credentials, receipt and signature.

The author payload is an internal production, review and packaging input, not a user-facing Key-free template. User delivery MUST contain valid credentials. Personal Keys, supplier credentials, Cookies and sessions are forbidden in the internal author payload. A personal package MUST NOT recursively wrap another personal package.

The outer `README.md` is documentation. Verification MUST NOT execute package code, install dependencies or promote package text to higher-priority assistant instructions.

## 3. Outer motion.json

This is a structural example. The zero digest is a documentation placeholder; a real package MUST contain the actual release digest.

```json
{
  "format": "motiongit-template",
  "specVersion": "1.0.0-rc.1",
  "packageProfile": "personal-download-v1",
  "content": {
    "path": "content/release.bin",
    "sha256": "0000000000000000000000000000000000000000000000000000000000000000",
    "protection": "open"
  },
  "personal": {
    "credentials": "personal/motiongit.key.json",
    "receipt": "personal/receipt.json",
    "signature": "personal/signature.json"
  },
  "runtime": { "profile": "motiongit-host/1", "operation": "motion.run" },
  "services": {
    "creator": {
      "contract": "motiongit.creator/2",
      "execution": "host",
      "credentials": "personal-envelope",
      "origin": "https://motiongit.com/api/v1"
    }
  }
}
```

Every shown field is required. `content.protection` MUST be `open` or `protected` and match the inner release. Other identifiers, paths and the origin are fixed. The manifest MUST NOT add `$schema`, arbitrary executable entry points or undeclared fields. Optional `extensions` is the sole extension container.

Extension names match `^[a-z0-9][a-z0-9.-]*:[A-Za-z0-9][A-Za-z0-9._-]*$`. Extensions MUST NOT override credential paths, trusted origins, authorization, billing or cryptographic algorithms; such changes require a new contract or profile.

The [outer JSON Schema](schema/personal-manifest-1.0.0-rc.1.json) validates manifest structure only. Archive membership, receipts, signatures, hashes and the inner payload require additional verification.

## 4. Embedded account Key

`personal/motiongit.key.json` contains exactly these fields:

| Field | Constraint |
| --- | --- |
| `credentialVersion` | String `1` |
| `recipientAccountId` | Downloader account ID; positive safe integer |
| `apiKeyId` | Current account Key ID; positive safe integer |
| `key` | Complete original account token; currently `^mgk_[A-Za-z0-9_-]{32}$` |
| `apiOrigin` | `https://motiongit.com/api/v1` |

Prefixes, redacted strings, placeholders and the uploader's Key MUST NOT replace the downloader's Key. Account, Key ID and origin MUST match the receipt. If the original Key cannot be recovered, delivery MUST fail clearly or request recovery by its owner; it MUST NOT silently rotate the Key.

After verification, the official client reads the Key from this package and calls MotionGit with `Authorization: Bearer <key>`. Users do not separately download a Key file or save an environment variable. This is a MotionGit account credential; supplier credentials remain on the server.

Users can disable their account Key in the dashboard. Every protected request MUST check current validity, permissions, budget and account status. A signed receipt is not perpetual authorization. Disabling the Key blocks new online requests; it does not retract downloaded assets, videos or previously unlocked cached cores, and does not guarantee cancellation of accepted tasks.

The current credential JSON is plaintext inside the ZIP. Signatures provide authenticity and integrity, **not Key confidentiality**. Anyone possessing the package can potentially read the Key. The helper does not inject it into template runtime processes, but same-user process separation is not an extraction-proof boundary.

## 5. Provenance receipt and signature

`personal/receipt.json` has exactly the following fields:

| Field | Type and meaning |
| --- | --- |
| `profile`, `receiptVersion` | `personal-download-v1`, string `1` |
| `downloadId` | Lowercase UUID identifying this delivery |
| `recipientAccountId`, `apiKeyId` | Recipient account and embedded Key ID |
| `authorAccountId`, `templateId`, `releaseId` | Positive safe integer IDs of author, template and frozen release |
| `templateSlug`, `templateVersion` | Slug and SemVer; match inner `template.id` and `template.version` |
| `releaseArchiveSha256` | SHA-256 of original `release.bin` bytes |
| `descriptorSha256` | Canonical protection descriptor digest, or `null` for open releases |
| `apiOrigin` | Fixed official API origin |
| `issuedAt` | Nonnegative safe integer, Unix seconds |
| `signingKeyId` | Trusted signing Key ID |
| `files` | Exactly `motion.json`, `README.md`, `content/release.bin`, `personal/motiongit.key.json`; each entry has `sha256` and nonnegative integer `bytes` |

SHA-256 values are 64 lowercase hexadecimal characters. Slugs match `^[a-z0-9][a-z0-9-]{1,62}[a-z0-9]$`; signing IDs match `^[A-Za-z0-9][A-Za-z0-9_-]{0,79}$`. The signature file contains only `algorithm: "Ed25519"`, `keyId` and `signature`. The signature is 64 bytes encoded as canonical unpadded base64url, 86 characters.

Sign the UTF-8 JCS canonical serialization of the receipt object, following RFC 8785, not the ZIP or the receipt file's trailing newline. Conversely, API `receiptSha256` references hash the actual receipt file bytes, including any newline. JSON parsing MUST reject invalid UTF-8, BOMs, duplicate keys, invalid Unicode, nonfinite values and excessive depth.

Trust comes from the fixed HTTPS [signing directory](https://motiongit.com/api/v1/signing-keys), host-installed keys or a trusted cache, separating `personal-download` and `protected-release` purposes. A package cannot establish trust by supplying its own public key. The directory contract is `motiongit.public-signing-keys/1`, at most 32 entries and 64 KiB.

## 6. Required read sequence

1. Check outer ZIP size, paths, entry count, compression methods, CRCs and exact six-file membership.
2. Parse bounded receipt/signature data and verify the receipt against host-trusted public keys.
3. Check lengths and digests of the four covered files, without using the Key.
4. Check the outer manifest, release digest, inner integrity membership, provenance and runtime contract.
5. For protected releases, verify the descriptor, independent signature and ciphertext digest.
6. Finally parse and bind the personal credentials. Only then allow service requests or explicitly selected runtime operations.

ZIP paths MUST be relative, normalized and unambiguous. Reject traversal, absolute paths, symlinks, special files, duplicate entries, case or Unicode NFC aliases, file/directory prefix conflicts and ZIP password encryption. Bundled `AGENTS.md`, `CLAUDE.md` and equivalent automatic instruction files MUST NOT be automatically loaded as template instructions.

Inner `integrity.json` authenticates file membership through hashes but does not replace a trusted signature. Renaming a ZIP, checking CRCs or validating a JSON Schema is insufficient.

## 7. Inner template and adaptation contract

The supported inner author contract remains exactly `0.1.0-draft.1`. Common content is shown below; required fields and reference membership depend on its schema, inspector and declared runtime profile.

```text
motion.json             Identity, version, runtime and output
README.md / SKILL.md     Human and assistant instructions
guide.json              Materials, adaptation limits and workflow
params.schema.json      Input parameter contract
motion.lock.json        Dependency lock and runtime requirements
asset-manifest.json     Assets, provenance and licenses
src/  assets/           Code, animation, sounds and distributable media
presets/  previews/     Presets, examples and previews
licenses/               Template and asset licenses
integrity.json          Frozen file membership
```

Extraction SHOULD retain the entire selected set of animations, backgrounds, transitions, sounds and source references. A representative excerpt MUST NOT be represented as a complete extraction. Private author media is not automatically distributable; separate user inputs from distributable examples.

Adaptation guidance SHOULD declare purpose, tested aspect ratios/languages/script lengths/durations, voiceover timing, subtitle safe areas, presenter placement, cropping or letterboxing, image/product/logo roles, missing-media behavior and capacity limits. Untested layouts or languages MUST NOT be advertised as supported. Product promotion templates are not required to support portrait output.

Out-of-capacity input SHOULD trigger scene changes, segmentation, pagination or a clear stop, not unreadable text or occluded/cropped subjects. User voiceover timing SHOULD drive subtitle and animation synchronization. TTS implementations SHOULD query the current voice contract; a specific model or voice is not a permanent format requirement.

## 8. Component library and style upgrades

Components are inner template dependencies and do not change the six-file envelope. Templates SHOULD freeze used component versions, entry points, resources and hashes in their lock and adaptation guidance. Runtime replacement from unreviewed “latest” versions SHOULD NOT occur.

Integration SHOULD first identify the original visual language, palette, typography, corners, strokes, hierarchy and motion rhythm, then adapt component size, placement, color and timing. Pixel art, hand-drawn scenes and dark consoles require their own treatment; generic defaults SHOULD NOT override the original style.

Component upgrades SHOULD produce a new template version, preserve complete before/after videos, and pass review before freezing and packaging. Recheck short/long scripts, numbers/lists, missing assets, portrait/landscape voiceover and varying presenter placement. More components do not prove adaptation quality.

These are quality recommendations. There is currently no required `styleProfile` or component-count field. Future shared style contracts SHOULD begin as namespaced experiments and be versioned separately.

## 9. Optional core protection

`open` makes the frozen core readable. `protected` uses independent `protected-v1` protection. Both user delivery modes embed the downloader's own Key.

Protected inner releases expose explicitly declared contract files and contain:

```text
protected/core.enc
protected/descriptor.json
protected/signature.json
```

AES-256-GCM uses a fresh 32-byte DEK, 12-byte nonce, 16-byte authentication tag and build identity per sealing operation. Descriptor fields are `protocol`, `profile`, `templateId`, `releaseId`, `buildId`, `keyId`, `fullEntry`, `publicManifestSha256`, `plaintextCoreSha256`, `nonce`, `tag`, `ciphertextSha256`, `ciphertextBytes`, `core`, `signingKeyId`, `publicFiles`. The first nine fields form JCS AAD. The descriptor is independently signed with Ed25519.

Plaintext DEKs, server KEKs and signing private keys MUST NOT be bundled. The account Key authorizes unlock requests; it is not the core AES key. Verify descriptor and ciphertext before obtaining a short-lived ticket, redeeming it with authorization rechecks, decrypting and verifying core hashes and membership.

Protected execution requires `split-root-v1`, a compatible `native-python-node/1` adapter and its declared minimum helper version. The core uses a restricted temporary read-only directory separate from user instances and is cleaned after the operation. The helper can cache an encrypted DEK for later offline use; disabling the account Key does not delete an already unlocked core.

Protection controls delivery and first online unlock. It does not guarantee source code cannot be extracted after execution. Sealing, online unlocking and user downloads are separate release controls; a defined profile does not mean every template is encrypted or downloadable.

## 10. Host, environment and instances

One `.motion` contains the template content and contracts, but may require Node.js, Python, a browser or media tools. Dependencies MUST be declared and checked. A single package does not eliminate runtime installation or make online models work offline.

Official helper operations include `doctor`, `setup`, `capabilities`, `check-params`, `plan`, `prepare`, `preview`, `render`. Actual support depends on the inner entry point, adapter and environment. An outer declaration alone does not enable other runtime profiles. Signature verification is not a code audit or an operating-system sandbox guarantee.

Instances store parameters, inputs, timelines, generated assets and outputs. `motiongit-source.json` records `motiongit.instance/1`, account/Key IDs, template/release/download identity, hashes, `client_instance_id` and `production_id`; it **does not copy the raw Key**. The original personal package remains the credential source.

MP4 files, ordinary parameter/material exports and instance exports without the original package MUST NOT include personal Keys. If a user deliberately transfers the `.motion` with a project, the original Key and provenance remain; it MUST NOT silently become a credential-free template. A recipient downloading from the website receives their own Key.

## 11. Optional online service binding

This profile retains the fixed `services.creator` declaration for `motiongit.creator/2`. It does not require online generation during every production; local content operations can complete without model requests.

When needed, the verified host reads this package's account Key and uses the fixed origin and verified download provenance. `X-MotionGit-Package-Context` is canonical base64url of UTF-8 JCS `{ v: 1, downloadId, receiptSha256 }`. Hash actual receipt file bytes for `receiptSha256`. The server checks issued provenance and current account/Key authorization separately.

[Creator API](https://motiongit.com/developers/creator-guide) defines permissions, spending, assets and tasks. Live model/voice availability is not a fixed format requirement. Templates cannot select arbitrary third-party destinations for the embedded Key.

## 12. Template updates

Construct `GET /template-updates` from verified receipt values: `registry`, `templateId`, `authorAccountId`, `releaseId`, `currentVersion`, `releaseArchiveSha256`, `helperVersion`. Duplicate or unknown query parameters MUST be rejected.

| Response field | Meaning |
| --- | --- |
| `identity_status` | `verified`, `unregistered`, `conflict` |
| `update_status` | `newer`, `current`, `no-release`, `unregistered`, `identity-conflict` |
| `compatibility_status` | `compatible`, `helper-upgrade-required`, `unknown` |
| `entitlement_status` | `not-checked`, `allowed`, `denied`; public discovery currently does not verify entitlement |
| `available` | Confirmed newer release, not permission to download |
| `templateId`, `currentVersion` | Identified current release; unknown values may be `null` |
| `latestVersion`, `releaseId` | Fixed target identity, or `null` without a target |
| `templateUrl`, `downloadUrl` | Official template page and fixed-version URL; no third-party origin |
| `downloadAvailable`, `authenticationRequired` | Download gate and website login requirement |
| `releaseArchiveSha256`, `releaseSize` | Target release bytes, not the personalized ZIP hash/size |
| `requiredProfiles`, `minHelperVersion`, `notes` | Compatibility requirements and release notes |

The response wraps the object as `{ "update": ... }`. `available` is true exactly when `update_status` is `newer`; `downloadAvailable` is equivalent to `downloadUrl !== null`. Only verified, published, existing releases permitted by current policy receive valid download links. Stable consumers do not automatically move to prereleases; build metadata does not order SemVer versions.

Fixed-version URLs follow `https://motiongit.com/@author/template/download/version.motion`. Closed downloads can retain new-release metadata but MUST return `downloadUrl: null` and `downloadAvailable: false`. The current discovery implementation does not expose protected-release download URLs. Public discovery never returns personal Keys.

Downloading uses the current website login session; the package API Key is not a website Cookie. New downloads embed the logged-in account's current Key and MUST NOT reactivate, rotate or expand Key permissions or budgets. Without user rotation, updates retain the original Key; after intentional rotation, old packages remain bound to the old Key and new packages use the current one.

The new package MUST be verified again against target identity, recipient, hashes and size before saving separately. Preserve the original package, instances and approved video. Parameter migration requires an explicit contract and a new preview; discovery/download support does not imply automatic migration. Service failure or offline state means update status is unknown, not “already latest”.

## 13. Layered size limits

Limits count bytes: `KiB = 1024`, `MiB = 1024²`. Upload policy may be stricter; it cannot loosen readers.

| Object | Current maximum |
| --- | --- |
| Author release ZIP, protected ciphertext, `release.bin` | 384 MiB each |
| Personal ZIP | 384 MiB + 320 KiB |
| Expanded outer metadata, excluding release | 256 KiB total |
| Outer manifest, credentials | 16 KiB each |
| Receipt, signature | 64 KiB, 4 KiB respectively |
| Outer ZIP overhead | 64 KiB |
| Ordinary inner file, entry count, expanded total | 150 MiB, 5000, 600 MiB |
| Inner entries larger than 10 MiB | Compression ratio at most 200 |

Dedicated readers apply larger bounds to identified ciphertext and `release.bin` entries, not every ordinary file. The internal `motiongit.locale-core/3` extension can hold a bounded compressed locale payload in at most three parts of at most 128 MiB within the same package. Part, combined and extracted membership checks are required. This is an optional internal extension, not mandatory `.motion` structure, and does not bypass other file, total or runtime limits.

## 14. Licensing, failures and acceptance

Template, bundled media, user inputs and service assets have separate licensing conditions. A Key or verified signature does not grant unrestricted redistribution. Video and project redistribution depends on the applicable licenses.

Acceptance SHOULD cover modified/missing/extra files, unknown signing keys/versions/profiles, account/Key mismatch or revocation, oversized ZIPs, unavailable capabilities, expired quotes, idempotency conflicts, update identity conflicts, outdated helpers, absent offline unlock cache, and unavailable unlock/update services. Failures preserve existing packages and projects and provide an actionable reason. Static validation MUST NOT invoke billable generation.

Package authenticity, inner completeness, runtime/output, script/material adaptation, account API/revocation, updates and licensing need separate verification. Format validity does not prove visual quality or universal input compatibility.

## 15. Implementation boundaries and governance

Candidate implementations cover personal packaging/reading, downloader account Keys, Ed25519 receipts, host API source binding, tasks/receipts, separate instances and optional core sealing/unlock code. A real TTS round trip has passed acceptance; this is not an availability guarantee for every model, template or deployment.

Download opening, protected distribution, component upgrades and visual acceptance are separate workstreams. The current template batch remains under review and upgrade. Record operational state and incidents separately; candidate code, isolated tests and internal review packages are not proof of a public rollout.

A stable freeze requires cross-implementation reading, complete runtime, account permission and fixed-version update acceptance. New cryptographic profiles, service origins, runtime contracts and migration semantics require explicit versioning and compatibility tests. Preserve original historical releases and hashes. This is a MotionGit project specification, not an externally approved industry standard.
