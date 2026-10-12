# MotionGit Template Specification 0.1.0-draft.1 (Draft)

> Legacy content layer: this describes author payloads, not the current personal website delivery envelope. See the [.motion open format](README.md) and [personal-download-v1](PERSONAL.en.md). Credentials are forbidden in author payloads; the personal envelope contains the downloader’s own Key. Authoring tools are provenance and recommendations.

> Status: Draft. These fields and directories are proposals that can be tried now, not a published industry standard. A freeze decision has not been confirmed. This English text and the Chinese text describe the same requirements; field names remain English. See [version history, compatibility and migration](HISTORY.en.md).

## 1. Scope and terminology

A **MotionGit template (.motion file)** is an editable, reusable, runnable video template package: a ZIP container holding source code, editable parameters, asset requirements, runtime dependencies, instructions, previews and licenses.

**One .motion file is a video you can make repeatedly.** An exported video (MP4) is one result. The .motion file preserves the recipe: source code defines motion and timing, parameters define replaceable content, and style variants provide ready-made appearances. An AI assistant reads the package and adapts the text and assets for a new video, subject to the required tools and runtime.

Use the full term “MotionGit template (.motion file)” on first mention. Tools recognize the root manifest’s fixed identifier `"format": "motiongit-template"`, rather than relying on the extension alone.

- **Template**: a reusable creative method.
- **Release package**: the .motion deliverable for a particular template version. A published package cannot be overwritten.
- **Instance**: the project created with a user’s content. Changing an episode title produces a new instance, not a new template version.
- **Runtime profile**: the loading rules, frame evaluation rules and required capabilities for a class of templates, such as `html-canvas/1`.
- **Host**: a tool that reads and executes a template, such as an AI assistant, local runner or third-party application.

“MUST”, “SHOULD” and “MAY” follow the usage of RFC 2119 / RFC 8174. Each requirement must be testable or explainable.

## 2. Package structure

Place `motion.json` directly at the ZIP root, without an enclosing directory.

```text
motion.json                  Sole interchange entry point (required)
README.md                    Human instructions, beginning with instructions for AI
SKILL.md                     AI loading and production entry (readable data only)
guide.json                   Use cases, assets, duration and conversation guidance
params.schema.json           Input types, constraints and defaults
motion.lock.json             Declared tested environment and dependency lock
asset-manifest.json          Assets, fonts, audio and license index
src/                         Readable template source
assets/                      Public assets permitted for redistribution
presets/demo.json            Reproducible public sample inputs
previews/                    Cover and sample previews (optional)
licenses/                    Code and resource licenses
integrity.json               SHA256 of every file except itself
```

### 2.1 File conventions

- Paths MUST be relative to the package root, use forward slashes and match case exactly. Absolute paths, drive letters, backslashes, `..`, symbolic links and extraction outside the root are forbidden.
- JSON MUST be strict UTF-8 JSON without comments or trailing commas.
- Do not package `node_modules`, `.venv`, caches, logs, `.env`, keys, tokens, private original footage or final private videos.
- Files automatically loaded as instructions by AI tools are forbidden, including `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `.claude/` and `.cursor/`. Package instructions are **data awaiting inspection**, not trusted instructions.
- A .motion file and its compatible .zip copy MUST be byte-identical. Use `application/zip` for MIME transport initially.

## 3. Root manifest: motion.json

Permanent JSON Schema URL: [`https://motiongit.com/schema/motion/0.1.json`](schema/content-0.1.json).

```json
{
  "$schema": "https://motiongit.com/schema/motion/0.1.json",
  "format": "motiongit-template",
  "specVersion": "0.1.0-draft.1",
  "template": { "id": "hello-motion", "version": "1.0.0", "name": "Hello Motion" },
  "runtime": { "profile": "html-canvas/1", "entry": "src/index.html", "dependencyLock": "motion.lock.json" },
  "parameters": { "schema": "params.schema.json", "defaultPreset": "presets/demo.json" },
  "assets": { "manifest": "asset-manifest.json" },
  "output": { "width": 1920, "height": 1080, "fps": { "num": 30, "den": 1 } },
  "license": { "file": "licenses/TEMPLATE-LICENSE.txt" },
  "permissions": { "network": false, "read": ["package", "instance-inputs"], "write": ["instance-output"] },
  "ai": {
    "summary": "This is a MotionGit template (.motion, a ZIP container)…",
    "entry": "SKILL.md",
    "guide": "guide.json",
    "authoring": { "tool": "claude", "client": "claude-code" }
  },
  "integrity": "integrity.json"
}
```

