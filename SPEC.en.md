# .motion Open Animation Template Format

Technical specification · Candidate · Updated: 2026-10-12

`.motion` packages reusable animation templates using ZIP, JSON and animation project source files. It combines animation implementation, replaceable parameters, material guidance, dependency declarations, presets and licenses so people, AI assistants and compatible tools can inspect, adapt and render the same template.

The format emphasizes **understandable content, traceable dependencies, reproducible execution and explicit extensions**. It does not mandate an authoring model or animation engine. Online generation is optional: understanding the content, validating inputs and rendering local animation do not inherently require MotionGit API calls.

[Version history](HISTORY.en.md) · [Existing content contract](LEGACY.en.md) · [Optional platform delivery profile](PERSONAL.en.md)

## 1. Protocol layers

| Layer | Responsibility | Version location |
| --- | --- | --- |
| Template content | Source, parameters, assets, presets, previews and licenses | Content manifest `specVersion` |
| Runtime adapter | Environment checks, input handling, frame rendering and output | `runtime.profile` and adapter contract |
| Optional distribution | Release identity, signed provenance, personal credentials or core protection | Outer `packageProfile` and separate contracts |

The existing content version is `0.1.0-draft.1`. MotionGit personal delivery uses outer version `1.0.0-rc.1` and `personal-download-v1`. These describe different layers and MUST NOT be interchanged. Template `template.version`, helpers and component libraries have separate versions. Stable `1.0.0` is not frozen.

Optional means a protocol integration choice for implementations. MotionGit website user downloads still use personal delivery with the downloader's own account Key; this does not introduce a “public template” product or separate Key download workflow.

## 2. Openness and interoperability

The format uses standard containers and text contracts. Manifest fields, paths, runtime profiles and version semantics are inspectable. Developers can implement readers, static validators, parameter editors, engine adapters or renderers.

Template content is separate from platform services; understanding a content manifest does not require a private database. Signed or credentialed distribution profiles have additional trust and authorization contracts. An open format does not imply open licensing for every template, font, soundtrack or asset.

MUST and MUST NOT indicate requirements; SHOULD indicates recommendations. Implementations SHOULD declare precisely supported versions, profiles and extensions rather than accept unknown versions by prefix. Preserve unknown optional metadata; unsupported execution, permission or compatibility semantics require a clear warning or stopping the affected operation.

## 3. Content package structure

Content is a ZIP with a root `motion.json`. This is a typical layout; required entries depend on the exact version and references:

```text
motion.json
README.md
SKILL.md
guide.json
params.schema.json
motion.lock.json
asset-manifest.json
src/
assets/
presets/demo.json
previews/
licenses/
integrity.json
```

| Entry | Responsibility |
| --- | --- |
| `motion.json` | Identity, runtime, parameter references and output |
| `README.md`, `SKILL.md` | Human/assistant instructions; not automatically registered or executed |
| `guide.json` | Purpose, material roles, duration and adaptation limits |
| `params.schema.json` | JSON Schema for replaceable content |
| `motion.lock.json` | Dependency versions and tested environments |
| `asset-manifest.json` | Media/fonts, provenance, hashes and licensing |
| `src/`, `assets/` | Animation source and distributable resources |
| `presets/`, `previews/` | Inputs, style variants and previews |
| `licenses/` | Template and third-party licenses |
| `integrity.json` | Frozen membership and SHA-256 |

Relative paths MUST resolve accurately inside the package. Complete extraction retains the selected animation, backgrounds, transitions, sounds and source dependencies. An excerpt MUST NOT be presented as a complete template. Private user material is not automatically redistributable.

## 4. Content manifest

This example describes the current content layer, not a complete publishable archive. All referenced files must exist and pass validation.

```json
{
  "$schema": "https://motiongit.com/schema/motion/0.1.json",
  "format": "motiongit-template",
  "specVersion": "0.1.0-draft.1",
  "template": {
    "id": "example-title",
    "version": "1.0.0",
    "name": "Example Title",
    "status": "candidate"
  },
  "runtime": {
    "profile": "html-canvas/1",
    "entry": "src/index.html",
    "dependencyLock": "motion.lock.json"
  },
  "parameters": {
    "schema": "params.schema.json",
    "defaultPreset": "presets/demo.json"
  },
  "assets": { "manifest": "asset-manifest.json" },
  "output": {
    "width": 1920,
    "height": 1080,
    "fps": { "num": 30, "den": 1 },
    "durationPolicy": "fixed",
    "durationSeconds": 6
  },
  "license": { "file": "licenses/TEMPLATE-LICENSE.txt" },
  "permissions": { "network": false },
  "ai": { "entry": "SKILL.md", "guide": "guide.json" },
  "integrity": "integrity.json"
}
```

