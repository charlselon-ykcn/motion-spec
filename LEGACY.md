# MotionGit 模板规范 0.1.0-draft.1（Draft 草案）

> 兼容内容层说明：此页描述既有作者载荷，不是当前官网用户个人下载包的外层协议。最新版见 [.motion 开放格式说明](SPEC.zh.md)；个人交付见 [personal-download-v1](PERSONAL.md)。作者载荷禁止凭据，个人交付外层按独立合同包含下载者自己的 Key。制作工具只作来源与建议。

> 状态：Draft。字段和目录是可立即试做的提议，不是已发布的行业标准。冻结决定尚未确认，继续保持 Draft。中英文正文描述相同要求，字段名只用英文。参见[版本历史、兼容规则与迁移](HISTORY.md)。

## 1. 范围与术语

**MotionGit 模板（.motion 文件）** 是一种可编辑、可复用、可运行的视频模板包：用 ZIP 容器装入源码、可改参数、素材要求、运行依赖、使用说明、预览和许可。

一句话理解：**一个 .motion 文件，就是一条可以反复制作的视频。** 成片（MP4）只是某一次的结果；.motion 保存的是做出这条视频的配方——源码负责动作与节奏，参数负责可替换的内容，样式分支提供几种现成的风格。AI 助手读取它，换上你的文字和素材，就能做出属于你的新视频。

文档与引导文字统一使用“MotionGit 模板（.motion 文件）”的完整称呼，工具通过根清单中的固定标识 `"format": "motiongit-template"` 识别，而不是只看扩展名。

- **模板**：可以反复使用的创作方法。
- **发布包**：某个模板版本的交付物（.motion）。发布后不可覆盖。
- **制作实例**：使用者换上自己的内容后得到的项目。改本期标题只产生新实例，不产生模板新版本。
- **运行配置**（profile）：某一类模板如何加载、如何按帧求值、需要哪些能力，例如 `html-canvas/1`。
- **宿主**：读取并执行模板的工具，如 AI 助手、本地运行器或第三方软件。

规范中的“必须”“应当”“可以”参考 RFC 2119 / RFC 8174 的用法，每条都对应可测试或可解释的要求。

## 2. 包结构

ZIP 根目录直接放 `motion.json`，不再套一层同名目录。

```text
motion.json                  交换层唯一入口（必须）
README.md                    人能直接照做的说明，开头包含“给 AI 的说明”
SKILL.md                     AI 加载与制作入口（只是可读文件）
guide.json                   用户引导与内容适配：适合什么视频、素材怎么用、时长怎么定、怎么对话
params.schema.json           输入类型、约束与默认值
motion.lock.json             实测运行环境与依赖锁
asset-manifest.json          素材、字体、声音与许可索引
src/                         可读的模板源码
assets/                      获准随包分发的公共素材
presets/demo.json            可重现的公开样例输入
previews/                    封面与样例预览（可选）
licenses/                    代码及各资源许可
integrity.json               除自身外所有文件的 SHA256
```

### 2.1 文件约定

- 路径必须是相对于包根的正斜杠路径，大小写严格匹配。禁止绝对路径、盘符、反斜杠、`..`、符号链接和解包越界。
- JSON 必须是 UTF-8 严格 JSON，不含注释或尾逗号。
- 禁止打包：`node_modules`、`.venv`、缓存、日志、`.env`、密钥、令牌、私人原片和最终私人影片。
- 禁止包含会被 AI 工具自动加载为指令的文件：`CLAUDE.md`、`AGENTS.md`、`.cursorrules`、`.claude/`、`.cursor/` 等。包内说明属于**待检查数据**，不能当作可信指令。
- `.motion` 与兼容 `.zip` 副本必须字节相同。MIME 传输先按 `application/zip` 处理。

## 3. 根清单 motion.json