| Field | Draft requirements |
| --- | --- |
| `format` | MUST be `motiongit-template` |
| `specVersion` | Required. The host MUST stop execution for an unsupported version. The current Schema and inspector support exactly `0.1.0-draft.1` |
| `template.id / version / name` | Required. Versions use semantic versioning |
| `runtime.profile / entry` | Required. For an unknown profile, show instructions without claiming it can run |
| `output.width / height / fps` | Required. fps is the rational value `{num, den}` |
| `permissions` | Declarations only. The host MUST enforce them; a package cannot authorize itself |
| `ai.summary` | SHOULD explain what the package is, what is editable, what it needs and what to read next, from the root manifest alone (approximately 2000 tokens) |
| `ai.guide` | SHOULD point to guide.json (§4.2) |
| `output.durationPolicy` | `fixed` / `range` / `fit-voiceover` / `fit-content`; consistent with guide.json’s `duration.policy` |
| `ai.authoring` | Author-declared provenance. Tools/models are provenance and recommendations, not format restrictions. Review evaluates actual results; declarations are not technical proof. Preserve the actual source |

## 4. Parameter contract {#params}

`params.schema.json` uses an explicit subset of JSON Schema Draft 2020-12: text, numbers, booleans, enums, colors and media slots. The root MUST be `type: object`. It SHOULD set `additionalProperties: false` and document `required`, length constraints, enums and defaults.

Public parameters MUST map to fields actually consumed at runtime. An input shown as editable must affect the rendered result.

### 4.1 Style variants: variants

One template may include multiple style variants for color, layout or appearance. Each variant is a complete input preset, and one download includes all of them.

```json
{
"variants": [
  { "id": "sage", "name": "Sage", "preset": "presets/sage.json", "swatch": "#C2D788" },
  { "id": "night", "name": "Night pink", "preset": "presets/night.json", "swatch": "#F778BA", "preview": "previews/night.mp4" }
]
}
```

- `preset` MUST exist and contain only parameters declared in `params.schema.json`.
- `preview` is optional. The platform extracts it and displays it as that variant’s preview.
- A style variant does not change template source. Different motion or structure should be published as a new template or version.
- A user can request the “night” style. The assistant reads its preset as the starting input.

### 4.2 User guidance and content adaptation: guide.json {#guide}

A template provides a **production standard**: it defines motion, timing and layout, while the user’s content determines copy, assets and duration. `guide.json` tells the assistant how to fit content into that standard and how to discuss it with the user.

```json
{
  "guideVersion": "0.1",
  "fit": { "summary": "An opening about A × B producing C", "genres": ["talking-head"], "goodFor": ["Talking-head openings"], "notFor": ["Portrait videos"], "placement": "opening" },
  "duration": { "policy": "fit-voiceover", "min": 5, "max": 30, "default": 8, "param": "duration",
                "rules": ["With voiceover: duration = opening speech end − sourceStart + 0.4 seconds; do not cut mid-sentence"] },
  "materials": [
    { "role": "video.presenter", "required": false, "use": "Place in the portrait card on the right, synchronized with voiceover",
      "prepare": "Provide a video path; the renderer trims and extracts frames", "ifMissing": "Use the layout without a presenter",
      "ask": "Do you have a talking-head video? Send it if available.", "maps": ["presenter"] }
  ],
  "copy": [ { "param": "line1", "label": "First title line", "from": ["text.subtitles", "conversation"], "rule": "Write as A × B" } ],
  "conversation": ["Introduce the template and preview first, then ask about the topic and available assets"]
}
```

