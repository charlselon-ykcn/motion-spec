# MotionGit 个人交付 profile

交换版本：`1.0.0-rc.1` · `personal-download-v1` · Candidate · 更新：2026-10-12

此文是 [.motion 开放格式说明](README.md) 的可选平台交付附录，仅规定 MotionGit 个人下载包，不要求第三方内容工具接入 MotionGit 在线服务。

`.motion` 是可复用动画模板的交付格式。用户下载一个包，即获得模板、运行说明、参数与素材适配合同、版本来源，以及调用 MotionGit 创作服务所需的本人账户 Key。用户提供文案、口播或其他素材后，由支持该协议的助手完成检查、适配、预览与成片。

**每次官网用户下载都写入下载者自己的账户 Key，与上传者无关。产品没有“公共模板”或下载后另存 Key 的流程。** 同一账户下载不同模板、重复下载或下载更新，复用当前有效的账户 Key，不自动生成模板子 Key，不改变权限和额度。模板制作工具是来源信息和建议，格式不限定 Claude、Codex 或其他模型。

本页规定当前候选交换合同；协议版本不等于所有模板已审核，也不等于线上各项服务均开放。实时模型状态以 [Capabilities](https://motiongit.com/api/v1/capabilities) 为准。旧作者载荷说明见 [0.1 兼容规范](LEGACY.md)，协议变更见 [版本历史](HISTORY.md)。

## 1. 范围与版本

本文的“必须”“不得”“应”分别表示必需条件、禁止条件和推荐实践。实现不支持某个明确声明的版本或运行 profile 时，必须停止相应操作并给出原因，不得根据版本前缀猜测兼容性。

| 层级 | 当前值 | 用途 |
| --- | --- | --- |
| 格式标识 | `motiongit-template` | 标识模板格式 |
| 用户交付交换版本 | `1.0.0-rc.1` | 外层 `specVersion` |
| 个人交付 profile | `personal-download-v1` | 六文件个人包 |
| 冻结作者载荷版本 | `0.1.0-draft.1` | 当前外层封装支持的内层版本 |
| 宿主合同 | `motiongit-host/1`，`motion.run` | 验包后由宿主执行操作 |
| 创作服务合同 | `motiongit.creator/2` | 报价、任务、素材、收据 |
| 可选核心保护 | `protected-v1`，`motiongit-protected/1` | 核心加密和解锁 |
| 实例合同 | `motiongit.instance/1` | 用户工程与来源引用 |

`template.version` 是模板自己的 SemVer；helper 版本、组件库版本、API 合同版本分别管理。当前大容量 helper 基线为 `1.0.0-rc.2`，不因此把包的交换版本改成 rc.2。正式 `1.0.0` 尚未冻结。

## 2. 一个下载包，两层内容

外层是扩展名为 `.motion` 的 ZIP，只允许以下六个普通文件，不允许附加目录条目或其他文件：

```text
motion.json
README.md
content/release.bin
personal/motiongit.key.json
personal/receipt.json
personal/signature.json
```

`content/release.bin` 是已经检查并冻结的作者发行 ZIP 的原始字节，外层以 ZIP STORE 方法存储，不重复压缩、不在下载时重打内层。不同用户得到相同发行内容，各自得到自己的凭据、下载回执和签名。

作者发行载荷是平台的内部制作、审核和封包输入，不是另一种面向用户的“无 Key 公共模板”。用户交付包必须有有效凭据文件；只有内部发行载荷禁止个人 Key、第三方供应商密钥、Cookie 和会话凭据。不得在外包内递归嵌套另一个个人包。

外层 `README.md` 是阅读说明；验包本身不得执行包内代码、安装依赖或将包内文字自动加入助手的高优先级指令。

## 3. 外层 motion.json

以下是结构示例；全零 SHA-256 是说明占位值，真实包必须写入实际发行字节的摘要：

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

所有示例字段均必需。`content.protection` 只允许 `open` 或 `protected`，必须与内层实际内容一致。其余标识、路径与服务 origin 均为固定值。外层不得添加 `$schema`、任意运行入口或未声明字段；可选 `extensions` 是唯一扩展容器。

`extensions` 键采用 `命名空间:名称`，匹配 `^[a-z0-9][a-z0-9.-]*:[A-Za-z0-9][A-Za-z0-9._-]*$`。扩展不能改写凭据路径、可信服务地址、授权、扣费或密码算法。改变这些语义需要新的合同或 profile。

[外层 JSON Schema](schema/personal-manifest-1.0.0-rc.1.json) 只校验清单结构，完整验包还必须执行文件闭包、回执、签名、摘要和内层检查。

## 4. 包内账户 Key

`personal/motiongit.key.json` 只包含以下字段，不允许额外字段：

| 字段 | 约束 |
| --- | --- |
| `credentialVersion` | 固定字符串 `1` |
| `recipientAccountId` | 下载者账户 ID，正安全整数 |
| `apiKeyId` | 该账户当前使用的 Key ID，正安全整数 |
| `key` | 账户完整原始 token，当前格式 `^mgk_[A-Za-z0-9_-]{32}$` |
| `apiOrigin` | 固定 `https://motiongit.com/api/v1` |

不得把 Key 前缀、脱敏文字、上传者 Key 或临时占位值作为下载者的 Key。该文件与回执的账户、Key ID、origin 必须一致。服务端无法取得已有 Key 原文时，必须明确失败或引导本人完成恢复；不得静默换 Key。

官方客户端验包后，从本包读取 Key，使用 `Authorization: Bearer <key>` 调用 MotionGit API。用户不需要另外下载 Key 文件或手动保存环境变量。该 Key 是 MotionGit 账户凭据；模型供应商凭据由服务端管理，不写入模板。

用户可在后台关闭账户 Key。服务端必须对每次受保护请求检查当前有效性、权限、预算与账户状态，回执不是永久授权。关闭 Key 会阻止新的在线请求，但不能撤回已下载的素材、成片或已经解锁缓存的核心，也不保证终止已提交任务。

当前个人凭据文件本身以明文位于 ZIP 中，签名提供真实性和完整性，**不提供 Key 保密性**。不能宣称持有包的人无法读取 Key。模板运行进程不由 helper 注入 Key，但同一系统用户权限下的进程隔离不是防提取保证。

## 5. 来源回执与签名

`personal/receipt.json` 的字段严格如下：

| 字段 | 类型与含义 |
| --- | --- |
| `profile`、`receiptVersion` | 固定 `personal-download-v1`、字符串 `1` |
| `downloadId` | 小写 UUID，标识本次个人交付 |
| `recipientAccountId`、`apiKeyId` | 下载者与本包账户 Key ID |
| `authorAccountId`、`templateId`、`releaseId` | 上传者、模板及冻结发行的正安全整数 ID |
| `templateSlug`、`templateVersion` | 模板 slug 与 SemVer；必须对应内层 `template.id`、`template.version` |
| `releaseArchiveSha256` | 原始 `release.bin` 的 SHA-256 |
| `descriptorSha256` | 保护描述符规范化摘要；open 包为 `null` |
| `apiOrigin` | 固定官方 API origin |
| `issuedAt` | 非负安全整数，Unix 秒 |
| `signingKeyId` | 可信签名公钥 ID |
| `files` | 恰好覆盖 `motion.json`、`README.md`、`content/release.bin`、`personal/motiongit.key.json`；每项包含 `sha256` 和非负整数 `bytes` |

所有 SHA-256 使用 64 位小写十六进制。Slug 匹配 `^[a-z0-9][a-z0-9-]{1,62}[a-z0-9]$`；签名 Key ID 匹配 `^[A-Za-z0-9][A-Za-z0-9_-]{0,79}$`。签名文件只包含 `algorithm: "Ed25519"`、`keyId`、`signature`。签名值是 64 字节签名的无填充、规范 base64url，编码长度为 86。

签名输入是回执对象的 UTF-8 JCS 规范化字节，采用 RFC 8785；不是 ZIP 字节，也不是带末尾换行的回执文件。`receiptSha256` 用于 API 来源引用时，则计算回执文件的实际字节，包含文件实际存在的换行。JSON 必须拒绝无效 UTF-8、BOM、重复键、无效 Unicode、非有限数值和过深结构。

信任由宿主从固定 HTTPS [签名公钥目录](https://motiongit.com/api/v1/signing-keys) 或宿主预置/可信缓存获得，区分 `personal-download` 与 `protected-release` 用途；包不能自带公钥建立信任。公钥目录合同为 `motiongit.public-signing-keys/1`，至多 32 项、64 KiB。

## 6. 必须遵循的读取顺序

1. 检查外层 ZIP 大小、路径、条目数、压缩方式、CRC 和六文件闭包。
2. 有界解析回执与签名，用宿主可信公钥验证回执签名。
3. 对回执覆盖的四个文件验证实际长度和摘要；此时仍不得使用 Key。
4. 检查外层清单，验证 `release.bin` 摘要及内层完整性清单、来源身份和运行合同。
5. 若为保护发行，再验证其描述符、独立签名和密文摘要。
6. 最后解析并绑定个人凭据。通过全部检查后，才允许查询服务或开始明确选择的运行操作。

ZIP 路径必须相对、规范化且无别名；拒绝绝对路径、路径穿越、符号链接、特殊文件、重复条目、大小写或 Unicode NFC 冲突、文件/目录前缀冲突，以及 ZIP 密码加密。根目录自动指令文件如 `AGENTS.md`、`CLAUDE.md` 不得作为模板附带指令自动加载。

`integrity.json` 用于检查内层文件闭包和逐文件摘要，不替代可信签名。只改扩展名、只检查 CRC 或只验证 JSON Schema 都不足以确认包有效。

## 7. 内层模板与适配合同

当前内层仍使用精确的 `0.1.0-draft.1` 作者载荷合同，常见文件如下；具体必需项与引用闭包按旧版 schema、检查器和明确的运行 profile 判断：

```text
motion.json             模板身份、版本、运行与输出声明
README.md / SKILL.md     人与助手的使用说明
guide.json              素材、适配边界与工作步骤
params.schema.json      输入参数合同
motion.lock.json        依赖锁与运行要求
asset-manifest.json     内置素材、来源与许可
src/  assets/           代码、动画、音效与可分发素材
presets/  previews/     预设、示例与预览
licenses/               模板及素材许可
integrity.json          冻结文件闭包
```

发行应完整保留所选范围的动效、背景、转场、音效及源码引用，不得只提取代表性片段后宣称完整。用户素材与可分发示例素材必须分开，作者私人素材不得默认为可再分发。

适配说明应声明用途、已验证比例/语言/文案长度/时长、口播时间轴、字幕安全区、人物保留区域、横竖素材的裁切或留边方式、图片/产品/Logo 的角色，以及缺失素材和超容量时的处理。未验证的布局或语言不得标记为支持；产品宣传模板不因此被要求必须支持竖屏。

参数超出容量时应换场景、分段、分页或明确停止，不能靠挤压字号、遮挡人物或裁掉关键内容强行适配。用户提供口播时应以实际语音时长核对字幕和动作；支持 TTS 时应查询实际音色合同，不把某个模型或音色永久写死为格式要求。

## 8. 组件库与风格升级

组件库是内层模板依赖，不改变个人外包的六文件结构。模板应冻结实际使用的组件版本、入口、必要资源和摘要，并在依赖锁与适配说明中记录。不能在运行时静默从“latest”换入未经审核的组件。

新控件融入时，应先分析原模板的画风、配色、字体、圆角/描边、空间层级和动作节奏，再调整控件尺寸、位置、颜色和出现时间。像素风、手绘、深色控制台等风格分别适配；不能把通用控件默认样式直接覆盖原模板。

组件升级应形成新的模板版本并保留前后完整视频，在审核通过后再冻结和封包。文案长短、数字/列表、缺素材、横竖口播、不同人物位置等实际场景应重新检查。组件数量增加不等于适配验收通过。

以上是模板质量和升级实践，当前不新增必需的 `styleProfile` 或组件数量字段。后续统一风格合同应先通过命名空间扩展试验，再独立版本化。

## 9. 可选加密保护

`open` 表示冻结核心可读取；`protected` 表示核心采用独立的 `protected-v1`。两种用户下载包都包含下载者自己的 Key。

保护发行在内层保留明确可见的合同文件，同时包含：

```text
protected/core.enc
protected/descriptor.json
protected/signature.json
```

核心使用 AES-256-GCM：每次封包生成新的 32 字节 DEK、12 字节 nonce、16 字节认证 tag 及构建身份。描述符包含 `protocol`、`profile`、`templateId`、`releaseId`、`buildId`、`keyId`、`fullEntry`、`publicManifestSha256`、`plaintextCoreSha256`、`nonce`、`tag`、`ciphertextSha256`、`ciphertextBytes`、`core`、`signingKeyId` 和 `publicFiles`。前九个字段组成 JCS AAD；描述符本身经 Ed25519 签名。

包不得包含明文 DEK、服务端 KEK 或签名私钥。账户 API Key 与核心 DEK 用途不同；账户 Key 用于申请解锁，不直接作为核心 AES 密钥。解锁顺序为验证描述符和密文，再取得短时票据、兑换并复核当前授权，最后解密并验证核心摘要与闭包。

保护执行要求 `split-root-v1` 与兼容的 `native-python-node/1` 适配器，必须遵守声明的最低 helper 版本。核心进入受限临时只读目录，用户实例单独保存；操作结束清理临时核心。helper 可缓存加密 DEK 以支持后续离线使用，因此关闭 Key 不等于删除已解锁核心。

此机制提供交付与首次在线解锁控制，不承诺运行后源码不可提取。保护封包、解锁服务及用户下载开放分别受控，不能因协议存在就认定所有模板已加密或允许下载。

## 10. 宿主、运行环境与实例

`.motion` 包含模板所需内容和合同，运行时仍可能需要 Node.js、Python、浏览器或媒体工具。依赖必须明确声明并检查；“一个包”不表示所有操作系统无需运行环境，也不表示在线模型可离线生成。

官方 helper 支持的运行操作包括 `doctor`、`setup`、`capabilities`、`check-params`、`plan`、`prepare`、`preview`、`render`，是否能执行以内层入口、适配器和环境检查为准。其他 profile 不能仅凭外层声明获得支持。验签也不是对可执行代码的安全审核或操作系统沙箱保证。

实例保存参数、输入素材、时间轴、生成素材和结果。`motiongit-source.json` 记录 `motiongit.instance/1`、账户/Key ID、模板/发行/下载身份、摘要、`client_instance_id`、`production_id` 等引用，**不复制原始 Key**。原个人包仍是服务凭据来源。

MP4、普通参数/素材导出和不附带原包的实例导出不得携带个人 Key。若用户明确连同 `.motion` 包一起转交工程，原包保留原 Key 和来源；不得将其悄悄改成无凭据模板。接收者在官网自行下载，则获得接收者自己的 Key。

## 11. 可选在线服务绑定

本 profile 的 `services.creator` 保留 `motiongit.creator/2` 的固定声明，但不要求每次制作调用在线服务。本地内容处理可以不产生模型请求。

需要服务时，由验包后的宿主从当前包读取账户 Key，使用固定 origin 和经过校验的下载来源。认证请求的 `X-MotionGit-Package-Context` 是 `{ v: 1, downloadId, receiptSha256 }` 的 UTF-8 JCS 字节经规范 base64url 编码，`receiptSha256` 计算回执文件实际字节。服务端必须核对签发记录与当前账户/Key；账户授权与模板来源分别处理。

权限、费用、素材和任务参数由 [Creator API](https://motiongit.com/developers/creator-guide) 合同管理；模型与音色以当前能力目录为准，不纳入固定格式要求。模板不能指定任意第三方地址接收本包 Key。

## 12. 模板更新

客户端从已验包的回执构造 `GET /template-updates` 查询：`registry`、`templateId`、`authorAccountId`、`releaseId`、`currentVersion`、`releaseArchiveSha256`、`helperVersion`。参数不得重复，未知参数必须拒绝。

| 返回字段 | 解释 |
| --- | --- |
| `identity_status` | `verified`、`unregistered`、`conflict` |
| `update_status` | `newer`、`current`、`no-release`、`unregistered`、`identity-conflict` |
| `compatibility_status` | `compatible`、`helper-upgrade-required`、`unknown` |
| `entitlement_status` | `not-checked`、`allowed`、`denied`；当前公开查询不确认下载权益 |
| `available` | 是否确认存在更高版本，不代表可下载 |
| `templateId`、`currentVersion` | 当前识别结果；未识别值可为 `null` |
| `latestVersion`、`releaseId` | 固定的目标发行；无目标为 `null` |
| `templateUrl`、`downloadUrl` | 官网模板页及固定版本下载地址；不允许第三方 origin |
| `downloadAvailable`、`authenticationRequired` | 下载入口是否开放，以及是否仍需要官网登录 |
| `releaseArchiveSha256`、`releaseSize` | 目标原始发行摘要与字节数；不等于个人外包摘要 |
| `requiredProfiles`、`minHelperVersion`、`notes` | 所需 profile、最低 helper、升级说明 |

响应外层为 `{ "update": ... }`。`available` 当且仅当 `update_status` 为 `newer`；`downloadAvailable` 与 `downloadUrl !== null` 必须一致。只有身份已核验、已发布、实际存在且符合策略的版本才能给出有效更新入口。稳定版本不自动升级到预发布；SemVer 构建元数据不决定版本先后。

固定版本下载地址形式为 `https://motiongit.com/@作者/模板/download/版本.motion`。关闭下载时可以保留新版信息，但必须返回 `downloadUrl: null`、`downloadAvailable: false`。当前更新查询实现不给保护发行开放下载地址；公开查询也不返回用户 Key。

更新下载使用官网当前登录会话，包内 API Key 不是网站 Cookie。新包按登录者账户写入当前 Key；更新不得自动激活旧 Key、轮换 Key、扩大权限或预算。正常未轮换场景下仍是用户原 Key；用户主动轮换后，旧包失效，新包使用当前有效 Key。

下载后必须重新验签，核对目标发行、下载者、摘要和大小，通过后另存新包。原包、实例和已审核成片保留，参数迁移需有明确合同并重新预览；当前查询与下载实现不等于已支持自动参数迁移。查询服务报错或离线时，显示更新暂不可确认，不得伪报“已是最新”。

## 13. 分层大小限制

所有大小以字节判断，`KiB = 1024`，`MiB = 1024²`。网站上传限制可以更低，不能放宽协议读取边界。

| 对象 | 当前上限 |
| --- | --- |
| 作者发行 ZIP、保护密文、外包 `release.bin` | 各 384 MiB |
| 个人 ZIP 总字节 | 384 MiB + 320 KiB |
| 外层非发行文件解包合计 | 256 KiB |
| 外层 `motion.json`、个人凭据 | 各 16 KiB |
| 外层回执、签名 | 分别 64 KiB、4 KiB |
| 外层 ZIP 固定开销 | 64 KiB |
| 普通内层单文件、条目数、解包合计 | 150 MiB、5000、600 MiB |
| 超过 10 MiB 的内层文件 | 压缩比至多 200 |

保护 reader 和个人 reader 对明确的密文/`release.bin` 条目采用专用上限，不把所有普通文件上限抬高到 384 MiB。语言资源的 `motiongit.locale-core/3` 是独立内部扩展，可将有界压缩载荷放在同一个包内的至多三块、每块至多 128 MiB；必须验分块、整体和解包闭包。该扩展不是所有 `.motion` 的必需结构，也不替代单文件、总量和运行环境限制。

## 14. 许可、失败与验收

模板许可、内置素材许可、用户素材权利与服务素材下载条件分别声明；持有 Key 或通过签名验证不自动获得任意再分发许可。成片和工程能否再分发以实际许可为准。

协议实现至少应覆盖这些失败：文件篡改/增删、未知签名公钥、未知版本/profile、账户或 Key 不匹配、Key 已关闭、超限 ZIP、不可用能力、报价过期、幂等冲突、更新身份冲突、helper 过旧、无离线解锁缓存，以及解锁或更新服务暂不可用。失败应保持原包与已有工程，并返回可行动原因；静态验证不得触发收费模型请求。

合格交付需要分别验证外包真实性、内层完整性、实际运行与输出、素材/文案适配、账户 API 与撤销、版本更新和许可。格式正确不等于视觉质量合格，组件丰富不等于所有用户输入都已验证。

## 15. 当前实施边界与后续治理

当前候选实现已覆盖个人封包与读取、下载者原 Key、Ed25519 回执、宿主 API 来源绑定、任务与收据、独立实例以及可选保护的封包/解锁代码。已经完成真实 TTS 闭环验收，但这不是对所有模型、模板或部署的可用性承诺。

模板开放下载、保护发行上线、组件库升级与视觉验收分别推进；本批模板仍处于审核和升级流程。线上运行状态与故障应独立记录，规范不能把候选实现、隔离测试或内部审核包写成公开上线证明。

稳定冻结前应完成跨实现读取、完整模板运行、账户权限与固定版本更新验收。新增核心保护方式、服务 origin、运行合同或迁移语义应明确版本化并提供兼容测试；旧版本原始文件及摘要保留。此为 MotionGit 项目规范，不宣称是其他组织批准的行业标准。