Core fields are `format`, `specVersion`, `template`, `runtime`, `output`. The format identifier is `motiongit-template`. IDs remain stable; versions follow their declared contract's SemVer rules. Frame rate uses positive integer `num/den` to avoid floating-point ambiguity.

Parameter, preset, asset, assistant and license fields reference package files. `permissions` declares requirements; the host enforces them. Packages cannot authorize themselves. Authoring models/tools are provenance or recommendations, not format admission restrictions.

The [content manifest Schema](schema/content-0.1.json) validates field structure. Reference completeness, actual execution and visual quality require separate checks.

## 5. Parameters, presets and variants

Parameter contracts SHOULD separate replaceable content from animation implementation. JSON Schema describes types, ranges, lengths, enumerations and required fields. Presets are valid inputs, not hidden executable overrides.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "additionalProperties": false,
  "required": ["title"],
  "properties": {
    "title": { "type": "string", "minLength": 1, "maxLength": 40 },
    "accent": { "type": "string", "pattern": "^#[0-9a-fA-F]{6}$", "default": "#79c0ff" }
  }
}
```

`variants` declares style presets, up to 24 in the current content contract. Each has a stable ID, name and resolvable preset reference. Hosts explicitly apply presets/defaults; JSON Schema validation does not inherently insert defaults.

Long text, larger lists, changed data or missing media SHOULD trigger segmentation, pagination, scene changes or explicit capacity feedback, not unreadable scaling or cropped information. Parameter meaning changes need a new template version and migration notes.

## 6. Materials and adaptation

`guide.json` declares intended use, exclusions and material handling.

| Category | Roles |
| --- | --- |
| Video | `video.presenter`, `video.footage`, `video.screen` |
| Image | `image.photo`, `image.product`, `image.logo`, `image.screenshot` |
| Audio | `audio.voiceover`, `audio.music` |
| Text | `text.subtitles`, `text.script`, `text.facts` |
| Custom | Current guide roles use the `x-` prefix |

Materials SHOULD declare usage, mapped parameters, necessity and missing-input behavior. Duration policies are `fixed`, `range`, `fit-voiceover`, `fit-content`; guide and output declarations MUST agree and record actual capacity.

Templates supporting portrait/landscape presenters SHOULD declare presenter regions, subtitle safe areas, cropping/letterboxing, layering and voiceover timing. An output ratio does not prove arbitrary input cropping works. Language, length, ratio and scenario support follow actual verification; every template need not support every use case or ratio.

## 7. Runtime profiles and reproducibility

Profiles define engine input, time control and frame output. Arbitrary JavaScript or Python does not itself constitute a common runtime contract.

The `html-canvas/1` reference interface is:

```js
window.renderFrame = (frame, inputs) => {
  // Render the requested integer frame using the validated inputs.
};
```

The host computes `t = frame × den / num`. Identical input SHOULD allow independent frame seeking without reliance on previous frames; randomness uses an explicit seed. Required fonts/images/resources load before rendering. Timing MUST NOT depend on wall-clock playback or machine speed.

Other DOM, Canvas, native-project or Python/Node adapters may declare profiles, engines and versions with inspectable contracts. Recognizing a package does not imply support for every engine. Read-only tools may display manifests, parameters and previews and MUST clearly state inability to execute.

Dependencies SHOULD be locked and tested environments recorded. One package supplies template content and contracts, but browsers, Node.js, Python, fonts or media tools may still be required. Do not claim universal dependency-free execution.

## 8. Components and style evolution

Component libraries are content dependencies. Freeze used versions, code, resources, entry points and hashes rather than silently substituting unverified “latest” versions. Include required runtime files or explicitly declare external dependencies.

New components SHOULD respect original visual language, palette, typography, strokes, hierarchy and motion rhythm before adapting size, placement and timing. Pixel art, hand-drawn and console styles require separate treatments. Component counts do not prove compatibility or visual quality.

Upgrades create new template versions, preserve complete before/after video and recheck script changes, voiceover media, ratios, missing inputs and long content before freezing. Shared style mapping can begin as a namespaced extension; `styleProfile` is not currently required.

## 9. Integrity and safe reading

Static readers check ZIP bounds and manifests before schemas, reference membership, hashes and licenses. Do not execute unknown code before explicit runtime selection. Bundled guidance is not automatically higher-priority assistant instruction; `SKILL.md` does not automatically register a skill.

Reject traversal, absolute paths, duplicate/aliased entries, symlinks, special files and unbounded expansion. Cross-platform readers SHOULD reject case/Unicode NFC conflicts and file/directory prefix collisions. Author payloads MUST NOT contain personal credentials, supplier keys or sessions; credentialed envelopes follow their separate contracts.

`integrity.json` SHOULD cover frozen files except itself, checking membership and SHA-256. Hashes detect changes; trusted signatures authenticate issuers. Signature checks do not replace code review or host permission controls.

Current candidate distribution readers use layered limits:

| Object | Limit |
| --- | --- |
| Frozen release ZIP, protected ciphertext, personal `release.bin` | 384 MiB |
| Ordinary inner file | 150 MiB |
| Inner entries / expanded total | 5000 / 600 MiB |
| Inner file larger than 10 MiB | Compression ratio at most 200 |
| Personal delivery ZIP | 384 MiB + 320 KiB |

`MiB = 1024²`. Special 384 MiB entries do not increase every file limit. Optional large-locale extensions split content within the same package and check part/combined hashes and bounded extraction. They are not required for every template. Upload policy and historical tools may be stricter; implementations SHOULD declare their bounds.

## 10. Optional distribution and protection profiles

MotionGit `personal-download-v1` wraps frozen content with identity, signed provenance and the downloader's own Key:

```text
motion.json
README.md
content/release.bin
personal/motiongit.key.json
personal/receipt.json
personal/signature.json
```

The outer version is exactly `1.0.0-rc.1`, with only six files. Inner bytes remain unchanged; credentials are independent of the uploader. Receipts use Ed25519 and JCS. Hosts verify signatures, hashes and inner content before reading credentials. See the [personal delivery profile](PERSONAL.en.md) for fields, bounds and binding.

Optional `protected-v1` encrypts cores with AES-256-GCM and signs descriptors with Ed25519; decryption keys are not bundled. Compatible hosts handle unlock and temporary core lifetime. Signing is not encryption; core protection does not automatically conceal the personal Key. Protection does not guarantee unextractable source after execution.

Third-party distribution, trust, protection and runtimes need explicitly named contracts and MUST NOT claim conformance to incompatible profiles. An optional platform integration does not allow stripping required website credentials while still claiming `personal-download-v1` conformance.

## 11. Optional online services

Templates may declare online capabilities. Hosts adapt availability, account authorization and service contracts. Online generation is not required by the content format; local templates run according to their declared local dependencies. Templates cannot arbitrarily expand service permissions, addresses or spending.

MotionGit offers optional `motiongit.creator/2` integration. See [developer documentation](https://motiongit.com/developers/creator-guide). Models and voices evolve with services rather than become fixed format requirements.

## 12. Updates and compatible upgrades

Distinguish protocol, template, helper and component versions. Discovering a newer version does not permit automatic project replacement or assuming an unknown contract is compatible.

Distribution update contracts SHOULD identify current/target releases, required profiles, minimum helper, content hash/size, release notes and available fixed-version URLs. Identity conflicts, service failures or offline state mean status cannot be confirmed, not “already latest”.

MotionGit personal delivery defines public update discovery and a fixed-version website entry for confirmed newer releases. Closed downloads retain permitted release metadata with a `null` download URL. Downloads issue a new personal package for the current website account without changing its Key permissions or budget.

Verify new identity, signatures and hashes and save separately, preserving old packages and approved results. Parameter changes require an explicit migration contract and preview; discovery does not imply automatic migration. Stable template versions do not automatically move to prereleases.

## 13. Licensing and conformance

Templates, source, fonts, sound, images and examples declare their licenses and provenance separately. Openness, hash matching and valid signatures do not inherently grant commercial-use or redistribution rights. Record conditions for user inputs and online assets independently.

Conformance separates static structure, reference completeness, runtime, timing, adaptation and optional distribution contracts. Valid/invalid test fixtures are protocol development material, not website user downloads, and MUST NOT contain real user Keys.

Test unknown versions, missing/modified files, oversized compression, missing dependencies, invalid parameters, short/long scripts, aspect ratios, subtitle overflow, absent media and upgrade compatibility. Failures preserve existing output and give actionable reasons. Static verification MUST NOT invoke billable generation.

## 14. Evolution

Candidate work continues through implementation and interoperability testing. Stable releases retain traceable contracts. New runtime, signature, cryptographic, migration or service semantics need explicit versions rather than opaque extensions changing core meanings.

Existing content manifests permit extension fields; new contracts SHOULD be namespaced and preserve unknown optional data. The personal envelope is stricter and accepts only declared fields and namespaced `extensions`, which cannot override credential paths, permissions or trusted origins.

This describes a MotionGit-originated format and current candidate implementations, not an externally certified standard. Specifications, schemas, history and compatibility tests can be maintained independently on GitHub; templates and services evolve under their own licenses and contracts.