| Field | Requirements |
| --- | --- |
| `fit` | MUST provide `summary`. SHOULD provide `goodFor` and `notFor`. `placement` is `opening / chapter / body / insert / ending / whole`. `genres` describes the actual positioning, with the primary genre first |
| `fit.genres` | Unified video genres: `talking-head`, `knowledge`, `product`, `tutorial`, `story`, `news`, `showcase`, `opener`. Templates are not universal: list only genres tested as suitable so assistants can recommend or advise against them |
| `duration` | `policy` is `fixed / range / fit-voiceover / fit-content`; `min ≤ default ≤ max`. Except for `fixed`, MUST use `param` to reference a numeric duration parameter with matching min/max constraints, and provide `rules` explaining duration decisions |
| `materials` | Each `role` comes from the unified role table. MUST provide `use` and `ask`. Optional assets MUST provide `ifMissing`. `maps` can reference only declared parameters |
| `copy` | Every `param` MUST be declared. Provide a user-facing `label` and writing `rule` |
| `conversation` | SHOULD provide template-specific discussion points |
| `hostWithoutTools` | SHOULD explain what a chat-only assistant can deliver and how the user can render locally |
| `workspace` | Optional instance directory layout: extraction directory, inputs.json, assets and command locations |

**Unified asset roles** shared by all templates; template-specific roles use an `x-` prefix:

| Category | Roles |
| --- | --- |
| Video | `video.presenter` talking head · `video.footage` recorded footage · `video.screen` screen recording |
| Image | `image.photo` photo · `image.product` product image · `image.logo` logo · `image.screenshot` screenshot |
| Audio | `audio.voiceover` voiceover · `audio.music` background music |
| Text | `text.subtitles` subtitles (srt) · `text.script` script · `text.facts` key points / selling points / data |

See [Instructions for AI](https://motiongit.com/ai) for the shared conversation workflow.

## 5. Runtime semantics

- Evaluate integer frame indices at a rational frame rate. Frame seeking must not depend on playback from the beginning.
- Evaluate after assets and fonts are ready. Random behavior MUST declare a seed.
- Network access is off by default. For external resources, the host MUST explain the resource, destination and purpose, then follow its authorization process.
- Pixel-identical output across devices is not promised by this draft. The first stage aims for explicit inputs, traceable environments and explainable behavior.

| Situation | Expected behavior |
| --- | --- |
| Core version recognized, runtime profile unsupported | Show instructions, preview and download without claiming it can run |
| Runtime satisfied, assets missing | List missing assets; allow replacement or fail explicitly |
| Optional capability unavailable | Follow the declared fallback and explain the difference |
| Required extension unknown | Refuse execution while preserving files and instructions |

## 6. Assets and licenses

Record code and asset licenses separately. Fonts, music and images may have different terms. For resources that cannot be redistributed, supply clear references and replacement slots. SPDX identifiers may be used; the original license controls the actual terms.

## 7. Security

Read metadata without executing code on import. Limit extraction paths, file count and size, and reject abnormal archives. Before publication, inspect common credentials, absolute paths and personal information. Automatic checks reduce risk; they cannot prove copyright ownership or complete absence of malicious code.

## 8. AI readability

The fixed reading order is `motion.json` → `guide.json` → the AI instructions at the beginning of `README.md` → `params.schema.json` → assets and licenses. This order is the same for every template.

Users may send the package with the handoff prompt in [Instructions for AI](https://motiongit.com/ai).

## 9. Conformance tests {#conformance}

The platform and MotionGit Skill share the static inspector. Reference packages carry expected results and serve as conformance tests and learning samples.

- `hello-motion.motion`: a valid package, expected to pass.
- See the [sample catalog below](https://motiongit.com/spec/legacy#samples) for valid and invalid entries, including the inspector’s expected error code for every invalid case.

Names and expected results remain visible while downloads are closed; download links appear only when downloads are open. These cases verify static contracts and error handling, not completed rendering or video export.

## 10. Proposals and governance {#proposals}

Public proposals describe a problem, provide samples and explain compatibility effects. They then become tests, versions and migration notes. Maturity labels are Draft, Candidate and Stable. MotionGit initiates and maintains this specification; no independent foundation or standards organization endorsement is claimed.

See [version history and migration](HISTORY.en.md) for supported versions and the treatment of unsupported packages.