JSON Schema 永久网址：[`https://motiongit.com/schema/motion/0.1.json`](schema/content-0.1.json)

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
    "summary": "这是 MotionGit 模板（.motion，ZIP 容器）……",
    "entry": "SKILL.md",
    "guide": "guide.json",
    "authoring": { "tool": "claude", "client": "claude-code" }
  },
  "integrity": "integrity.json"
}
```

| 字段 | v0.1 要求 |
| --- | --- |
| `format` | 必须为 `motiongit-template` |
| `specVersion` | 必须。当前 Schema 与检查器精确支持 `0.1.0-draft.1`；宿主不支持时必须停止执行 |
| `template.id / version / name` | 必须。版本使用语义化版本 |
| `runtime.profile / entry` | 必须。不认识的 profile 只展示说明，不假装可运行 |
| `output.width / height / fps` | 必须。fps 用有理数 `{num, den}` |
| `permissions` | 只是声明，必须由宿主强制执行，不能作为包自行授权的依据 |
| `ai.summary` | 应当提供。只读根清单（约 2000 token 内）就能回答：这是什么、能改什么、需要什么、下一步读哪个文件 |
| `ai.guide` | 应当提供。指向 guide.json（见 §4.2） |
| `output.durationPolicy` | `fixed` / `range` / `fit-voiceover` / `fit-content`，与 guide.json 的 `duration.policy` 一致 |
| `ai.authoring` | 制作来源声明。工具与模型是来源信息和制作建议，格式不限定品牌；平台以实际效果审核为准，声明不构成技术证明 |

## 4. 参数契约 {#params}

`params.schema.json` 采用 JSON Schema Draft 2020-12 的明确子集：文字、数字、布尔值、枚举、颜色和媒体槽位。顶层必须是 `type: object`，应当设置 `additionalProperties: false`，写清 `required`、长度、枚举和默认值。

公开参数必须映射到运行时真实消费的字段，不能出现“参数面板能改、渲染不生效”。

### 4.1 样式分支 `variants`

同一个模板可以带多种样式分支（配色、版式、风格）。每个分支是一份完整的输入预设，一次下载全部包含：

```json
{
"variants": [
  { "id": "sage",  "name": "鼠尾草", "preset": "presets/sage.json",  "swatch": "#C2D788" },
  { "id": "night", "name": "夜粉",   "preset": "presets/night.json", "swatch": "#F778BA", "preview": "previews/night.mp4" }
]
}
```

- `preset` 必须存在，且只能包含 `params.schema.json` 声明过的参数。
- `preview` 可选，平台会从包内提取并展示为该分支的预览。
- 样式分支不改变模板源码；需要不同动作或结构时，应作为新模板或新版本发布。
- 用户可以告诉助手“使用 night 样式”，助手读取对应预设作为本次输入的起点。

### 4.2 用户引导与内容适配 `guide.json` {#guide}

模板提供的是一套**出片标准**：动作、节奏、版式由模板决定；文案、素材和时长由用户的内容决定。`guide.json` 告诉助手怎么把用户的内容装进这套标准，以及怎么和用户对话。

```json
{
  "guideVersion": "0.1",
  "fit": { "summary": "视频开头的「A × B 做成了 C」", "genres": ["talking-head"], "goodFor": ["口播开场"], "notFor": ["竖屏视频"], "placement": "opening" },
  "duration": { "policy": "fit-voiceover", "min": 5, "max": 30, "default": 8, "param": "duration",
                "rules": ["有配音时：时长 = 开场白结束时间 − sourceStart + 0.4 秒，不在半句话处截断"] },
  "materials": [
    { "role": "video.presenter", "required": false, "use": "放进右侧竖卡，和配音同步",
      "prepare": "直接填视频路径，渲染器自动截取并拆帧", "ifMissing": "改用无人物版式",
      "ask": "有口播视频吗？有的话直接发我。", "maps": ["presenter"] }
  ],
  "copy": [ { "param": "line1", "label": "第一行标题", "from": ["text.subtitles", "conversation"], "rule": "写成「A × B」" } ],
  "conversation": ["先介绍模板并给出预览，再问这条视频讲什么、手上有什么素材"]
}
```

| 字段 | 要求 |
| --- | --- |
| `fit` | 必须有 `summary`；应当写 `goodFor` 与 `notFor`。`placement` 取 `opening / chapter / body / insert / ending / whole`。`genres` 写模板的真实定位（第一项为主定位）：`talking-head` 真人口播、`knowledge` 知识讲解、`product` 产品介绍/品牌、`tutorial` 教程录屏、`story` 叙事、`news` 资讯、`showcase` 作品展示、`opener` 片头片尾插件。模板不是通用的：只写实测擅长的类型，助手据此推荐或劝退 |
| `duration` | `policy` 取 `fixed / range / fit-voiceover / fit-content`；`min ≤ default ≤ max`。非 `fixed` 时必须用 `param` 指向控制时长的数字参数（范围与 min / max 一致），并写 `rules` 说明助手怎么决定时长 |
| `materials` | 每项 `role` 来自统一角色表；必须写 `use` 与 `ask`；可选素材必须写 `ifMissing`；`maps` 只能引用已声明的参数 |
| `copy` | 每项 `param` 必须已声明，写 `label`（给用户看的名字）与 `rule`（怎么写） |
| `conversation` | 应当提供。本模板的对话要点 |
| `hostWithoutTools` | 应当提供。只能聊天、不能运行命令的助手该交付什么，用户在自己电脑上怎么出片 |
| `workspace` | 可选。制作实例目录的布局（解包目录、inputs.json 与素材的位置、运行命令的位置） |

**统一素材角色**（所有模板共用；模板特有素材用 `x-` 前缀）：

| 类别 | 角色 |
| --- | --- |
| 视频 | `video.presenter` 口播 · `video.footage` 拍摄素材 · `video.screen` 录屏 |
| 图片 | `image.photo` 照片 · `image.product` 产品图 · `image.logo` Logo · `image.screenshot` 截图 |
| 声音 | `audio.voiceover` 配音 · `audio.music` 背景音乐 |
| 文字 | `text.subtitles` 字幕（srt）· `text.script` 文稿 · `text.facts` 要点 / 卖点 / 数据 |

助手的通用对话流程见 [给 AI 的说明](https://motiongit.com/ai)。

## 5. 运行语义

- 按整数帧索引与有理数帧率求值；寻帧不依赖先从头播放。
- 素材和字体就绪后再求值；随机行为必须声明种子。
- 默认不联网。需要外部资源时，宿主必须说明具体资源、去向和用途，并按宿主授权流程处理。
- 跨设备逐像素一致不是 v0.1 的承诺。第一阶段追求输入明确、环境可追踪、行为可解释。

| 情况 | 合理行为 |
| --- | --- |
| 核心版本可识别，配置规范不支持 | 展示说明、预览和下载，不假装可运行 |
| 运行环境满足，资源缺失 | 列出缺口，允许替换或明确失败 |
| 可选能力不可用 | 使用声明的降级路径并显示差异 |
| 不认识的必需扩展 | 拒绝执行，保留文件与说明 |

## 6. 素材与许可

代码许可与素材许可分别记录。字体、音乐、图片可能有不同条件；不能随包分发的资源，提供清晰的引用与替换槽位。许可标识可借鉴 SPDX，具体条款以原许可为准。

## 7. 安全

导入时先读元数据，不执行代码。解压限制路径、文件数量与体积，拒绝异常压缩包。发布前检查常见凭据、绝对路径和个人信息。自动检查能降低风险，但不能宣称已证明版权或彻底消除恶意代码。

## 8. AI 可读性

助手的固定阅读顺序：`motion.json` → `guide.json` → `README.md` 开头的“给 AI 的说明” → `params.schema.json` → 资源与许可。所有模板顺序相同。

用户可以连同文件发送一段固定的转交文字，见 [给 AI 的说明](https://motiongit.com/ai)。

## 9. 一致性测试 {#conformance}

平台与 MotionGit Skill 使用同一套静态检查器。下面的样例包附带预期结果，同时作为一致性测试与模型学习样例：

- `hello-motion.motion`：有效包，预期通过。
- 有效与无效条目统一见下方[样例包清单](https://motiongit.com/spec/legacy#samples)，每个无效条目列出真实检查器的预期错误码。

下载未开放时仍可查看名称与预期结果；获取入口只在下载开放时显示。这套清单验证静态契约与错误处理，不代表模板已完成渲染或视频导出验收。

## 10. 提案与治理 {#proposals}

公开提案说明问题、附上样例和兼容性影响，再形成测试、版本与迁移说明。成熟度用 Draft、Candidate、Stable 表达。MotionGit 发起并维护本规范，不宣称存在独立基金会或标准组织背书。
