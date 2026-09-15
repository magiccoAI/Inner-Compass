# Inner Compass（内在罗盘）变更记录

## v5.51（2026-09-14）— 发布审计缺陷修复（BUG-01 P1 + 4 项 a11y）【基于 v5.50】

**产出**：`最终公开版inner_compass_v5_optimized_v5.51.html`（v5.50 原封不动，作为被审计基线保留）
**验证**：`_qa_v550/run_all.js` 跑 `core dataview final a11yaudit a11yverify offscreen bug01 compat` —— dataview 81/81、compat 36/36（Chrome/Edge/Firefox 各 12/12）、a11yaudit/a11yverify/offscreen 全 OK、bug01 修复后 PASS；core/final 剩余 3 项失败与 v5.50 完全一致（见下「未动 / 非回归」），均为审计已判定的测试侧误报，非产品缺陷。

### 触发
v5.50 发布前 Release Audit（`RELEASE_AUDIT_v5.50.md`）结论：**唯一阻断 = BUG-01（P1）**，另有 4 项 a11y（A11Y-01~04）与若干 P3 文案/元数据。本版只做最小修复，不改信息架构 / 文案 / Map 美学 / 数据结构。

### 改动（6 处，全部局部）
1. **BUG-01（P1）**：`renderCompassShell()` 的 **Record 分支**新增 `closePmClueLayer(true);`。线索层由 `openPmClueLayer()` 用 `document.body.appendChild` 挂在 app shell 兄弟节点，Map→Record 切换时 `#compassContent` 重建不会带走它；此前只有 Map 分支经 `initPmField()` 收起它，Record 分支无人收 → 残留覆盖记录正文并吞点击。修复后切到 Record 线索层随之收起。**1 行、幂等、silent、不抢焦点**；Map 分支与 `initPmField` 时序完全不动。（≥901px 100% 复现的缺陷，≤768px 本就不创建该层，不受影响）
2. **A11Y-01（P2）**：`.compass-detail-panel:not(.active), .progress-drawer:not(.active) { visibility: hidden; }` —— 关闭态侧面板移出可聚焦序列，键盘 Tab 不再落到屏外元素（x=1838/1463/1798）。打开态仍 `visible`，focus trap 与滑入动画不受影响；仅关闭滑出动画变为即隐（轻微 UX 取舍，可接受）。
3. **A11Y-02（P2）**：`#mainContent` 的 `<div class="shell">` 加 `role="main"`，补 main 地标（CSS 仅用 `#mainContent`/`.shell` 选择器，无标签选择器依赖）。
4. **A11Y-03（P2）**：`body:not(.dark) .pm-eyebrow { color: #646e64; }` —— 浅色主题下 6 处站点标号对比度 4.295:1 → ≈4.65:1（达 AA 4.5:1）；仅浅色主题覆盖，深色主题沿用 `--ink-faint` 不受影响，且不改全局变量（`.pm-note`/`.pm-clue-text` 同色点不受影响）。
5. **A11Y-04（P3）**：Map 模板 `<svg class="pm-contours">` 加 `aria-hidden="true"`（与 `.pm-trail-layer` 对齐，纯装饰退出 a11y 树）。只加属性、未动 SVG 几何 / `viewBox` / `#pmFieldInk`，避免触发渐变节点失配。
6. **META-02（P3）**：`<meta name="version">` 与 description 版本号由 `v5.22` 升到 `v5.51`（分发的 meta 不再显示过时版本；canonical / favicon / sw.js 等托管项留待上线时处理，属产品决策非缺陷）。

### 未动 / 非回归
STATE_VERSION(7) / STORAGE_KEY / STEPS(7 阶段 18 题) / 每题 id·type·max / PM_ANCHORS / buildMapNodes / buildMapRelationships / Map·Record 呈现 / 继续探索 / 导出 / 加密备份 / 文案。
core 与 final 仍各有 1/2 项失败，但**与 v5.50 完全一致**：core `E-10`（种子 `answers:null` 畸形态，正常使用不可达）、final `N-15r`（测试用 `element.click` 不移动焦点，真实鼠标点击 14/14 通过）、final `T-02`（测试误查 desktop 元素 display，真实移动端 footer nav 可见项=0）。三者均为审计已判定的测试侧误报，本版零新引入回归。

### 上传就绪化（2026-09-15）
面向 GitHub 公开发布做的零功能改动（仅署名 / 许可 / 资源自包含）：
1. **去外部依赖**：删除 3 行 Google Fonts `<link>`，改纯系统字体栈（`Noto Sans SC, Microsoft YaHei, PingFang SC, system-ui`）→ 文件完全离线、零外部请求（顺手解决审计 PRIV-01 隐私项，也契合项目「单文件离线、无外部字体」硬约束）。
2. **署名 / 许可收敛**：页脚移除 `ic-license-note` 许可行（原文「© 2026 Leah…非商业用途，转载/改编须注明作者·详见 LICENSE」）；作品署名与许可改由 ① 首页 `<meta author>` / `<meta copyright>` ② 仓库 `README.md` 的 License 段 ③ 仓库 `LICENSE` 文件 三处一致承担。
3. **图标自包含**：favicon 改为内联 SVG data URI（🪐）；移除 `favicon.ico` / `apple-touch-icon.png` / `manifest.webmanifest` 三个外部引用 → GitHub Pages 部署后零 404。
4. **仓库资产补齐**：新增根目录 `LICENSE`（CC BY-NC 4.0 中文条款）、`README.md`、`INNER_COMPASS_BUILD_LOG.md`（设计过程记录，README 已引用）；`canonical` 仍为 `https://YOUR_DOMAIN_HERE/...` 占位符，待上线时替换为真实仓库 URL。

---

## v5.50（2026-09-14）— 完整记录 PDF 排版优化（呼吸感 + 回答不再加粗）【基于 v5.49】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.50.html`（v5.49 原封不动）  
**配套脚本**：`_apply_v550_patch.py`（EOL-safe，2 处锚点替换，断言各命中 1 次；CHANGELOG 单独改）  
**验证脚本**：`_verify_v550.js`（puppeteer 真实打印模拟 + A4 PDF，**8 PASS / 0 FAIL**）  

只改 `buildReportHTML()` 内联打印 `<style>`，不碰任何结构 / 字段 / 文案 / 其它功能。

### 改动
- **回答不再加粗**：`.rp-text`（Compass Summary 原话）与 `.rp-list li`（Journey / 回访原话）显式 `font-weight: 400` + 柔和墨色 `#3a463d`；提问 `h3`/`h4` 保留为**唯一加粗锚点**（`h3` 700、`h4` 600），形成「提问重、回答轻」的层次——回答不再与提问争夺视觉重量，也不再读成整片黑体。
- **题目间留白（呼吸感）**：`.rp-item` 下边距 10px → 18px、左内边距 10px → 14px 并加深浅色竖线 `#e6e0d4`；`.rp-phase` 14px → 22px、`.rp-section` 20px → 28px；提问与回答间距 4px → 7px；列表项间距 2px → 7px；正文行高 1.85 → 1.95。
- **未填写项**改斜体灰字（`font-style: italic`），与已填写内容区分。

### 明确没动
- 引导语 / 题目 label / placeholder / Phase 7 文案 / 标签上限逻辑 / 导出 PDF 触发与兜底 / `exportMarkdown` / Map / Record / Insight / 加密备份 —— 全部未动；STATE_VERSION 仍 7。

## v5.49（2026-09-14）— 标签上限一致性 + 完整记录导出 PDF + Phase 7 第二问文案【基于 v5.48】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.49.html`（v5.48 原封不动）  
**配套脚本**：`_apply_v549_patch.py`（EOL-safe，14 处锚点替换，每个锚点断言命中 1 次）  
**验证脚本**：`_verify_v549.js`（jsdom，**65 PASS / 0 FAIL**）、`_shot_v549.js`（puppeteer 1440 / 390 + 真实 A4 PDF，**37 PASS / 0 FAIL**）、`_probe_v549_report.js`（报告文档几何探针）  

---

### 目标（用户提出的三件事）

**task1 — 标签上限「看起来也一致」**。用户截图：work_constraints（Phase 5 第二问，`hint` 写「最多 3 项」）实际出现 6 个 tag（预设 + 自定义混排）。先确认机制：**预设与自定义本就共用同一个 `question.max`**（`renderTagOptions` 的 `current.length < question.max` 与 `commitCustomValue` 的 `current.length >= question.max` 数的是同一个「合计已选」），所以「合并计数」模型本无 bug；6 项是 **v5.35 把 max 由 5 降到 3 时故意保留的旧存档**（当时明确「不截断，避免静默删数据」）。

**task2 — 「下载我的完整记录」改成 PDF**。理由：潜在用户不一定装了 Markdown 阅读器，希望下载物本身就是通用可读的 PDF，内容是完整问卷记录。

**task3 — Phase 7 第二问（experiment_action）文案微调**，并把 4 条例子从提示词里拆成独立的「例如：」示例块。

---

### 改动清单（14 处 hunk）

**task1（5 处）**
1. `hint` 明确合计口径：`从下面选出最多 3 项：…预设词与自定义词一起计算，合计不超过 3 项。`
2. 新增 CSS `.tag.is-locked`（`opacity:.5` + `cursor:not-allowed`）。
3. `renderTagOptions` 增加 `atLimit = !!question.max && selected.length >= question.max`；达上限时**未选中**的标签加 `.is-locked` 与 `aria-disabled="true"`（选中项不加）。**仍然可聚焦、可点击**——点击会说明原因，而不是变成不可达的死控件。
4. `multitag` 分支新增 `.tag-counter` 实时计数器（`已选 N / max`，达上限加 `.at-limit`）+ `.tag-overlimit` 越界提示。
5. 两条添加路径统一给出原因：`data-tag` 分支此前是**静默无反应**（`next = current`），现补 `showToast("最多只能选择 N 项，可先取消一项再添加")` 并 `return`；`data-valuepick` 分支同样处理。样式新增 `.tag-counter` / `.tag-overlimit` / `.tag-counter + .helper-row`。

**task2（4 处）**
6. 按钮文案 `下载我的完整记录` → `下载我的完整记录（PDF）`。
7. `downloadCompassReport()` 改调 `exportReportPDF()`，toast / announce 文案改为「已打开打印窗口，选择『另存为 PDF』…」。
8. 新增 `reportAnswerLines(question, value)` —— 与 `exportMarkdown()` 同口径的取值函数（字符串 / 数组 / `reality_classification` 四档 / structured 对象）。
9. 新增 `buildReportHTML()` + `exportReportPDF()` + `canPrint(win)` + `printDocument(win)`。

**task3（5 处）**
10. `experiment_action.guide` 改「先试一次、做几天、找个人聊聊、观察一阵子」「准备做什么，是试一次，还是持续一段时间」。
11. `scaffold.prompts` 由 3 条改 2 条（第一条改「是试一次、做几天，还是观察一段时间」；第二条改「回头看看前面写下的内容：它和哪些想法有点关系」）。
12. 新增可选字段 `scaffold.examplesLabel` + `scaffold.examples`（4 条，示例文字同步为「聊聊他的日常」）。
13. `renderScaffold` 支持 `examplesLabel`（`sc.hasExamplesLabel` 未给时默认「例如：」）。
14. 新增 CSS `.scaffold-examples-label`。

---

### 关键设计决定

**A. 越界处理按「上限是否曾经合法」分流，而不是一刀切。**
| 题 | 上限历史 | 越界性质 | 处理 |
|---|---|---|---|
| `core_values`（valuesorter） | **一直是 5** | 无效越界状态 | v5.48 一次性截断到上限（保留用户自排顺序） |
| `work_constraints`（multitag） | **曾 5、v5.35 降到 3** | 第 4/5 项是用户在旧上限下**真实选过**的 | v5.49 温和重选：**全部保留、绝不静默删除**，提示「你之前选了 N 项，现在这题最多 M 项…想留下哪 M 项？」，并在降到 ≤M 之前禁止继续添加 |

v5.48 的 `clampValueMaxSelections()` 只作用于 `type === "valuesorter"`，因此天然不会碰到 `work_constraints`——两套策略在同一份数据上不冲突（验证 C1/C2 同时断言了这两件事）。

**B. 为什么 PDF 走「打印视图 + 浏览器原生打印」，而不是自己写 PDF。**
项目铁律是单文件离线：**无 CDN / 无第三方库 / 不引入外部字体**。中文 PDF 必须内嵌字体（子集化也需字体数据），手写 PDF 要么体积不可行、要么只能靠 `STSong-Light` 这类非内嵌 CID 字体——在 Chrome/PDFium 与部分阅读器上会退化成豆腐块，不可接受。因此改为：
`buildReportHTML()` 生成一份**自包含 A4 打印文档**（系统字体、真实中文、可选中文本、`@page { size:A4; margin:16mm 15mm }`、`break-inside: avoid`）→ 塞进**离屏但保持渲染**的 `<iframe srcdoc>` → `iframe.contentWindow.print()`。
用户在打印对话框选「另存为 PDF」即得到任何设备都能打开的文件；默认文件名取文档 `<title>`。环境不支持打印时**退回下载同内容的 `.html`**，保证内容不丢。

> ⚠️ 与 v5.47h 的区别（避免被误读为回退）：v5.47h 修的 bug 是按钮调 `window.print()` 打印的是 **Map Mode 页面**（看不到问卷原文）。本版打印的是**专门构造的报告文档**，内容 = Compass Summary → Journey 记录（18 题全部题面 + 用户原话）→ 试过之后（回访），与 `exportMarkdown()` 同口径同覆盖。`exportMarkdown()` 本体与「更多 › 导出 Markdown 文本」入口**完全保留**（验证 D17 断言同种子下 md 输出与 v5.48 逐字节一致）。

**C. 打印容器必须「离屏」而不是「`display:none`」**——`display:none` / `visibility:hidden` 的 iframe 会被浏览器优化掉，打印出空白页。用 `position:fixed; left:-10000px; width:1024px; height:768px`（实测容器内文档 `scrollHeight` = 2140px，真实排版）。

---

### 明确没有修改
`STATE_VERSION`(7) / `STORAGE_KEY`(`inner_compass_v5_3`) / `STEPS` 结构 · 阶段数 · 题目数（7 阶段 / 18 题）· 每题 `id`/`type`/`max`（`work_constraints` 仍 3、`core_values` 仍 5、`adaptability` 仍 5）· `buildCompass` / `buildMapNodes` / `buildMapRelationships` / `findNodeRelations` / `PM_ANCHORS` / `PM_TRAILS` / Map Mode 与 Record Mode 呈现 / Insight（继续探索）/ 加密备份 / 导出 JSON / `exportMarkdown()` 本体 / Experiment Feedback 数据层 / 7 阶段标题与引言。

---

### 验证

**`_apply_v549_patch.py`**：EOL 探测 = 纯 LF（CRLF 0 / loneLF 10559），14 处锚点各命中 1 次，写盘后 EOL 不变（CRLF 仍为 0）。

**byte-exact 反向 diff**：复用补丁脚本自身的 `OPS` 对新文件逆序逆应用 14 处 → **与 v5.48 逐字节相等**。证明确实只动了这 14 处 hunk。

**`_verify_v549.js`（jsdom）— 65 PASS / 0 FAIL**
- **A 结构零改动（10 项）**：把 v5.48 与 v5.49 的 STEPS 元数据投影出来逐项对比——阶段 id / 题目数 / 每题 `id·type·max·options·fields` 全部一致；**文案差异集合恰为 `{experiment_action, work_constraints}`**（即只动了本轮该动的两题）；标题与引言 `===`。
- **B 上限一致性（15 项）**：初始 `已选 0 / 3` 且无置灰；选中 3 项后 8 个未选中标签全部 `is-locked` + `aria-disabled="true"`（选中项不带）；计数器 `已选 3 / 3` + `.at-limit`；点第 4 个不改变答案**且给出原因**；**自定义添加同样被拦下**（证明「预设+自定义合计同一个 max」）；取消一项后置灰解除、计数器回落、再自定义可加。
- **C 旧存档越界（9 项）**：预置 5 项旧存档 → **完整保留未被截断**；同时 `core_values` 6 项被 v5.48 截断为 5 项保序（两条策略并存）；越界提示文案含「你之前选了 5 项」「最多 3 项」「不会自动删掉」；计数器标注「（超出上限）」；越界时选中项仍可取消但禁止新增；降到 3 项提示消失 → 再降 1 项后可正常添加（取舍回到用户手里）。
- **D 导出 PDF（18 项）**：报告为合法完整 HTML（`@page A4`）；含 **7 个阶段标题 + 18 道题题面**；含用户原话（字符串 / structured / `reality_classification` 四档「守住」）；回访章节按同口径出现；未填项显示「（未填写）」；`<script` 被转义；`exportReportPDF()` 走打印路径且**只触发一次**、未误走兜底；离屏容器属性正确；**不支持打印时退回 `.html`**；按钮点击确实调用 `exportReportPDF()` 一次并恢复可用与原文案；**`exportMarkdown()` 输出与 v5.48 逐字节一致**。
- **E task3 文案（7 项）**：label / placeholder 未变；guide 与要求**逐字一致**；脚手架 2 条提示与要求逐字一致；示例块 =「例如：」+ 4 条（含「聊聊他的日常」）；渲染出 `scaffold-examples-label` + 4 个 `.scaffold-example`；**仅此一题使用示例块**。
- **F 其它题上限（3 项）**：`core_values` 连点 6 停在 5、`adaptability` 连点 6 停在 5。
- **G 旧存档 / 极端输入（3 项）**：无 `version` 的旧存档加载 + 全量渲染不报错；异常类型（字符串 / `null` / 数字）不产生越界提示。

**`_shot_v549.js`（puppeteer-core + Chrome 131，1440×900 / 390×844）— 37 PASS / 0 FAIL**
- 置灰真实生效（`opacity<1` + `cursor:not-allowed`）；计数器与越界提示在两种视口都不横向溢出、标签互不重叠、提示条有实际宽度。
- 示例引出行与 4 条示例在桌面 / 移动都正常排版、保留左侧强调线。
- 打印容器 `display` 非 `none` / `visibility:visible`；**容器内文档真实排版 `scrollHeight` = 2140px**（不是空白页）；三章内容齐全。
- 用同一份报告文档生成**真实 A4 PDF**：合法 `%PDF-` 头、211 KB、**多页**、**内嵌字体子集**（中文不会变豆腐块）。

**过程中修正的两个 harness 陷阱（已写回 skill）**
1. **只调 `renderJourney()` 不会切视图**——HTML 被写进 `display:none` 的容器，所有 `getBoundingClientRect()` 返回 0，**「无横向溢出 / 互不重叠」这类断言会静默通过**（第一轮 37 项里有 3 项是这样假通过的）。必须调 `switchView('journey')`，并加一条「题目区宽高 > 0」的守门断言。
2. **同一个 jsdom 内多次 seed 时，一次性迁移的幂等标记会串场**——第一次 `loadState()` 把 `_clampedValueMax` 置 true 后，后续 seed 的迁移被跳过，导致「v5.48 截断仍在」假失败。seed 前需 `delete state._clampedValueMax; delete state._purgedLegacyWorkTags;`。

---

## v5.48（2026-09-14）— 价值排序上限修复（一次性数据迁移）【基于 v5.47】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.48.html`（v5.47 原封不动）  
**配套脚本**：`_apply_v548_patch.py`（EOL-safe，5 处锚点替换：头部版本号 + changelog 条目 + 新增 `clampValueMaxSelections` + `loadState` 接入 + `applyImportedState` 接入）  
**验证脚本**：`_verify_v548.js`（jsdom，**12 PASS / 0 FAIL**）  

**本轮目标**：修复「引导语写着最多 5 项、答案却出现 6 项」的错位。UI 选额上限（valuesorter 的 `current.length < question.max`）自 v5.30 起就一直为 5（连点 6 个 chip 停在 5，历史 jsdom 已验证），但**旧版存档 / 外部导入**数据里可能残留超过上限的答案，与引导语矛盾。

**根因判定**：不是当前 UI 能选出 6（当前选额上限始终生效），而是历史遗留 / 导入数据越界。因此采用**一次性迁移**而非改文案或放宽逻辑——与 v5.35 处理 work_constraints（max 5→3）时「不静默删数据」的先例不同：那里是**合法降低上限**，旧选额是用户在旧上限下真实选的，故不截断；而本题 max 一直是 5，6 项本就是无效越界状态，修复=截断到上限并保留用户自排顺序，属合理纠正。

**改动清单（仅 1 个新函数 + 2 个接入点）**：
1. 新增 `clampValueMaxSelections(force)`：遍历 `STEPS`，对 `type === "valuesorter" && q.max` 的题目，若 `state.answers[q.id]` 数组长度 > `q.max`，`slice(0, q.max)` 截断；带幂等标记 `state._clampedValueMax`，正常路径只跑一次。
2. `loadState()`：在 `purgeLegacyWorkConstraintTags()` 之后接入 `clampValueMaxSelections()`，若 trimmed>0 立即落盘。
3. `applyImportedState(incoming)`：在 `purgeLegacyWorkConstraintTags(true)` 之后接入 `clampValueMaxSelections(true)`（force，导入必修）。

**明确没有修改**：STATE_VERSION(7) / STORAGE_KEY / STEPS / 18 题 id 与每题 type·max / 6 个 PM_ANCHORS / buildMapNodes / buildMapRelationships / 地图呈现 / Record / Map / Insight / 加密备份 / 导出 / 任何文案 / 任何用户可见选额上限（仍是「最多 5 项」）。

**验证（_verify_v548.js，jsdom）**：
- 结构零改动：`clampValueMaxSelections` 唯一定义；`loadState`/`applyImportedState` 均含调用；`core_values` 仍 `valuesorter` + `max===5`；`STATE_VERSION===7`。
- 迁移逻辑：6 项→5 项且保留前 5 项顺序（=此刻最看重）；返回 trimmed 计数正确。
- 真实加载路径：预置 7 项旧存档 → 启动后 `state.answers.core_values` 修复为 5 项且顺序保留；幂等标记写盘。
- 实时上限：连点 6 个 chip → 答案停在 5 项。
- 额外：v5.48 对 v5.47 做 **byte-exact 反向 diff**（5 处补丁逆向后逐字节相等），证明只有这 5 处 hunk 改动。

---

## v5.47（2026-09-14）— Map Mode · Interactive Cartography（会回应用户注意力的个人地图）【基于 v5.46】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.47.html`（v5.46 原封不动；v5.47i 直接在此文件上修改）  
**配套脚本**：`_apply_v547_patch.py`（主补丁，38 处 EOL-safe 锚点替换）、`_apply_v547b_field.py`（场域首次标定）、`_apply_v547c_field.py`（遮罩 58%→34%）、`_apply_v547d_baseline.py`（**修正「Rest 有假注意力中心」**）、`_apply_v547e_pool.py` + `_apply_v547f_pool290.py`（注意力池半径收紧到 330px）、`_apply_v547g_reading_island.py`（给 anchor 文字加页面同色阅读底色，避免等高线穿字）  
**验证脚本**：`_verify_v547.js`（jsdom 结构/数据层，**125 PASS / 0 FAIL**）、`_shot_v547.js`（puppeteer 桌面 1440·1024 / 移动 390·375·430 / reduced-motion）、`_probe_v547_ink.js`（白底墨量比值探针，定量标定）、`_probe_v547_diag.js`（ASCII 墨迹图）、`_probe_v547_inkalign.js`（坐标系对齐）、`_shot_v547i_confetti.js`（v5.47i 礼花多帧截图）、`_shot_v547i_confetti_rm.js`（v5.47i reduced-motion 零粒子验证）  
**jsdom 结果**：`PASS: 125 FAIL: 0`。A 组 24 项断言 v5.46 与 v5.47 数据层**完全一致**（STATE_VERSION / STORAGE_KEY / STEPS / PM_ANCHORS / PM_TRAILS / PM_DENSITY / PM_TRAIL_SHAPE / buildMapNodes / buildMapRelationships / findNodeRelations / getCompassData / exportMarkdown / renderCompassRecord 全部 `===`）。

**本轮目标**：不是增加动画数量，而是把 Map Mode 从「带动画的个人方向海报」升级成「**会回应用户注意力的个人地图**」。判据（贯穿全部改动）：每个 motion 必须能回答「它是否帮助用户理解地图空间、注意力或关系」，否则删除。

**硬性边界（全部满足）**：不改 STATE_VERSION(7) / state schema / answer keys / `PM_ANCHORS` / `PM_TRAILS` / `buildMapNodes` / `buildMapRelationships`；不新增 AI 推断、不新增关系；不引入 Canvas / WebGL / Three.js / GSAP / CDN；保持既有 accessibility / reduced-motion / keyboard focus 体系；保持 Mobile Vertical Perceptual Field。

**① P0 — Motion Grammar 四态化（不让所有状态都靠 opacity 完成）**
- **Rest**：contour 只有 96s 的 `pmFieldBreath` 呼吸（0.93↔1.00，几乎察觉不到），**无** dash movement、**无**全局 parallax、**无**粒子、**无** glow；Anchor 完整可读。删除了 v5.46 的 `--pmx/--pmy` 全局 translate（指针不再移动整张图）。
- **Explore / Hover**：改为 **Local Attention Field**——地标三级可读性 `nearest = 1 / 半径内 = .90 / 半径外 = .86`（CSS `.is-att-near` / `.is-att-mid`），删除 v5.46 的 `.is-hovering .pm-anchor { opacity: .62 }` 全局压暗。Hover **不打开** Detail Panel，只做 marker 微动画（点 → crosshair）、标题下划线、related trail preview、以及局部地形响应。
- **Focus**：click / Enter 后形成地图叙事——Step A rest→focus 0ms → Step B related 提升 130ms → Step C 真实 `PM_TRAILS` 路线 reveal 780ms 一次性 `stroke-dashoffset`，播完**停止**（`.is-revealing` 由「移除 → 强制回流 → 重加」实现一次性重播，不无限循环）。
- **Trail Motion 分级**：Rest `.18` / Related `.32` / Active `.64`；infinite `.pm-trail-current` 从 15s·`.04` 降为 26s·`.022`（环境层，不得成为第一眼感知的动画）。

**② P0 — Local Contour Response（本轮真正的技术核心，含一次根本性纠错）**
机制：`#pmFieldInk`（`userSpaceOnUse` 径向渐变，cx/cy 由 `pmUpdateInkGradient()` 每帧跟随 `--fx/--fy`）负责「**注视处加深**」；`.pm-contours` 的 `mask-image`（`--field-k` 控制半径外退让）负责「**半径外退出**」。两者都**不改任何 path 的 `d` / `dasharray` / `transform`**，几何永不变动。

- **A. Rest 必须均匀（根本纠错）**：初版把渐变写死为「核心 14% / 边缘 5%」，于是 Rest 状态地图中央就已经有一个**假的注意力中心**；指针一移动，这个核心跟着走，整条原本被照亮的带子同时变暗 —— 产生与注意力无关的大幅整体变暗。实测远端只有 `0.51~0.54`（= 核心移走 0.59 × 遮罩 0.66 × 提亮 1.296），而站在同一块地形上的地标只退到 `.86` —— **地形比它自己的地标退得狠 2.6 倍**。修正：两档在 Rest 时**同值**（`stop-opacity: calc(.10 + var(--field-k,0) * .04)` / `.10`），只有注意力出现才把核心档拉到 `.14`。有效墨色 `.10 × .34 = .034`，与 v5.46 的 `.08 × .42 = .0336` 一致（**Rest 基准不漂移**）。
- **B. 不做整幅提亮**：删除 Explore 的 `--contour-att` 统一提亮项（`opacity: calc(.34 + att*.28)` → `opacity: .34`）。`--contour-att` 改为只服务 `.pm-stage.is-focused`（`.34 → .4012`，1.18×），token 不变成死变量。
- **C. 地形与地标一起退**：遮罩衰减系数由 `.58` 校正到 `.14`，使半径外 = `1.00 × .86 = .86`，**恰好等于地标 far 档**。
- **D. 一个场域只有一枚注意力半径**：池半径 `r=290`（userSpace）→ 渲染约 334×366px，与 JS 既有的地标注意力半径 `clamp(min(w,h) * .34, 200, 330) = 330px` 对齐。330px 既决定哪些地标算 near/mid/far，也决定地形在哪里加深 —— 两套机制共享同一个空间语义。（实测 `r=620` ≈ 714×782px 会盖住半张地图，把「整幅」抬到 1.23×。）

**③ 场域定量标定（白底墨量法，`_probe_v547_ink.js`）**
方法：隐藏 `.pm-trail-layer` / 全部 `.pm-anchor` / `.pm-datum` / `.pm-field-attention`，把 `#pmStage` 与 `.pm-atmos` 背景强制纯白 → 画面里**只剩流场描边是暗的**；指标 = `Σ(255 − 亮度) / 像素数`（平均墨量），墨量与描边 alpha 成正比，故 `explore ÷ rest` 的比值**就是有效 alpha 比值**。截图前重新 pin 视口（实测 drift = 0px），并关闭 96s 呼吸以消除 ±3.5% 相位噪声。（早期版本用高通滤波 + 固定方块，把地标文字与场域色池也算进高频能量，reads 全部塌成 ≈1.00，已废弃。）

| 版本 | 整幅 ratio | 注视处 ratio | 半径外 ratio | 局部对比 |
|---|---|---|---|---|
| v5.46 | 1.049 | 1.044 | 1.057 | **0.99 : 1**（无局部机制，只有 `--contour-att .35` 的统一提亮 +5%） |
| v5.47 | **1.126** | **1.403** | **0.828** | **1.69 : 1** |

- Rest 静息场域（跨版本逐格对比，139 个有线格）：**逐格相关 r = 0.9764**（场域形态与 v5.46 一致），总墨量比 **0.955**（v5.47 略淡 4.5%），单格最大绝对差 0.14。两者有效墨色代数上几乎相等（v5.46 `.08 × .42`，v5.47 `.10 × .34`），4.5% 的差额量级约 **0.4 灰阶**（流线单像素暗度仅 ≈8.7 灰阶）——属于遮罩离屏合成 / 8-bit 量化的跨版本系统性偏差，不是机制差异。仪器自洽度对照：同版本、同状态、同视口重拍两次的比值 = **1.0000**（完全确定性）。
- 注视处 `1.403` ≈ 设计值 `1.40`；半径外 `0.828` 落在设计语言 `.82~.86` 带内；整幅 `1.126` 未超出「不是整张地图亮起来」的上界。
- 坐标系对齐实测：`.pm-contours` 为 `preserveAspectRatio="none"` + `viewBox="0 0 1000 1000"`，`cx/cy = PM.fx × 10` 的换算**误差 0.2px**（已确认不是坐标 bug）。
- 顺带确认：`calc()` + `var(--field-k)` 在 SVG `<stop>` 的 `stop-opacity` 上可正常解析（Chrome 131 实测 `0.1/0.1` → `0.13992/0.1`）。

**⑤ P1 — Reading Island（v5.47g 同日追加）**
- 问题：用户 hover 到「我想去哪里（vision）」地标时，`.pm-contours` 的等势线从长段用户文字背后穿过，形成「叉号/涂鸦」感，干扰阅读。
- 修复：给 `.pm-anchor-inner` 增加与页面同色的阅读底色（`background: var(--bg)`），让等高线退到文字后面，不再与文字竞争。圆角 6px、内边距 10px 12px，视觉重量仍保持「地图注记」而非「卡片」。
- 范围：桌面端所有 anchor（vision 长文本立即受益）、移动端 `.pm-anchor.is-open` 就地展开同样继承；不动动画/关系/数据结构。

**⑥ P1 — 结果页操作微打磨（v5.47h 同日追加）**
- **「查看将复制的内容」更易点击**：`.ai-preview-toggle` 从纯文本下划线链接改为轻量按钮样式（`display: inline-flex`、14px 字重 500、44px 触控高度、边框 + 面板底色、hover 时切换为 accent-soft、带 chevron 旋转指示展开/收起）。保留 focus-visible 轮廓，不破坏 modal 内的信息层级。
- **「下载我的完整记录」现在下载 Record Mode 全量 Markdown**：原 `#downloadSummaryBtn` 调用 `window.print()`，会打印当前 Map Mode 视图（看不到完整问卷原文）；改为调用既有 `exportMarkdown()`，输出与「更多 › 导出 Markdown」同口径的完整 Journey 记录（含 Compass Summary、逐题回答、实验后回访）。按钮文案同步从「下载我的 Compass 报告」改为「下载我的完整记录」。

**⑦ P1 — 首次完成礼花动效升级（v5.47i）**
- 问题：首次点击「生成我的 Compass」是用户的一个重要完成时刻，但旧礼花是 56 片从底部缓缓上升的「纸片漂流」，缺少庆祝感和爆发张力，正向反馈偏弱。
- 方向：保持单文件原生 JS/CSS（无 Canvas/WebGL/GSAP/CDN）、保持 `prefers-reduced-motion` 关闭、保持仅首次触发（`hasCelebrated` 逻辑不动），在此前提下把礼花升级为「中心扇形爆发 + 柔光爆闪 + 两波节奏 + 重力下坠」的庆祝事件。
- 具体改动：
  - **爆发原点**：从底部 `top:90%` 改为视口中心 `left/top: 50%`，粒子以中心为原点向四周迸发，强化「成就从罗盘中心释放」的空间隐喻。
  - **粒子形态**：由 4 种增至 6 种——circle、square、rect（丝带）、bar（细条）、tri（三角）、glyph（✦✧✶★✺❉ 星形），每种带随机缩放（0.7–1.4）和三轴旋转（rotate/rotateX/rotateY），纸片会在飘落中翻滚闪烁。
  - **色彩搭配**：在原有低饱和纸感色基础上略提亮，加入香槟金 `#F2C879`/`#F4D58A` 与玫瑰 `#E89BAE` 作为成就/庆祝锚点，整体仍保持温暖、不刺眼。
  - **爆发范围与节奏**：96 片分两波——第 1 波 66 片、delay 0–0.18s、最大射程 46% 视口短边；第 2 波 30 片、delay 0.42–0.95s、射程 72%，形成「砰——噗」的层次感与惊喜余韵。角度以向上为主（±112°），再叠加重力下坠与轻微横摆。
  - **持续时间**：单粒子 2.6–4.3s，整体清理时间 5.6s，比旧版（2.0–3.2s / 3.6s 清理）稍长，让完成感延续。
  - **运动节奏**：新 keyframe `confetti-burst` 分 0%（收缩透明）→ 6%（显形）→ 24%（爆发到 60% 射程，带弹起 scale）→ 58%（到顶并开始下坠）→ 100%（飘落出屏并淡出），用 `cubic-bezier(.2,.62,.26,1)` 让爆发先快后慢、下落自然。
  - **中心柔光爆闪**：新增 `.celebration-flash`，在触发瞬间从中心展开一个暖金+玫瑰的径向光晕（0.9s），给「达成」一个明确的视觉锚点，提升惊喜感与成就感。
- 验证：puppeteer 直接调用 `triggerCelebration()` 输出 `confetti:96 / flash:1`，帧截图 250ms/650ms/1150ms/1900ms 显示中心爆发、扩散、下坠、消散全过程；reduced-motion 下 0 粒子/0 闪光；jsdom `_verify_v547.js` 仍 **125 PASS / 0 FAIL**。

**④ P1 — Clue Layer / Map Intro / Mobile**
- **Clue Layer**：保留既有架构（`role=dialog`、不用 `aria-modal`、`aria-labelledby="pmClueTitle"`、Escape 分层、焦点归还），视觉向 Field Note / Cartographic Annotation 靠拢：`NN / 地标名` → `FIELD NOTE` → 分隔线 → 记录列表 → 「还有 N 条记录」。Hover 只出现 accent line，Click 才进既有 Detail Panel。
- **Map Intro**：只讲「这张图不替你决定方向。它只是把你刚才写下的线索放回同一张地图。」+ 一行阅读顺序（生活图景 → 价值 → 边界 → 工作 → 现实 → 尝试）+ 极轻提示「移动鼠标探索 · 点击地标深入」，不加长教程。
- **Mobile（≤900px）**：保持 Vertical Perceptual Field。`getBoundingClientRect` 视口中心带取代 IntersectionObserver 的「进/出」（当前 = 1 / 上下相邻 = 2），无横向 pan / 无 drag-to-pan / 无 desktop pointer parallax；contour 只在局部改 visibility，不移动。感受是「我在沿着自己的地图往前走」，不是「网页自动播放动画」。
- **Reduced motion**：关闭 contour 动画 / route reveal / marker 动画 / parallax，但**保留** focus / related / hover 语义状态 / route visibility / clue layer / 可访问性焦点，信息层级完整。

**未动**：STATE_VERSION=7 / STORAGE_KEY=`inner_compass_v5_3` / STEPS / 全部 answer key / 18 题结构 / `PM_ANCHORS` / `PM_TRAILS` / `PM_DENSITY` / `PM_TRAIL_SHAPE` / `PM_FIELD_SOURCES` / `buildMapNodes` / `buildMapRelationships` / `findNodeRelations` / `getCompassData` / Record Mode / Detail Panel / Export(JSON·Markdown) / 加密备份 / 继续探索 / 实验后回访 / Welcome / Journey 全部未动。

---

## v5.46.x（2026-09-13）— 流场柔化与节点 hover 交互增强【基于 v5.46】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.46.html`（v5.46 原地修改，未另开新文件）  
**配套脚本**：`_apply_v546x_field_and_hover.py`（CSS/JS 补丁，幂等）、`_make_demo_v546.py`（生成带虚拟数据的 `_demo.html`）  
**验证**：puppeteer 桌面 1440 / 移动 390；结构探针：`strokeDasharray: none`、`animationName: none`、contour opacity 0.42；6 anchors / 3 real trails / datum 在位 / 0 重叠。

**改动原因**：用户反馈 v5.46 Map Mode 背景流线因 `stroke-dasharray` + `pmContourDrift` 动画呈现「毛毛虫」观感，且节点交互反馈不足。

**① P0 — 消除「毛毛虫」背景线**：
- `.pm-contours path` 移除 `stroke-dasharray` 与 `pmContourDrift` 动画；改为连续、极淡的开放流线。
- stroke 从 `color-mix(in srgb, var(--ink) 20%, transparent)` 降至 `8%`，基础 opacity 从 `.72/.58/.14` 降至 `.42/.32/.05` 量级，focus 提升也相应降低。
- 删除 `@keyframes pmContourDrift`；`pmCurrent`（路线 current 动画）保留但仅在真实 path 上，量级已极淡。
- `.pm-field-attention` accent 色池从 `.14/.1` 降至 `.09/.06`，避免抢夺文字主体。

**② P1 — 在 v5.46 设计约束内增强节点交互**：
- `.pm-anchor` 增加 `opacity` 与 `.pm-anchor-inner` 的平滑过渡；hover 时无关 anchor 降至 `.62`，被 hover anchor 完全亮起。
- 为每个 anchor 添加 `mouseenter/mouseleave` 监听；hover 时调用 `pmHoverAnchor()`，高亮相连的**真实 PM_TRAILS**（仅 3 条可证明路径：here→experiment、experiment→vision、values→vision），不新增任何关系或伪连线。
- 键盘聚焦 `:focus-within` 保留并增加极淡 accent 轮廓；已有 `pmOnFocusIn` 键盘进入 anchor 时自动 `pmFocusAnchor` 的行为不变。
- 未做「节点拖拽」或「任意节点间自由连线」——两者与 v5.46 设计铁律冲突（Map 只呈现可证明关系、禁止 fake relationship lines / 数据可视化式 network）。如后续明确需要，可作为独立实验分支（非 v5.46 主线）处理。

**③ P1 — Demo 文件同步更新**：
- `_make_demo_v546.py` 重新生成 `最终公开版inner_compass_v5_optimized_v5.46_demo.html`，使用独立 `inner_compass_v5_3_demo` key，首次打开即写入完整虚拟人设并渲染 Map Mode。

**未动**：STATE_VERSION=7 / STORAGE_KEY / STEPS / answer key / PM_ANCHORS / PM_TRAILS / buildMapNodes / buildMapRelationships / getCompassData / 6 锚点关系语义 / Mobile 纵向场 / Record Mode / Detail Panel / Clue Layer。

---

## v5.46（2026-09-13）— Map Mode 视觉范式重构：Living Field / 个人方向地图【基于 v5.45】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.46.html`（v5.45 不动）  
**配套脚本**：`_apply_v546_patch.py`（保留原始呈现层替换）、`_apply_v546_fixes.py`（修复 Edit 工具偶发未落盘的边角）、`_apply_v546_presence.py`（场域存在感最终强度）、`_render_v546.js` / `_render_v546_full.js`（puppeteer 桌面/移动端截图与探针）  
**验证**：JS `node --check` PASS；puppeteer 桌面 1440 / 移动 390 全页截图；结构探针：16 条流场线、1 条当前 route、datum 存在、6 anchors、**0 重叠**、pointer 触发 `--pmx` 0 → 0.227（场域本身响应，文字节点不平移）。

**本轮目标**：不是继续在 v5.45 的「六站式网格 + 淡装饰等势线」上调颜色/动效，而是**重新思考 Map Mode 的视觉构图逻辑**：从「Dashboard / 六个 Section / 数据可视化」推进为「一张正在被阅读的私人方向地图（Personal Atlas + Living Field）」。地图本身是主要视觉对象，六个 anchor 只是嵌入流场的 landmark / annotation。

**① P0 — 构图逻辑重构（结构层）**：
- 删除 v5.45 的 `.pm-stage` 三列 CSS Grid station 布局，改为 **block-flow 交错文档流**：01 居中、02 左、03 右、04 中、05 左、06 右，margin 由内容驱动，**内容再高也零重叠**（不回退绝对定位百分比）。
- 每个 anchor 左侧不再画整高的 vertical rule（station mark），改为 tiny crosshair map-mark（11×11px `+`），明确表达「这是地图上的注记点，不是卡片」。
- 05「我现在站在哪里」升格为空间原点 / 测地 datum：新增 `.pm-datum` 十字 + 坐标刻度 +「⌖ 这里」标签；场源 `PM_FIELD_SOURCES` 末项特意放在 here 附近，形成轻微汇聚。

**② P0 — 场域从「装饰等势线」变为「地图主体」**：
- 删除 `PM_FIELD_CENTERS` + `pmFieldLoop` + `pmClosedSmooth` 的同心闭合等势线。
- 新增 `PM_FIELD_SOURCES`（4 个确定性场源，含 here 弱吸引子）+ `pmFieldVel()` 向量速度场 + `pmFlowStreamline()` 双向积分 + `pmSmoothOpen()` 平滑开放曲线。
- 从四条舞台边缘释放 streamline，形成**连续、非均匀、非闭合、汇聚/疏散/转向**的向量流场；同一份 Compass 形态稳定（无 random）。密度仍由 `PM_DENSITY.contours` 控制，当前中密度渲染 16 条开放流线。
- 场域线加粗、不透明度从 `.5/.4/.09` 提升到 `.72/.58/.14`（desktop stroke-width 1.25），使地图本身成为视觉主体；路线骨架 `.pm-trail` 透明度从 `.13` 提升到 `.24`，focus 到 `.65`。

**③ P0 — 交互从「移动文字节点」改为「扰动场域」**：
- `.pm-anchor-inner` 不再读取 `--pmx/--pmy` 做视差平移；文字节点**完全不动**。
- `.pm-contours` 与 `.pm-trail-layer` 读取 `--pmx/--pmy`，随 pointer 做极轻微整体 parallax，让「地图在回应你」。
- 保留 `.pm-field-attention` accent 色池 + focus 呼吸（当前 1 / 相关 ≈ .8 / 不相关 ≈ .5），但完全由 opacity 承担，不 scale/translate 文字。
- 删除 anchor 的 `pmBreathe` 上下浮动动画（卡片感来源）；地图的生命感改为来自流场 120s drift 与 field-attention。

**④ P1 — 清理「卡片/装饰/仪表盘」痕迹**：
- 删除 `.pm-anchor-inner::before` 的 soft radial glow（视觉上像卡片光晕）。
- 删除 `--pk` 深度视差系数（文字节点 pointer 平移的残留）。
- 删除 dead `.atlas-note-grid` 移动端规则与相关 `pm-anchor-inner::before` override。
- 保留并复用 `PM_ANCHORS`、`PM_TRAILS`、`buildMapNodes`、`buildMapRelationships`、`getCompassData`、Detail Panel、Clue Layer、Focus、键盘可达性。

**⑤ P2 — Mobile 零改动**：
- 移动端仍使用 Vertical Perceptual Field（纵向滚动 + IntersectionObserver），流场退为极淡静态背景（`.pm-atmos { opacity: .38; }` + 动画关闭），不抢纯文字阅读。
- 桌面 Living Field 重构不牵连任何 mobile CSS/JS。

**验证结果**

| 项目 | 结果 |
|---|---|
| JS 语法 | `node --check` PASS（提取全量 app script） |
| 数据层回归 | STATE_VERSION=7、PM_ANCHORS=6、PM_TRAILS=3、buildMapNodes / buildMapRelationships / getCompassData 全部保留，无新语义关系 |
| 结构探针（puppeteer 1440×900） | 16 条 `.pm-contour`、6 个 `.pm-anchor`、`.pm-datum` 存在、0 重叠（>2px tolerance） |
| pointer 交互 | `--pmx` 0 → 0.227（场域/路线 parallax 响应），文字节点无 `transform: translate3d` 依赖 `--pk` |
| 视觉范式 | 全页截图显示流场作为空间骨架贯穿六个 anchor，路线可见，datum 位于 05，第一眼更像「方向地图」而非「六个 section」 |
| Mobile 390 | 纵向场完整保留、无横向溢出、地图 tab 正常 |
| 禁止项扫描 | 无同心圆/六卡片/dashboard/glassmorphism/neon/粒子/真实地形插画；无新增语义关系；无装饰性元素解决「普通」问题 |

**未动（逐项核实）**：STATE_VERSION(7) / STORAGE_KEY / STEPS / answer key / state schema / localStorage 写入 / `getCompassData` / `buildMapNodes` / `buildMapRelationships` / `findNodeRelations` / `buildPerceptualModel` / `PM_ANCHORS` / `PM_TRAILS`（3 条） / Record Mode / Export / Experiment Feedback / 继续探索 / 加密备份 / Detail Panel / Clue Layer / 单文件离线约束。

---

## v5.45（2026-09-13）— Living Field / Map Atmosphere Refinement（活场域 · 地图氛围打磨）【基于 v5.44】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.45.html`（v5.44 不动）
**配套脚本**：`_apply_v545_patch.py`（17 处锚点全部一次命中，纯 LF，EOL 自适应，可重放生成 v5.45）
**验证**：`_verify_v545.js`（无依赖 vm 冒烟测试，28 PASS / 0 FAIL）· `_shot_v545.js`（puppeteer 真实渲染 1440/390/reduced-motion，22 PASS / 0 FAIL）· `_shot_v545b.js`（高密度 + 长文 + resize + 键盘焦点，12 PASS / 0 FAIL）
**种子**：复用 `_seed_v542_dense.json` + v5.45 自带长文/高密度种子（极长 life_vision + 6 core_values + 4 work_constraints + 全 reality_context）

**本轮目标**：在 v5.44「六站式个人地图」的空间结构之上，只打磨 **Map Mode 的呈现氛围**——把「六个排布整齐的 editorial 内容块 + 很淡的背景圆圈」推进为「一张连续、有空间感的抽象地形场域」。**不重做架构、不动任何数据层与六站布局**，只调呈现层。

**① 定位与硬边界**：本轮是 Atmosphere Refinement，不是第六次结构改造。明确不碰：STEPS / answer key / state schema / STATE_VERSION(7) / STORAGE_KEY / localStorage / `getCompassData` / `buildMapNodes` / `buildMapRelationships` / `findNodeRelations` / `PM_ANCHORS` 语义 / `PM_TRAILS` 语义 / Record Mode / Export / Experiment Feedback / 继续探索 / Journey Flow / 六站网格布局。新增视觉 token ≤ 2 个（`--fx` / `--fy`，且复用既有 `--pmx/--pmy` 管线派生）。

**② P0 — 重建背景场域（Background Field）**：删除 v5.44 残留的同心 `PM_CONTOUR_PATHS`（[70,165,260,355,445,525] 被 `circle` 均匀拉伸为扁椭圆）。改为 `PM_FIELD_CENTERS`（3 个确定性的 field center）+ `pmFieldContours()` / `pmFieldLoop()` / `pmClosedSmooth()` 生成的**闭合等势线**——沿角度采样 `半径 = base·(1 + 正弦叠加扰动)`，确定性、**无 random**，同一份 Compass 每次刷新形态稳定；轻微不对称、间距不一致、局部波动，呈现「等势线」而非「轨道」。层数仍由 `PM_DENSITY.contours`（2/3/4）控制。描边沿用 `--pm-hair`（alpha .07），极淡、绝不抢文字。

**③ P0 — Pointer → Field（复用既有管线，不移动地图）**：`pmOnPointerMove` 在写 `PM.tx/ty`（既有视差）的同时，新增写 `PM.fxt/fyt`（0–100% 落点）并调 `pmUpdateField()`；`pmTick`（既有 rAF）在写 `--pmx/--pmy` 的同屏追加缓动写 `--fx/--fy`（`.10` 系数）。`pmUpdateField()` 只切换 `has-field` 类 + 计算落点，**绝不**改任何节点位置或文字。`.pm-field-attention` 是一个 `position:absolute; inset:0; z-index:0; pointer-events:none` 的极淡 accent 色池，`radial-gradient(... at calc(var(--fx,50)*1%) calc(var(--fy,50)*1% ...)`，`has-field` 时 opacity `.14`（reduced-motion `.1`）。整条链路零新监听、零新布局读。

**④ P1 — 路线变「current」+ 空间聚焦呼吸式**：`updatePmGeometry()` 为每条 trail 追加一条 `.pm-trail-current`（同 `d`，极淡宽带 `stroke-width:5; opacity:.045; dasharray 46 230; animation pmCurrent 13s`），让路线像「内部有非常轻的流动」，不新增任何关系（`PM_TRAILS` 一字节未动）。聚焦层级从「硬降透明度」改为呼吸式：**当前 = 1 / 相关 ≈ .8 / 不相关 ≈ .5**（非 reduced-motion 与 reduced-motion 两处同步），focus 时整片场域 contour 微提（`opacity 1` / `.82`），让「被注意的区域更在」，但不喧宾夺主。

**⑤ P2 — 缓慢生命感 + 全 reduced-motion + Mobile 区分**：contour 加极慢 `pmContourDrift`（86s / 2n 112s，dashoffset 漂移）；anchor 呼吸（`pmBreathe` 19–30s，v5.44 已有，本轮保留并校验）。`prefers-reduced-motion` 下：contour drift 关、current 动画关（保留 `.06` 静态线 + 完整信息层级 + focus 可用）。Mobile（≤900px）：contour 强制静态（`.pm-contours path { animation: none }`）、trail-layer 隐藏（不复制 Desktop pointer 模式）、纵向滚动 + IntersectionObserver 不变——**Desktop 改动零牵连 Mobile**。

**⑥ JS / CSS 改动清单（共 17 处，全部以 assert 命中）**：JS 9 处（PM_FIELD_CENTERS + 3 个生成函数；`renderCompassMap` 用 `pmFieldContours`；atmos 加 `.pm-field-attention`；`PM` 加 `fx/fy/fxt/fyt`；`pmOnPointerMove` 写落点；`pmOnPointerLeave` 调 `pmUpdateField`；新增 `pmUpdateField`；`pmTick` 写 `--fx/--fy`；`pmFocusAnchor` 调 `pmUpdateField`；`updatePmGeometry` 加 `.pm-trail-current`；`pmUpdateTrailActive` 选择器纳入 `.pm-trail-current`）。CSS 8 处（`.pm-contours circle→path` + drift + focus 微提；`.pm-field-attention`；`.pm-trail-current` + `pmCurrent`；keyframes `pmContourDrift`/`pmCurrent`；两处 focus 层级；移动端 contour 静态）。

**验证结果**

| 项目 | 结果 |
|---|---|
| vm 冒烟（结构零回归） | **28 PASS / 0 FAIL**：STATE_VERSION===7、PM_ANCHORS===6、PM_TRAILS===3、PM_TRAIL_SHAPE===3、STORAGE_KEY 不变、新函数存在、`pmFieldContours` 确定性且输出 `<path>` 非 `<circle>`、填充渲染含 `.pm-field-attention`/contours/6 站、空数据→空态、旧 `PM_CONTOUR_PATHS`/`.pm-contours circle` 已移除 |
| Desktop 1440 真实渲染 | **22 PASS / 0 FAIL**：六站全渲染、contour 为 path、3 条 current、field-attention 存在、contour 层数≥4、静止 anchor opacity≈1、pointer→`has-field`+`--fx` 写入、pointerleave 关、focus 呼吸收敛到 1、无横向溢出、无 console error |
| 高密度 + 极长 life_vision 回归 | **12 PASS / 0 FAIL**：六站全有内容、任意两 anchor 真实矩形零相交（v5.44 重叠真因未回归）、current 路线 opacity .045（不穿透正文）、contour 描边 alpha .070（极淡）、resize 1024/1280 零溢出、键盘 Tab 进被 dim 站点自动提升焦点、无 console error |
| 横向 overflow | Desktop 1440 / 1024 / 1280、Mobile 390 全部 `scrollWidth ≤ clientWidth` |
| 路线不穿透正文 | `.pm-trail-current` opacity .045、`.pm-contours path` 描边 alpha .07，文字始终清晰可读 |
| prefers-reduced-motion | contour / current 动画关闭，3 条 current 与 focus 状态仍保留 |
| Mobile 375 / 390 / 430 | 纵向流、trail-layer 隐藏、contour 静态、零横向溢出 |

**未动（逐项核实）**：STATE_VERSION(7) / STORAGE_KEY / STEPS / answer key / state schema / localStorage 写入 / `getCompassData` / `buildMapNodes` / `buildMapRelationships` / `findNodeRelations` / `buildPerceptualModel` / `PM_ANCHORS` / `PM_TRAILS`（3 条、6 字段一字节未动）/ `PM_TRAIL_SHAPE` / `PM_DENSITY` / `pmSatText` / `pmClip` / Record Mode / `buildCompass()` / Export(JSON|Markdown) / Experiment Feedback / 继续探索 / 加密备份 / 六站 CSS Grid 布局（`pmClampAnchors` 仍只清残留）/ Detail Panel / Clue Layer / 无 Canvas / 无 WebGL / 无 Three.js / 无 GSAP / 无 CDN / 无新依赖。

**已知遗留 / 自我批评**：① 场域注意力源是「整片 radial-gradient 色池跟随落点」，不是逐站高光——这是 P0 刻意选择（只让「被注意的区域更在」，不重画地图），但首次进入时注意力源较弱，需 pointer 移动或 focus 才显形。② contour 漂移是纯 dashoffset 位移（沿固定路径滑动），不是「地形在生长」；要更「活」需引入多帧相位，但那会逼近动画成本红线，本轮克制。③ `.pm-field-attention` 的 `radial-gradient` 在 `preserveAspectRatio="none"` 的 stage 上会被非均匀拉伸（与 v5.44 contour 同源限制），落点坐标按 stage 百分比计算，宽屏下径向略扁——视觉可接受，未做补偿。④ `--fx/--fy` 由 `pmTick` 的 rAF 缓动写入，headless 下若 rAF 不推进则 computed 值滞后，真实浏览器正常；测试已改为读 inline 落点。⑤ 该 Chrome 构建 `1px` 边框 computed 仍报 `0.57971px`（环境级量化，沿用 v5.44 判据）。

---

## v5.45.x（2026-09-13）— Map Mode · Clue Layer 浏览流修复（原地修改 v5.45）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.45.html`（v5.45 原地更新，v5.44 仍不动）
**修复点**：Desktop Map Mode 中，用户打开某地标「深入阅读」（Clue Layer）→ 点击某条线索 → 打开 Detail Panel → 关闭 Detail 后，原本的 Clue Layer 被 document 级点击委托误关，导致用户必须重新点开「深入阅读」才能继续浏览同地标的其他线索。
**改动**：`onCompassClick` 中「点击 Clue Layer 之外收起」的逻辑，增加对 `#compassDetailPanel` / `#compassDetailBackdrop` 的例外：当点击发生在 Detail Panel 内部（含关闭按钮）或 Backdrop 上时，不自动收起 Clue Layer。关闭 Detail 后，用户仍停留在同一地标的 Clue Layer，可继续点其他线索。
**验证**：新增 `_shot_v545_fix_cluelayer.js`，8 项断言 ALL PASS（打开 Clue Layer、点击线索打开 Detail、Detail 内点击不关闭 Clue Layer、关闭 Detail 后 Clue Layer 仍在、可继续打开另一条线索的 Detail）。
**未动**：Detail Panel / Clue Layer 的结构与 DOM、answer key、state、Map/Record 逻辑均不变；仅调整 document 委托的关闭条件。

---

## v5.44（2026-09-12）— Map Mode · Cartographic Stations（六站式个人地图）【基于 v5.43】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.44.html`（v5.43 不动）
**配套脚本**：`_apply_v544_patch.py`（纯 CSS + 一处 JS 局部调整，纯 LF）
**验证**：`_verify_v544.js`（jsdom 双文件对比，49 PASS / 0 FAIL）· `_probe_v544.js`（puppeteer 多视口压力测试，149 PASS / 0 FAIL）· `_cmp_geom.js` / `_ovl.js`（几何与重叠复现对照）· `_shot_v544.js` / `_shot_ovl.js`（对照截图）
**高密度种子**：复用 `_seed_v542_dense.json`

**本轮目标**：在保留 Personal Cartography / Perceptual Field 视觉语言、数据结构与认知逻辑的前提下，把 Desktop Map Mode 从「六个自由漂浮的内容块」推进为「六站式个人地图」——解决的问题是**空间结构**，不是装饰。

**① 根因（实测，非推测）**：旧版 Desktop `.pm-anchor` 是 `position:absolute` + `left/top` 百分比固定 + `width` 固定，而 `height` 完全由用户内容决定；`.pm-stage` 桌面又是固定高度 + `overflow:hidden` 的画布，内容装不下无处可去，`pmClampAnchors()` 只把 Anchor 夹回舞台边界、没有「邻居占位」概念。三者相冲时必然出问题。
实测复现（`_ovl.js`，同一份高密度存档 + 点击 life_vision「↕ 展开」）：

| 视口 | v5.43 | v5.44 |
|---|---|---|
| 1440 | **vision × experiment 重叠 288×199px** | 0 重叠，舞台 1087 → 2042px 随内容长高 |
| 1024 | **vision × experiment 重叠 215×248px**，且内容底 1422px 超出舞台底 1346px → **76px 被 `overflow:hidden` 裁掉** | 0 重叠，舞台 1061 → 2016px |

需要说明的边界：**仅「内容变多」但未展开时，v5.43 在本项目的测试数据下并未重叠**（`_cmp_geom.js`：dense / extreme 两种存档 × 1440/1280/1024/960 全部 0 重叠、0 裁切）。真正稳定复现重叠的是「Anchor 高度出现无界增长」的路径（如 life_vision 展开），以及用户内容显著超过既有上限的情形。因此本轮不是修补某个坐标，而是移除「固定 x/y + 可变高度」这个结构冲突本身。

**② 结构解法**：Desktop `.pm-stage` 改为 CSS Grid station 布局。

```css
@media (min-width: 901px) {
  .pm-stage {
    display: grid;
    grid-template-columns: minmax(0, 1fr) minmax(0, .94fr) minmax(0, 1fr);
    grid-template-rows: repeat(4, auto);
    column-gap: clamp(32px, 6.4vw, 116px);
    row-gap: clamp(44px, 6.5vh, 92px);
    align-items: start;
    height: auto;
    min-height: clamp(560px, 74vh, 780px);
    overflow: visible;
  }
}
```

行高由该行内容决定、行间距恒定 → 同一行的两个站点各占一列，**重叠从结构上不可能发生**；地图随之可以变高（一张「随内容变高的测绘图」），不再是「把内容硬塞进固定画布」。

**③ 空间语法（六站 = `PM_ANCHORS` 声明顺序，语义一字节未动）**

| 站点 | 区域 | 认知角色 |
|---|---|---|
| 01 `vision` 我想去哪里 | r1，跨三列居中 | 远景 / 唯一长文本例外 |
| 02 `values` 我在意什么 | r2c1，右对齐 | 与 03 左右对望 |
| 03 `boundary` 我不想走向哪里 | r2c3，左对齐 | 与 02 同层对望 |
| 04 `direction` 我想往哪里靠近 | r3，居中 | 工作方式层 |
| 05 `here` 我现在站在哪里 | r4c1，右对齐 | 与 06 同层 |
| 06 `experiment` 我想弄清楚什么 | r4c3，左对齐 | 与 05 同层 |

阅读方向固定为：**想去 → 判断（在意 / 边界）→ 工作方向 → 现在 → 下一步**，不再是六个平级模块。

**④ Anchor 去卡片化**：新增 01–06 站点编号（取 `PM_ANCHORS` 声明顺序，**不新增语义**，桌面 `aria-hidden`，移动端纵向顺序与之天然一致）；新增极细 vertical rule 作为 station mark（不是卡片描边、不是容器）；去 `left/top` 坐标定位，宽度改由列宽决定（`width:100%` / `max-width:300px`）。

**⑤ `.pm-sat` 从 tag 退成 cartographic annotation**：无 capsule、无矩形边、无内边距、去圆角；hover / focus 才出现一道极轻的高亮下划线（`inset 0 -.5em 0` 的 accent-soft），focus-visible 保留 2px outline。注记横向间距放宽（`gap` 列 9→16px），避免又读成一行 chips。语义点 / 字号 / 层级 / `type` / `sourceKey` / 关系逻辑 / `PM_DENSITY` 全部不变。

**⑥ 路线减法（`PM_TRAILS` 真实关系一字节未动）**：新增纯呈现层 `PM_TRAIL_SHAPE`，按 trail id 索引「怎么画」——`{ bow }`（弧度，最多 64 → 38px）与 `{ via: "right-corridor" }`（走中列与右列之间的空走廊）。`PM_TRAILS` 仍是 3 条、每条仍是原有 6 个字段，未混入任何呈现参数。线更克制：线宽 1.1→1、静息 opacity .16→.13、active .76→.62；注记移到沿真实路径取中点，并加 `paint-order` 底色描边，避免被站点文字吃掉（不额外加背景块）。特别强化三条的语义：`here → experiment`（从这里出发）、`experiment → vision`（下一步仍朝向想去的方向）、`values → vision`（为什么这个方向对我有意义）。

**⑦ 大气减法，不加**：删除 6 个按百分比漂浮的 `.pm-glow` 光斑（站式布局下它们的坐标已与站点脱钩，会漂到空地上，且与 `.pm-anchor-inner::before` 的 soft glow 职责重复）；等高线由 3/4/6 降为 2/3/4；Anchor soft glow 起值 opacity .4→.22；次级地标静息下限 .84→.88、未成形地标 .72→.78；`lang="zh-CN"` 与 `.pm-haze` 大气层保留。「⌖ 这里」从固定百分比画布标记改为挂在 05 站自身左上角的 field mark，随站点移动。

**⑧ Detail Panel 只改视觉语言**：从「右侧 dashboard drawer」推进为 Field Note / 地图注记阅读页——`.cd-kicker`（letter-spacing .24em）、题签与「相关线索」各一道 `1px` section divider、原话 16.5px/1.95、阴影由 dashboard 感降为 `-14px 0 40px rgba(24,31,27,.07)`。**功能、数据来源、交互逻辑、focus trap、Escape 分层、`role=dialog` / `aria-modal` 全部不变**，未新造第二套全文阅读器。

**⑨ JS 改动只有一处**：`pmClampAnchors()` 不再写 inline `left/top`（px 覆盖会破坏 grid item 定位），只负责清理旧版残留。

**验证结果**

| 项目 | 结果 |
|---|---|
| jsdom 零回归（v5.43 vs v5.44） | **49 PASS / 0 FAIL**（Record / Export / state / storage / Detail 逻辑逐字节对比） |
| 多视口压力测试 | **149 PASS / 0 FAIL** |
| Case A 少量回答 @1440/1280/1024 | 内容盒 / 外框零重叠，无横向溢出 |
| Case B 六站全有内容 | 零重叠，三列落位稳定 |
| Case C–G 长文 + 多线索 + 全 latent | `vision` 262 / `values` 163 / `boundary` 130 / `direction` 201 / `here` 205 / `experiment` 221px，零重叠 |
| resize 1440 → 1024 → 1440 | 零重叠、无横向溢出、仍无 inline px |
| Clue Layer + focus 同开（4 个地标逐个） | 站点几何不变、实例数恒为 1、Esc 只关一层、每条 clue 有 `aria-label`、无 `aria-modal` |
| focus 态 | 仅被聚焦站点保持满不透明（`vision`=1，`values`/`experiment` 作为真实 related 提升，其余降到 .26–.38）；状态类 `vision* values+ boundary· direction· here· experiment+` 与 `PM_TRAILS` 完全对应；`direction` 不在任何真实 Path 中 → 不提升、也不虚构路线 |
| 键盘 | 可进入站点；Tab 进入被 dim 的站点时自动提升 |
| prefers-reduced-motion | 呼吸动画 / 过渡关闭，6 站与信息层级完整，零重叠 |
| Mobile 375 / 390 / 430 | 仍是纵向流（`position:static`、无网格、无绝对定位）、零重叠、无横向溢出、「更多线索」仍就地展开（无浮层） |
| Detail Panel | 打开正常、两道实线分隔生效、`role=dialog` + `aria-modal` 未变 |
| 站点入口结构（v5.43 vs v5.44，同种子） | `vision:btn/read` … `experiment:btn/read` **逐项一致** |

**未动（逐项核实）**：STATE_VERSION(7) / STORAGE_KEY / STEPS / answer key / state schema / localStorage 写入 / `buildMapNodes` / `buildMapRelationships` / `findNodeRelations` / `buildPerceptualModel` / `getMapDensity` / `PM_ANCHORS` / `PM_TRAILS` / `PM_DENSITY`（含 `sat: 5/4/3`）/ `pmSatText` / `pmClip` / Record Mode / `buildCompass()` / Export(JSON|Markdown) / Experiment Feedback / 继续探索 / 加密备份 / Mobile Vertical Perceptual Field（CSS 区块实测仅 1 行差异 = 移除已删除 `.pm-glow` 的陈旧 `display:none` 选择器）/ 无 Canvas / 无 WebGL / 无 Three.js / 无 GSAP / 无 CDN / 无新依赖。

**已知遗留 / 自我批评**：① 站点行高是「由内容决定 + 恒定间距」，所以行距是**下限保证**而非绝对值——内容差异大时留白仍会变多，稳定性来自「不会重叠」而非「间距永远相同」。② 01 与 04 是跨列居中，01 与 03/05 之间的左右空隙在宽屏下偏大，六站仍有可能被读成「三行两块」而不是一张连续的地图。③ 三次视觉减法之后，`PM_TRAILS` 的线更接近「结构记号」，代价是**首次进入时不一定注意到路线**，位置关系仍主要靠间距与编号承担。④ 部分地标在特定数据下没有主摘要（由 `summaryKeys` 命中情况决定），此时站点渲染 `.pm-head.is-static`、也没有「深入阅读」入口，只能通过线索进入完整原文——**这是 v5.43 之前就存在的数据层行为，本轮未改**。⑤ 该 Chrome 构建里 `1px` 边框的 computed 值报 `0.57971px`（裸页面亦然，属环境级 device-scale 量化），断言改为「存在实线分隔」。

---

## v5.43（2026-09-10）— Map Mode · Anchor Content Density【基于 v5.42】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.43.html`（v5.42 不动）
**配套脚本**：`_apply_v543_patch.py`

纯 CSS + 一处 JS 常量，收窄默认地图上各 Anchor 的内容展示量，避免「想去的方向」因内容过长在视觉上压到相邻节点：① Desktop `.pm-summary.is-vision` `max-height` 5.6em → 3.4em（≈2 行，保留渐隐）；② 非 vision summary 的 `pmClip` 上限 46 → 28 字符；③ `.pm-summary` 行高微调，确保截断点稳定。

**未改**：STEPS / answer key / state schema / localStorage / STATE_VERSION / `buildMapNodes` / `buildMapRelationships` / `PM_ANCHORS` / `PM_TRAILS` / `PM_DENSITY` / `buildPerceptualModel` / Clue Layer / Detail Panel / Record Mode / Export。

**说明**：这一版属缓解而非解决——它压低的是「摘要长度」，Anchor 的高度仍由内容决定，`position:absolute` + 固定百分比坐标的结构冲突未被触及。该冲突在 v5.44 被移除。

---

## v5.42（2026-09-10）— Map Mode · Perceptual Field Refinement【基于 v5.41】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.42.html`（v5.41 不动）
**配套脚本**：`_apply_v542_patch.py`（18 处锚点，全部一次命中，纯 LF）
**验证**：`_verify_v542.js`（jsdom 双文件对比，51 PASS / 0 FAIL）· `_accept_v542.js`（puppeteer 真实渲染，58 PASS / 0 FAIL）· `_edge_v542.js`（边界场景 7 PASS / 0 FAIL）· `_probe_v542.js`（几何/重叠/溢出）· `_probe_v542_text.js`（跨 Anchor 文本块重叠）· `_shot_v542.js`（21 张截图 light/dark × 1440/1280/1024/430/390）
**高密度种子**：`_make_seed_v542_dense.py` → `_seed_v542_dense.json`（6 Anchor 全有内容：vision 3+8 sats / values 3+7 / direction 3+6 / here 3+10 / experiment 3+1 / 长 vision 文本 / 10 个 core_values）

**核心原则**：让地图保持稳定，让内容按注意力逐层出现。
旧机制冲突：`.pm-anchor` 是绝对定位，点击「＋N 更多线索」把 `.pm-sat--latent` inline 展开 → Anchor 自身高度增长 → 不会触发其他 Anchor 重排 → 重叠（实测 v5.41：点 here 后 values×here 重叠 70–131px，here 高度 253→658px）。

**P0 —「更多线索」展开机制重做（Desktop）**

| 项 | v5.41 | v5.42 |
|---|---|---|
| 点击「＋N 更多线索」 | latent 展开进 Anchor（卡片长高） | 打开轻量 **Clue Layer**，地图几何零变化 |
| latent 默认状态 | 隐藏在 Anchor 内 | 不进 Anchor（`.pm-latent` 槽位整体 display:none） |
| 完整原文 | 既有 Compass Detail Panel | 不变（Map → Clue Layer → 既有 Detail Panel） |
| 浮层形态 | — | `#pmClueLayer`：340px / max-height min(66vh,560px) / 内部滚动 / 无 backdrop / 不锁 body scroll / 不移动任何 Anchor |
| 多实例 | — | 同一时间最多 1 个；切换地标 = 换内容 |
| 关闭 | 收起 | × 按钮 / Escape / 点击层外 / 切换地标 |

**新增 JS**（纯 UI，全部复用既有派生与焦点系统）：`openPmClueLayer` / `closePmClueLayer` / `renderPmClueLayer` / `pmPositionClueLayer` / `pmClueItemHTML` / `pmClueLayerEl` / `pmResetMoreButton`；clue 列表完全来自当前 Anchor 的 `sats`（`buildPerceptualModel` 派生），文字沿用 `pmSatText`。
**复用**：`pmFocusAnchor()` / `.pm-stage.is-focused` / `.is-focus` / `.is-related` / `trapFocus()` / `announce()` / `openCompassDetail()`。Escape 分层改为 Modal → Detail → Drawer → **Clue Layer** → Map focus，一次只关一层。

**新增 CSS**：`.pm-clue-layer`（+ `.pm-clue-head/-eyebrow/-title/-count/-close/-list/-item/-dot/-text`）、`.pm-anchor.is-clue-open { z-index: 3 }`、`.pm-latent`。

**P1 — Constellation 视觉减法（Desktop only，`min-width:901px`）**：`.pm-sat` 默认 border/background 全透明，只留语义点 + 文字（「· 自主 · 创造 · 时间自由」）；hover / focus 才出现极轻的 border + accent-soft 底。层级、字号、sourceKey、type、关系逻辑不变。附带修复：latent 槽位此前只隐藏 button，`<li>` 仍是 flex item，会在 `.pm-constellation` 留下多余 gap（10 个 latent ≈ 70px 空隙），现在 li 一起收起，默认地图各 Anchor 高度普遍变矮（如 here 312→307px @1024、direction 248→188px @1440）。

**P1 — 更多线索文案记忆**：`data-more-label` 记录「＋N 更多线索」原始文案；修复旧版收起后退化为「更多线索」丢掉条数的问题。

**P2 — life_vision 弱化卡片感**：accent 线 3px→2px 且降到 45% 透明度；accent-soft 底 60%→34% 且 75%→62% 更快淡出；去掉右上角圆角；左 padding 16→18px、上下 padding 14→10/12px；字号 17px / 行距 1.88 / 5 行折叠 / 「↕ 展开」按钮 / 完整原文在 DOM 全部保留。

**Mobile（≤900px）完全不动**：保持 Vertical Perceptual Field，`.pm-more` 就地展开（`@media (max-width:900px)` 内保留 `.is-open` 展开），无浮层，390/430/375 无横向溢出。

**可访问性**：`role="dialog"` 但**不写 `aria-modal`**（它不是 modal）；`aria-labelledby="pmClueTitle"`；打开时焦点进第一条线索；Tab / Shift+Tab 在层内循环（复用 `trapFocus`，实测连按 20 次不逃逸）；Escape 关闭并把焦点归还「＋N 更多线索」触发按钮；每条 clue 有 `aria-label="打开线索：…，查看这项线索的完整记录"`；不暴露 sourceKey / 节点 id / phase；reduced-motion 下关闭过渡但功能完整；print 隐藏。

**未动（逐项核实）**：STATE_VERSION(7) / STORAGE_KEY / STEPS / answer key / state schema / localStorage 写入（交互全程字节级不变）/ `buildMapNodes` / `buildMapRelationships` / `buildPerceptualModel` / `getMapDensity` / `PM_ANCHORS` / `PM_TRAILS` / `PM_DENSITY`（含 `sat: 5/4/3` 代表性线索数）/ `pmSatText` / `pmClip` / Record Mode 渲染 / `buildCompass()` / Markdown 导出 / Compass Detail Panel / Experiment Feedback / 加密备份 / Insight Layer / 无 AI / 无 API / 无 CDN / 无新依赖。剔除 `.pm-latent` 与 `data-more-label` 后 `renderCompassMap()` 输出与 v5.41 **逐字符一致**。

**已知遗留（非本轮引入，v5.30 即存在）**：① 1024px 下 `here` Anchor 底边距舞台底仅 ~14px，`pmClampAnchors` 会把它夹回，视觉略贴边；② SVG trail label（「从这里出发」等）在大段文字附近会与等高线交叠，属既有装饰层行为。

---

## v5.41（2026-09-10）— 继续探索两个提示词重构：报告式 → 阅读式【基于 v5.40】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.41.html`（v5.40 不动）
**配套脚本**：`_apply_v541_patch.py`（生成）/ `_verify_v541.js`（周屿实渲染 24 PASS / 0 FAIL）；真实 prompt 产物 `_prompt_v541/deep_reading.txt`、`_prompt_v541/career_direction.txt`

**问题**：旧 prompt 过度强调「完整覆盖 + 逐项对应 + 事实/归纳/未知三分 + 严格原文依据」，模型输出像结构化分析报告（一板一眼、机械复述、堆已知/未知、洞察不新），用户感觉「AI 只是把我写过的话重新整理了一遍」。

**改动**（仅 `AI_INSIGHT_TASKS` 内两个 `instruction` + 两个 `desc`）：
| 项 | 旧 | 新 |
|---|---|---|
| 深入阅读任务 | 找出反复主题 + 逐条原文依据 + 三分（原文/归纳/未知），且三分**重复出现 3 次** | 按「反复出现 / 彼此呼应 / 张力 / 被忽略的线索 / 值得继续想的问题」五类找**关系**；明确「优先找关系而不是逐题复述」 |
| 洞察预算 | 无（默认覆盖全部） | 3–5 个发现，真正重要的只有 2–3 个就只写 2–3 个 |
| 职业方向任务 | 「然后完成四步」+「每个方向都要说明对应原始线索」+「证据弱就不列」 | 先理解线索共同指向的**工作形态**，再提出 2–4 个真正有区别的**工作世界**；每方向回答「为什么浮现 / 可能长什么样 / 现在真正还不知道的是什么」 |
| 验证方向 | 1–3 个 | 收窄到 **1–2 个**，且必须给出具体验证动作（从业者聊 30 分钟 / 看 3 个真实案例 / 拆解岗位一周 / 小任务等） |
| 防幻觉 | 三分法（事实/归纳/未知）分栏输出 | 一句话：事实不能编造；解释可以有；解释必须保持为解释；没有足够依据就不下结论 |
| 深入阅读→职业方向 | 无 | 增加借用规则：可用其观察，但必须回原始记录核对，原始材料不支持的判断应放弃（不做数据流改造） |

**清理**：删除重复三次的三分要求、「请只做三件事」「请以：1. 反复出现的主题…」「然后完成四步」「证据弱就不列」等报告式约束；校验 `instruction` 定义仍为 **2 个**（无重复/残留）。

**未动**：keys 数据范围（13 / 17）、`aiAnswerUnits` / `buildAIInsightContext` / `buildAIInsightPrompt` 拼接方式与顺序、复制与剪贴板、隐私一次确认、modal / bottom sheet、STATE_VERSION(7) / STORAGE_KEY / STEPS / 数据结构 / Export(JSON|Markdown)（导出内容本就不含提示词，实测 5,585 字符正常）/ Map Mode / Record Mode / Insight Layer。

---

## v5.40（2026-09-10）—「AI 洞察」→「继续探索」轻量 Bonus 出口重构【基于 v5.39】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.40.html`（v5.39 不动）
**配套脚本**：
- `_apply_v540_patch.py`（25 处锚点 assert 补丁，全部一次命中，EOL-safe）
- `_verify_v540.js`（Puppeteer 68 断言·全 PASS）
- `_shot_v540.js`（16 张多视口截图 → `_shots_v540/`）

**性质**：产品权重重构——把 AI 从结果页的「正式结果分析模块」降级为结果之后**可选的 Bonus / 外部阅读出口**。用户第一眼感受到「这是我的 Compass」，滚到底部才发现有两个可以带走的提示词。零结构 / 零字段 / 零 answer key / 零 state / 零 localStorage 改动，STATE_VERSION 维持 7。

### 改动
- **名称统一**：用户可见「AI 洞察」→「继续探索」（section heading / modal eyebrow / close aria / announce）；内部函数名（`aiInsightSectionHTML` 等）保留旧命名降回归风险。
- **入口 3 → 2**：删除 `tensions`（找出张力）用户入口，只留 deep_reading / career_direction；`aiAnswerUnits` / `buildAIInsightContext` / `buildAIInsightPrompt` / `copyAIInsightPrompt` / 剪贴板回退 / 隐私确认 / modal-focus-Escape 全部保留。
- **删除顶部「AI 洞察 ↓」**：HTML + `#aiInsightJump` + listener + `.ai-jump-link` CSS 一并移除，不用「继续探索 ↓」替换（Bonus 靠滚动自然发现）。
- **删除 sparse 机制**：移除 `AI_SPARSE_THRESHOLD` 与「先完成更多 Compass 记录」引导；不评分、不阻断、不劝填；完全为空保留一句兜底 toast。计数改「本次会带上你已经写下的相关内容 · 约 N 字」（不出现 x/y、百分比、未填写数）。
- **视觉降权**：`.ai-insight`（3 列卡片）→ `.continue-exploring` editorial aside（纵向 2 张轻量卡、无背景填充、无阴影、hover 仅 border、margin-top 56px 认知断句）；modal 内分享范围 radio 降为次级（legend「本次会带上」、无 accent-soft 底 / 无描边）；主 CTA「复制提示词 + 我的回答」→「复制提示词」。
- **Prompt 角色重定义**：两个 prompt 均加入「请阅读这些材料，而不是解释这个人」+ 统一防幻觉纪律（原文明确出现 / 你的归纳 / 仍然未知 三分；材料不足直接说明；不补全信息）；显式禁用「你是一个……」「你的底层人格是……」「你潜意识里其实……」「你属于……」「你适合做……」「你天生适合……」等句式。
- **隐私文案微调**：「这些内容来自你自己写下的记录。复制后，只有在你主动粘贴到外部 AI 时，才会交给对应平台处理。」保留一次确认，不加第二层 modal / checkbox / 协议。
- print 隐藏选择器同步 `.ai-insight` → `.continue-exploring`。

### 未改
- STATE_VERSION(7)、STORAGE_KEY、STEPS（7 阶段 / 18 题）、全部 question id / answer key、`getCompassData()`、Map / Record / `buildMapNodes` / `buildMapRelationships` / `PM_ANCHORS` / `PM_TRAILS`、Experiment Feedback、Export(JSON|Markdown)、加密备份、Insight Layer、`aiPrivacyAcknowledged` 不落盘。

### 验证
- `_verify_v540.js`：68 PASS / 0 FAIL（结构 / 入口删除 / 两卡 / 视觉降权数值 / modal 减法 / 隐私流 / 二次直接复制 / Escape 与焦点归还 / 空内容兜底无劝填 / 7 视口无横向溢出 / bottom sheet / print 隐藏 / 无新增 state）。
- 截图目检：1440 / 1280 / 1024 / 768 / 430 / 390 / 375 底部 + modal + sheet + Map / Record 全貌，正式结果未受影响。

## v5.39（2026-09-09）— AI 洞察首次复制交互收尾（P1-6 视觉层级）【基于 v5.38】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.39.html`（v5.38 不动）
**配套脚本**：
- `_apply_v539_patch.py`（版本号 + changelog + 1 处 CSS 锚点 assert，EOL-safe）
- `_verify_v539.js`（Puppeteer 9 断言·全 PASS：首次点击不复制 / 主 CTA 变「确认并复制」/ 二次点击才复制 / 主 CTA 实心 primary 且框内确认键降为次级 panel 底无阴影）

**性质**：AI 洞察首次复制交互的视觉收尾。逻辑与隐私流（v5.37 已实现）复核确认正确；仅把隐私框内「我已看过，继续复制」从 `.soft-btn`（accent-soft 实心底，与隐私框同色、又和主 CTA 同色系，视觉上像第二个主操作）降为明确次级（panel 底 + accent 字 + 细描边、无阴影）。主 CTA「确认并复制」成为唯一强 primary。零结构 / 零字段 / 零 state / 零 localStorage 改动，STATE_VERSION 维持 7。

### 改动
- 复核 P1-6 逻辑（v5.37）：首次点击主 CTA → 仅弹「复制前看一眼」+ 主 CTA 文案改「确认并复制」+ 不复制；二次点击（框内确认键或再次点主 CTA）→ 置 `aiPrivacyAcknowledged`、隐藏框、真正复制。
- CSS `.ai-privacy-confirm` 改为次级样式。

### 未改
- AI 数据范围（13/11/17 keys）、sparse 逻辑、提示词中性化（v5.38）、Map / Record / Experiment Feedback / Export / Insight Layer、用户原话渲染、STATE_VERSION(7)、STORAGE_KEY、STEPS。

### 验证
- `_verify_v539.js`（Puppeteer 实渲染）：9 PASS / 0 FAIL。

## v5.38（2026-09-09）— AI 洞察提示词去性别化【基于 v5.37】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.38.html`（v5.37 不动）
**配套脚本**：
- `_apply_v538_patch.py`（18 处「他」→「填写者」锚点 assert）
- `_verify_v538.js`（jsdom 18 断言·全 PASS）

**性质**：把三段阅读提示（深入阅读 / 找出张力 / 寻找职业方向）中所有指代填写者的「他」改为中性「填写者」，避免女性 / 非二元填写者看到提示词被默认男他称呼冒犯。「用户」本就中性保持不变。零结构 / 零字段 / 零 answer key / 零 state / 零 localStorage 改动，STATE_VERSION 维持 7。

### 改动
- 18 处「他」→「填写者」（deep_reading 1；career_direction 17）。九个职业探索线索的第三人称全部以「填写者」中性重写；禁用规则「宣称填写者天生适合」「编造填写者未提供的经历」同步中性化。

### 未改
- 数据范围（13/11/17 keys）、sparse 逻辑、交互与隐私流、DOM/CSS 结构、STATE_VERSION(7)。

### 验证
- `_verify_v538.js`（jsdom）：18 PASS / 0 FAIL。

## v5.37（2026-09-09）— AI 洞察 UX 修补：三任务数据范围分化 + 结果页定位与复制反馈【基于 v5.36】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.37.html`（v5.36 不动）
**配套脚本**：
- `_apply_v537_patch.py`（17 处锚点 assert）
- `_verify_v537.js`（jsdom 41 断言·全 PASS）
- `_probe_v537.js`（Puppeteer 1440/390 smoke test + 响应式 / 溢出 / modal 探针）

**性质**：AI 洞察模块 UX 点版本修补。零结构 / 零字段 / 零 answer key / 零 state / 零 localStorage 改动，STATE_VERSION 维持 7，不接入任何 AI API / SDK / CDN。

### 目标

修补 v5.36 AI 洞察中真实存在的 8 个 UX 断点，让「把 Compass 带出去」更像一个顺手按钮，而不是需要学习的功能。

### 改动明细

| # | 优先级 | 位置 | 改动 |
|---|---|---|---|
| 1 | P0-1 | `AI_INSIGHT_TASKS.deep_reading.keys` | 由 19 个 key 收窄为 13 个生活/工作线索 key，不再近似完整 Compass；移除 `adaptability` / `experiment_goal` / `experiment_action` / `success_criteria` / `experiment_feedback` / `reality_classification` |
| 2 | P0-1 | `AI_INSIGHT_TASKS.career_direction.keys` | 由 19 个 key 调整为 17 个职业探索所需 key，聚焦「想去的方向 / 在意与取舍 / 边界 / 工作方式 / 现在的处境 / 下一步」；移除 `experiment_feedback`，`muted_version` 不默认纳入 |
| 3 | P0-1 | `AI_INSIGHT_TASKS.career_direction.instruction` | 压缩约 25%，去除「不是职业推荐 / 不是结论 / 不替用户决定」的重复表述，保留总原则、四步顺序与全部禁用规则 |
| 4 | P0-2 | `renderAIInsightModal` | sparse 判断改基于 `buildAIInsightContext(taskId, null, aiInsightScope).count`，不再基于整个 Compass；`aiEmptyNote` 随 scope 切换同步显隐 |
| 5 | P1-1 | `aiInsightSectionHTML()` | 结果页 AI 洞察头部增加轻说明：「这里不会直接生成 AI 回答。选择一种阅读方式，复制后带到你常用的 AI。」 |
| 6 | P1-2 | Compass Header | 在 `.compass-header-actions` 内增加轻量文字按钮「AI 洞察 ↓」，点击平滑滚动到 `#aiInsightHost`；非 primary、不新增 view state |
| 7 | P1-3 | CSS `.ai-radio` | 选中态强化：`border-color: var(--accent)` + `background: var(--accent-soft)`，`.ai-radio-main` `font-weight: 500`；同时用 `:has(input:checked)` + `.is-selected` 双保险保证兼容性 |
| 8 | P1-4 | `updateAIInsightPreview()` | 计数文案由「将复制 N 段你写下的内容」改为「已选 N 段内容 · 约 M 字」，基于 `estimateAITextLength()` 统计中文字符数 |
| 9 | P1-5 | CSS `.ai-preview` @media ≤720px | `max-height` 由 `34svh` 改为 `24svh` 并增加 `overscroll-behavior: contain`，减少 nested scroll |
| 10 | P1-6 | `copyAIInsightPrompt()` / `setAICopyLabel()` | 首次点击复制按钮触发隐私提示时，主 CTA 文案变为「确认并复制」 |
| 11 | P2 | `onAIInsightClick()` | 隐私确认后隐藏 `#aiPrivacyNotice`；主按钮在提示展开时再次点击也可作为「确认并复制」 |
| 12 | P2 | `copyAIInsightPrompt()` / `setAICopyLabel()` | 复制成功后按钮短暂显示「已复制 ✓」约 1800ms 后恢复基础文案；不新增 toast system，仍走既有 `showToast` + `announce()` |

### 明确没有修改

- 数据结构：`STATE_VERSION(7)` / `STORAGE_KEY` / `STEPS` / answer key 集合 / state schema / localStorage schema
- 现有模块：Map Mode（6 Anchor / 关系层 / `buildMapNodes` / `buildMapRelationships`）/ Record Mode / Detail Panel / Progress Drawer / Experiment Feedback / Export（JSON|Markdown）/ 加密备份 / 现有 Insight Layer
- AI 边界：不接任何 AI API / SDK / CDN；Inner Compass 自身不产生 AI 回答

### 验证

- jsdom（`_verify_v537.js`）：**41 PASS / 0 FAIL**
  - 三任务 keys 范围符合预期；`related` vs `full` 过滤正确；sparse 基于当前任务+范围
  - 结果页新增 hint / 选择一种阅读方式 / AI 洞察 ↓ 按钮 / 预览 `<pre>`
  - 辅助函数存在；函数无重复定义；STATE_VERSION 仍为 7；STORAGE_KEY 不变
  - CSS 规则存在；用户原话在 preview 中不二次 escape
- Puppeteer（`_probe_v537.js`）：**1440 / 390 全 PASS**
  - 无横向溢出；结果页 AI 洞察区块渲染
  - radio 选中态应用；复制计数文案正确
  - 切到「完整 Compass」后计数仍基于当前范围
- 语法检查：提取主脚本 `node --check` 通过

---

## v5.36（2026-09-09）— 「领导方式」清理 + 结果页新增「AI 洞察」出口【基于 v5.35】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.36.html`（v5.35 不动）
**配套脚本**：
- `_apply_v536_patch.py`（领导方式清理，4 处锚点 assert）
- `_verify_v536.js`（jsdom 48 断言·全 PASS）+ `_smoke_v536.js`（Puppeteer 真实存档回归）
- `_apply_v536_ai_insight_patch.py`（AI 洞察，40 KB 锚点 assert）
- `_verify_v536_ai.js`（jsdom 107 断言·全 PASS）
- `_probe_v536_ai.js` / `_probe_v536_ai2.js` / `_probe_v536_ai3.js` / `_probe_v536_ai4.js` / `_probe_v536_ai5.js`（多视口 + 极端长内容 + dark mode 探针）

---

### 变更一：把「领导方式」加入遗留标签清理清单

**性质**：一次性数据清理 + 幂等标记升级。零结构 / 零字段 / 零 answer key 改动，STATE_VERSION 维持 7。

**背景**：`领导方式` 在 v5.33 是 `work_constraints` 的预设选项，v5.35 换组时已并入「团队、管理与氛围」——**代码里早已不存在**（仅存在于文件头 changelog 注释）。用户仍能看到，是因为与 v5.34 同源的「幽灵 tag」机制：`renderTagOptions()` 会把「已选中但不在 options 里」的值渲染成 custom chip，并写入 `_custom_options` 持久化，**取消勾选也去不掉**。

| # | 位置 | 改动 |
|---|---|---|
| 1 | `LEGACY_WORK_CONSTRAINT_TAGS` | 追加 `领导方式`（6 项 → **7 项**） |
| 2 | 幂等标记 | `_purgedLegacyWorkTags` 由**布尔**升级为**版本号** `LEGACY_PURGE_VERSION = 2` |
| 3 | 判定条件 | `=== true` → `Number(state[LEGACY_PURGE_FLAG]) >= LEGACY_PURGE_VERSION` |
| 4 | 写入值 | `true` → `LEGACY_PURGE_VERSION` |

**为什么要版本化**：v5.34 的清理在很多存档里已经跑过、标记已是 `true`。直接往清单里加词不会触发。改成版本号后：`Number(true) === 1 < 2` → 升级到 v5.36 仍会再执行一次；跑完写入 `2`，之后不再重复。**以后再往清单加词，只需把版本号 +1**。

**清理范围**（不变，仍只有 3 处）：`work_constraints` / `_custom_options.work_constraints` / `reality_classification` 孤儿条目。

**未触碰**：`core_values`（「身体健康」仍是活预设项）、`work_environment` 的「协作方式」字段标签、`STATE_VERSION(7)`、`STORAGE_KEY`、Map anchor / 关系层 / `experiment_feedback`、v5.35 的 `hint` 与 `max: 3`。

---

### 变更二：结果页新增「AI 洞察」出口

**性质**：纯 presentation + 纯派生文本生成。**不是 AI 产品**，不调用任何 AI API，不新增后端 / SDK / CDN / 第三方依赖。

**核心定位**：把用户已经写下的 Compass 整理成适合交给外部 AI 阅读的一份文本 → 一键复制 → 用户自己粘贴到 ChatGPT / Claude / Gemini / Deepseek / 豆包。Inner Compass 自身不做 AI 分析 / 性格判断 / 心理诊断 / 职业推荐（与现有 Insight Layer 严格区分，**Insight = 产品内部透明规则；AI 洞察 = 把 Compass 带出去**）。

**新增代码**（仅纯函数 + UI 渲染 + 少量交互）：
- 1 个常量 `AI_INSIGHT_TASKS`（3 个任务配置）
- 1 个分组常量 `AI_INSIGHT_GROUP`（6 个自然中文分组）
- 3 个派生函数 `aiAnswerUnits()` / `buildAIInsightContext()` / `buildAIInsightPrompt()`
- 5 个渲染/交互函数 `renderAIInsightSection()` / `renderAIInsightModal()` / `openAIInsightModal()` / `closeAIInsightModal()` / `copyAIInsightPrompt()`
- 1 个模块级变量 `aiPrivacyAcknowledged = false`（仅页面生命周期，不落盘）
- 结果页共享区块 `aiInsightHost`（Map / Record 两种模式共用）+ 同一 Modal 节点（Desktop 居中 / Mobile bottom sheet 仅 CSS 切换）

**数据来源**：只复用既有 `getCompassData()`，**不直接读 `state.answers` 原始 JSON**——避免暴露 `life_vision` / `work_environment` 等内部字段名给用户。用户看到的永远是自然中文分组标题：

| 自然中文分组 | 对应 answer keys |
|---|---|
| 【想去的方向】 | `life_vision` / `ideal_day` / `end_feeling` / `desired_identity` / `muted_version` |
| 【在意与取舍】 | `core_values` / `trade_off` / `anti_future` |
| 【边界】 | `work_constraints` / `reality_classification` |
| 【工作方式】 | `work_environment` / `activity_types` |
| 【现在的处境】 | `reality_context` / `existing_stock` / `adaptability` |
| 【下一步】 | `experiment_goal` / `experiment_action` / `success_criteria` / `experiment_feedback`（如有） |

**三个任务（AI_INSIGHT_TASKS）**：

1. **深入阅读**：`看看不同回答放在一起后，哪些线索反复出现`。完整阅读提示（含三个阅读小节：反复出现的线索 / 值得继续看的地方 / 目前仍不确定的地方；明确禁止人格类型判断、心理诊断、职业适配测试）。
2. **找出张力**：`看看哪些愿望、选择或现实条件正在同时成立`。完整阅读提示（找出 2–5 个张力，引述原话，说明两端；明确禁止把普通差异夸大成冲突；信息不足时直说「目前信息不足」）。
3. **寻找职业方向**：`从你写下的生活与工作线索中，看看哪些方向值得继续探索`。完整阅读提示（四段：提炼职业方向线索 / 形成可能的职业探索方向 / 指出方向之间的差异 / 指出最值得验证的方向；明确禁止人格类型分析、心理诊断、MBTI、宣称"天生适合"、单关键词匹配职业、编造经历、把兴趣等同于职业适配、把某方向说成唯一答案、用"你真正适合的是……"等确定性语言）。

**v5.36 内最终迭代**：第三个入口原为「事实与猜想」，现替换为「寻找职业方向」。理由——「事实与猜想」更像 AI 输出时自带的方法纪律（区分事实/推测），而不是用户专门选择的一个任务；把它作为固定入口会让价值阶梯落到「学习认识论」而非真实用户会走的路。替换后三入口形成清晰价值阶梯：
```text
深入阅读      → 看见
找出张力      → 看清冲突
寻找职业方向  → 往职业探索延伸
```
「事实与猜想」的方法内核（区分用户原话 / AI 解释 / 未知）作为 AI 输出纪律被吸收进**每一个**任务的 instruction，不再单独占入口。任务 id 由 `fact_vs_guess` 改为 `career_direction`（keys 用全部 19 个 answer key——该任务语义就是「基于整个 Compass 提炼职业线索」，因此「相关内容」=「完整 Compass」输出一致，UI 仍允许切换）。

> **禁用词（本任务入口绝不使用）**：职业推荐 / 适合我的职业 / 职业匹配 / AI 职业分析 / 我适合做什么。这些词都会把产品语义推向「测试 → 匹配 → 答案」，而 Inner Compass 做的是从生活图景、工作方式、现实处境走向「下一步想弄清楚什么」。

**分享范围**：Modal 第一层只显示两个单选——「与本次阅读相关的内容（推荐）」/「我的完整 Compass」。**不在第一层显示 17 个字段 checkbox**。

**Preview**：点击「查看将复制的内容」打开只读 preview（`<pre>` 等价容器），顶部「将复制的内容」，底部「复制提示词 + 我的回答」按钮。**Preview 不允许编辑 Prompt**——这是稳定的产品模板，不是 Prompt Playground。

**Desktop 交互**：点击任务卡片 → 居中 modal，max-width **740px**，max-height **80svh**；header（标题 + 关闭）与 footer（主按钮）sticky，中间内容可滚动。

**Mobile 交互**：≤720px viewport → **bottom sheet**，`position: fixed; inset: auto 0 0`，max-height **90svh**，从底部滑入，sticky 操作栏，44px 最小触控区，无横向溢出，无 body 横向滚动。**禁止**：双栏缩小 / horizontal stepper / 多级 modal 套 modal / 左右滑动 / 复杂 accordion / 拖拽调整分享范围。

**可访问性（复用既有 v5.30.3 dialog 规范）**：
- `role="dialog"` + `aria-modal="true"` + `aria-labelledby`
- 打开记录 trigger，focus 到 close button / heading
- 关闭 focus 回到 trigger
- Escape **只关当前 AI Modal**（不影响 Detail Panel / Progress Drawer / Map focus）
- Tab trap / Shift+Tab 正常

**剪贴板双路径**：
```js
navigator.clipboard.writeText(text)        // 优先
catch → 隐藏 textarea + select() + execCommand('copy')  // 回退
```
成功用既有 `showToast('已复制。打开你常用的 AI，粘贴后直接发送即可。')` + 既有 `liveRegion` 播报（一次成功只发一次播报，不制造连续播报）。

**隐私双层**：
1. **首次复制前**显示「复制前看一眼」弹层（仅在 `aiPrivacyAcknowledged === false` 时触发），用户点「我已看过，继续复制」后置位，本次会话不再弹。**不写入 state / localStorage**，生命周期只限当前页面。
2. AI 区块下常驻一句：**「复制只会把内容放进你的剪贴板；是否发送给外部 AI，由你决定。」**——和 footer「所有数据仅保存在当前设备」定位一致，**绝不写"Inner Compass 会把回答发给 AI"**。

**空字段处理**：绝不输出「暂无数据 / N/A / undefined / null」——直接跳过。Compass 几乎为空时按钮显示「目前可供 AI 阅读的内容还比较少。你可以先完成更多 Compass 记录，再回来使用这里的阅读方式。」**不报错，不阻止 Results 页面**。

**注入防护**：用户原话可能含 `<` / `>` / `&` / Markdown / Emoji / 代码 / 网址。preview 全部 `textContent` 渲染（不二次 escape）；剪贴板文本原样保留——`用户写：<test>` → 复制仍是 `<test>`，不是 `&lt;test&gt;`。

**视觉系统**：复用现有 `--bg / --panel / --panel-strong / --ink / --ink-soft / --ink-faint / --line / --accent / --accent-soft / --radius-* / --shadow-*`。**禁止**：neon / cyberpunk / glassmorphism / 巨型渐变 / AI 星星 / 魔法闪光 / 聊天气泡 / 机器人 icon / `✨ AI`。

**响应式验证视口**：1440 / 1280 / 1024 / 768 / 430 / 390 / 375（light + dark），全无横向溢出；Modal 不溢出 viewport；bottom sheet 底部 sticky 操作栏始终可见；长中文 prompt 自然换行；button 不超过 viewport；focus ring 在所有交互元素存在；reduced motion 关闭 transition / transform。

**验证**（`_verify_v536_ai.js`）：jsdom **107 PASS / 0 FAIL**——三个任务可打开 / Prompt 互不相同 / 默认「相关内容」 / 可切换完整 Compass / preview 内容正确 / Clipboard 成功 / fallback copy 成功 / Toast 成功 / Privacy notice 首次出现且不重复 / 空内容不生成 undefined/null；不新增 answer key、STATE_VERSION 仍 7、STEPS 不动、localStorage schema 不动、Map 数据不动、Experiment Feedback 数据不动。

**Puppeteer 探针**：1440/1280/1024/768/430/390/375 + 极端长内容 + 真实 dark mode（state.theme = "dark"）——全绿。

**最终验收（10 秒完成）**：
```text
看见 AI 洞察 → 点「深入阅读」→ 看到隐私提醒 → 点「复制提示词 + 我的回答」
→ 打开 ChatGPT → 粘贴 → 发送
```

**零侵入边界（必须保持）**：
- 零新增 state / answer key / localStorage 字段
- 零修改 `STATE_VERSION(7)` / `STORAGE_KEY` / `STEPS` / `PM_ANCHORS` / `buildMapNodes` / `buildMapRelationships`
- 零修改 Map Mode（6 Anchor / L1 Landmark / L2 Constellation / L3 Field Detail / mobile vertical perceptual field / reduced motion / 无 WebGL）
- 零修改 Record Mode / Detail Panel / Progress Drawer / Export（JSON|Markdown）/ Experiment Feedback / 加密备份 / 现有 Insight Layer

---

## v5.35（2026-09-09）— 工作条件那一问：文案 + 选项组 + 选额上限【基于 v5.34】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.35.html`（v5.34 不动）
**配套脚本**：`_apply_v535_patch.py`（5 处锚点 assert）、`_verify_v535.js`（jsdom 45 断言·全 PASS）、`_smoke_v535.js`（Puppeteer 1440/390）
**性质**：纯文案层 + 选额上限（max 5→3）。零结构 / 零字段 / 零 answer key 改动，STATE_VERSION 维持 7。

### 改动明细

| # | 位置 | v5.34 | v5.35 |
|---|---|---|---|
| 1 | `work_constraints.guide` | 「……才能和你想靠近的生活相容？**哪些条件对你来说比较重要，不想轻易让步？**」 | 去掉与 label 重复的收尾句，只留「……才能和你想靠近的生活相容？」 |
| 2 | `work_constraints.hint`（新增字段） | — | 「从下面选出最多 3 项：哪些条件对你来说比较重要？」 |
| 3 | `work_constraints.max` | 5 | **3** |
| 4 | `work_constraints.options` | 13 条 | **11 条**（见下） |
| 5 | multitag 渲染分支 | 只渲染 `.helper-row` | 有 `hint` 时在其上方渲染 `.tag-hint`；无 hint 的题目输出与旧版一致 |
| 6 | CSS | — | 新增 `.tag-hint` + `.tag-hint + .helper-row` |

**label 未变**：`为了过上这样的生活，工作上有哪些条件你不想轻易让步？`

**新选项组（11 条，顺序即代码顺序）**：收入保障 / 工作稳定性 / 工作时间与灵活度 / 工作与生活边界 / 工作地点与通勤 / 出差与流动性 / 工作强度 / 自主空间 / 成长空间 / 团队、管理与氛围 / 对身体的负担

**选项映射**：工作时间规律性 → 工作时间与灵活度；出差频率 → 出差与流动性；学习与成长 → 成长空间；团队氛围 + 领导方式 + 组织文化 + 协作方式 → 合并为「团队、管理与氛围」；新增「对身体的负担」。

### 边界处理
- **未触碰**：`work_environment` 的「协作方式」字段标签（属另一题，勿混）、`core_values`（「身体健康」仍是活预设项）、`reality_classification` 的取数来源与四档 priority 数据值（must/prefer/flexible/unknown）、`STATE_VERSION(7)`、`STORAGE_KEY`、Map anchor / 关系层 / `experiment_feedback`、v5.34 的一次性清理逻辑。
- **旧存档兼容**：旧选项由 `renderTagOptions` 降级为自定义 chip 保留；已选数量超过新上限 3 时**不做截断**（避免静默删数据），取消勾选始终可用，不会锁死。

## v5.34（2026-09-09）— 清理早期版本遗留的工作条件标签【基于 v5.33】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.34.html`（v5.33 不动）
**配套脚本**：`_apply_v534_patch.py`（5 处锚点 assert）、`_verify_v534.js`（jsdom 33 断言·全 PASS）、`_smoke_v534.js`（Puppeteer 1440/390）
**性质**：一次性数据清理 + 极小代码增量（一个函数 + 两个入口调用）。

### 问题
用户要求删掉这 6 个 tag：自主安排空间 / 项目自主权 / 学习成长空间 / 家庭陪伴时间 / 身体健康 / 学习与成长空间。
核查结果：**其中 5 个在代码里早已不存在**（它们是 v5.31 / v5.32 的 work_constraints 预设选项，换组时已移除）。之所以还在界面上出现，是因为：
- `renderTagOptions()` 会把「已选中但不在 options 里」的值渲染成 **custom chip**；
- 并且这些值会被写进 `state.answers._custom_options` **持久化**——所以即使取消勾选，下次打开仍然显示。

### 做法
新增 `purgeLegacyWorkConstraintTags(force)`，在两个入口执行：
| 入口 | 行为 |
|---|---|
| `loadState()` | 载入后清理；清出东西就 `saveState()` 落盘（否则下次仍从旧存档重建） |
| `applyImportedState()` | `force=true`，导入外部 JSON / 解密备份时强制再清一次 |

清理范围（仅这 3 处）：
- `answers.work_constraints`
- `answers._custom_options.work_constraints`
- `answers.reality_classification` 中的孤儿条目（待分类项来自 work_constraints，标签没了分类也要跟着没）

### 明确没有删除的东西
- **`core_values` 里的「身体健康」保留不动** —— 它是 Phase 3「核心价值」的**活的预设选项**（还带专属 placeholder 释义），不是遗留项。本次只在「工作条件」语境下清除它。
- 任何其它 answer key、STEPS 结构 / 题目数 / id / type / max、STATE_VERSION(7)、STORAGE_KEY、Map anchor 与关系层、`experiment_feedback` —— 全部未动。

### 幂等设计
用 `state._purgedLegacyWorkTags` 标记，**只跑一次**。跑完之后用户仍然可以把这 6 个词当作自定义项自由添加（不会被反复吞掉）。导入路径用 `force=true` 绕过标记。

### 验证
- jsdom **33 PASS / 0 FAIL**：结构零改动、清除数量 8（3+3+2）、正常选项与用户自定义项保留、孤儿分类移除、core_values 不受影响、**真实加载路径**（往 localStorage 预置旧数据后启动页面）清理并落盘、幂等、force 生效、13 个新选项 / 四档新标签 / 实验回访模块仍完好。
- Puppeteer：1440×900 与 390×844，13 个 chip、每条目 4 个分类按钮、图例可展开、无横向溢出、无 JS 报错。

---

## v5.33（2026-09-09）— 工作条件选项再换组 + 现实分层四档标签改写【基于 v5.32】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.33.html`（v5.32 不动）
**配套脚本**：`_apply_v533_patch.py`（18 处锚点 assert，LF 保持）、`_verify_v533.js`（jsdom 56 断言·全 PASS）、`_smoke_v533.js`（Puppeteer 1440/390 冒烟）
**性质**：纯文案层。零结构 / 零字段 / 零 answer key / 零数据模型改动，STATE_VERSION 维持 7。

### A. Phase 5「从生活到工作」第二问 work_constraints
| 项 | 改动 |
|---|---|
| label | 不变：`为了过上这样的生活，工作上有哪些条件你不想轻易让步？` |
| guide | 去掉引出句「刚才写的是你想要的生活。现在把这些生活线索再往工作里翻译一步：」，直接进问题 |
| options | 换为 13 条：收入保障 / 工作稳定性 / 工作时间规律性 / 工作与生活边界 / 工作地点与通勤 / 出差频率 / 工作强度 / 自主空间 / 学习与成长 / 团队氛围 / 领导方式 / 组织文化 / 协作方式 |
| 自定义入口 | `+ 自定义` → `＋ 自定义`（全角，与文件内其它 ＋ 入口一致；该组件为 multitag 共用，故 core_values / end_feeling / adaptability 同步生效）|
| type / max | 仍 `multitag` / `5`（未改） |

### B. Phase 6「把方向放回现实」reality_classification
| 项 | 改动 |
|---|---|
| label | 不变：`现实里的调整空间` |
| guide | 重写为「前面想的是：什么对你重要。这里再往前一步：如果现实需要你做取舍，哪些条件你想守住，哪些还有调整空间？不是每个重要的条件都必须一成不变。你可以保留它对自己的意义，同时允许它换一种形式。」 |
| 四档按钮 | 必须保留→**守住**／希望保留→**尽量保留**／可以调整→（不变）／暂不确定→**还不确定** |
| 图例触发 | 改为 `不知道怎么区分？`（原句尾的自问句移入展开区） |
| 图例展开区 | 先给「问问自己：『这个条件改变以后，我愿意接受多大的变化？』」，再列四档释义（守住／尽量保留／可以调整／还不确定，释义全部换新） |
| 稍后再答 | 既有组件，未改动 |

**priority 数据值 `must / prefer / flexible / unknown` 一个都没变** → 旧存档、导入导出、Map 分层、localStorage 全部兼容。

### C. 下游同步（6 处，避免新旧标签并存）
`priorityLabel()` 映射 · Markdown 导出映射 · `buildCompass()` 现实分层四行 · `PRIORITY_META` 的 label 与 desc · 一条 Insight 文案 · 新增 `.classify-legend-ask` 最小样式。

### 明确没有修改
STEPS 结构 / 7 阶段 / 18 题 id 序列 / 每题 type·max / STATE_VERSION(7) / STORAGE_KEY / 6 个 PM_ANCHORS / buildMapNodes / 关系层结构 / `experiment_feedback` / 加密备份 / Phase 1–4 与 7 的任何文案。

### 验证
- jsdom **56 PASS / 0 FAIL**：结构零改动、13 选项顺序、新 guide、四档按钮与 data-priority 值、图例、6 处下游同步、v5.32 旧选项降级为 custom chip 且保留、旧 priority 值仍有效、活跃代码无旧标签残留。
- Puppeteer：1440×900 与 390×844 均渲染 13 个 chip、每条目 4 个分类按钮（守住／尽量保留／可以调整／还不确定）、图例可展开且含自问句、无横向溢出、无 JS 报错。

### 待你定的一点
选项从 11 增到 13，但**多选题上限仍是 5**（未擅自改）。13 条覆盖的维度比原来更细，5 个可能偏紧——要不要放宽到 6 或 7？

---

## v5.32（2026-09-09）— Phase 5 工作条件选项换组 + Phase 7 末问重写【基于 v5.31】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.32.html`（v5.31 不动）
**配套脚本**：`_apply_v532_patch.py`（9 处锚点 assert 补丁，LF 保持）、`_verify_v532.js`（jsdom 56 断言·全 PASS）、`_smoke_v532.js`（Puppeteer 1440/390 冒烟）
**性质**：纯文案层。零结构 / 零字段 / 零 answer key / 零数据模型改动，STATE_VERSION 维持 7。

### 改动清单

#### A. Phase 5「从生活到工作」第二问 work_constraints — 选项组整体替换
- 旧 11 项 → 新 11 项（顺序即最终顺序）：固定收入底线 / 工作时间边界 / 工作地点 / 通勤范围 / 加班与工作强度 / 出差频率限制 / 工作稳定性 / 工作生活边界 / 自主安排空间 / 学习与成长空间 / 团队与管理方式 / 家庭与个人时间。
- 移除：工作地点要求、时间自由度、学习成长空间、团队文化匹配、项目自主权、身体健康、家庭陪伴时间、稳定的人际关系。
- `type` 仍为 `multitag`、`max` 仍为 5、题目数与 id 不变；reality_classification 仍只从本题取待分类项。

#### B. Phase 7「还有想知道的事」最后一问 success_criteria — 问题 / 引导 / 三字段重写
- label：`试过之后，你想留意什么？` → `试的时候，你想留意什么？`（时间点前移到行动之前，不再要求预设评判标准）
- guide：改为「接下来真正去试的时候，可以留意几件事……这里不是提前给自己定标准，只是先想想，到时候你愿意看看什么。」
- 三字段 label + placeholder 全部换新示例（第二个字段 `这次发现了什么` → `看看有没有发现什么`，避免默认一定会有发现）。
- field key（energy_signal / learning_signal / identity_signal）不变。

#### C. 下游同步（4 处，避免新旧标签并存）
| 位置 | 改动 |
|---|---|
| `FEEDBACK_SIGNAL_FIELDS` | learning_signal label 同步为新标签 |
| `getCompassData()` 的 `experiment.successCriteria.fields` | 同上（Record / Map 字段标签自动跟随） |
| `buildCompass()` 实验坐标文本 | `这次发现了什么：` → `看看有没有发现什么：` |
| `buildMapRelationships()` evidence 引文 | 改为引用新 guide「这里不是提前给自己定标准……」 |

### 明确没有修改
STEPS 结构 / 7 阶段 / 18 题 id 序列 / 每题 type·max / STATE_VERSION(7) / STORAGE_KEY / 6 个 PM_ANCHORS / buildMapNodes / 关系层结构 / `experiment_feedback` 数据结构 / 加密备份 / Phase 1–4、6 任何文案。

### task3 审计结论：experiment_feedback 模块完整存在，未做改动
对 v5.31 与 v5.32 分别做了同一组存在性检查，结果完全一致：

| 检查项 | v5.31 | v5.32 |
|---|---|---|
| 6 个回访函数（get / write / has / build / renderRecord / open） | 全部存在 | 全部存在 |
| `.record-fb` 系列 CSS | 存在 | 存在 |
| Record Mode 07 章内「这次尝试之后」区块 | 渲染 | 渲染 |
| 区块位于 07 章内（即 Record Mode 最底部） | 是 | 是 |
| 入口按钮 `data-feedback-open="experiment"` | 存在 | 存在 |
| Markdown 导出「试过之后（探索回访）」章节 | 存在 | 存在 |

**找不到它的两个原因（非缺陷）**：
1. Compass 默认视图是 **Map Mode**（`currentCompassView = "map"`），而 Map Mode 的回访入口卡片在 **v5.30.5 被主动移除**（为给地图视区减负）；入口现在只存在于 Record Mode。
2. Record Mode 的回访区块有前置条件 `if (!exp.answered) return ""`——需先答过 Phase 7 任一题才会出现。

### 验证
- jsdom：**PASS 56 / FAIL 0**（结构零改动、新选项组、新文案、4 处下游同步、旧选项值降级为 custom chip 且保留、回访数据只写 experiment_feedback 不污染其它 key、Record 区块位置、Markdown 导出、面板可开）。
- Puppeteer 冒烟：1440×900 与 390×844 均正常加载、无横向溢出、无 JS 报错（仅 2 个 favicon/字体 404，属单文件离线预期）。

---

## v5.30.8（2026-09-08）— Welcome 首页文案修订 + 紧凑排版【基于 v5.30.7】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.30.html`（就地修改）
**配套脚本**：`_apply_v5308_welcome.py`（ops 可复现记录：forward 重建 == 主文件、reverse 还原 == baseline，双向逐字节）、`_verify_v5308.js`（jsdom 6 节·45 断言·全 PASS）、`_shot_v5308.js`（Puppeteer 4 视口 light + 2 视口 dark）、`_probe_v5308_compare.js`（base vs new 布局数值对比）
**报告**：`_report_v5308.md`

### 性质
- 范围：**仅 Welcome 首页文案 + 支撑多段的紧凑排版 CSS**。零数据层改动：STATE_VERSION=7、STORAGE_KEY、STEPS、7 阶段、18 题 id 序列与每题 type / max / placeholder / scaffold、6 个 PM_ANCHORS、buildMapNodes / buildMapRelationships、`experiment_feedback` 数据结构、Poster 装饰文案、加密备份、Markdown 导出 — **全部未动**。
- 全部改动为定向字符串替换 + 少量间距 CSS，无算法 / 渲染函数改动。

### 解决的问题
Welcome 首页文案整体修订：h1 措辞、左侧「把话变成自己的话」区块由超长单段重写为 5 段短句、chip 文案、hero-hint、右侧三张卡片标题与正文全部重写（「Compass 报告」→「方向记录」口径）。因 5 段正文高于原超长单段约 190px，若不处理会让 CTA「开始这一轮探索」被推出首屏（桌面 900px 视口底边 952>900、390 移动端 981>844），故同步做紧凑排版把 CTA 拉回全部目标视口的首屏内。

### 改动清单（按区域）

#### A. 文件头版本注记
- 顶部插入 v5.30.8 一行（不逐字引用文案，避免制造重复锚点破坏反向还原唯一性）。

#### B. hero 左侧文案卡（hero-copy）
- h1：「不是急着找答案。是慢慢整理线索。」→「不急着找答案，先慢慢把线索理清楚。」
- 原超长 `<p class="lead">` 整体重写并拆成 5 段：① lead「…一时说不清的感受，一点点变成自己的话。」② 从想要的生活开始…不想走向什么、现在站在哪里 ③ 留下一份方向记录：生活图景 / 看重的东西 / 想坚持的选择 / 还想弄清楚的问题 ④「这些都不是最终答案。」⑤ 把想法、现实和疑问放在一起，之后怎么走仍可调整。
- hero-note 三枚 chip：「先生活，后职业」→「先从生活出发」；「允许模糊，允许待定」→「可以模糊，也可以改主意」；「先观察，后定义」不变。
- hero-hint：「数据只保存在当前浏览器，不会上传服务器，可随时回来继续。」→「所有记录只保存在你的浏览器里，不会上传。你可以随时回来继续。」

#### C. 右侧三张卡片（panel-list）
- 卡1「你会从‘想过的生活’出发」→「你会从『想过的生活』出发」+ 正文「先看看你想靠近怎样的生活，再一点点把它想具体…」。
- 卡2「你会带走一份可回看的整理」→「你会留下一份可回看的记录」+ 正文「生活图景、在意的东西、想坚持的选择、现实处境和下一步尝试…属于你的方向记录」。
- 卡3 strong 不变，正文「可以留空，可以稍后再答，也可以下次回来修改。这份记录服务的是思考，不是完成率。」

#### D. 紧凑排版 CSS（保 CTA 首屏可见，文案一字未省）
- `.hero-copy` 上下 padding 42px → 24px（横向 clamp 不变）。
- `.hero h1` 上距 14px → 6px。
- `.hero p.lead` 规则改为 `.hero-copy p`（覆盖 5 段）+ 新增 `.hero-copy p + p { margin-top: 7px }`；段落字号 clamp(16px,2vw,19px) → clamp(14.5px,1.5vw,16.5px)、新增 line-height 1.58。
- `.hero-note` 上距 22px → 12px；`.hero-footer` 上距 32px → 14px。

### 明确没有修改
- 任何 STEPS / answer key / type / max / placeholder / scaffold；STATE_VERSION=7；STORAGE_KEY；buildMapNodes / buildMapRelationships / PM_ANCHORS；experiment_feedback 数据结构；Poster 装饰文案（Personal Compass / 把人生问题，慢慢变成自己的语言）；按钮「开始这一轮探索」文字与 id；Journey / Summary / Record / Map / 导出 / 备份全部未触碰。

### 验证
- jsdom：**45 PASS / 0 FAIL**（A 数据契约零改动 · B 新 Welcome 文案在场 · C 旧文案清零 · D 结构要点 · E 无 console 错误 · F 版本注记唯一性）。
- 可复现记录：`_apply_v5308_welcome.py` FORWARD（baseline→ops==主文件）与 REVERSE（主文件→baseline）双向逐字节 OK。
- Puppeteer 探针（vs v5.30.7 baseline）：desktop1440 / desktop1280 / mobile430 / mobile390 全部 `ctaInViewport=true`、`overflowX=0`；hero 高度 754 vs 基线 734（+20px 用于容纳 5 段）；3 chip 单行、3 卡高 106px 与基线一致；桌面 h1 三行 / 移动端三行与基线一致；hero-panel 右侧卡片等高于左卡（grid stretch），无横向溢出。
- 截图：`_shot_v5308_{desktop1440,desktop1280,mobile430,mobile390}_light.png` + `_shot_v5308_{dark1440,dark390}.png`。

## v5.30.7（2026-09-08）— Phase 7 认知重构：从「猜想与尝试」到「还有想知道的事」【基于 v5.30.6】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.30.html`（就地修改）
**配套脚本**：`_apply_v5307_phase7.py`（锚点 assert 补丁，纯 LF）、`_verify_v5307.js`（jsdom 6 节·55 断言·全 PASS）、`_shot_v5307.js`（Puppeteer 5 截图：Welcome / Phase 7 desktop+mobile / Summary / Map）
**报告**：`_report_v5307.md`

### 性质
- 范围：**仅文案 / label / 关系层 evidence 文案层**。零数据层改动：STATE_VERSION=7、STORAGE_KEY、STEPS、7 阶段、18 题 id 序列与每题 type / max / placeholder / scaffold 引导语、6 个 PM_ANCHORS、buildMapNodes、buildMapRelationships 数据结构、`experiment_feedback` 数据结构、加密备份、Markdown 导出结构 — **全部未动**。
- 所有改动均为定向字符串替换（~30 处），无算法 / 渲染函数改动。

### 解决的问题
用户完成前 6 阶段后突然被要求「挑一个念头去试」——产生「为什么现在突然要我行动」「是不是要把所有事都变成职业问题」等认知断点。Phase 7 旧版以「念头 → 行动 → 观察」三段式，把行动写成终点而非获得信息的手段，且示例严重偏职业（独处工作、自主时间、找从业者聊），让 Phase 7 ≈ 「职业验证」。

本次重构在 Phase 6「我现在站在哪里」与 Phase 7「下一步做什么」之间补上一层缺失的认知台阶——「我还有什么没弄清楚」，并把行动重新定位为「获得信息的一种方式」（与去 / 聊 / 观察 / 停 / 体验并列）。

### 改动清单（按区域）

#### A. 文件头版本注记（注释区）
- `Updated: 2026-08-17 / v5.25.5` 顶位插入 v5.30.7 改动说明（仅 changelog）

#### B. Welcome 轻量桥接
- `.lead` 末尾：「走完这一轮……以及一个可以开始尝试的小实验」→「以及一个想带去生活里弄清楚的问题。这份 Compass 不会替你决定答案——有些事情，只有放进真实生活里走过一点，才会慢慢变清楚。」
- Welcome 第二卡片「你会带走一份可回看的整理」：`现实地图和下一步尝试` → `现实地图和想弄清楚的问题`

#### C. Phase 7 phase / title / intro
- `phase: "猜想与尝试"` → `"还有想知道的事"`
- `title: "先试，再看"` → `"把问题带回生活"`
- `intro`：从「挑一个你还想知道的念头，先做一次尝试，再看看现实会告诉你什么」改为「从里面挑一个你还想弄清楚的问题，带它去现实里走一走」（明示「写出来 ≠ 有答案」+「不用做决定」）

#### D. Q1 `experiment_goal`（未决问题）
- `label`：「有没有一个关于生活或工作的念头，值得拿到现实里试试看？」→「回看前面写下的这些——现在还有什么，是你还没有答案、但很想弄清楚的？」
- `guide`：明示可同时有多个、一次先挑一个、不必挑「最重要」
- `placeholder`：同步
- `scaffold.prompts`（3 条）：① 强调「从来没真的验证过」② 增加「光靠想是想不清的」开放式追问 ③ 示例覆盖：长期独立 / 减社交 / 身份 vs 想象中 / 信息探索

#### E. Q2 `experiment_action`（靠近现实）
- `label`：「接下来一段时间里，你愿意怎么试一次？」→「为了多知道一点，你愿意先去做点什么？」
- `guide`：列出去 / 体验 / 聊 / 观察 / 刻意停下五种方式；不再以「行动」为单一指向
- `scaffold.prompts`（3 条）：① 最小版本仍是周期问题 ② 改为「它和前面写下的哪条线索有关」中性引导 ③ 示例覆盖：晚间不安排 / 能量高峰 / 聊普通一天 / 换城住几天

#### F. Q3 `success_criteria`（想留意什么）
- `label`：「这次尝试之后，你想留意什么？」→「试过之后，你想留意什么？」（去掉「这次尝试」预设）
- `guide`：明示「这不是给自己定成功标准」「不是为了证明你选对了」「只是为了比现在多知道一点」
- `fields[learning_signal].label`：`学习信号` → `发现信号`（语义从「学到新技能」转向「发现新信息」）
- 同步 `FEEDBACK_SIGNAL_FIELDS`（key 不变）

#### G. Phase 6 阶段反馈补承接句
- `generateStepFeedback("reality_bridge")` 末尾追加：「知道自己站在哪里之后，不必马上做决定——有时候，下一步只是去弄清一件还没有答案的事。」

#### H. Phase 7 阶段反馈（已填 / 未填 两态）
- 已填：「你已经把一个猜想变成了一次真实的尝试」→「你已经把一个还没有答案的问题，变成了一次可以去做的探索。它不是用来证明什么的——做过之后，现实会带回一点新的信息……」
- 未填：「挑一个你还想知道的念头，设计一次成本不高的小尝试」→「挑一个你还想弄清楚的问题，想一个成本不高的方式去碰碰现实」

#### I. Insight #8
- `content`：「从模糊到行动的关键一步」→「从模糊走向真实信息的关键一步」
- `sources`：`["现实地图", "实验设计"]` → `["现实地图", "还有想知道的事"]`

#### J. Record / 区块标题
- Compass Data 07 章 `title: "猜想与尝试"` → `"还有想知道的事"`
- `experiment: "猜想与尝试"`（Record sections） → `"还有想知道的事"`
- Detail Panel meta：`猜想与尝试 · 实验后回访` → `还有想知道的事 · 试过之后`
- Feedback 面板 announce / close-note：去掉「实验」二字
- `// 猜想与尝试` 注释 → `// 还有想知道的事`

#### K. Map 节点 / Anchor / Trail
- Map node field：`想试的念头` → `想弄清楚的问题`、`这次怎么试` → `准备怎么接触现实`
- PM_ANCHORS.experiment：`title: "我正在尝试什么"` → `"我想弄清楚什么"`；`empty.text` 同步
- PM_TRAILS：`reality_context → experiment_goal` evidence 文案改写；`summaryKeys` 把 experiment_goal 提前（问题优先于行动）
- 第二条 action evidence 同步 `experiment_action` 脚手架原文

#### L. Summary
- 报告项 label：`想拿去试一次的念头` → `还想弄清楚的问题`
- 报告项：`打算怎么试` → `准备怎么接触现实`
- 反馈章节表头：`— 实验后回访 —` → `— 试过之后（回访） —`
- successCriteria learning_signal 行 label → `发现信号`
- completion side note：`把下一步尝试留住：${experimentAction}` → `把想弄清楚的事带回生活里：${experimentAction}`

#### M. `buildMapRelationships`
- Journey Flow `reality_context → experiment_goal`：`label / source / evidence` 全部改写（去掉「试」，强调「弄清」，明示「不限于工作」）
- Experiment Chain 第一环：`念头 → 最小行动` → `问题 → 靠近现实`，`source` 同步阶段名
- Experiment Chain 第二环：`行动 → 判断标准` → `探索 → 想留意什么`，evidence 用新 guide 原文
- Stock → Experiment：`to` 后缀「下一步尝试」→「下一步探索」，label 「已经有的 → 想试什么」→「已经有的 → 还想弄清楚什么」

#### N. Markdown 导出
- `## 实验后回访（这次尝试之后）` → `## 试过之后（探索回访）`

### 明确没有修改
- 所有 `answer key`：`experiment_goal` / `experiment_action` / `success_criteria` / `experiment_feedback` 及其子字段 `energy_signal` / `learning_signal` / `identity_signal` —— 全部未动
- STEPS 7 阶段 id、18 题 id、每题 type / max —— 全部未动
- STATE_VERSION=7、STORAGE_KEY=`inner_compass_v5_3` —— 全部未动
- `experiment_feedback` 数据结构、Record / Map 节点 region / sourceKey —— 全部未动
- 顶部历史 changelog 注释（含 v5.10 的 `想拿去试一次的念头`、v5.5 的 `决定职业方向` 等）—— **故意保留**（这些是历史版本的说明，不是 UI 文案）

### QA 报告（55 PASS / 0 FAIL）
- A 数据契约：STATE_VERSION=7 / STORAGE_KEY / 7 阶段 id / 18 题 id / 每题 type / 每题 max / experiment_goal 键可用 / 三字段 key 不变 / experiment_feedback 可选不计入进度 / 全站引导语无成功/失败/KPI 词汇
- B Phase 7 新文案：phase / title / intro / Q1 label+guide+scaffold / Q2 label+guide+scaffold / Q3 label+guide+fields label（发现信号）
- C 旧 UI 文案清零（10 项，全部已清除）
- D 同步点：Welcome / Phase 6 反馈桥 / Phase 7 反馈 / Insight 8 / Map anchor / Summary / 关系层三处 / 导出章节名（12 项）
- E 渲染层：切到 stepIndex=6 后 Phase 7 新文案在 DOM 中渲染、无 console / jsdom 错误
- F 旧档兼容：旧档无 `experiment_feedback` 启动无错

### 配套截图（5 张）
- `_shot_v5307_01_welcome.png` — 1280×900，Welcome 新 lead 在 DOM 中呈现
- `_shot_v5307_02_phase7.png` — 1280×900，Phase 7 步骤条 / phase / title / Q1+guide / Q2+guide 渲染
- `_shot_v5307_03_summary.png` — 1280×900，Summary Report 视图（方向地图 tab）
- `_shot_v5307_04_map.png` — 1280×900，Map 模式（部分 anchor 在 viewport 内）
- `_shot_v5307_05_phase7_mobile.png` — 390×844，移动端 Phase 7（标题 / Q1 label 自然换行，无横向溢出）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.30.html`（就地修改，v5.30.5 状态保留为基线快照 `_pre_v5306_baseline.html`）
**配套脚本**：`_apply_v5306_copy.py`（6 处锚点补丁，纯 LF）、`_verify_v5306.js`（jsdom 8 节·86 断言）、`_shot_v5306_p6.js`（Puppeteer 4 视口 × 27 断言）
**截图**：`_shot_v5306_desktop-1440.png` / `_shot_v5306_desktop-1024.png` / `_shot_v5306_mobile-390.png` / `_shot_v5306_mobile-430.png`

### 性质
- 范围：**仅文案层**。零数据层改动：STATE_VERSION=7、STORAGE_KEY、STEPS、7 阶段、18 题 id、每题 type / max、6 个 PM_ANCHORS、buildMapNodes、buildMapRelationships、Record、Map、Insight、加密备份、导出 — **全部未动**。
- 全部 6 处改动均可在 v5.30.5 基线上做「精确字符串反向替换」还原（详见 §B）。

### 改动清单（6 处）
| # | 位置 | 旧 | 新 |
| --- | --- | --- | --- |
| 1 | `reality_context.fields[income].placeholder` | 例如：稳定 / 可接受变化 / 有不能退让的底线 | 例如：目前较稳定 / 近几个月需要控制支出 / 可以接受一段时间的变化 |
| 2 | `reality_context.fields[location].placeholder` | 例如：留在本市 / 可考虑迁移 / 暂不受限 | 例如：目前最好留在本地 / 可以考虑换城市 / 暂时没有固定地点要求 |
| 3 | `reality_context.fields[time].placeholder` | 例如：时间有限 / 每周可固定投入 / 可阶段集中 | 例如：只能零碎投入 / 每周有固定时间 / 某一阶段可以集中投入 |
| 4 | `reality_context.fields[responsibility].placeholder` | 例如：家庭照顾 / 工作承诺 / 暂无特别 | 例如：需要照顾家人 / 已有一些工作或生活承诺 / 目前没有特别需要配合的安排 |
| 5 | `reality_context.fields[energy].placeholder` | 例如：稳定 / 易疲惫需恢复 / 有长期状况 | 例如：精力比较稳定 / 只能低强度投入 / 现阶段需要优先留出恢复时间 |
| 6 | `adaptability.options`（7 → 15，含改名 1 + 新增 8；保留用户给的顺序） | ["学习新的技能", "调整工作方式", "减少非必要消耗", "改变当前生活节奏", "尝试新的城市或环境", "接受短期不确定性", "寻找新的合作方式"] | ["学习新的技能", "调整工作方式", "尝试不同的工作节奏", "探索新的收入来源", "寻找新的合作方式", "改变当前生活节奏", "调整居住安排", "尝试新的城市或环境", "减少非必要消耗", "暂时减少一些固定承诺", "先从小规模尝试开始", "给自己留出更多试错空间", "接受一段时间的不确定性", "主动寻找新的信息或资源", "找人一起讨论或尝试"] |

### 明确没有修改
- 题目 label / guide / scaffold 引导语：均未动。
- `adaptability.max` 仍为 **5**（用户没提，未改；多选保留「挑最重要的几个」语义）。
- `reality_context.fields[*].label`（收入与经济情况 / 地点与生活环境 / 可投入时间 / 其他需要考虑的责任 / 健康与精力状态）：未动。
- 「＋ 自定义」内联入口：未动（multitag 已原生支持，新旧选项均能写入 `_custom_options`）。
- 5 个新选项（"尝试不同的工作节奏" / "探索新的收入来源" / "调整居住安排" / "暂时减少一些固定承诺" / "先从小规模尝试开始" / "给自己留出更多试错空间" / "主动寻找新的信息或资源" / "找人一起讨论或尝试"）和改名后的「接受一段时间的不确定性」均按用户给的顺序排入，**未擅自打散或归类**。
- 移动端 chip 行高未动（沿用既有 .tag padding 3×9），chip 内部尺寸允许文本在 chip 内换行（无 nowrap 约束），实测最长项「先从小规模尝试开始」（10 字）未超单 chip。

### 旧存档兼容
- 旧存档里若勾选了「接受短期不确定性」，不会丢：`renderTagOptions` 中 `!question.options.includes(v)` 的判断把它降级为「custom 类 chip」继续渲染并保留在答案数组里。
- 跨验证：seed = `{adaptability: ["接受短期不确定性", "学习新的技能", "我自己的旧自定义词"]}` → 渲染 17 chip（15 预设 + 2 旧值），旧词呈 `tag custom active` 三态，勾选其它项后旧词仍在答案数组里。

### 视觉实测（puppeteer-core，P6 步骤 = reality_bridge）
| 视口 | chip 数 | 行数 | 单 chip 宽 | 单 chip 高 | 文档横向溢出 | 结果 |
| --- | --- | --- | --- | --- | --- | --- |
| 1440 light | 15 | 4 | ≤205px | 49px | 0 | ✓ |
| 1024 light | 15 | 4 | ≤205px | 49px | 0 | ✓ |
| 390 light | 15 | 13 | ≤205px | 49px | 0 | ✓（长但单 chip 触控舒适） |
| 430 light | 15 | 10 | ≤205px | 49px | 0 | ✓ |

- 移动端 chip 行数较多（≈ 10–13 行 ≈ 490–640px 纵向空间），chip 区纵向变长是文案扩展的**直接代价**；如后续觉得偏长可考虑给 chip 区加二级分组（学习 / 生活节奏 / 居住与地点 / 心态）— **本轮未做**。
- 5 个 placeholder 全部正确落到 `<textarea>` 节点。

### 验证汇总
| 套件 | 节数 | 断言数 | 结果 |
| --- | --- | --- | --- |
| jsdom `_verify_v5306.js`（数据契约 / 精确 diff / 5 个示例 / 15 选项 / 渲染层 / max=5 / 旧存档兼容 / Record+Markdown） | 8 | **86** | ✓ 0 FAIL |
| Puppeteer `_shot_v5306_p6.js`（4 视口 × 5–7 断言） | 4 | **27** | ✓ 0 FAIL |
| **合计** | | **113** | **0 FAIL** |

### 教训（写入技能候选）
1. **`max` 不是建议，是渲染上限**。本轮扩容后 `max: 5` 仍生效，jsdom 通过模拟连点 6 个 chip → 答案数组停留在 5 个（`if (current.length < question.max) next = [...current, value]`）。升级选项时不调 `max` 不会自动放宽选上限。
2. **jsdom 无 `URL.createObjectURL` / Blob**：拦下 `downloadFile` 取字符串比试造 jsdom polyfill 简单得多：`window.eval("downloadFile = function(c){ window.__md = c; };")` → `exportMarkdown()` → `window.__md`。
3. **STEPS 类型断言**：每题 type 是字面字符串常量，必须用探针得到当前真实序列做精确断言（不可凭印象，否则 v5.x 的 `valuesorter` 会被误写为 `value_ranking`）。

## v5.30.5（2026-09-07）— Map Mode 视觉收尾【基于 v5.30.4】

### 改动清单（8 处锚点补丁）
- **CSS** (`.pm-summary.is-vision`)：默认加 `max-height: 5.6em` + 上下 mask-image 渐隐尾段；新增 `.is-vision-open { max-height: none; mask-image: none }`；新增 `.pm-vision-toggle`（圆形紧凑 12px 按钮，aria-expanded=true 时字色加深）。
- **CSS 移动端**：≤900px 收紧到 `max-height: 4.2em`，≤560px 再紧到 `max-height: 4em`。
- **JS renderPmAnchor**：新增 `const visionOpen = false;`（生命周期仅在当次渲染内）；vision summary class 化、附 `<button class="pm-vision-toggle" data-anchor-vision="vision" aria-expanded="...">↕ 展开/收起</button>`。
- **JS pmToggleVision(anchorKey)**：本地切换 is-vision-open / aria-expanded / 文案，不影响其它 anchor 与 trail。
- **JS 事件代理**：在 `pmToggleMore` 同链 `closest("[data-anchor-vision]")` 判断中插入（与既有捕获链一致，零副作用）。
- **JS 取消 map mode 渲染**：`key === "experiment" ? renderTrailheadFeedback(model.data) : ""` 改成 `${"" /* v5.30.5：map mode 不再渲染此卡，入口仅在 Record Mode 保留 */}`。
- `renderTrailheadFeedback` 函数本身**保留未删**（CSS `.trailhead-feedback` 也保留）；零数据层改动（STATE_VERSION / STEPS / answer key 不变）。

### 删除清单
- map mode 内的 `.trailhead-feedback` DOM 节点（卡片 + 提示文案）：本轮完全不再渲染。
- map mode 内的 `tf-entry` / `tf-card` 入口：同上消失。

### Record Mode 保留的入口（不改）
- 「＋ 记录后来发生了什么」(6367 行)
- 「继续补充 / 修改」(6397 行)

### 验证（283 PASS / 0 FAIL）
- 既有 5 个 jsdom 套件 = 215 ✓
- v5.30 多视口截图回归（375/390/430/1024/1440 × light/dark）= 50 ✓
- v5.30.5 视觉探针（4 视口 × 17 断言：vision 高度收敛 + map 无 trailhead + toggle 切换逻辑 + document 无横向溢出）= 68 ✓

### 视觉实测（puppeteer-core）
| 视口 | vision 默认高 | 阈值 | 通过 |
| --- | --- | --- | --- |
| 1440 light | 95px | ≤230 | ✓ |
| 1440 dark | 95px | ≤230 | ✓ |
| 390 light | 64px | ≤180 | ✓ |
| 430 light | 64px | ≤180 | ✓ |

### 教训（写入技能）
1. **JS 注释陷阱**：模板字符串里的 `/* ... */` 或 `//` 会被原样渲染成可见文本。本轮 OP6 第一版就踩了，截图里出现了「/* v5.30.5：... */」字面文本。**修法**：注释必须写在 `${}` 插值表达式内（`${"" /* comment */}`），那里才是 JS 语法节点（永远不输出）。
2. **`.pm-stage` 设计内溢出**：该容器在 desktop 是 `overflow:hidden` 的固定高度舞台，6 个 anchor 用 `position:absolute` 围圆环布局，`scrollWidth > clientWidth` 属设计内。**真正的横向溢出判据**是 `documentElement.scrollWidth ≤ clientWidth`。
3. **探针勿断言 `getBoundingClientRect()`**：desktop 圆环布局使部分 anchor 一开始就不在视口内，rect.top 可能为负或 NaN。应断言「DOM 存在」而非「可视可达」。

## v5.30.4（2026-09-07）— Map Mode 字体层次 + 生活图景 Vision Board 化【基于 v5.30.3】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.30.html`（就地修改，v5.30.3 状态保留为基线快照 `_pre_v5304_baseline.html`）
**配套脚本**：`_apply_v5304_vision.py`（5 处锚点补丁，纯 LF）、`_probe_v5304_vision.js`（Puppeteer 4 视口 × 11–12 断言 = 46 PASS / 0 FAIL）、`_report_v5304.md`
**截图**：`_shot_v5304_desktop_light.png` / `_shot_v5304_desktop_dark.png` / `_shot_v5304_mobile390_light.png` / `_shot_v5304_mobile430_light.png`

### 性质
- 范围：仅 Map Mode 样式 + `renderPmAnchor` 一处 JS。零数据层改动：STATE_VERSION=7、STEPS、answer key、6 个 PM_ANCHORS、PM_TRAILS、PM_DENSITY、buildMapNodes、buildMapRelationships、Record Mode、Journey、Detail Panel、Progress Drawer、Escape 分层、focus 归还、trapFocus、Experiment Feedback、Export / Import、加密备份——**全部未动**。
- 不引入新锚点 / 新关系 / 新 region / AI 推断 / 关键词匹配 / 新装饰元素。
- 字体策略：保持现有 system stack（`"Noto Sans SC", "Microsoft YaHei", "PingFang SC", system-ui, sans-serif`），不引入外部字体（项目硬约束：单文件离线、无 CDN、无外部字体）；通过字号 + 字重 + 行距 + 颜色对比提升清晰度。

### CSS 调整
| 区块 | 旧值 | 新值 |
|---|---|---|
| `.pm-summary`（默认） | 13px / `white-space:nowrap` / `text-overflow:ellipsis` / line-height 1.72 | 15px / `white-space:normal`（多行）/ line-height 1.65 / color ink-soft |
| `.pm-summary.is-vision`（新增，仅 life_vision） | — | **17px** / line-height 1.88 / w450 / ink / 左 3px accent 边 / padding 14/16px / margin 14/16px / accent-soft 渐变底 |
| `.pm-sat` 普通 | 11.5px / padding 2×7 | 13.5px / padding 3×9 |
| `.pm-sat` 高密度 | 11px / padding 1×6 | 13px / padding 2×8 |
| `.pm-anchor[data-anchor="values"] .pm-sat` | 13.5 / w500 / opacity .88 | **15px** / w500 / ink / opacity .92 / dot opacity 1 |
| 移动端 `.pm-summary` / `.pm-sat` / values | 13.5 / 12.5 / — | **15.5** / **14** / **15.5** |
| 移动端 `.pm-summary.is-vision` | — | 17px / line-height 1.78 / padding 14px |
| 移动端 ≤560 `.pm-summary.is-vision` | — | 16px |

### JS 调整
- `renderPmAnchor` 渲染 summary 时判断 `a.summary.sourceKey === "life_vision"` → 加 class `is-vision` + 取消 `pmClip(46)` 字符截断，原样输出 `String(a.summary.label)`。其它 summary 仍走 `pmClip(rawText, 46)` 短截（紧凑预览；完整内容仍只在 Focus Panel / Record Mode 可读）。

### 不影响
Journey / Record / Detail Panel / Drawer / Panel / Toast / 输入项 / a11y / Trap Focus / Escape 分层 / focus 归还 / focus-within / 路径注记 / 移动触控区（≥36px 维持）。

### 验证构成
| 套件 | 断言 | 状态 |
|---|---|---|
| 既有 jsdom 回归（smoke / subtract / circles / v5.30.3 / case_6） | 215 | 0 fail |
| 多视口截图回归 `_shot_v530sub.js` | 50 | 0 fail |
| v5.30.4 视觉探针 `_probe_v5304_vision.js` | 46 | 0 fail |
| **合计** | **311** | **0 fail** |

### 实测数据（puppeteer-core 真视口）
| 维度 | 桌面 1440 | 移动 390 | 移动 430 |
|---|---|---|---|
| `.pm-title`（vision） | 26px / w500 | 19px | 19px |
| `.pm-summary.is-vision`（生活图景） | **17px** / 31.96px / w450 / ink + 3px accent 边 | **16px** / 28.48 | **16px** / 28.48 |
| 同一段 135 字 life_vision 文本 | 完整显示（h=348px, 9 行）/ 无截断 | 完整显示 / 无截断 | 完整显示 / 无截断 |
| `.pm-summary` 普通（边界·麻木、一成不变） | 15px / 24.75px / ink-soft | 15.5px | 15.5px |
| `.pm-sat` values（创造） | **15px** / 22.5 / w500 / ink | 15.5px | 15.5px |
| `.pm-sat` 普通（我愿意继续的选择） | 13px / 19.5px / ink-soft | 13px | 13px |
| 横向溢出 | 无 | 无 | 无 |

### 风险 / 待你确认（P2，非缺陷）
1. Vision summary 在桌面占 348px / 移动 223px，是 anchor 主体最显眼的一块——视觉重心偏向 vision anchor，符合 vision board 设计意图但请你看截图确认。
2. title 26px → vision 17px 比例 1.53；若想"标题更突出"，可单提 title 上限（任务外调整，需拍板）。
3. 其它 summary 不再 ellipsis——目前都是短答案效果仍是单行；若以后用户填长段（>80 字）会自然多行延展，不会撑破 anchor。

---

## v5.30.3（2026-09-06）— Release Candidate Polish【基于 v5.30.2】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.30.html`（就地修改，v5.30.2 保留不动）
**配套脚本**：`_apply_v5303_rc.py`（30 处锚点补丁，纯 LF）、`_verify_v5303_rc.js`（jsdom 62 断言）、`_probe_v5303_visual.js`（Puppeteer 视觉探针 40 断言）、`_report_v5303.md`

### 性质

发布候选版交互 / 可访问性 / 可读性与完成反馈审计。**零结构 / 零字段 / 零数据模型改动，STATE_VERSION 维持 7**；STEPS / answer key / 6 个 PM_ANCHORS / PM_TRAILS / density 系统 / getCompassData / buildMapNodes / buildMapRelationships / findNodeRelations / Record Mode / JSON+Markdown Export / 实验回访 / 加密备份全部未动。

### 改动

- **CSS（6 处）**：Core Values sat 提亮（`--ink` + weight 500 + opacity .88，dot opacity 1，hover 1，不加光不加彩不加字号）；`is-focused` 下 `focus-within` 自动提升被 dim 元素到 opacity 1；confetti 尺寸减档 + `prefers-reduced-motion` 下 `display:none`；移动端触控区 `.pm-sat` min-height 36px / `.pm-more,.pm-read` min-height 34px / `.pm-jump` padding（视觉不变大，可点区域更宽）；mode-btn 与 `#compassPageTitle` 的 `:focus-visible`。
- **HTML（7 处，仅静态壳）**：Detail Panel 与 Progress Drawer 补 `role="dialog" aria-modal="true" aria-labelledby`（`compassDetailTitle` / `progressDrawerTitle`）；Map/Record 切换 `role="tablist"/tab` → `role="group"` + `aria-pressed`（语义准确，不引入完整 Tab widget）；`#compassPageTitle` 加 `tabindex="-1"`。
- **JS（11 组 / 17 处补丁）**：通用 `trapFocus(container,event)` 抽出，`trapModalFocus` 复用；Escape 严格分层（Modal → Detail → Drawer → Map focus，一次一层）；Detail/Drawer 打开记录 `_lastFocusedBeforeDetail/_Drawer` 并 focus 关闭按钮、关闭后归还触发者（feedback 面板经 `_feedbackTriggerSel` 归还到重建后的同一入口）；sat HTML 去掉 title、改 `aria-label="打开线索：…，查看这项线索的完整记录"`（不暴露 sourceKey/id/phase/evidence）；Detail 标题改 `textContent` 直写（修含 `&`/`<` 原话二次转义）；`updateSaveState` 只更新 `#saveText` 保留 `.save-dot`；防抖自动保存不逐次 announce；confetti 96→56 片、2.8–4.6s→2.0–3.2s、10 色低饱和、reduce 下直接 return 不建粒子；完成 announce 改中性「这轮 Journey 已完成，你的个人罗盘已经生成。」；完成后 rAF focus `#compassPageTitle`；`focusin` 提升被 dim anchor（去重绑一次）。

### 验证

jsdom 回归 `_smoke_v530.js` 101 / `_verify_v530_subtract.js` 22 / `_verify_v5302_circles.js` 9 / `_case_6.js` 21 + 新 `_verify_v5303_rc.js` 62 + Puppeteer 多视口 `_shot_v530sub.js` 50 + 视觉探针 `_probe_v5303_visual.js` 40 = **共 305 项断言，0 失败**。详见 `_report_v5303.md`。

## v5.30.2（2026-09-06）— 减法延续：contour 改同心圆【基于 v5.30.1】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.30.html`（就地修改，v5.30.1 保留不动）

### 改动（纯呈现层 / 零结构 / 零字段 / 零数据模型改动，STATE_VERSION 维持 7）

- terrain contour 从 6 条 SVG `path d="C…"` 改为 6 个数字半径数组，渲染为 `<circle r="…" vector-effect="non-scaling-stroke">`，CSS 选择器 `.pm-contours path` → `.pm-contours circle`；circle 在 stage `preserveAspectRatio="none"` 下被均匀双向拉伸为扁椭圆叠层（同心轨道质感）。线条数量 / 密度分档（3/4/6）/ reduced-motion 不变。
- 修「core_values 选词补充在 Map 看不见」：`getNodeAnswerText` 对 `sourceKey === "core_values"` 短路返回 `node.sublabel`（即用户为该词写的 explanation，v5.21 已存于 `core_values_explanations`，本次仅让它在地图可见）；空态显示可执行引导「这个词你还没补充它对你具体意味着什么。可以回到 Journey 在词旁点『＋ 补充』写下。」

### 验证

`_smoke_v530.js` 101 / `_verify_v530_subtract.js` 22 / `_verify_v5302_circles.js` 9 / `_case_6.js` 21 = 153 PASS / 0 FAIL。报告 `_report_v5302.md`。

## v5.30.1（2026-09-06）— Detail Panel 移除冗余标题【基于 v5.30】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.30.html`（就地修改，v5.30 保留不动）

### 改动（纯文案 / 零结构 / 零字段 / 零数据模型改动，STATE_VERSION 维持 7）

- Map Mode 点开任意节点弹出的 Focus Panel 中，删除了「这部分在探索中处于什么位置」标题（原 `detail-relations-title`）。该标题在 v5.25.6 引入，每个节点都重复出现、且与面板已有的「阶段 · 题目标题」meta、`来自：…·…` 的关系证据列表语义重叠，属冗余信息。
- 关系列表本身（`detail-relations` / `relation-item`，含 type / label / 来自 source · evidence）完整保留——用户仍能看到该线索与哪些阶段相连、证据来源。
- 同步修正 Map Intro 引导语（原「点任意一个关键词，可以看见它在探索中处于什么位置、与哪些线索相连」）为「…可以看见它与哪些线索相连」，消除对已被删除标题的悬空引用。
- 未触碰：`buildMapNodes` / `buildMapRelationships` / `findNodeRelations`、6 个 Anchor、`STATE_VERSION`、答案键、Record / Export / 实验回访。

## v5.28（2026-09-02）— Map Mode 视觉语言重构：Personal Cartography / 个人方向地图【基于 v5.25.7】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.28.html`（由 v5.25.7 干净打补丁生成，v5.25.7 保留不动）  
**配套脚本**：`_apply_v528_visual.py`（12 处字符串补丁，保持 CRLF）、`_check_v528_syntax.js` / `_check_v528_braces.py`

### 目标

把 Map Mode 从「卡片式结果页」升级为真正具有辨识度的 **Personal Atlas / 个人方向地图**：

- 建立 **Atlas Header**：`INNER COMPASS / PERSONAL ATLAS` + 地图编号 / 日期 / 指南针标记
- 建立 **Territory 区域网格**：Ideal（01/02）→ Translation（03）→ Reality（04）→ Experiment（05）
- 用 **SVG Journey Path** 动态连接区域：Ideal（实线）→ Translation（虚线）→ Reality（淡暖色线）→ Experiment（实线箭头）
- 用 **节点形状** 区分信息类型：圆点（想靠近）、菱形（想避免）、靶心圆（下一步尝试）、方块（已拥有的）
- Reality Terrain 用 **地形带**（等高线装饰 + 不同边框样式）呈现 must / prefer / flexible / unknown
- Detail Panel 加入 **FIELD NOTE** 地图注释感
- 进入动画按顺序绘制：标题 → 区域 → 路径 → 节点

### 硬性边界（本轮严格遵守）

| 约束 | 落实方式 |
| --- | --- |
| 不动数据层 | `getCompassData()` / `buildMapNodes()` / `buildMapRelationships()` / `findNodeRelations()` 全部原样保留 |
| 不动问卷逻辑 | `STEPS` / 7 阶段 / 18 题 / answer key / `STATE_VERSION`(7) / `STORAGE_KEY` 全部未变 |
| 不动关系语义 | 没有新增任何 `buildMapRelationships()` 关系；没有从文本自动推导关系 |
| 不做人格推断 | 没有新增任何「人格 / 指数 / 倾向」类标签或视觉评分 |
| 节点形状只表示信息类型 | 大小一致，不表示重要性、权重、强弱 |

### 改动清单（12 处补丁）

1. **版本注释** — 追加 v5.28 说明。
2. **CSS** — 注入约 300 行新制图语言：`.atlas-header`、`.atlas-canvas`、`.territory-grid`、`.territory`、`.journey-path`、`.map-node-shape`、`.terrain-band`、`.atlas-legend`、响应式与减少动画降级。
3. **`renderCompassMap()`** — 重写为 atlas 布局：header + SVG 路径层 + territory grid + legend。
4. **`mapNodeHTML()`** — 用 `.map-node-shape` 替代 `.map-node-dot`，按 `type` 输出不同形状。
5. **`renderCompassShell()`** — DOM 就位后调用 `updateAtlasPath()`。
6. **`updateAtlasPath()`（新增）** — 根据 5 个 territory 实际位置计算贝塞尔 Journey Path。
7. **`window resize` 监听** — 重算路径。
8. **关系高亮迁移** — `clearRelationshipHighlights` / `highlightSections` / `highlightTfPath` / `focusMapRegion` / `highlightNodeRelationship` 全部从 `.spine-section` 迁移到 `.territory` / `.journey-path`。
9. **Detail Panel 风格** — 左侧加粗色边 + `FIELD NOTE` 伪标题。
10. **移动端响应式** — 820px 以下单列、路径/节点自适应；560px 以下字号缩小。
11. ** prefers-reduced-motion 降级** — 关闭 territory / path / node 动画。
12. **JS 语法与 CSS 花括号检查** — 全部通过。

### 明确没有修改的部分

- `STEPS` / 阶段数 / 题目数 / 题目类型 / answer key 列表
- `STATE_VERSION`（7）、`STORAGE_KEY`、`state` schema、`IMPORT_WHITELIST`
- `getCompassData()` / `buildMapNodes()` / `buildMapRelationships()` / `findNodeRelations()`
- Phase 1–7 任何题目、文案、引导、脚手架
- Record Mode 结构与渲染
- 导入 / 导出 / 加密备份 / 打印流程
- `renderMapIntro()` 等函数仍保留在代码中（只是不再被 `renderCompassMap()` 调用）

---

## v5.25.7（2026-09-02）— 实验后回访（Experiment Feedback）：Phase 7 的时间闭环【optimized 公开版轨道，基于 v5.25.6】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.25.7.html`（由 v5.25.6 干净打补丁生成，v5.25.6 保留不动）
**配套脚本**：`_apply_v5257_patch.py`（13 处字符串补丁，CRLF 保持）、`_verify_v5257.js`（jsdom 验证 **85/0 PASS**）

### 目标

把 Phase 7 从单次的「猜想 → 行动 → 观察什么」补成跨时间的闭环：

```
猜想 → 行动 → 观察什么 → 实际发生什么 → 判断发生什么变化
```

Feedback 记录的是**实验发生后的真实信息**与**用户自己的判断变化**——不是 success_criteria 的再次填写，也不是实验结果评分。

### 硬性边界（本轮严格遵守）

| 约束 | 落实方式 |
| --- | --- |
| 不新增第 8 阶段 | `STEPS` 仍为 7 阶段 / 18 题，逐项比对与 v5.25.6 完全一致 |
| 不把反馈做成新问卷 | `experiment_feedback` **不在 STEPS 中** → 不进入 `getStepCompletion` / `getOverallProgress`，不是必答题 |
| 不新增固定必答题 | 三部分均可只填一部分、可全留空；无必填校验 |
| 不新增独立大地图区域 | 不新增 region / 节点 / spine-section（`.spine-section` 仍为 5 个）；Feedback 只是既有 Experiment（Trailhead）节点的第二状态，挂在既有 section 底部 |
| 不破坏 answer key / localStorage / 旧数据 | `STATE_VERSION` 维持 **7**、`STORAGE_KEY` 维持 `inner_compass_v5_3`；新增且仅新增一个**可选**答案键 `experiment_feedback`，旧存档无此键一律走默认空值 |
| 不动其他阶段 | Phase 1–6 全部题目的 label / guide / placeholder / options / scaffold 与 v5.25.6 逐项比对完全一致 |

### 新增的数据结构（唯一一处）

```js
state.answers.experiment_feedback      // 可选对象，缺省 = 从未回访
{
  happened: "",                        // ① 后来实际发生了什么
  signals: {                           // ② 回看当时想留意的信号（key 与 success_criteria 严格对齐）
    energy_signal: "", learning_signal: "", identity_signal: ""
  },
  change: "",                          // ③ 这次尝试之后，你对原来的想法有什么变化
  updatedAt: null                      // 写入时间戳
}
```

- 唯一写入口 `writeExperimentFeedback(field, key, value)`，只写这个键，不触碰任何既有 answer key（已验证写入前后其它 key 全部原样）。
- `getExperimentFeedback()` 对脏数据（`null` / 数组 / 字符串 / 缺字段）一律降级为空结构。
- 派生视图 `buildExperimentFeedback(successCriteria)` 为纯只读、不落盘。

### 三个认知层级（只此三层，不新增观察指标）

| # | 层级 | 形态 | 说明 |
| --- | --- | --- | --- |
| ① | 后来实际发生了什么？ | 自由文本 | 不评价做得好不好 |
| ② | 回看当时想留意的信号 | 引用 + 补充 | 自动引用已有 `success_criteria.energy_signal / learning_signal / identity_signal` **原文**，用户只补「后来观察到」；**不重新定义观察指标**，未写过的 criteria 不凭空生成条目 |
| ③ | 这次尝试之后，你对原来的想法有什么变化？ | 自由文本 | 「想法没变」「更确定了」「开始怀疑」「还是不知道」都算 |

不设成功 / 失败评分，不要求用户证明实验有效。「没有变化」「还是不知道」「这次没看出什么」均为有效反馈（已在 UI 文案与引导中显式说明）。

### 改动清单（13 处补丁）

1. **文件头版本注释** — 追加 v5.25.7 说明。
2. **CSS** — 新增 36 条规则：`.trailhead-feedback` / `.tf-*`、`.fb-*`（Feedback Card）、`.record-fb-*`（Record 区块）。全部复用既有 CSS 变量，浅色 / 深色主题自动适配。
3. **JS 数据层** — `FEEDBACK_LABELS`、`FEEDBACK_SIGNAL_FIELDS`、`getExperimentFeedback()`、`hasExperimentFeedback()`、`buildExperimentFeedback()`、`writeExperimentFeedback()`。
4. **`getCompassData()`** — `experiment` 下新增只读 `feedback` 派生对象（`exists` / `availableCount` / `filledCount` / `preview` / `updatedAt`）；`experiment.answered` 语义**未变**。
5. **Feedback Card / Detail Panel** — `openExperimentFeedback()` / `feedbackFormHTML()` / `updateFeedbackPanelStatus()` / `onCompassInput()`。**复用既有 `compassDetailPanel`**，不新增面板 DOM；输入走 document 级一次性委托（不产生监听器累积），边写边存。
6. **`openCompassDetail()`** — 复位 foot 按钮文案，避免沿用回访面板的「在完整记录中查看」。
7. **`closeCompassDetail()`** — 支持 `announceText` 参数（同时修复了原本会把 MouseEvent 传给 `announce()` 的隐患）；回访面板关闭时立即落盘并刷新 Map / Record。
8. **`renderCompassMap()`** — 向 `renderTrailhead` 传入完整 `data`。
9. **`renderTrailhead(data)`** + 新增 `renderTrailheadFeedback(data)` — Experiment 第二状态（等待反馈 / 已有反馈）。
10. **`renderRecordExperiment()`** + 新增 `renderRecordExperimentFeedback(data)` — Record 07 章内追加「这次尝试之后」，不新增章节号（章节数仍为 7）。
11. **`onCompassClick()`** — 新增 `[data-feedback-open]`（打开）与 `[data-feedback-done]`（完成落盘）两条委托分支。
12. **`buildCompass()`** — 实验坐标追加「— 实验后回访 —」内容（与 exportMarkdown 同口径：实验未答时不输出，避免孤儿记录）。
13. **`exportMarkdown()`** — Journey 记录之后新增「## 实验后回访（这次尝试之后）」独立章节，含记录时间与「这不是评分」说明。

### 明确没有修改的部分

- `STEPS` / 阶段数 / 题目数 / 题目类型 / answer key 列表（18 题全部原样）
- `STATE_VERSION`（7）、`STORAGE_KEY`（`inner_compass_v5_3`）、`state` schema 主干、`IMPORT_WHITELIST`
- Phase 1–6 任何题目、文案、引导、脚手架
- `buildMapNodes()` / `buildMapRelationships()` / `findNodeRelations()`（jsdom 逐项比对：节点 id / region / sourceKey / sourceField 与关系层输出与 v5.25.6 **完全一致**）
- Map 区域数量（`.spine-section` 仍为 5）、Map 节点数量、Journey Spine 算法
- Insight Layer（`generateInsights()` 8 条规则一条未动）
- Journey Mode 的 Phase 7 三题（未插入任何回访题目，避免把反馈做成问卷）
- 导入 / 导出 / 加密备份 / 打印 的既有流程

### 完整 walkthrough

**① 第一次完成 Experiment 时看到什么**

- Map Mode → Trailhead section → 三条链（想试的念头 / 这次怎么试 / 想看清楚什么）下方，出现一条**默认收起**的虚线轻量入口：`＋ 记录这次尝试后来发生了什么`，下方一行小字「等你真的试过之后再来写也不迟。没有变化、还是不知道、这次没看出什么，都是有效记录。」
- 没有展开的表单、没有新增必填项、进度统计不变（不计入 18 题）。
- Record Mode → 07 章末尾出现同类入口（虚线区块 + `＋ 记录后来发生了什么`）。

**② 几周后回来从哪里进入**

- 打开页面 → 因 `showSummary = true` 直接落在 Compass → 两个入口之一：
  - **Map Mode**：Trailhead 底部入口（等待反馈态）或「已记录回访」卡片（已有反馈态，`N / M 部分已填写` + 预览文本 + 记录时间 + `查看 / 继续补充`）；
  - **Record Mode**：07 章「这次尝试之后」区块的 `＋ 记录后来发生了什么` / `继续补充 / 修改`。
- 点击任一入口 → 展开 Feedback Card（复用右侧既有 Detail Panel），三个层级，边写边存；底部有 `N / M 部分已填写` 计数与「内容会自动保存」。

**③ 保存 Feedback 后各处的显示**

| 位置 | 显示 |
| --- | --- |
| **Map** | Trailhead 底部变为实线「已记录回访」卡片：`2 / 3 部分已填写`、最早一段的预览文字（60 字截断）、记录时间、`查看 / 继续补充` |
| **Record** | 07 章内新增「这次尝试之后」：① 后来实际发生了什么 → 全文；② 回看当时想留意的信号 → 每条「能量信号 / 学习信号 / 还想不想继续」先列「当时想留意：〈原文〉」再列用户补充；③ 想法变化 → 全文；未填部分不渲染，另给一行「这次回访还有 N 部分没有写。想补的时候随时可以补。」 |
| **Markdown 导出** | Journey 记录之后新增 `## 实验后回访（这次尝试之后）` 章节，含三段（② 以 `- 能量信号（当时想留意：…）：…` 形式带上原文）、记录时间、「这不是对这次尝试的评分」说明 |
| **Compass Summary（buildCompass）** | 「想拿去试一次的念头」坐标下追加 `— 实验后回访 —` 与 `回访 · …` 各行 |
| **JSON 导出 / 加密备份** | `experiment_feedback` 随 `state.answers` 自动包含，无需额外处理 |

### 验证

`_verify_v5257.js`（jsdom，**85 / 0 PASS**）：A 结构零改动（10 项）· B Phase 1–6 label/guide/placeholder/options/scaffold 逐项一致 · C 数据层默认值与脏数据 / 旧存档兼容（6 项）· D Map 第二状态与入口（6 项）· E Feedback Card 三层级 + 引用 criteria 原文 + 写入隔离 + 落盘（12 项）· F 部分填写与低信息量反馈（6 项，含「没有变化」「还是不知道」有效）· G/H Record 展示与入口（13 项）· I Markdown 与 buildCompass（14 项）· J 地图结构 / 关系层零改动（4 项）· K 旧存档加载（4 项）。

---

## v5.25.6（2026-09-02）— Phase 5 → 6 → 7 认知分工保护性修复（纯文案 / label，零结构 / 零字段 / 零数据模型改动，STATE_VERSION 维持 7）【optimized 公开版轨道，基于 v5.25.5】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.25.6.html`（由 v5.25.5 干净打补丁生成，v5.25.5 保留不动）
**配套脚本**：`_apply_v5256_patch.py`（字符串补丁，CRLF 保持）、`_verify_v5256.js`（jsdom 验证 **64/0 PASS**）

### 目标
保护三段的认知分工：Phase 5 = 我想往哪靠近（不做职业测评、不被现实提前纠正）；Phase 6 = 我现在站在哪里（现实是「当前位置」不是「理想纠错器」）；Phase 7 = 我准备先试什么（实验保持开放，不因 Phase 5 有工作探索而收窄成职业验证）。允许 Phase 5 与 Phase 6 之间存在距离，该距离本身就是信息。

### 改动（8 处，全部不涉及数据结构）

| # | 位置 | 原文 | 新文 |
| --- | --- | --- | --- |
| 1 | `STEPS.reality_bridge.title`（Phase 6 标题） | 从「想怎样」回到「现在能怎样」 | 从「想怎样」看到「现在在哪里」 |
| 2 | `questions.reality_classification.guide` | 前面想的是：什么对你重要。这里想的是：这些条件放在现在的现实里，哪些需要守住，哪些还有空间。 | 追加：「一个条件即使现在暂时做不到，也可以先保留下来；这里不是要把前面的方向改成现实可行方案。」 |
| 3 | `questions.adaptability.label` | 为了靠近想要的方向，你愿意在哪些方面试一试调整？ | 就现在的处境，你愿意在哪些方面试一试调整？ |
| 4 | `questions.adaptability.guide` | 不是立刻改变，是看看你愿意在哪些地方主动试一试调整——这和上一题「现实里哪些能调整」是两回事：上一题是判断现实的弹性，这一题是你愿意亲自迈出的一步。 | 这不是要求你马上改变。只是看看，在你现在的处境里，哪些部分你愿意主动试一试。这和上一题是两回事：上一题是判断现实本身的弹性，这一题只看你自己愿不愿意先动一动。 |
| 5 | `generateStepFeedback().reality_bridge` | 「你已经把想要的生活放回了现实里……」 | 「你现在把想去的方向与当前处境放在一起看了……想去的地方和现在的位置之间可以先留着距离，这个距离本身就是有用的信息。」（按真实答案动态生成，无分类 / 无 adaptability 时不虚构判断） |
| 6 | `buildMapRelationships()` Journey Flow 两条 label | 工作偏好 → 放回现实 / 现实 → 下一步尝试 | 工作方向与现实处境并置 / 从当前处境出发，看下一步想试什么（evidence 同步去因果化） |
| 7 | `renderMapIntro()` | 从想要怎样的生活（Ideal）→ 经过翻译成工作语言（Translation）→ 进入现实判断（Reality）→ 形成下一步尝试（Experiment） | 从想要怎样的生活（Ideal）→ 把其中与工作有关的部分说具体（Translation）→ 与当前现实处境并置（Reality）→ 从现在的位置挑一个想试的念头（Experiment）；说明段加入「现实作为当前出发位置，可供下一步参考，它不负责修正前面写下的方向」，并去掉「发生了关系」 |
| 8 | `openCompassDetail()` 标题 / `renderTerrainNodes()` DEPTH 标签 | 这部分和什么发生了关系 / 必须保留的现实 · 希望保留的现实 · 可以调整的现实 · 暂不确定的现实 | 这部分在探索中处于什么位置 / 仍想守住 · 希望保留 · 可以调整 · 暂不确定 |

### 明确未做
- 未新增 / 删除 Journey 阶段或题目（仍 7 阶段 18 题，1/2/2/3/3/4/3，与 v5.25.5 逐项一致）
- 未改 answer key、`state` schema、`STORAGE_KEY`、`STATE_VERSION`（7）
- 未扩大 `reality_classification` 输入来源（`extractRealityClassifyItems()` 仍只读 `work_constraints`）
- 未新增语义推断 / 关键词归类 / 职业分类系统 / 强制关系
- Phase 7 三题文案原文未动（经核对已满足「关于生活或工作 / 不必是一次职业尝试 / 不是判断成败而是收集信息」）
- **Insight Layer（`generateInsights()`）本轮未改**，风险另列（见下方）

### 已知风险（本轮不处理，建议下一轮评估）
- Insight #8：把「现实分类 + 实验」说成「已经不只是停留在想象层面……从模糊到行动的关键一步」，隐含「现实 + 实验 = 进步」的进步叙事，与「允许理想—现实距离」存在张力。
- Insight #2：即使 `reality_classification` 为空，也会输出「你在「现实地图」里的分类也指向了这个方向」（对用户未做动作的事实断言）。
- Insight #4 / #6：关键词过宽（`#4` 匹配「少」、`#6` 匹配「比」，会命中「比如」等无关文本），存在误报与人格结论风险。

---

## v5.25.5（2026-09-01）— Reality Bridge 文案回归「用户自然判断」（不破坏数据结构 / 不改 classification key / STATE_VERSION 维持 7）【optimized 公开版轨道，基于 v5.25.4】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.25.5.html`（由 v5.25.4 干净打补丁生成，v5.25.4 保留不动）
**配套脚本**：`_apply_v5255_patch.py`（字符串补丁）、`_verify_v5255.js`（jsdom 验证 **56/0 PASS**）

### 问题（来自审稿）
v5.25.x 改版后 reality_classification（Reality Bridge）模块口吻生硬、偏设计说明、不像用户自然判断：四档语义被动作化（如「最难动 / 可松动」），标题带心理咨询腔（「现实里的摩擦：哪些最难动，哪些先松动？」），item 强塞系统关系文案（「XX 来自『工作条件』」），skip 区要求用户「挑最难动 1–2 项 + 最愿松动 1 项」——与四档体系不一致。

> 说明：用户描述的「摩擦 / 松动」原始文案实际落在 v5.25.1（该版本曾把四档 enum 改为两档 locked/movable）。本次目标文件 v5.25.4 已是四档「必须保留 / 希望保留 / 可以调整 / 还不确定」，但标题与 item 来源标签仍不够自然。按用户确认，以 v5.25.4 为基底另存新文件，不回滚 v5.25.1。

### 改动（仅文案 / label / 标题，数据结构零改动）
- **标题（Spine 段名）**：phase / spine section 采用用户首选候选「把方向放回现实」（承接 Phase 5「从生活到工作」→ 回到现实条件，而非进入咨询工具）。
- **问题卡 label**：`reality_classification` 卡 label 改为「现实里的调整空间」。
- **说明文案（guide）**：改为「前面想的是：什么对你重要。这里想的是：这些条件放在现在的现实里，哪些需要守住，哪些还有空间。」——删除「这里不重复问 / 只做一个极轻的判断」等设计解释语气。
- **四档 label 统一**（仅改显示文字，data key / enum 不变）：`must`→必须保留 / `prefer`→希望保留 / `flexible`→可以调整 / `unknown`→暂不确定（原「还不确定」）。
- **item 来源标签删除**：`reality-classify-label` 只渲染条件本身（如「项目自主权」），不再出现「来自『工作条件』」系统关系文案；"工作条件"仅作为内部数据来源（`extractRealityClassifyItems` 从 `work_constraints` 派生），不再对用户暴露。
- **skip / 空状态文案**：删除「挑最难动 + 最愿松动」二元任务；改为轻量「前面还没留下工作条件。可以回到『从生活到工作』补上几条，也可以先跳过这一题，之后再回来。」状态提示「已经判断了 X / Y 项，剩下的可以之后再补。」/「每一项都已经有了一个判断。之后想改，随时可以回来。」——不再替用户预设「真正知道往往要等一次小实验」。
- **图例（默认折叠）**：四档释义用自然中文（少了它现在的生活就转不动 / 希望留住，形式可以商量 / 换个形式也能接受 / 现在还说不准），无动作化 / 咨询腔。

### 严格边界（全守）
- **不改** classification 数据 key（must/prefer/flexible/unknown）、`reality_classification` 的 `[{label,priority}]` 结构、`STATE_VERSION`(7)、`STORAGE_KEY`、answer key、题目数量、Map 关系层逻辑。
- **不新增**交互 / 评分 / 职业测评语义；Reality Bridge 仍只是「把方向放回现实」的判断动作。

### 下游一致性（jsdom 验证通过）
- Map Mode 关系层 `#7 classification` 仍基于 `reality_classification` 数组长度渲染，label 改为「工作条件 → 你自己的现实判断」，未破坏。
- Compass Report / Record Mode 沿用 `priority` enum，label 改动不影响数据。
- 脏值回归：旧档 `locked/prefer` 等异常 priority 不崩溃，渲染仍为 12 按钮（3 项 × 4 档）。
- 验证 `_verify_v5255.js` **56/0 PASS**（A 静态禁语 / B 按钮与 item / C 落库 / D legend / E Map 关系 / F skip / G 空状态 / H 脏值回归 / 导出 markdown 含四档）。

## v5.25.4（2026-08-31）— Experiment 阶段时间边界去 KPI 化【optimized 公开版轨道，基于 v5.25.3】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.25.4.html`（由 v5.25.3 干净打补丁生成，v5.25.3 保留不动）
**配套脚本**：`_apply_v5254_patch.py`（6 步字符串补丁）、`_verify_v5254.js`（jsdom 验证 **31/0 PASS**）

### 目标
把 experiment_action 的「接下来 30 天」固定周期改为弹性周期——时间成为实验的一部分，而非系统预设的唯一正确周期；保留「把猜想拿到现实里验证」的行动性，避免 KPI 式约束。

### 严格边界（全守）
- **不删除** Experiment 阶段；**不删除** experiment_goal / experiment_action / success_criteria；`STATE_VERSION` 仍为 7、`STORAGE_KEY` 不变、answer key / state schema / localStorage 格式均未改；题目总数仍为 18。
- **不新增**任何周期选择 UI / input 字段 / 任务管理系统；周期表达只通过自然语言（「这周试一次」「连续两周」「一个月观察」）落在现有 textarea 文本里。

### 文案改动（仅改 label / guide / placeholder / scaffold + 地图一处渲染 label）
- `experiment_action`：
  - label：「接下来 30 天里，你愿意怎么试一次？」→「接下来一段时间里，你愿意怎么试一次？」
  - guide：新增「周期由你定：可以是一次、几天、一周、两周，或一个月观察。选一个你真正做得到、并且足以让你获得一点信息的周期就行；如果动作太大，就把它缩小到你能轻松开始的规模；如果一次还看不出什么，就把周期拉长一点。」
  - placeholder：去掉「30 天」，提示「包括你准备用多长周期」。
  - scaffold[0]：由「1 次而不是 10 次？1 周而不是 1 个月？」改为「一次就够，还是连续几天、一周、两周，甚至一个月观察？挑一个你真的做得到、也足以让你获得一点信息的周期」。
- `success_criteria`：保留「不是判断成败，而是收集信息」；轻量对齐时间逻辑——「无论这次尝试持续多久……都按它实际发生的时长来观察，不必另设一个期限」。三信号字段（energy/learning/identity）未动。
- Map Mode / Journey Spine：renderTrailhead 第三步 label「怎么算有收获」→「想看清楚什么」（表达收集信息、而非成功阈值）；"完成率"两处原有表述本就是否定式（「不是完成率」），保留。

### 三者承接（核对一致）
1. goal = 我还不知道什么（guide：「挑一件你现在还不确定、但很想知道答案的事」）
2. action = 我准备怎样获得一点真实信息（guide 显式「获得一点信息」）
3. criteria = 我准备观察什么（「收集信息」「留意……信号」）
4. 不需要回答「最终成功还是失败」（「不必提前规定它应该证明什么」）

### 克制性
experiment_action 的 label/guide/placeholder 及全文件均不含「30 天 / 效率 / KPI / 完成率（肯定式）/ 必须达成」。验证 `_verify_v5254.js` 31/0 PASS（A 静态 / B action / C criteria / D 承接 / E Map / F 克制 / G 旧存档回归）。

## v5.25.3（2026-08-31）— Reality = Constraints + Existing Stock：「手里的牌」正式入档【optimized 公开版轨道，基于 v5.25.2】

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.25.3.html`（由 v5.25.2 干净打补丁生成，v5.25.2 保留不动）
**配套脚本**：`_apply_v5253_patch.py`（17 步字符串补丁，从 v5.25.1 生成，可复跑）、`_verify_v5253.js`（jsdom 验证 **73/0 PASS**）

> 注：此版本属于「optimized 公开版」轨道（无 LLM），与 dev 主线的 v5.26 / v5.27 / v5.28 并行。v5.25.2 是在 v5.25 基础上已发布的公开版，本轮在其之上做最小范围增强。

### 目标
把过去只存在于 reality_bridge 阶段 guide / scaffold 里的「经历存量」提升为正式、可被 Compass / Map / AI 使用的用户输入——`Reality = Constraints + Existing Stock`，但克制，不做简历采集 / 职业能力盘点。

### 严格边界（全守）
- **不修改** reality_context 五字段（income / location / time / responsibility / energy）；**不修改**任何已有 answer key；`STATE_VERSION` 仍为 7、`STORAGE_KEY` 仍为 `inner_compass_v5_3`、IMPORT_WHITELIST 未变；**不删除**任何题目。
- 只 +1 题：`existing_stock`（阶段内位置 reality_context → existing_stock → reality_classification → adaptability）；题目总数 17 → 18，阶段数仍为 7。

### 新增题目 `existing_stock`
- **label**：「过去走过的路，现在有哪些东西还在你手里？」（无简历腔）
- **guide**：「不只写『我会什么』。也可以是做过的事、做出来的东西、认识的人、对某个领域慢慢攒下的了解，还有那些试过又没继续的经历留下的判断——包括你已经确定『不适合自己』的部分。只写 1–3 个你最想带走的就够，不用做完整盘点。」
- **placeholder**：「过去走过的路里，有什么是你还想带着继续走的？」
- **scaffold（4 条非诱导提示）**：做过且有结果的事 / 停下但留下什么的尝试 / 还联系得上的人 / 越来越确定的适合与不适合。

### 成为正式 Compass 数据的落点（最小修改）
1. `getCompassData()`：`reality.existingStock = { questionLabel, text }`（text 为用户原文，未改写）。
2. `buildMapNodes()`：Reality 区域下新增正式节点 `reality-stock`（region=existingStock，sourceKey=existing_stock，type=stock，label=用户原话，无优先级/评分）。
3. Map Mode：Reality 地图分两个轻量 subsection——「现实地形 · 现在的处境」（原 5 字段 + 四档分类）与「手里的牌 · 过去留下、还能带着走的」（暖色区分，不评级）。
4. Journey Spine：Reality 段内 reality-stock 进 `.terrain-band[data-priority=stock]`。
5. Record Mode：原文回看，独立条目 `data-record-qid=existing_stock`。
6. Detail Panel：点开 reality-stock 显示完整原文 + 可追溯关系。
7. Report / Markdown 导出：`buildCompass()` 的「我的现实地图」纳入「手里的牌」原文；AI payload 可见。
8. 关系层：`buildMapRelationships` 新增可证明关系 `stock`（existing_stock → experiment_goal + experiment_action，「已经有的 → 想试什么」），source 与 evidence 来自 question intent 设计，非语义推断；`getNodeRelationGroup` 归入 terrain（点击高亮 Reality 相位），future Signal 可引用。
9. Insight #7（经历存量）：可由真实「手里的牌」独立触发，措辞只说明「你已经写下它」，不评价好坏/价值/是否优势。

### 阶段说明修正
- reality_bridge intro 重写为同时点出四类现实（动不了 / 可调整 / 看不准 / 已经拥有），去掉「现实=限制」旧暗示。
- reality_context guide 收回「已经拥有的」提示，转交下一题；scaffold 改为「现实条件三问」（最难动 / 还有空间 / 看不准），与下一题职责分离。

### 克制性
全文件无新增「资源等级 / 优势评分」字样；`existingStock` 只含 `questionLabel` + `text` 两键；题目定义无 score / weight / level / tier 字段。

### 旧数据兼容
无 `existing_stock` 的存档：`existingStock.text` 为空串（非 undefined）、`reality.answered` 仍为 true、不生成 reality-stock 节点、Map / Record 不渲染空 subsection、存档 version 与顶层键集合不变、answers 不含空占位。零崩溃。

### 验证
`_verify_v5253.js`：73/0 PASS（A 静态结构 / B 新题定义 / C getCompassData / D buildMapNodes / E Map Mode / F Record / G Detail / H 报告+导出 / I 关系层 / J 旧数据兼容 / K 克制性）。

## v5.28（2026-08-30）— 可插拔 LLM Insight Provider（AI 增强层，仍可完全无 AI）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.28.html`（由 v5.27 复制升级，v5.27 保留不动）
**配套脚本**：`_verify_v528.js`（jsdom 验证，**156/0 PASS** = 139 项 v5.27 回归 + 17 项 v5.28 新断言）、`_diag_insight.js` / `_diag_insight2.js`（定位 harness bug 用，可删）
**部署骨架**：`llm-insight-provider/`（local-proxy.mjs / vercel / cloudflare / shared/prompt.mjs / README.md）

### 目标与定位
在 v5.27 的「规则 Pattern Layer」之上，新增一层 **可插拔的 LLM Insight Provider**：用户点击「生成 Compass 洞察」后，由 LLM 生成 pattern / tension / signal 三类洞察。**AI 是增强不是必需**——默认 `rule` 模式完全不调用 LLM，页面永远不空白（生成失败自动回退规则层）。

### 严格边界（承 v5.27 评审结论）
- **不改动**任何既有数据结构 / Map Node 结构 / 用户本地存档机制（`STATE_VERSION` 仍为 7、`STORAGE_KEY` 不变、零新增持久化字段、零 localStorage 格式变动）。v5.28 验证已确认「存档 key 集合未变化」「answers 未新增 key」。
- **不引入 chat / 自由对话**功能。仅「本轮一次性的 AI 辅助 Pattern/Tension/Signal 层」。
- LLM 只分析用户**本次** Journey 写下内容；不是咨询师 / 职业规划师 / 人格分析器（系统提示词硬约束）。
- **产品边界**：核心是帮用户重新看见自己的「生活 / 价值 / 边界 / 现实 / 尝试」线索，不是预测未来、不是给职业答案。没有证据就没有洞察。

### Provider 抽象（四 + 一模式）
- `generateCompassInsights(compassData, patternCandidates, options) → Promise<InsightResult>`，`InsightResult = { insights: [{ type, title, summary, confidence, evidence: [{nodeId, sourceKey, phase, excerpt}] }] }`。
- 模式：`rule`（默认，无 AI）/ `mock`（开发自检，复用 suggestedCandidates 证据）/ `local`（本地代理）/ `serverless`（Vercel/Cloudflare）/ `userKey`（用户自填 Key 直连）。
- **最小 Insight Payload**：前端只构造 `{ schema, note, nodes:[{nodeId,sourceKey,phase,text}], experiment, suggestedCandidates }`，**绝不发送原始 localStorage / 完整 state** 给 LLM。

### 关键实现
- **证据多重兜底**：JSON schema 校验 → 证据 nodeId 必须存在于 `buildMapNodes()` 输出（否则丢弃该证据、不足则整条丢弃）→ 本地重算 confidence → 跨阶段 / 最小证据数门槛 → 按 type+nodeIds 去重。最终每一条洞察都可通过 nodeId 回跳地图原话。
- **四态 UI**：未生成 / 正在生成 / 生成成功 / 生成失败；失败 → 保留 v5.27 规则层（页面不空）。
- **轻量 AI 说明**：「这些洞察来自你本次填写的内容。它们是对线索的整理，不是对你的定义。」+ 「查看依据」机制回跳 Map Node。
- **无 AI 默认**：`rule` 模式不显示「生成 Compass 洞察」按钮；只有非 rule 模式才出现，且必须用户显式点击才发请求，无后台上传。
- **密钥铁律**：前端是纯静态单文件，绝不硬编码 / 假装能安全保存 Key。密钥只在 server 端（env / secret）或用户内存（userKey，仅本次会话、不落盘）。纯静态 GitHub Pages 无法在前端保护密钥——需要 Key 时只能选 serverless 代理 / userKey / 本地代理三者之一。

### 部署骨架（llm-insight-provider/）
- `local-proxy.mjs`：Node 内置、零依赖，读 `.env`，对应 `local` 模式默认 `http://localhost:3000/api/compass-insights`。已 smoke test（启动 + 优雅 500 + CORS 预检 204）。
- `vercel/api/compass-insights.js` + `shared/prompt.mjs`：对应 `serverless` 模式，Key 走 Vercel 环境变量。
- `cloudflare/worker.js` + `wrangler.toml`：对应 `serverless` 模式填 Worker 完整 URL，Key 走 `wrangler secret put`。
- `shared/prompt.mjs`：server 端提示词副本，**必须与前端 `INSIGHT_SYSTEM_PROMPT` 逐字同步**（改一侧另一侧也要改，否则证据重叠校验 / 措辞审计失真）。
- `README.md`：三种选项对比、契约、分步部署、安全隐私注意。

### 验证结果（_verify_v528.js，jsdom 156/0 PASS）
- v5.27 全部 139 项回归（Persona A/B/C 候选生成 / 证据门槛 / 交互 / Signal 定位 / Tension 双向 / 空状态 / spine/Record/localStorage/STATE_VERSION）继续全绿。
- 17 项 v5.28 新断言：`__insightEngine` 暴露、payload 最小字段（含 nodes/experiment/suggestedCandidates、不含 state）、Mock 生成 InsightResult 且每条 type/title/summary/evidence 齐、证据 nodeId 全真实、每条 ≥2 证据且跨 ≥2 阶段、LLM 与 Rule 证据高度重叠（Mock 复用 suggestedCandidates）、非法 nodeId 丢弃整条、合法 nodeId 保留、围栏/非 JSON 回退、初始 rule/none、切 Mock 后出现按钮 + 轻量说明、默认 rule 无按钮。
- 注：初版 `_verify_v528.js` 顶部 `FILE` 误指 v5.27 文件导致 2 项误 FAIL；修正为 v5.28 文件后 156/0（此为测试脚本 bug，非产品 bug）。

### 后续补充（同版本追加）
- **新增 Persona D（v5.28 追加）**：`PATTERN_MOCK_PERSONAS.D` ——「独处沉淀 ↔ 渴望被看见（Pattern + Tension + Signal 同框）」，用于在本地无风险演示 LLM Insight Provider 的三种产物同时出现。验证脚本 `_verify_v528.js` 已把 Persona D 纳入循环，验证通过。
- **新增演示脚本 `_demo_v528_persona_d.js`**：以 Persona D 为数据基础，依次输出 ① 规则层产物 ② Mock LLM 产物 ③ local/serverless 会发送的脱敏 payload ④ userKey 会构造的 OpenAI 请求体 ⑤ 模式切换后的 UI 状态。
- **`__insightEngine` 暴露 `INSIGHT_SYSTEM_PROMPT` 与 `buildInsightUserPrompt`**：方便 demo/调试时查看前端真实 prompt，server 端同步时也可对比。
- **`llm-insight-provider/README.md` 新增「十、五种模式使用速查」**：直接对应 UI 下拉框的 5 个选项，给出每种模式「是否联网 / 谁持有 Key / 怎么点生成」的操作步骤。
- **新增 `DESIGN_ALIGNMENT_v528.md`**：对照「如何将用户的非结构化自我反思转化为结构化数据，并利用 LLM 在跨模块信息之间发现模式、张力与可验证问题，同时避免模型直接对用户人格下结论」逐项评估，指出架构已对齐、真实能力需在接 LLM 后验证。
- **验证更新**：`_verify_v528.js` 加入 Persona D 循环后，由 156 项增至 **196/0 PASS**。

### 未改动（严格遵守范围）
17 题 / answer key / `STEPS` / `QUESTIONS` / 数据结构 / `STATE_VERSION`(7) / `STORAGE_KEY` / Map 关系层 / Journey Spine / 节点内容 / 地图空间布局 / 导出格式 / 零新增持久化字段 / 零 localStorage 格式变动。

## v5.27（2026-08-30）— Pattern Layer 正式化：Map Mode 第二认知层（仍不接外部 LLM API）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.27.html`（由 v5.26 复制升级，v5.26 保留不动）
**配套脚本**：`_apply_v527_patch.py`（带花括号守恒断言的补丁）、`_verify_v527.js`（jsdom 验证，**138/0 PASS**）、`_shot_v527.js`（Playwright 真实浏览器复核）

### 目标与定位
把 v5.26 的「开发实验组件」升级为正式 Compass Experience，但本轮仍不接外部 LLM API。Pattern Layer 成为 Map Mode 的**第二认知层**：第一层 = Journey / Map（你留下了什么），第二层 = Patterns（哪些内容在跨阶段反复出现）。不是独立 Report 页。

### 核心改动（保留 v5.26 的 buildPatternCandidates 与结构化 schema，不推翻）
- **只展示证据充分的候选**：默认 ≤3 条、每类最多 1 条、某类无合格候选就空着，禁止为「完整」补齐三类。证据门槛收紧为「≥2 条证据 且 跨 ≥2 个阶段」（跨阶段是这一层的定义性证据）。评分分母放宽（frequency /5、crossPhase /3、sourceDiversity /5）以消除多阶段顶满的饱和。
- **证据交互升级**：点卡片 → 地图上多个跨阶段节点同时高亮（带顺序序号）+ 无关节点适度 dim + 浮动提示条可取消；点任一证据 → 复用现有 Compass Detail Panel 看回原始回答与现有关系（Pattern 摘要不替代原话）；打开详情时地图上的跨阶段高亮保持不丢。
- **「为什么会出现这条」轻量透明说明**：只显示「出现在 X 个阶段 · 关联 Y 条回答（其中 Z 条来自这次尝试）」+ 一句大白话门槛规则，不暴露内部分数 / breakdown。
- **Tension 双向结构**：两端各显示方向名、规模与代表性证据，中间「同时成立」；缺任何一端都不生成。措辞从「冲突=问题」改为「两件事目前同时存在」。
- **Signal 与 Experiment 对接（本轮关键修正）**：优先生成 `experiment-link` 类型——反复出现的方向，且用户写下的**具体行动 / 标准（experiment_action / success_criteria）真的碰到了它**，说明「你正在通过哪一个实验获得什么信息」，直接展示三题原文，点击定位到实验区。新增 `goal-link` 弱档：方向反复出现且只在 `experiment_goal` 里被命名、但行动/标准还没碰到——诚实呈现为「被命名、尚未被试过」，不暗示闭环。不做「你应该做什么」的建议。
- **措辞规则**：`PATTERN_COPY_BANNED` 黑名单 + `patternCopyAudit()` 硬检查。只描述「你写下的东西呈现出的模式」，禁止人格化结论（你本质上 / 你其实是 / 你的深层需求 / 说明你属于 …）。
- **空状态与低置信度状态**：证据不足时显示「目前还没有足够重复的线索形成一个明显模式」+ 具体原因；该容器从 v5.27 起固定为未来 LLM Layer 的统一出口。
- **开发指标** `computePatternMetrics()`（生成候选数 / 通过 filter 数 / 显示条数 / 每条证据数 / 哪一类最稳定 / 过滤原因分布），仅本地 console 与 dev 面板，不上传。
- **最小布局调整**：v5.25 的 North Star「生活图景」是纯文本块而非 `.map-node`，导致证据无法标出它；仅补 `data-node-id` 一个属性 + 最小聚焦/dim 样式，不改动任何视觉结构，不新增第二条视觉主线。

### 验证结果（_verify_v527.js，jsdom 138/0 PASS）
- **Persona A（低密度）**：nodes=16 / hits=5 / generated=1 / passed=1 / displayed=1 / mostStable=pattern(16)。仅 1 张 Pattern 卡「收入」在 4 个不同部分里都出现了（4 证据 / 4 阶段）。无 Tension / 无 Signal。**符合「只有 1 条就只展示 1 条」。**
- **Persona B（高密度）**：nodes=42 / hits=51 / generated=17 / passed=17 / displayed=3 / mostStable=tension(16)。Pattern「创造」在 5 个不同部分里都出现了 + Tension「创造」与「收入」目前同时存在 + Signal「你正在通过这次尝试，获得关于「创造」的信息」(experiment-link)。
- **Persona C（高张力）**：nodes=40 / hits=76 / generated=20 / passed=20 / displayed=3 / mostStable=pattern(12)。Pattern「稳定」在 7 个不同部分里都出现了 + Tension「稳定」与「探索」目前同时存在 + Signal「你正在通过这次尝试，获得关于「稳定」的信息」(experiment-link)。**注**：C 的「稳定」确实出现在 experiment_action / success_criteria，故为强闭环 experiment-link；`goal-link` 为新增防御性弱档（A/B/C 未触发，但已注册并参与排序）。

### 产品级评审（A/B/C 三类 Persona）
三个准入问题，至少两个「是」才进下一阶段 LLM API：
1. **用户是否看到了自己原本不容易看到的跨阶段模式？** → **是**。A 在 4 个阶段反复写「收入」、B 在 5 个阶段反复写「创造」、C 在 7 个阶段反复写「稳定」，且都以「在 X 个不同部分里都出现了」呈现——单题界面下用户无法横向比对，只有 Pattern Layer 把跨阶段复现显影。
2. **Tension 是否比普通 summary 更有价值？** → **是（对 B/C 明确，A 不适用）**。B「创造 vs 收入」、C「稳定 vs 探索」都以双向结构并置两端证据与代表性原话，比「你似乎在创造与收入之间纠结」的总结更克制、更可证伪——用户能沿两端证据回跳地图验证，而非接受一个结论。A 无 Tension（证据不足则不生成，符合设计）。
3. **Signal 是否真的让 Compass 与 Experiment 形成闭环？** → **是**。B/C 的 Signal 都是 experiment-link：反复出现的方向正好被用户写下的具体实验（行动/标准）碰到，且点击 Signal 直接定位到实验区、实验三题原文直接展示。用户看到的不是「建议」，而是「你正在用这次尝试检验你前面反复写下的东西」。

**结论：三个问题全部为「是」→ 可以进入下一阶段（接真实 LLM API 作为 candidate validator）。** 注意：LLM 应只对现有候选做重排/过滤（validator 角色），不自由做文本语义分析，保持本层可解释、可追溯。

### 未改动（严格遵守范围）
17 题 / answer key / `STEPS` / `QUESTIONS` / 数据结构 / `STATE_VERSION`(7) / `STORAGE_KEY` / Map 关系层 / Journey Spine / 节点内容 / 地图空间布局 / 导出格式 / 零新增持久化字段 / 零 localStorage 格式变动。

### 关键工程纪律（已内化，建议沉淀为 skill）
- CSS 媒体查询边界处做字符串替换时，anchor 与 replacement 的花括号必须成对守恒（v5.26 hotfix 教训）；patch 脚本每步用 Python 显式平衡 `{`/`}` 差值，不匹配即 `sys.exit(1)`。
- jsdom（解析 DOM、跑逻辑、断言 138 项）与 Playwright（真实浏览器复核 CSS computed style / 聚焦交互）双层验证；jsdom 不解析媒体查询，必须在真实浏览器复核 CSS。

## v5.26（2026-08-30）— Pattern Engine（Mock）· 跨阶段模式发现层（受控实验）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.26.html`（由 v5.25.html 复制后增量，v5.25.html 保留不动）
**配套脚本**：`_apply_v526_patch.py`（补丁）、`_verify_v526.js`（验证，57/57 PASS）

**来源**：用户发起的受控产品实验——验证「仅按原问卷结构重新排列」是否足够有价值，还是需要一层「跨阶段模式发现层」才能产生 Aha。本轮**不重设计地图视觉、不接真实 LLM**，只验证信息价值。

### 核心改动（**纯派生层 + 最小可用 UI**，不推翻任何现有结构）

- **新增 `buildPatternCandidates(data)`（纯派生，不持久化）**：从现有 Compass View Model（buildMapNodes 产物）召回候选，输出 `pattern / tension / signal` 三类，每条带 `evidence`（直指现有 `nodeId / sourceKey / phase / excerpt`）。刷新重算，不写 localStorage。
- **Seed 词典（召回机制，非洞察）**：`time_autonomy / stability / exploration / creativity / learning / space / belonging / income / control / social_connection`，每个含 `keywords`（自由文本命中）+ `options`（固定选项词表命中）。
- **四维度可解释评分**：`frequency=(跨节点数-1)/4`、`crossPhase=(阶段数-1)/3`、`sourceDiversity=(sourceKey数-1)/4`、`specificity`（基础 0.55/0.70 + 按 sourceRole / reality 档位加成）。权重 `0.35/0.30/0.20/0.15`。门槛 `PATTERN_MIN_SCORE=0.45`。全部可在 dev 面板复算。
- **证据门槛**：≥2 条证据且（跨≥2 阶段 **或** 来自≥2 独立 sourceKey），否则过滤；允许「无候选」不凑数。
- **三类生成器**：
  - Pattern：同一 seed 跨≥2 阶段复现（seed-recurrence）。
  - Tension 三档（按信息增益排序 user-action > declared-axis）：TG1 价值×现实分层（你把它排进价值，却在现实里留松动）；TG2 让步×现实底线；TG3 声明式张力轴（两词义相对主题都出现）。
  - Signal：反复出现但未进入「猜想与尝试」阶段的缺口（gap）。
- **3 个开发用 Mock Persona**（仅 `?patternMock=A|B|C` 启用，通过 `getAnswer` 只读覆盖层注入，`saveState` 提前 return，**永不写 localStorage**）：
  - A 低信息密度（工作驱动、答案短、长期图景弱）；B 高信息密度（图景丰富、价值/边界清晰）；C 高张力（稳定↔探索、自主↔收入明显冲突）。
- **Pattern Layer UI（主地图之后、Record Mode 之前）**：标题「你这次反复写下的东西」；默认 ≤3 卡（Pattern/Tension/Signal 各 ≤1，且指向不同 seed 避免同义重复）；三类型用 glyph+label+border-style 区分（不依赖颜色 alone）；「查看地图中的证据」按钮复用现有 `openCompassDetail` 与关系高亮，真正跳回 Map Node/Section，不复制详情系统。
- **Dev 调试面板**：`?patternDebug=1` 输出候选主题、命中 sourceKey、score 拆解、最终入选/过滤原因（console.groupCollapsed + 页面 `.pattern-dev` 块）；正式用户（无参数）不可见。
- **视觉克制**：延续绿/暖色、圆角、轻量卡片；无大标题、无大面积渐变、无复杂动画；不新增地图地理元素、不扩大 Spine、不加等高线、不改现有空间布局。

### 验证结果（`_verify_v526.js`，jsdom 57/57 PASS）

- **Persona A（低密度）**：nodes=16 / hits=6 / selected=1 → 仅 1 张 Pattern 卡（「收入与经济基础」跨 4 阶段），无 Tension/Signal。**符合预期：低密度用户 Pattern Layer 几乎没有内容。**
- **Persona B（高密度）**：nodes=42 / hits=54 / candidates=16 / selected=3 → Pattern「时间自主」(5 阶段) + Tension「创造×收入」(declared-axis) + Signal「自主与掌控」(gap)。Pattern 明确给出「原来我在不同问题里反复谈同一件事」的跨阶段发现。
- **Persona C（高张力）**：nodes=40 / hits=80 / candidates=15 / selected=3 → Pattern「收入」(5 阶段) + **Tension「学习与积累」(user-action：价值排第 2，但现实分层归「可调整」)** + Signal「关系与连接」(gap)。该 Tension 明显比普通总结更有信息增益——把用户两处自己的判断并置。
- **功能回归零**：Journey/Record/Map/导出/保存/节点详情均未回归；存档 version 仍=7、key 集合未变；Mock 会话 localStorage 零写入；正式用户无候选时 Pattern Layer 不渲染、dev 面板不出现。

### 已知局限（若后续接真实 LLM validator 的改进点）

1. **评分饱和**：多阶段答案下 frequency/crossPhase/sourceDiversity 常顶到 1，仅 specificity 区分，score 不能强分候选；选择靠 score 排序，弱档 declared-axis 张力可能压过强档 user-action 张力（如 B 选中创造力×收入而非 time_autonomy 的 TG1）。建议后续按 info-gain tier 加权或交 LLM 重排。
2. **Seed 召回天花板**：仅关键词/固定选项命中，自由文本语义、跨文本聚类、心理推断一律不做（有意为之）；因此非 seed 覆盖的主题不会被发现。
3. **本轮结论**：对高密度用户，Pattern（跨阶段复现）已能产生「沿证据回跳地图」的真实 Aha；低密度用户几乎无内容。实验达到「验证信息价值」目标，未进入视觉扩展。

### 未改动（严格遵守范围）

- 17 题 / answer key / `STEPS` / `QUESTIONS` / `getCompassData()` / `buildMapNodes()` / `buildMapRelationships()` / `findNodeRelations()` / `openCompassDetail()` / Journey Spine / 5 个 section 渲染 / `STATE_VERSION`(7) / `STORAGE_KEY` / 节点内容 / 地图空间布局 / 导出格式。

### v5.26 hotfix（同日 17:40）— 修复 Pattern Layer 在默认浏览器下不显示的 CSS 漏闭合 bug

**症状**：在 Chrome/Edge 默认设置（prefers-reduced-motion: no-preference）下打开 v5.26，Pattern Layer 区域出现一个巨型黑色填充块（约 1070×1070），盖住整个卡片区域。

**根因**：v5.26 patch 脚本 `_apply_v526_patch.py` 的 `CSS_ANCHOR` 把 v5.25 末尾 `@media (prefers-reduced-motion: reduce) { ... }` 块的**闭合 `}` 一起匹配并吞掉**，但 `PATTERN_CSS` 开头没有补回这个 `}`。结果整段 Pattern Layer CSS（约 305 行）被解析进 `prefers-reduced-motion` 媒体查询内——只有开启 reduce motion 的浏览器才生效。Chrome/Edge 默认 reduce-motion 关闭，导致：
- `.pattern-glyph` 失去 `width:22px / height:22px` → 容器被拉成 1070px 宽
- 内嵌 SVG 失去 `width:14px / height:14px` → 默认填满父容器
- `<path>` 失去 `fill:none / stroke:currentColor` → 默认 fill: black，巨型黑色填充块

**修复**：
1. v5.26 HTML 已在 `prefers-reduced-motion` 块与 Pattern Layer 注释之间补回 `    }`（见 `最终公开版inner_compass_v5_optimized_v5.26.html:3948`）
2. `_apply_v526_patch.py` 的 `CSS_ANCHOR` 保留原状（仍含闭合 `}`），但 `PATTERN_CSS` 开头先补一个 `    }`，保证 anchor 吞掉的 `}` 被还原
3. Playwright 验证：`.pattern-glyph` 现在 22×22、SVG 14×14、计算样式正确，_verify_v526.js 57/57 PASS
4. 幂等性：基于 v5.25 重新运行 patch 脚本三次结果一致，结构正确

**反推教训**：在 CSS 媒体查询边界处做字符串替换时，**anchor 与 replacement 的花括号必须成对守恒**——anchor 吞掉多少 `}`，replacement 开头就要补回多少 `}`。后续 patch 在该处用 Python 显式平衡花括号更稳。

## v5.25（2026-08-29）— Map Mode Spine 唯一主线重构（左置路线轨道 + 合并 Transformation Path）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.25.html`（由 v5.24.html 复制后重构，v5.24.html 保留不动）

**来源**：用户对 v5.24 Map Mode 的审阅反馈——
- 「Journey Spine 仍是页面中央的装饰性中分线，未成为 Ideal → Translation → Reality → Experiment 的连续探索路径」
- 移动端「中分线」问题需真正解决（v5.24 的 ≤640 修复只把 Spine 移到左边，但 TF overlay 仍隐藏，路径连续性丢失）
- 「Transformation Path 与 Journey Spine 应合并为一个视觉主线」
- 「不要通过增加粗线、阴影、颜色强度简单强化 Spine，也不要新增第二条主路径」

### 核心改动（**仅 Spine 视觉语义重构**——零数据结构 / 零字段 / 零 Map Intro / 零节点内容 / 零 IA）

- **Journey Spine 左置为「路线轨道」（route rail）**：`.journey-spine` 从居中（`left:50%` 虚线）改为左置全宽绝对容器，轨道线（2px）位于卡片左缘 gutter（`--rail-x: 11px` 桌面 / `9px` 手机）。桌面端与移动端采用**同一语义、不同布局**。
- **每个 map section 成为 route station**：
  - `.spine-section::before`（station 标记）从卡片顶部居中（`top:-6px; left:50%`）改为卡片**垂直中心**（`top:50%; translateY(-50%)`），坐在轨道线 `x=--rail-x` 上。
  - 新增 `.spine-section::after`（station 挂接线）：从轨道线到卡片左缘的 1.5px 短水平线，让卡片**真正「挂」在路线上**，而不是被线穿过——彻底消除「中分线」感。
  - North Star station 用 4 角星 `clip-path` 标记（cartographic 锚点）。
  - Trailhead 段在轨道终点（`bottom:-1px`）加 9px 水平收束条（`var(--good)`）——路线在「出发点」处收束，呼应 cartographic 的 trailhead 含义。
- **Transformation Path 并入 Journey Spine（唯一视觉主线）**：
  - 删除独立的 `.transformation-overlay` 容器（`id="transformationOverlay"`、`tf-path--*` 类、`tf-label` 类、`tf-path--active` 类、`tfReveal` 动画全部移除）。
  - 4 段相位渐变（`linear-gradient(accent→good)` / `(good→#8db8a0)` / `(#8db8a0→warn)` / `(warn→accent-warm)`）合并为 `spine-rail-seg--ideal/translation/reality/experiment` 4 段，**沿用原 cartographic 颜色语言**。
  - 段与段在 section 间隙**中点相接**（`gapMid(a,b) = (a.offsetTop + a.offsetHeight + b.offsetTop) / 2`），构成一条无断点的连续路径。验证：4 段边界 16 → 1103.5 → 1766 → 2514 → 3084，完全连续。
  - 移动端 ≤760 旧版「隐藏 TF overlay」策略取消——合并后**移动端也保留 4 段相位连续性**。
- **轨道段标签（spine-rail-label）**：竖排显示相位名（Ideal/Translation/Reality/Experiment + sub），位于轨道与卡片之间的 gutter 中。桌面 always-on @ opacity .45（取代 v5.24 的 hover-only `tf-label`），移动端隐藏（阶段名已由各 section kicker 承担）。
- **`updateTransformationOverlay` → `updateJourneySpine`**：用段间中点连续边界替代旧「每段覆盖对应 section 范围」算法；resize 防抖重算保留。
- **不加粗、不加阴影、不加颜色强度**：
  - 轨道宽度 2px（与 v5.23 spine-line 同宽），opacity .5（与原 tf-path 的 .55 相当）。
  - station 圆点 10px @ opacity .55（桌面）/ .75 + 描边（手机，与 v5.24 手机端一致）。
  - 不使用 drop-shadow / 不加 box-shadow / 不新增第二条主路径。
- **可访问性 / 动画**：`.journey-spine` 仍 `aria-hidden="true"`（装饰层）；`railReveal` 动画使用 `backwards` fill，结束后回归静态 opacity（避免与 `--active` 状态冲突）；`prefers-reduced-motion` 块同步更新为 `spine-rail-seg`。

### 未改动（严格遵守范围）

- `QUESTIONS`、`STEPS`、17 个 answer key、`getCompassData()`、`buildMapNodes()`、`buildMapRelationships()`、`STATE_VERSION`(7)、`STORAGE_KEY`、`Direction Field` 三分法、`Terrain` 抽象属性、`Detail Panel` 关系上下文、Map Intro 文案、节点渲染（`mapNodeHTML`）。
- `renderMapIntro()` / `renderNorthStar` / `renderBearings` / `renderTranslationField` / `renderRealityTerrain` / `renderTrailhead` 五个 section 渲染函数零改动。

### 验证

- `node _check_v525.js`：JS 语法 OK；技术约束、17 answer key、关系可追溯、唯一主线约束、视觉克制、reduced-motion、移动端断点全过（41 项全 PASS，含「无第二条主线」「轨道宽度保持 2px」「无 drop-shadow」三条 v5.25 新增断言）。
- `node _smoke_v525.js`（jsdom 真实渲染）：10/10 用例通过，控制台无错误。Case C 验证 `.journey-spine` 内生成 4 段 `spine-rail-seg` + 4 个相位标签；Case G 验证点击工作类节点激活 `.spine-rail-seg--translation.spine-rail-seg--active`。
- 结构 dump-dom 验证：4 段轨道 top/height 连续相接（16 / 1103.5 / 1766 / 2514，无断点），与设计预期一致。
- 视觉截图：`_shot_map_desktop.png` / `_shot_map_mobile.png`（实际设计克制呈现）；`_shot_debug_desktop.png`（高对比定位验证，仅开发用）。

### 文件清单

- `最终公开版inner_compass_v5_optimized_v5.25.html`（主交付物）
- `_check_v525.js` / `_smoke_v525.js`（自检 + 冒烟）
- `_shot_map_seed.js` / `_shot_map_debug_seed.js` / `_shot_v525_map.html` / `_shot_v525_map_debug.html`（截图 harness，可重现）
- `_shot_map_desktop.png` / `_shot_map_mobile.png` / `_shot_debug_desktop.png`（视觉参考）

---

## v5.24（2026-08-29）— 3 处视觉修复：标题间距 / 罗盘指针 / 移动端 Journey Spine

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.24.html`（由 v5.23.html 复制后修复，v5.23.html 保留不动）

**来源**：用户在 v5.23 预览后提供的 3 条视觉反馈：
1. 摘要页/尾页标题与副标题间距过大；
2. Map Mode 顶部指南针徽章指针应朝北（朝上）；
3. 手机端 Journey Spine 居中虚线像「中分分割线」，看不出路径意义。

**改动（零数据结构 / 零字段 / 零 localStorage 改动，STATE_VERSION 维持 7，STORAGE_KEY 维持 inner_compass_v5_3）**：

1. **收紧标题与副标题间距**
   - `.compass-header-main h1`：`margin: 8px 0 6px` → `6px 0 2px`，标题与副标题更紧凑。
   - `.hero h1` / `.summary-hero h2` 底部间距同步收紧，避免「尾页」出现同类过大间距。
2. **罗盘指针朝北（朝上）**
   - `.map-needle-badge .needle`：`transform: rotate(196deg)` → `rotate(0deg)`，多边形尖端默认指向 SVG 上方（North）。
3. **手机端 Journey Spine 改为左侧时间线**
   - 新增 `@media (max-width: 640px)`：`.spine-body` 左侧留出 38px 时间线沟槽；`.journey-spine` 从居中移到 `left: 22px`；虚线改为半透实线。
   - `.spine-section::before` 站点圆点从居中移到卡片左侧 `-21px`，落在 Spine 上，并增强不透明度与描边，使其成为可见的「站点标记」。
   - 保留 TF path 在移动端隐藏的策略；通过左置 Spine + 站点标记建立「探索路径」的空间含义，避免中分分割线感。

**未改动**：`QUESTIONS`、`STEPS`、17 个 answer key、`getCompassData()`、`buildMapNodes()`、`STATE_VERSION`(7)、`STORAGE_KEY`、Relationship Spine 结构与关系层逻辑。

**验证**：
- `node _check_v524.js`：JS 语法 OK；技术约束、17 个 answer key、Relationship Spine 自检清单全过。
- `node _smoke_v524.js`（jsdom 真实渲染）：10/10 用例通过，控制台无错误。

## v5.23（2026-08-28）— 地图升级为 Relationship Spine（关系脊柱）+ 高层转化路径主线

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.23.html`（由 v5.22.html 复制后升级，v5.22.html 保留不动）

**来源**：用户反馈——v6 Living Map 的六区本质是内容分组，SVG 路线是装饰性的、不承载可解释的结构关系。要求地图呈现 Inner Compass 问卷本身已存在的思考关系，同时明确**不把 STEPS 填写顺序定义为"用户的真实思考脊柱"**。

**核心原则**：地图只呈现可从问卷结构、用户填写动作或明确设计意图中证明的关系；**禁止**跨文本语义聚类、自动主题归纳、AI 推断或伪造关联。用户文本仍是主角。

**改动（零数据结构 / 零字段 / 零 localStorage 改动，STATE_VERSION 维持 7，STORAGE_KEY 维持 inner_compass_v5_3）**：

1. **概念修正：Exploration Path / Journey Spine，不是"思考脊柱"**
   - STEPS 顺序只证明"用户在这次探索中走过的路径"，不声称还原了用户内在心理结构。
   - Journey Spine 视觉权重**低于** Transformation Path，避免填写顺序成为地图唯一空间意义。
2. **新增高层转化路径主线：Ideal → Translation → Reality → Experiment**
   - 基于问卷 4 层阶段划分（Phase 1-4 / 5 / 6 / 7），比"17 题依次填写"更值得成为空间叙事主线。
   - 左侧彩色渐变边条（`.transformation-overlay` + 4 段 `.tf-path`），hover 显示竖排阶段标签。
   - `updateTransformationOverlay()` 动态计算四段位置；resize 防抖重算；移动端隐藏（靠 section 配色区分）。
3. **Direction Field 三分法（基于 question intent，非文本语义）**
   - `DIRECTION_ROLES`：Attraction（想靠近）/ Orientation（重视与取舍）/ Avoidance（想避免）。
   - Bearings 区左右两栏：左栏 Attraction + Orientation，右栏 Avoidance。
   - life_vision / ideal_day / end_feeling 仍作为 North Star 最大锚点，Bearings 的 Attraction 栏只放 desired_identity。
4. **Translation Axis 加强**：Phase 5 不再是普通 Bridge 卡片
   - 三栏流动（工作方式 / 工作条件 / 活动形式）+ 渐变连接线 + 箭头。
   - 新增"翻译来源"标注（`#translationSourceTag`）：来自生活图景 + 在意与取舍 + 边界确认。
   - 重点呈现"生活 → 工作条件"的结构性转换过程，不伪造逐条文本对应。
5. **Terrain 抽象化**：降低 literal 地貌隐喻
   - 去掉"岩层 / 缓坡 / 沙地 / 迷雾"式插画表述，改用 CSS 变量控制 density / opacity / border-style / blur。
   - must（实心 4px）/ prefer / flexible（虚线 2px + 降透明度）/ unknown（轻微 blur + 最低透明度）。
   - priority 来自用户自己在 Reality Bridge 的分类动作，非系统判断。
6. **交互：点击高亮"实际参与的结构关系"，不是前后相邻 station**
   - `highlightNodeRelationship()` + `getNodeRelationGroup()`：按 sourceKey 分组。
   - 工作类节点 → 高亮整个 Translation Axis + 显示翻译来源标注；anti_future / muted_version → 高亮 dest + values（Contrast）；实验三题 → 高亮 Experiment Chain；现实类 → 高亮 Terrain。
   - 2.6s 后自动清除；点击其他区域立即清除。
7. **Detail Panel 新增"这部分和什么发生了关系"区块**
   - `findNodeRelations(nodeId)` 反查节点参与的关系，每条显示 `type` + `label` + 「来自：source · evidence」。
8. **三层架构明确分离**
   - 内容节点层（`.map-node`，用户原始文本，可读/可复制/可访问）+ Journey Spine（探索路径，aria-hidden）+ Relationship Layer（Transformation Path 与关系高亮）。
   - 新增价值来自关系层，不是重新排列内容。

**新增关系层（纯派生，不新增任何持久化 state）**：
- `buildMapRelationships(data)`：返回 6 类共 12 条关系，每条带 `source`（来源）与 `evidence`（证据）：
  - `flow` × 6 — 来自 STEPS 阶段顺序（承接式提问）
  - `translation` × 1 — 来自 Phase 5 question intent（work_constraints 引导语要求"把生活线索翻译成工作条件"）
  - `contrast` × 1 — 来自 Phase 1 Attraction 与 Phase 4 Avoidance 的对称设计
  - `chain` × 2 — 来自 Phase 7 三题递进（goal → action → criteria）
  - `dependency` × 1 — 来自 Phase 3 内部两题逻辑依赖（core_values → trade_off）
  - `classification` × 1 — 来自用户自己在 Reality Bridge 完成的分类动作
- `findNodeRelations(nodeId)`：反查某节点参与的关系（供 Detail Panel 与点击高亮）

**新增/修改组件**：`buildMapRelationships()`、`findNodeRelations()`、`updateTransformationOverlay()`、`renderNorthStar()`、`renderBearings()`、`renderTranslationField()`、`renderRealityTerrain()`、`renderTrailhead()`、`highlightNodeRelationship()`、`clearRelationshipHighlights()`、`getNodeRelationGroup()`、`highlightSections()`、`highlightTfPath()`；重构 `renderCompassMap()`（5 区）、`renderMapIntro()`（四阶段文案）、`focusMapRegion()`（兼容 `.spine-section`）、`onCompassClick()`（关系高亮）、`openCompassDetail()`（关系上下文）、`renderCompassShell()`（渲染后计算 TF path）；全部 `.spine-*` / `.tf-*` / `.terrain-band` / `.trailhead-*` / `.bearing-*` CSS。

**未改动**：`QUESTIONS`、`STEPS`、17 个 answer key、`getCompassData()`、`buildMapNodes()`、`STATE_VERSION`(7)、`STORAGE_KEY`、`Record Mode` 渲染逻辑。

**验证**：
- `node _check_v523.js`：JS 语法 OK；技术约束（STATE_VERSION / STORAGE_KEY / buildMapNodes / 无 AI 调用）全过；17 个 answer key 全保留；6 项自检清单全过。
- `node _smoke_v523.js`（jsdom 真实渲染）：10/10 用例通过，控制台无错误——覆盖空状态、5 个 section 渲染、TF path 四段生成、Direction Field 三容器、Terrain 四档、关系可追溯、点击高亮 Translation / Contrast、Detail Panel 关系上下文、用户文本不截断。

## v5.22（2026-08-17）— 地图主体升级为 v6 Living Map + 新增轻量「地图前言 / Map Intro」

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.22.html`（由 v5.21.html 复制后升级版本号，v5.21.html 保留不动）

**来源**：用户反馈——地图内部已有清晰的认知路径，但首次进入 Map Mode 时需要自己猜顺序；同时要求地图主体使用之前测试的 v6 Living Cartographic Map 版本（路线脊线 + 制图语义）。

**改动（零数据结构 / 零字段 / 零 localStorage 改动，STATE_VERSION 维持 7）**：

1. **地图主体升级为 v6 Living Cartographic Map**：
   - 从 v5.20 的 CSS Grid 卡片版升级为 v6 的制图语义地图：路线脊线 SVG（route spine）+ 六类地理元素（Destination / Landmarks / Boundary / Bridge / Terrain / Trailhead）。
   - 新增 `createCartographySVG()`：等高线（contour）、地形波（terrain-wave）、路线绘制动画（routeDraw pathLength 动画）、边界影线漂移（boundaryDrift）。
   - 节点入场动画（nodeReveal staggered delay）；边界区 hatched 背景图案；落点 camp 样式。
   - **数据层完全不变**：仍用 `buildMapNodes()` 派生节点，answer key / localStorage / STATE_VERSION 不受影响。
2. **Map Mode 顶部新增轻量 Map Intro**：
   - 位于真正地图 canvas 之前，视觉像"地图阅读说明 / map legend / expedition note"。
   - 标题「你的方向地图」+ 副标题 + 正文明确说明"这不是替你做决定的地图"。
   - **6-step reading rail**：生活图景 → 价值 → 边界 → 工作 → 现实 → 尝试，每站一句极短说明。
3. **reading rail 可交互**：
   - 每项可点击，通过 `data-area` 映射到对应 `.map-zone`（dest/values/bounds/bridge/terrain/landing）。
   - 点击后 `scrollIntoView` + 短暂 pulse 高亮；`focusMapRegion()` 兼容 `.map-zone` 和 `.map-region` 双选择器。
4. **v6 header 精简**：原 v6 header 的「你的方向地图」标题与 Map Intro 合并，header 只保留 "INNER COMPASS · LIVING MAP" kicker + 指南针图标。

**新增/修改组件**：`renderCompassMap()`（v6 Living Map 布局）、`createCartographySVG()`、`renderMapIntro()`、`focusMapRegion()`（双选择器兼容）、全部 `.compass-map-v6` / `.map-zone*` / `.living-map-*` CSS。

**验证**：`node --check` 抽取 `<script>` 校验 JS 语法 OK。

## v5.21（2026-08-17）— 四摩擦点交互与文案优化（基于 v5.20 虚拟走查）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.21.html`（由 v5.20.html 复制后升级版本号，v5.20.html 保留不动）

**来源**：v5.20 虚拟测试角色（TA）走查报告识别出 4 个摩擦点；本版针对其做可执行优化。

**改动（零数据结构 / 零字段 / 零 localStorage 改动，STATE_VERSION 维持 7）**：

1. **① 结构化多字段题（ideal_day / work_environment / reality_context / trade_off / anti_future / activity_types / success_criteria）**：
   - 顶部新增「已填 X / N」实时计数（struct 字段输入时即时更新，刷新后按已存答案重算）。
   - 底部新增「本题各空都可留空——不必每格都写满」显式许可，与「非任务感」基调一致。
   - reality_context 五字段占位从长示例句精简为短中性提示（如「稳定 / 可接受变化 / 有不能退让的底线」），减轻「信息墙」导致的浅填。
2. **② reality_classification（最高优先）**：
   - 四类释义**默认折叠**（作者反馈：常驻卡片降级为折叠，保留但默认隐藏）。
   - 折叠态常驻一行轻提示：「不知道怎么区分？试着问自己：「如果没有它，我现在的生活会不会明显改变？」」+「点击展开 ▾」；展开后显示四类释义（必须保留＝绝对底线不可妥协 / 希望保留＝尽量争取迫不得已才让步 / 可以调整＝形式不限满足核心目的即可 / 还不确定＝先放着之后再做决定）及一个通用归类示例。
   - 首个 item 额外给出引导示例（「先试着给『X』选一个：少了它就冲突，还是换种形式也行？」）。
   - 折叠切换为 per-render 绑定（`[data-classify-legend-toggle]`），沿用 document 级/元素级委托架构；`aria-expanded` + `hidden` 同步。
   - **data-priority 值（must/prefer/flexible/unknown）完全不变，旧数据 100% 兼容**。
3. **③ core_values 排序**：
   - 将「越靠前越重要（仅代表现在）」从 guide 段落搬入组件常驻文案（`.value-note`），用户不必回看引导即懂。
   - 列表顶端新增「↑ 越上越重要」视觉线索；每行前新增「≡」拖拽手柄图标（title=可拖拽排序）。
4. **④ 开放题示例回声削弱（life_vision / desired_identity / muted_version / experiment_goal / experiment_action）**：
   - placeholder 由完整示范句降级为提问式引子（差异化起头，避免统一模板感）。
   - 原示例句移入各题已有的折叠 scaffold（仅主动展开的用户可见），保留个体差异。

**回归验证**：`node --check` 抽取 `<script>` 校验 JS 语法 OK；`data-priority` 值不变、STATE_VERSION=7、字段结构与 STEPS 阶段数（7 阶段 17 题）均未改动；`.value-row` 仅由 valuesorter 使用，4 列网格（grip/rank/content/controls）不影响其他视图；print 媒体查询仅重置阴影，无网格冲突。

## v5.20（2026-08-15）— 方向地图重构为「语义方向地图」+ 事件去泄漏 + Record 折叠统一

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.20.html`（由 v5.19.html 复制后升级版本号，v5.19.html 保留不动）

**改动（零数据结构 / 零字段 / 零 localStorage 改动，STATE_VERSION 维持 7）**：

1. **方向地图从 SVG 硬坐标改为 CSS Grid 语义地图（修复 v5.19 实测缺陷）**：
   - 删除 `layoutMapRegions` / `renderMapRegionShapes` / `renderMapPaths` / `computeNeedleAngle` 四段基于固定 viewBox 坐标的布局逻辑；SVG 只保留「桥」与「指南针」两类抽象几何，不承载任何用户文本。
   - 地图主体改为 `.map-canvas` CSS Grid（dest / values / bounds / bridge / terrain / landing 六个区域），节点是原生 `<button>`（真实字号、可换行、可聚焦、可读屏），因此**任何数据量、任何屏宽都不会重叠、不会文本溢出、不会缩到不可读**。
   - 满填 38 节点实测无重叠/无溢出（v5.19 同场景实测 6/49 节点越框、29/49 文本溢出 SVG、life/work/reality 竖向间距 <26px 均已消除）。
2. **指南针 90° 朝向偏差修复**：v5.19 `computeNeedleAngle` 缺失 `+90°`，needle 实测偏差 90°；v5.20 改为纯视觉隐喻（固定指向下方落点），不再由答案计算「你的人生方向」，符合「地图只组织不解释」原则。
3. **事件去泄漏**：`bindCompassEvents()` 原每次 `renderCompassShell` 都用 `addEventListener` 重新绑定 `.mode-btn` / `.map-node` / `[data-expand]`，元素 innerHTML 重建后监听器累积（实测 mode-btn 来回切换 11 次后累积 11 个监听器，多次触发）。改为 `document` 级**一次性事件委托**（`_compassDelegated` 幂等守卫），无论视图重建多少次全局只绑定一次；`.progress-jump-btn` 同样改为委托。
4. **Record Mode 长文统一折叠**：结构化字段（reality_context / work / experiment 等 `.record-answer`）原本不折叠，长文会把页面撑到极长；新增 `recordAnswerHTML()` 统一对 >280 字回答折叠并提供「展开全文 / 收起」，**绝不静默截断**。去掉 01 章 descriptor 与问题重复。
5. **折叠遮罩与打印**：折叠渐变遮罩底色由 `var(--panel)` 改为 `var(--panel-strong)`（解决 print 时渐变露白）；新增 `@media print` 强制展开全部回答，导出 PDF 不再被截断。
6. **空区域「去这一题」**：未填写的区域渲染 `data-state="empty"` 占位 + `data-jump-key` 跳转按钮，点击经委托跳到对应阶段（与「不强行制造节点」一致）。
7. **移动端**：`@media (max-width: 760px)` 下 `.map-canvas` 退化为单列纵向阅读，节点仍为可聚焦 HTML 按钮；落点三步改纵向箭头。
8. **可访问性**：地图节点为原生 button + `aria-label`，Detail Panel 标题改用问题标签（node.field），不再把长回答当标题。

**新增/修改组件**：`renderCompassMap()`（Grid 语义地图）、`emptyMapStateHTML()` / `needleBadgeHTML()` / `bridgeFigureHTML()` / `renderGroupedNodes()` / `renderTerrainNodes()` / `renderLandingNodes()` / `mapNodeHTML()`、`recordAnswerHTML()`、`onCompassClick()` / `handleModeSwitch()` / `toggleRecordExpand()` / `jumpToQuestion()`；`buildMapNodes()` 节点增加 `field` / `priority` 字段（溯源不变）。

**验证**：`node --check` 抽取 `<script>` 校验 JS 语法 OK；jsdom 运行时回归 **11/11 通过、0 console error**（空状态 / 少量 / 混合完成 / 满填 38 节点 / 模式往返 / 节点详情 / 无双重转义 / 长文折叠 / 事件委托去泄漏 / saveState 落盘 / buildCompass）。重叠/溢出/needle 三项为结构性修复（Grid 不重叠、needle 改为静态隐喻），由代码审查确认。

## v5.19（2026-08-15）— Compass 展示页重构：个人方向地图 + 完整记录

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.19.html`（由 v5.18.html 复制后升级版本号，v5.18.html 保留不动）

**改动（零数据结构 / 零字段 / 零 localStorage 改动，STATE_VERSION 维持 7）**：
1. **双模式 Compass 页面**：最终页面从「本轮罗盘 + 填写总览」并列布局，重构为 **Map Mode（方向地图）** 与 **Record Mode（完整记录）**，默认进入 Map Mode。
2. **统一 View Model**：新增 `getCompassData()`，从现有 `state.answers` 派生统一的 Compass 数据模型。Map 与 Record 共享同一份数据，不新增 localStorage、不复制答案、不修改 answer schema。
3. **Map Mode（方向地图）**：
   - 使用 SVG 渲染六个区域：生活图景、在意与取舍、边界与自己的声音、从生活到工作、现实地图、猜想与尝试。
   - 每个地图节点均可追溯到原始 answer key，不生成新的「人格分析」「数值评分」或伪洞察。
   - 节点点击打开 Detail Panel，显示所属模块、对应问题与用户原始回答，并可跳转 Record Mode 对应位置。
   - 指南针 needle 指向最后的 experiment 节点，仅作视觉隐喻。
4. **Record Mode（完整记录）**：
   - 按 Journey 顺序连续展示 7 个章节的问题与原始答案。
   - 长文本默认完整展示，保留换行与段落；超过 280 字提供「展开全文 / 收起」。
   - 问题作为辅助信息缩小字号，回答作为主体放大字号，营造私人思考记录的阅读体验。
5. **填写总览改造**：原「填写总览」从 Compass 页面主视觉中移除，改为顶部 secondary action「填写进度」，点击打开右侧 Progress Drawer。
6. **响应式**：桌面端地图占据主空间（最大 1320px），移动端单列长文，地图保持可读尺寸，Detail / Progress 变为全宽 drawer。
7. **导出与 Insight 保持原样**：`buildCompass()`、`generateInsights()`、`exportMarkdown()`、`downloadCompassReport()` 等逻辑未改动，继续可用。

**新增组件/函数**：
- `getCompassData()`、`buildMapNodes()`、`renderCompassShell()`、`renderCompassMap()`、`renderCompassRecord()`
- `openCompassDetail()` / `closeCompassDetail()`、`openProgressDrawer()` / `closeProgressDrawer()`
- `renderProgressDrawer()`、`bindCompassEvents()`
- CSS：`.compass-shell`、`.compass-mode-toggle`、`.compass-map`、`.compass-record`、`.compass-detail-panel`、`.progress-drawer` 等。

**验证**：`node --check` 抽取 `<script>` 校验 JS 语法 OK；jsdom 运行时测试 8/8 通过（空状态、短文本、长文本、节点详情、混合完成状态、模式切换、Progress Drawer、Markdown 导出）。

## v5.18（2026-08-14）— 代码与交互性能优化：防抖保存 + 局部刷新

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.18.html`（由 v5.17.html 复制后升级版本号，v5.17.html 保留不动）

**改动（零结构 / 零字段 / 零数据模型，STATE_VERSION 维持 7）**：
1. **保存防抖**：`setAnswer()` 默认由 `saveState()`（立即同步写）改为 `saveState(false)`（600ms 防抖）。高频 change/click 事件不再每次同步写 localStorage；关键路径（步骤导航、导入、生成 Compass）仍显式 `saveState(true)` 立即落盘，`beforeunload` 兜底刷新未完成的防抖写入。文本输入类（textarea / struct / value-explain）此前已走 `saveState(false)`，本次统一收敛到同一策略。
2. **局部刷新（替代每次回答全量重建 DOM）**：新增 `softRefresh(changedId)` 作为回答后的默认刷新路径——仅更新受影响组件：
   - `renderPhaseTrack` 改为首次构建并绑定一次，之后 `updatePhaseTrack()` 就地更新 `active`/`done` 类与 `defer-count`，不再 `innerHTML` 全量重建、不再重复绑定点击监听；
   - 当前题目仅通过 `refreshQuestionInput(qid)` 重建其 `question-input-host` 并作用域绑定（scoped `bindQuestionEvents`），不重建整段旅程；
   - `renderOverviewModal` 仅在总览弹窗处于 `active` 时构建（`openOverviewModal` 打开前主动构建一次），关闭态不再随每次回答空转；
   - `updateStepFeedback()` 就地更新「你刚刚完成了什么」反馈块，不重建题目。
3. **defer 按钮**改为 `updateDeferUI + updatePhaseTrack + updateStepFeedback` 定向更新，不再触发整段旅程全量刷新。

**验证**：`node --check` JS 语法 OK；v5.17 旧存档可直载（数据模型未变）。

## v5.17（2026-08-14）— 模块四 Q3（muted_version）在「认同感 / 选择权审计」基础上强化「外部声音审计」

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.17.html`（由 v5.16.html 复制后升级版本号，v5.16.html 保留不动）

**范围**：仅调整 Q3 文案，未改数据结构 / 题目位置 / 交互方式 / 视觉设计，STATE_VERSION 维持 7，v5.16 旧存档可直接在 v5.17 加载。

设计目标（用户重新明确）：重新强化 Q3「外部声音审计」的设计初心——不是让用户简单区分"自己 vs 别人"，也不是要求回看前面答案，而是帮助识别那些已被内化、习以为常的"应该"，进一步判断：这些声音来自哪里，以及现在的自己是否仍然愿意选择它。外部影响不被默认描述为负面。

| 元素 | 修改前（v5.16） | 修改后（v5.17） |
|---|---|---|
| label | 回看前面写下的这些想法：哪些是你现在依然愿意选择的？哪些值得重新问一遍"这是我真的想要的吗？" | 有没有一些"应该"，你其实很少问过自己为什么？ |
| guide | 前面的"想要""应该"和"在意"，可能都受过外界影响……有些则只是因为太熟悉，一直被当成"应该"。现在，你还愿意选择哪些？ | 我们很容易把听了很久的话，当成自己的想法：这个年纪应该怎样、别人都在往哪里走、家人觉得什么才算好。它们不一定是错的，只是值得问一句：这是我现在真的想要，还是我只是太习惯这样想？ |
| placeholder | 我现在依然愿意选择的是……；我想重新问问自己的，是…… | 我脑子里的一个"应该"是……\n它可能来自……\n我现在对它的想法是…… |
| scaffold① | 有没有一个你一直觉得"应该如此"的选择，其实从没认真问过自己？ | 有没有一个你一直觉得"应该如此"的选择，其实从没认真问过自己为什么？ |
| scaffold② | 有些期待即使最初来自别人，你现在仍然认同吗？ | 这个"应该"更像从哪里来的：家人、同龄人、社会标准，还是过去的自己？ |
| scaffold③ | 当你觉得自己"落后"或"不够好"时，你真正害怕的是什么？ | 如果没有人因此评价你，你还会想要它吗？ |
| scaffold④ | 如果暂时不考虑别人怎么看，你还会保留哪些选择？ | 哪些外来的期待，你后来已经真的认同了？哪些你想慢慢放下？ |

**说明**：模块四整体框架（phase「边界与我的选择」/ title「看清自己的选择」/ intro / Insight #5「认同感 / 选择权审计」/ 报告章节 label「我愿意继续的选择」/ boundary 阶段反馈）沿用 v5.16，未在本次范围内调整——本次只动 Q3 一题文案。若希望模块叙事也回归"外部声音审计"一致口径，可再单独处理（不在本版本）。

### 验证

- `node --check` 抽取 `<script>` 校验 **JS 语法 OK**（修复了一处：placeholder 最初误写为真实换行符导致 JS 字符串跨行报错，已改为 `\n` 转义）。
- grep 确认 Q3 新文案（"有没有一些应该"、"我们很容易把听了很久的话"、"我脑子里的一个应该"、"更像从哪里来的：家人"、"如果没有人因此评价你"、"哪些外来的期待"）全部落地；旧 Q3 文案（"回看前面写下的这些想法"、"可能都受过外界影响"）在 live 已清除。
- 满足用户「重要要求」：Q3 不出现"回看前面写下的内容"类引导；不使用"自己的声音 / 别人的声音"二选一作为核心提问；外部影响不以负面默认（guide 明示"它们不一定是错的"）；零新增问题 / 字段。
- 遗留 live 的"回看前面"仅出现在 anti_future Q1 scaffold（回看生活图景）与 exploration_design Q1 label（回看前面线索），均属各自题目设计意图，非 Q3 引导。
- 版本标识（footer / meta version / description / 文件头 in-file CHANGELOG）全更新为 v5.17；STATE_VERSION 维持 7，字段 key 未动。

## v5.16（2026-08-14）— 模块四 Q3（muted_version）从「外部声音辨认」重构为「认同感 / 选择权审计」

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.16.html`（由 v5.15.html 复制后升级版本号，v5.15.html 保留不动）

零新增问题、零阶段变动（仍 7 阶段 17 题）、零字段改动、零数据模型改动，STATE_VERSION 维持 7，v5.15 旧存档可直接在 v5.16 加载。

### 重构动机（来自设计审阅）

原「外部声音辨认」框架的隐含假设是错的：它把问题设成"找出哪些想法是别人的 → 删掉"或"找到一个纯粹的'真正的我'"，并用「自己的声音 / 被别人写进去」「读起来像不像你自己的口气」「这样分一分，剩下的会更靠近你自己」这类表达，暗示"外部影响 = 污染"、"辨认声音 = 找到自己"。

真正该问的不是"这是谁说的？"，而是"我为什么相信它？我现在还选择它吗？"。所以这一题的核心从**来源纯度**改为**认同审计**：影响来源 → 当前认同。外部影响不等于要清除——有些期待后来也认同了，仍值得留下；只是有些因为太熟悉，一直被当成"应该"。审计的目的不是分清声音，而是让人能重新选一次，使最终的罗盘不至于"替别人规划人生"。

### 模块四三题定位（重构后）

- **Q1 anti_future**「什么生活，即使很好，也不是我想过的？」→ 负向边界
- **Q2 desired_identity**「我欣赏什么样的人…？」→ 正向自我
- **Q3 muted_version**「这些想法里，哪些是我现在真正愿意选择的？」→ 选择权审计

### ① muted_version 问题重写

| 元素 | 修改前 | 修改后 |
|---|---|---|
| label | 回看前面写下的图景、价值排序和边界——读起来，哪些更像是你自己的声音，哪些更像是被别人写进去的？ | 回看前面写下的这些想法：哪些是你现在依然愿意选择的？哪些值得重新问一遍"这是我真的想要的吗？" |
| guide | 你只是在回看前面写下的内容：哪些读起来像你自己的口气，哪些读起来像在替别人说话？不是要分析自己，只是看看——有些声音，是别人替你写进去的。这样分一分，剩下的会更靠近你自己。 | 前面的"想要""应该"和"在意"，可能都受过外界影响。我们不是要找一个完全不受别人影响的自己，也不用判断哪些声音是对的。只是停下来看看：有些期待，你后来也认同了；有些则只是因为太熟悉，一直被当成"应该"。现在，你还愿意选择哪些？ |
| placeholder | （写下一句你读起来最像"别人写进来的"，和一句最像"自己的"……） | 我现在依然愿意选择的是……；我想重新问问自己的，是…… |
| scaffold | 4 条"回看前面答案 / 心理溯源"混合提示 | 4 条"认同审计"提示（从来没认真问过的"应该" / 最初来自别人但现已认同 / 觉得"落后""不够好"时害怕什么 / 暂不考虑别人怎么看仍保留的选择） |

### ② 配套措辞同步（保持同一套语言）

- 模块四 `phase`「边界与自己的声音」→「边界与我的选择」
- 模块四 `title`「确认不想越过的线，也听见自己的声音」→「确认不想越过的线，也看清自己的选择」
- 模块四 `intro` 去掉"哪些是别人的声音或同龄比较不知不觉写进去的"
- Boundary 阶段反馈：去掉"把一部分别人的期待和真心想要的自己分开了"
- Insight #5 注释「自我声音的分辨（外部声音辨认）」→「认同感 / 选择权审计」，内容由"分辨哪些声音更像别人的期待…夺回方向盘的第一步"改为"把'想要'和'应该'重新过了一遍：哪些即使受过外界影响仍选择留下，哪些只是太熟悉才被当成'应该'…方向盘在你自己手里"
- Insight #3 / #5 / #6 的 `sources` 标签「更像我自己的声音」→「我愿意继续的选择」
- 报告章节 `label`「更像我自己的声音」→「我愿意继续的选择」
- v5.16 版本标识（footer / meta version / description / 文件头 in-file CHANGELOG）全更新

### 验证

- `node --check` 抽取 `<script>` 校验 **JS 语法 OK**（修复了一次脚手架替换导致的 brace 重复 SyntaxError，已通过"new_block 止于 scaffold 闭合 }、不含 question 闭合 }"方式修正）
- 旧框架文本「更像我自己的声音 / 被别人写进去 / 像在替别人说话 / 这样分一分 / 听见自己的声音 / 替别人规划」经 grep 确认**仅存于 CHANGELOG 历史行与 v5.16 meta description（用以说明重构来源），问卷正文全部清除**
- 新框架文本（「我愿意继续的选择」「回看前面写下的这些想法」「边界与我的选择」「看清自己的选择」「认同感 / 选择权审计」「我现在依然愿意选择的是」）全部落地
- `const STATE_VERSION = 7;` 未动，字段 key（muted_version 等）未动，报告 / Insight / Markdown 导出逻辑未动

## v5.15（2026-08-14）— 精修式修改：Trade-off 区分排序与真实冲突 + Reality Context 已有筹码进入回答 + 三个观察层级拉清

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.15.html`（由 v5.14.html 复制后升级版本号，v5.14.html 保留不动）

零新增问题、零阶段变动（仍 7 阶段 17 题）、零数据模型改动，STATE_VERSION 维持 7，v5.14 旧存档可直接在 v5.15 加载。

### ① Core Values：区分「排序（此刻的重要程度）」与「真实冲突（护住谁）」

| 元素 | 修改前 | 修改后 |
|---|---|---|
| core_values guide | 从下面的词里挑几项最贴近的词，最多 5 项，再按重要性排序。排在第一位不代表它永远最重要；只是现在如果必须取舍，你最不愿轻易放下它。 | 从下面的词里挑几项最贴近的词，最多 5 项，再按此刻的重要程度排个序。这个顺序只代表你现在怎么看，不代表它永远不变。 |
| trade_off guide | 不用想一个完整故事。只要想一想：如果真的只能留一个，你更不愿意失去哪一个？为了它，你可以暂时接受什么变化？ | 它们平时可以都重要。真正难的是只能保住一个的时候——你会先护住谁？为了它，你可以暂时接受什么变化？ |
| trade_off keep label / placeholder | 我更不愿意失去哪一个 / 我会先保留其中一个，因为…… | 我会先护住哪一个 / 我会先护住它，因为…… |
| trade_off scaffold | 存在（1 条 prompts + 1 条 example） | 已移除（prompt 与 guide 重复，字段 placeholder 已足够示意） |

**理由**：原 core_values guide 用「如果必须取舍，你最不愿轻易放下它」解释排序，让排序听起来就是取舍预演——这正是「我刚刚不是已经排过了吗？」的来源。本次把排序收敛为「此刻的相对重要程度」，把「冲突时刻」完整留给 trade_off；trade_off 以「平时可以都重要，真正难的是只能保住一个」自然区分两个问题，全程用「护住」替代「失去/牺牲/放下」，不制造心理压力。trade_off 的脚手架因与 guide 重复，已整体移除；guide 与字段占位符已足够引导回答。未使用「上一题测的是…本题测的是…」的产品说明式语言。

### ② Reality Context：让「已有筹码」真正进入回答

| 元素 | 修改前 | 修改后 |
|---|---|---|
| reality_bridge intro | …下面要写的不只是限制，如果过去的经历已经改变了你的选择、能力或判断，也可以一起写下来。 | …这里不只是在记限制，也可以写下你已经拥有的东西：做过的事、攒下的经验、身边的关系、对自己更清楚的判断。这些都可以带着走进下一步。 |
| reality_context guide | 写下每一项现在的状态就好，不用只写限制。如果过去的经历已经改变了你的能力、判断或选择，也可以顺手写进对应的地方。 | 写下每一项现在的状态就好，不用只写限制。已经拥有的也可以写：过去走过的路留下的能力、经验、关系，还有你对自己更清楚的判断——它们都是可以带着继续走的东西。 |
| reality_context scaffold① | 过去有没有「没走成」的尝试？它给你留下了什么——对某个领域的了解、一种能力，或一个更清楚的判断？ | 过去有没有「没走成」的尝试？它给你留下了什么——对某个领域的了解、一种会做的事、一份作品或关系，或一个更清楚的判断？ |
| reality_context scaffold② | 这些留下来的东西，此刻正怎样影响你在收入、地点或时间上的安排？ | 有什么是你已经会做、做过、拥有，或者已经知道自己不想再走的？它们影响着你此刻在收入、地点、时间上的安排，也是可以带着继续走的东西。 |

**理由**：存量此前只存在于提示层，用户进入四（五）个字段后仍只会写限制。本次把「已经拥有的」明确写进 intro / guide / scaffold，并补上「带着走进下一步 / 可以带着继续走」的未来连接——但全部复用现有五个字段（income/location/time/responsibility/energy），不新增输入框、不新增 answer key、scaffold 仍 2 条不增加负担；措辞刻意避开「核心竞争力 / 职业优势 / 资源盘点 / 可迁移能力」等职业规划腔。

### ③ life_vision / ideal_day / work_environment：三个观察层级拉清

| 位置 | 修改前 | 修改后 |
|---|---|---|
| life_vision guide | …身边有哪些人，以及你希望自己大多处于什么状态。 | …身边有哪些人、处在怎样的环境里，以及你希望自己大多处于什么状态。（补「环境」维度，强化「生活整体」观察；intro 原有「先不用急着定义职业或身份」保留） |
| ideal_day intro | 刚才写的是你想靠近的生活方向。现在，把它放进一个普通的星期二。 | 刚才写的是你想靠近的生活方向。现在，把它放进一个普通的星期二，看看它真正过起来是什么样。 |
| ideal_day guide | 不用写得特别理想，只要具体想想：早上怎么开始…越接近日常，越容易看见这套生活真正的样子。 | 不用再描述一遍理想生活，而是把刚才那幅图景放进具体的日子里：早上怎么开始…越具体，越接近这套生活真的过起来的样子。 |
| lifestyle_to_work intro | 先不急着找职业名称。回到你想要的日常：工作以什么样的方式进入你的生活… | 先不急着找职业名称。把前面那幅图景和那个星期二，再往前推一步：工作以什么样的方式进入这样的生活… |
| work_environment guide | 从节奏、地点和与人配合的方式想一想。先不用考虑具体职业。 | 从节奏、地点和与人配合的方式想一想，先不用考虑具体职业。这一题只看工作本身的形状——不用把整套生活再描述一遍。 |

**理由**：三个问题通过承接词自然形成「生活整体 → 图景落到一天 → 只看工作本身的形状」的递进，消除「自由、不坐班、陪家人」连写三遍的重复。Ideal Day 保留「普通星期二」设计，只强化「图景放进具体日子」的承接；Work Environment 用「只看工作本身的形状——不用把整套生活再描述一遍」轻轻拉回，无纠错口吻；未使用「第一层/第二层」等产品术语，未把逻辑直接展示给用户。

### 验证

`_verify_v515.js` 静态 + 运行时回归 **68/68 通过**：三处新文案全部落地、12 处旧文案全部清除、JS 语法有效、7 阶段 17 题结构不变、trade_off（keep/accept）与 reality_context（income/location/time/responsibility/energy）字段 key 不变、空/部分/完整三种填写进度计算正常、buildCompass 报告 9 项坐标正常（无 undefined / [object Object]）、generateInsights ≥3 条正常、7 阶段反馈正常、Markdown 导出（含 reality_classification 分类映射与 trade_off 双字段）正常。`diff v5.14 v5.15` 确认差异仅含上述文案与版本标识，零误伤其他区域。

## v5.14（2026-08-14）— 三处文案打磨（降心理审计感 + 去说教 + 去重复）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.14.html`（由 v5.13.html 复制后升级版本号，v5.13.html 保留不动）

零新增问题、零阶段变动（仍 7 阶段 17 题）、零数据模型改动，STATE_VERSION 维持 7，v5.13 旧存档可直接在 v5.14 加载。

- **① exploration_design intro**：删除"成本不高"，避免与 Q2 guide（"想一个成本不高、这段时间真的做得到的尝试"）重复
- **② Boundary Q3（muted_version）降低心理审计感**：
  - label 由"你能分出哪些更像你自己的声音，哪些更像别人的声音吗？"改为"读起来，哪些更像是你自己的声音，哪些更像是被别人写进去的？"
  - guide 由"把不同的声音分开看：哪些来自家人的期待、同龄人的比较、网络上的'标准'？"改为"你只是在回看前面写下的内容：哪些读起来像你自己的口气，哪些读起来像在替别人说话？"，并加"不是要分析自己"
  - scaffold 4 条全部从"心理溯源追问"改写为"回看前面的答案"的轻量提示（去掉"你在拿谁的坐标衡量自己？你们在同一条赛道、同一个起点上吗？"等分析式措辞）
- **③ Insight #6**：去掉"就像赛车没有后视镜"隐喻，保留洞察（横向比较只是分心而非导航，关注自己写下的方向与边界），降低说教感

**验证**：三处文案改动，结构与逻辑未动；STATE_VERSION 维持 7；v5.13 旧存档可直载。

## v5.13（2026-08-13）— 交互打磨：自动保存反馈降频 + 礼花升级

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.13.html`（由 v5.12.html 复制后升级版本号，v5.12.html 保留不动）

**① 自动保存反馈重构（解决"提示太频繁"）**
- 移除每次输入停顿就弹出的深色大胶囊 `toast("已自动保存")`（原 L2670 防抖后必弹）
- 改为 topbar **常驻状态灯**：左侧 7px 小圆点 + "已保存 HH:MM" 文字持续给安全感
- 成功保存时圆点变柔和绿并**重放一次极轻脉冲**（box-shadow 扩散 1s，非持续动效），用 reflow 重启动画确保每次保存都触发
- `updateSaveState(text, saved)` 新增 saved 参数驱动；失败分支显式传 false（保持琥珀色，不脉冲）
- `toast` 保留给真正需要打扰的事件（保存失败 / 加密保存 / 报告导出）

**② 完成礼花（confetti）升级（解决"不够盛大、单色单一"）**
- 由中心 26 个单色符号 → **底部喷发 96 片**，12 色温暖调色板（粉/琥珀/绿/蓝/紫等）
- 四种形状：圆点 / 方块 / 长条 / 星芒字形（✦✧★✺✸❉✪✶）
- keyframe 增加 45% 峰值弧线，模拟"上喷→飘落"的重力感；每片随机旋转 0–720°、时长 2.8–4.6s、错峰 0–0.4s 喷发

**验证**：`_verify_v513.js` 静态回归 17/17 通过（旧 toast 已移除、confetti 新方案落地、状态灯 + 弧线 keyframe 存在、STATE_VERSION 维持 7、JS 语法有效）。v5.12 旧存档可直载。

## v5.12（2026-08-13）— 落实整体审阅三轮修订（P0 + P1 + P2）

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.12.html`（由 v5.11.html 复制后升级版本号，v5.11.html 保留不动）

**来源**：`v5.10_问卷整体审阅与优化建议.md` 的分级优化方案。零新增问题、零阶段变动（仍 7 阶段 17 题）、零数据模型改动，STATE_VERSION 维持 7，v5.11 旧存档可直接在 v5.12 加载。

### P0 · 第一轮：6 处文案 + 1 处逻辑修复（约 30 分钟，零结构影响）
- ideal_day（白天字段）：placeholder 去「例如：例如：」复制笔误
- trade_off（label）：「如果前两项」→「如果刚才挑出的几项价值」，与 scaffold「其中两个」口径一致
- trade_off（accept placeholder）：去句末「……。」叠用
- work_constraints（选项）：「地理位置限制」→「工作地点要求」（消除歧义）
- experiment_action（guide）：去「缩小规模」与「成本不高」语义重复
- **reality_classify 完成度判定修复（逻辑 bug）**：`isAnswered` 对 reality_classify 加特判——work_constraints 派生的待分类项须全部归类才算已答，与 UI `allClassified` 口径一致；原判定只查数组非空，导致只归类 1/N 就被算「已完成」、进度虚高，可能过早显示"已完成 · 最终 Compass"

### P1 · 第二轮：报告与承接对齐
- 报告坐标「匹配的工作环境」→「匹配的工作方式」（与 v5.8 口径对齐；buildCompass 同时供屏幕报告与 Markdown 导出，一处改全覆盖）
- experiment_action（scaffold）：补一条「这个尝试，是为了靠近前面写下的哪一幅生活图景」的愿景连接（弥补愿景↔30 天实验的中期断点）
- adaptability（guide）：点明与上一题「现实里哪些能调整」的分工——上一题判断现实弹性，本题是你愿意亲自迈出的一步

### P2 · 第三轮：可选增强（随版本号推进）
- reality_context 新增「健康与精力状态」字段（增量 key `energy`，旧存档缺省即可，无需升 STATE_VERSION）；同步 buildCompass `realityText` 与 Insight#7 `hasRealityDepth` 的字段列表
- 报告「不想活成什么样」改为「场景 / 原因」分段展示（原 scene/reason 直接拼接换行，现带前缀提升可读性）

### 验证
- `_verify_v512.js` 精准回归：29/29 通过（新串落地、旧串清除、STATE_VERSION=7、JS 语法有效、无重复 id）

## v5.11（2026-08-13）— 整合发布：汇总 v5.10.2 ～ v5.10.9

**输出文件**：`最终公开版inner_compass_v5_optimized_v5.11.html`（由 v5.10.html 复制后升级版本号，v5.10.html 保留不动）

### 本次整合涵盖的全部增量（均已在 v5.10.x 原地落地）
- v5.10.2 模块四 Q1（anti_future）：问题/引导改为「不是你真正想靠近的生活」
- v5.10.3 模块三 Q2（trade_off）：去「想象冲突场景」，直接进入取舍
- v5.10.4 模块四 Q2（desired_identity）：双入口辨认「想成为的自己」，降低对前序依赖
- v5.10.5 模块五 Q2（work_constraints）：引导语去「工作侵占生活」预设
- v5.10.6 模块六 Q2（reality_classification）：标题/引导重定位为「判断边界在现实里的弹性」
- v5.10.7 清理两处残留脚手架（desired_identity / work_constraints）
- v5.10.8 模块七「先试，再看」实践循环升级（认识→实践→新认识）
- v5.10.9 模块七 Q3（success_criteria）清理残留脚手架

### 升级动作（仅版本标识，无逻辑改动）
- 文件头 `Version: v5.11`、`<meta name="version" content="v5.11">`、description、footer「Self Discovery Framework v5.11」
- 文件头新增 v5.11 整合说明注释
- **STATE_VERSION 维持 7**：数据模型零改动，v5.10 旧存档可直接在 v5.11 加载，不受影响

## v5.10.9（2026-08-13）— 模块七 Q3（success_criteria）清理残留脚手架

### 改了什么

| 元素 | 修改前 | 修改后 |
| --- | --- | --- |
| scaffold.prompts | 1 条「能量：做完之后你是想『再来一次』，还是『终于结束了』？」 | 整个 scaffold 对象删除（按钮与提示均不再渲染） |

### 影响
- 仅删除 scaffold 对象；label/guide/type/structured 三字段（energy_signal / learning_signal / identity_signal）/STATE_VERSION=7 / Insight 与报告对 `success_criteria` 的 2 处读取均不变，旧存档兼容。
- `renderScaffold` 在无 scaffold 时返回空，故 "💭 不知道从何写起？" 按钮不再渲染。

## v5.10.8（2026-08-13）— 模块七「猜想与尝试」（exploration_design）文案升级：认识→实践→新认识循环

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.10.html`（原地修改）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改 7 阶段 / 17 题、不改报告读取逻辑。借鉴"实践论"内核，把最后一步从"拿去试试"升级为明确的「先试，再看」实践循环：当前认识/猜想 → 实践 → 从现实获得信息 → 重新认识。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| 阶段标题 title | 把一个念头拿去试试 | 先试，再看 |
| 阶段引导 intro | 前面整理的是你的线索，不是答案。现在挑一个你还想知道的东西，给它一次低成本的现实尝试。它可以关于工作，也可以关于生活。重点不是证明自己，而是得到一点真实信息。 | 前面整理的是你目前的线索，不是最终答案。真正放进生活之后，很多事情才会变得清楚。现在挑一个你还想知道的念头，先做一次成本不高的尝试，再看看现实会告诉你什么。 |
| Q2（experiment_action）guide | …写不出具体动作，就先把规模缩小一半再写。 | …写不出具体动作，就先把规模缩小一半再写。不需要一次改变很多，想一个成本不高、这段时间真的做得到的尝试。 |
| Q3（success_criteria）guide | 不是判断成败，而是收集信息。可以观察：做的时候有没有投入感，这次尝试让你发现了什么，以及你还想不想再来一次。 | 不是判断成败，而是收集信息。可以观察：做的时候有没有投入感，这次尝试让你发现了什么，以及你还想不想再来一次。不必提前规定它应该证明什么。留意真实发生了什么，以及它有没有让你对这个念头多知道一点。 |

**未改动**：phase（猜想与尝试）、Q1（experiment_goal，用户确认已具备"当前认识/猜想"结构无需改）、Q2/Q3 的 label/type/placeholder/structured 字段、scaffold、STATE_VERSION=7、报告/Insight/导出。

**改动意图**：
- 标题「先试，再看」点明"实验不是证明猜想正确，而是获得更多信息"的方法论（与代码中 Q3 成功=获得信息 的设计原则一致）。
- intro 把"认识→实践→新认识"完整藏入一句，无哲学腔。
- Q3 guide 增补"不必提前规定它应该证明什么"，把实验从"证明我对"转向"让我知道更多"。

## v5.10.7（2026-08-13）— 清理两处残留脚手架与示例（desired_identity / work_constraints）

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.10.html`（原地修改）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改 7 阶段 / 17 题、不改报告读取。本轮收尾 v5.10.4 标注的两处残留，并删去 tag 选择题下的脚手架（与"不需要写"场景不匹配）。

### 修改前后对比

**① 模块四 Q2 desired_identity**（textarea）

| 元素 | 修改前 | 修改后 |
|---|---|---|
| placeholder | 例如："能安心做自己的事，也敢对不想要的说'不'的人"…… | （写下一个你欣赏的人、一类人，或希望自己身上慢慢长出来的某种状态……） |
| scaffold | 思考支架按钮"💭 不知道从何写起？" + 提示"图景里的那个你，最常处于什么状态？用一个词形容会是什么？" | 已移除整个 scaffold 对象（renderScaffold 在无 prompts/examples/starter 时返回空，故按钮不再渲染） |

**② 模块五 Q2 work_constraints**（multitag）

| 元素 | 修改前 | 修改后 |
|---|---|---|
| scaffold | 思考支架按钮"💭 不知道从何写起？" + 提示"回看刚写下的节奏、地点和协作方式——哪一条你其实最不愿让步？" | 已移除整个 scaffold 对象 |

**改动意图**：
- desired_identity：新引导语（v5.10.4）已双入口化并拆解"欣赏的究竟是什么"，原脚手架提示仍硬绑"图景里的你"已与新定位不符，删除；placeholder 内的具体品质示例也一并换为与新引导语对齐的中性表述，避免价值观植入。
- work_constraints：本题为多标签选择（multitag），不需要"不知道从何写起"的写作支架，删除整个 scaffold 以保持界面与题型一致。

**未改动**：label、guide、type、options、STATE_VERSION=7、报告/Insight/导出对 desired_identity / work_constraints 的读取逻辑。

**通用渲染逻辑确认**（v5.2 已有）：`renderScaffold` 在 `scaffold` 缺失或 prompts/examples/starter 全空时返回 `""`，因此删除整个 scaffold 对象与"保留空对象"在 UI 上等价，但数据更干净。本轮统一采用删除整个 scaffold 对象。

## v5.10.6（2026-08-13）— 模块六 Q2（reality_classification）标题+引导语重定位：从「不能动」到「现实里多大调整空间」

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.10.html`（原地修改）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改 7 阶段 / 17 题。只重写模块六「把方向放回现实」Q2（reality_classification）的标题（label）与引导语（guide）。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| 标题 label | 把想要的生活放回现实：哪些条件不能轻易动？ | 把想要的生活放回现实：哪些条件现在需要保留？ |
| 引导语 guide | 前面已经从生活里找出了你在意的东西，也把它们翻译成了具体的工作条件。现在不必重新排序，只看看：在你目前的现实里，哪些条件需要保留，哪些可以换一种方式实现，哪些可以暂时调整，哪些还需要更多信息。 | 前面已经把你在意的生活线索翻译成了具体的工作条件。现在换一个角度：不是再判断它们有多重要，而是看看在你目前的现实里，它们有多难调整。哪些现在必须保留，哪些希望保留但可以换一种方式实现，哪些可以调整，哪些还需要更多信息？ |

**改动意图**：
- 明确模块五 vs 模块六分工——模块五「形成边界」（我希望工作具备什么），模块六「判断边界在现实里的弹性」（这些条件在现实中有多大调整空间）。
- 删除"哪些条件需要保留、哪些可以……"的原有重复表达；引导语改为"换一个角度：不是判断多重要，而是看多难调整"。
- 标题由"不能轻易动"改为"现在需要保留"，不再提前暗示"不能动"，与四个分类（必须保留/希望保留/可以调整/还不确定）解耦。
- 保留四个分类不变；未改问题类型（reality_classify）、max=10、选项、STATE_VERSION=7、报告/Insight/导出对 reality_classification 的读取逻辑。

**标题歧义处理**：用户给"需要守住 / 现在需要保留"两个候选并说"更偏向第二个"。本版按"第二个=现在需要保留（更中性项）"落地；若用户原意指"需要守住"，将 label 改回即可，一行改动。

## v5.10.5（2026-08-13）— 模块五 Q2（work_constraints）引导语去「工作侵占生活」预设

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.10.html`（原地修改）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改 7 阶段 / 17 题。只重写模块五「把生活翻译成工作」Q2（work_constraints）的引导语。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| 引导语 | 刚才写的是"什么样比较适合"。现在把前面的生活线索再往现实里翻译一步：如果长期缺少某些条件，你会觉得工作正在侵占你想要的生活。哪些条件你不想轻易让步？ | 刚才写的是你想要的生活。现在把这些生活线索再往工作里翻译一步：如果一份工作要长期成为你生活的一部分，你希望它具备哪些条件，才能和你想靠近的生活相容？哪些条件对你来说比较重要，不想轻易让步？ |

**保留**：问题本身（label）、选项列表、type（multitag）/max=5、scaffold 提示、STATE_VERSION=7、报告 3 处对 work_constraints 的读取逻辑。
**改动意图**：保留"从生活图景反推工作条件"逻辑；删除"工作正在侵占你想要的生活"预设，避免暗示工作与生活必然对立。

## v5.10.4（2026-08-13）— 模块四 Q2（desired_identity）降低对前序填写的依赖：双入口辨认「想成为的自己」

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.10.html`（原地修改）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改 7 阶段 / 17 题。只重写模块四「边界与自己的声音」Q2（desired_identity）的提问与引导语。目标：保留"辨认自己希望成为怎样的人 / 希望拥有什么品质"的核心设计意图，但不再强依赖"必须先写好生活图景"，改为"欣赏他人"与"回看自己的图景"双入口，且不植入具体人物或品质示例。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| 问题 | 回看刚才的生活图景和价值排序——有没有一种你希望自己成为的人，已经隐约出现在里面？ | 你欣赏的人身上，有没有什么特质，是你也希望自己慢慢拥有的？ |
| 引导语 | 不用给自己贴标签。从你写下的图景里，能不能看出你希望自己身上拥有什么品质？ | 可以从一个具体的人、一类人，或者前面写下的生活图景里找。想想你欣赏的究竟是什么：一种做事的方式、对人的态度，或面对生活的状态。也可以不对应任何人，只写你希望自己拥有的品质。 |

**未改动**：type（textarea）、placeholder、scaffold（仍保留一处「图景里的那个你」提示）、其余 6 阶段与 STATE_VERSION=7、报告 3 处对 desired_identity 的读取逻辑。

**残留待确认**：① scaffold 提示仍写「图景里的那个你，最常处于什么状态？」，与"双入口、不强制前置图景"的新定位略有出入；② placeholder 仍含具体品质示例（"能安心做自己的事，也敢对不想要的说'不'的人"），与"不植入价值观"精神略有出入。两者均为改动前的既有内容，按"只改指定文案"原则本次未动，待用户确认是否一并调整。

## v5.10.3（2026-08-13）— 模块三 Q2（trade_off）去「想象冲突场景」：直接进入取舍

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.10.html`（原地修改，v5.10 原文件即本次修订对象）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改 7 阶段 / 17 题。只重写模块三「在意与取舍」Q2（trade_off），把"让用户想象一个价值冲突场景"改为"直接进入取舍"——不再要求编故事，而是问当两个都重要的东西不能同时被满足时，优先保留哪一个、为它愿意暂时接受什么变化。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| Q2 label | 想象一个真实场景：前两项只能选一个，你会怎么选？你愿意为这个选择放弃什么？为什么？ | 如果前两项暂时不能同时拥有，你会先保留哪一个？ |
| Q2 guide | 不用讲大道理。想一个可能发生的场景，说说你会怎么选，以及你愿意承担什么。 | 不用想一个完整故事。只要想一想：如果真的只能留一个，你更不愿意失去哪一个？为了它，你可以暂时接受什么变化？ |
| 类型 | textarea | structured（2 字段：keep / accept） |
| field `keep` label / placeholder | — | 我更不愿意失去哪一个 / 我会先保留其中一个，因为…… |
| field `accept` label / placeholder | — | 为了它，我可以暂时接受什么变化 / 另一个我可以暂时接受……。 |
| scaffold prompts | 什么情况下这两个会打架？比如一个机会只能满足其中一个——你会怎么选？ | 可以回看前面挑出的那几项价值：如果其中两个暂时不能同时满足，你更不愿意放下的是哪一个？ |
| scaffold examples | 含具体价值观（"如果稳定和自主需要二选一，我现在更想保留一点自主……"） | 只示范思考结构，无价值观植入（"例如：我会先保留其中一个，因为……；另一个我可以暂时接受……。"） |

### 刻意保留 / 未改动

- `trade_off` 原 answer 为字符串，报告 / Insight / 阶段反馈均未读取（grep `getAnswer("trade_off")` 无结果），改为 structured 双字段不影响任何下游逻辑。
- 模块三 Q1（core_values）、其余 6 阶段、STATE_VERSION=7、STORAGE_KEY、保存/恢复/导出结构均不变。
- 旧存档中若存在 trade_off 旧字符串答案，新结构不再读取，无副作用。

---

## v5.10.2（2026-08-13）— 模块四 Q1（anti_future）文案重构：从「不想靠近的未来」到「不是你真正想靠近的生活」

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.10.html`（原地修改，v5.10 原文件即本次修订对象）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改 7 阶段 / 17 题。只重写模块四「边界与自己的声音」Q1 的提问框架，把认知锚点从「想象一个别人看起来不错的未来」改为「回头看前面写下的生活图景——哪一种生活方式你其实不想这样过」，与用户「承接式提问」「结构优先于文案」的偏好保持一致。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| Q1 label | 哪一种未来，会让你觉得"看起来没问题，但我不想这样过"？ | 有没有一种生活，看起来也很好，却不是你真正想靠近的？ |
| Q1 guide | 想象一个别人看起来还不错、但你并不想靠近的未来。它常常能帮你更快看见自己的边界。 | 不必判断它好不好，也不用和别人比较。回头看看前面写下的生活图景：有没有一种生活方式，即使它本身没有什么问题，你却知道自己不太想把人生过成那样？ |
| field `scene` label | 那样的日子会是什么样 | 我不想慢慢习惯这样的生活 |
| field `scene` placeholder | 在那种未来里，我每天______，慢慢地我不再______。 | 我不想慢慢习惯这样的生活：…… |
| field `reason` label | 在这种状态下，我最怕失去什么 | 我为什么确定它不是我想要的 |
| field `reason` placeholder | 我最怕失去______。 | 它可能没有什么不好，只是我不想把自己的生活过成那样，因为…… |
| scaffold prompts | 3 条（未来里普通的一天 / 怕变成哪种状态 / 是否见过别人那样活成了参照） | 1 条（可以回看前面写下的生活图景：有没有什么状态，你曾经觉得"这样也可以"，现在却越来越确定自己不想长期如此？） |

### 刻意保留 / 未改动

- 字段 key（`scene` / `reason`）不变；`type: "structured"` 不变；报告读取 `anti.scene` / `anti.reason` 的 3 处逻辑（阶段反馈、Insight #1 关键词匹配、Compass 报告）不受影响。
- 模块四其余两题（desired_identity / muted_version）、其余 6 个阶段、STATE_VERSION=7、STORAGE_KEY、保存/恢复/导出结构均不变。
- 旧存档可直接加载，无需迁移。

---

## v5.10.1（2026-08-12）— Reality Bridge 重构：从「混合分类」到「价值 → 工作条件 → 现实判断」

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.10.html`（原地修改，v5.10 原文件即本次修订对象）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改 7 阶段 / 17 题。只调整 Reality Bridge 的「数据来源」与「表达」，把原本混在一起分类的三类内容（core_values + work_constraints + anti_future）收窄为只取 work_constraints，从而形成更清晰的认知链：价值（在意与取舍）→ 工作条件（从生活到工作）→ 现实判断（把方向放回现实）→ 小实验（猜想与尝试）。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| Reality Bridge 分类对象 | core_values + work_constraints + anti_future 混在同一坐标系 | 只取 work_constraints（来自「从生活到工作」） |
| 标题 | 把前面在意的东西放回现实：现在，它们可以怎样成立？ | 把想要的生活放回现实：哪些条件不能轻易动？ |
| guide | 前面已经说过什么对你重要……有些需要现在就满足，有些可以换一种方式实现，有些可以暂时让步，还有些需要更多信息才能判断。 | 前面已经从生活里找出了你在意的东西，也把它们翻译成了具体的工作条件……只看看：在你目前的现实里，哪些条件需要保留，哪些可以换一种方式实现，哪些可以暂时调整，哪些还需要更多信息。 |
| 按钮（must） | 现在就需要满足 | 必须保留 |
| 按钮（prefer） | 可以换一种方式实现 | 希望保留 |
| 按钮（flexible） | 可以暂时让步 | 可以调整 |
| 按钮（unknown） | 还需要更多信息 | 还不确定 |
| 分类项来源提示 | 来自「在意与取舍」/「工作条件」/「边界」 | 统一为「来自『工作条件』」 |
| 空状态 | 前面还没有留下可归类的内容。 | 前面还没有留下工作条件。先回到「从生活到工作」填写一些你不想轻易让步的条件…… |
| work_constraints guide | 刚才写的是"什么样比较适合"。现在再想想，其中哪些如果长期缺失…… | 刚才写的是"什么样比较适合"。现在把前面的生活线索再往现实里翻译一步：如果长期缺少某些条件……哪些条件你不想轻易让步？ |
| 报告「我的现实地图」分类标题 | 现在就需要满足 / 可以换一种方式实现 / 可以暂时让步 / 还需要更多信息 | 必须保留 / 希望保留 / 可以调整 / 还不确定 |
| Markdown 导出 priority 文案 | 同上旧名 | 同上新名 |
| Insight #8 | 在「现在就需要满足」和「可以暂时让步」之间画出了线 | 在「必须保留」和「可以调整」之间画出了线……去验证还不确定的猜想 |
| reality_bridge 阶段反馈 | 把前面在意的东西放回现实条件里…… | 把想要的生活放回现实条件里——分清哪些必须保留、哪些希望保留、哪些可以调整…… |

### 刻意保留 / 未改动

- **core_values 不删除**：仍用于用户 Compass（「我在意什么」）、Insight、前后阶段语义连接，作为「为什么这些工作条件重要」的上游信息。只是不再作为 Reality Bridge 的待分类条目。
- **anti_future 不删除**：仍用于 Compass（「不想活成什么样」）、Insight、边界相关内容。只是不再与 work_constraints 同层归类。
- **priority 字段值不变**：must / prefer / flexible / unknown 内部值未动，localStorage、JSON 导入导出、Markdown 导出、报告生成、旧数据兼容均不受影响。
- **available_assets**：原 `buildCompass` 中的读取无 UI 来源、不展示，本次移除该未使用读取，不做兼容性假数据。
- **STATE_VERSION = 7、STORAGE_KEY = inner_compass_v5_3、保存/恢复/加密备份/JSON/Markdown 导出结构均不变**。

## v5.10（2026-08-12）— exploration_design 去职业预设：从「验证一个职业方向」到「把一个念头拿去试试」

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.9.html` → `最终公开版inner_compass_v5_optimized_v5.10.html`（v5.9 原文件未作改动）

**定位**：不新增问题、不修改数据结构（STATE_VERSION 维持 7）、不改变三个问题 id（experiment_goal / experiment_action / success_criteria）与信号字段 key（energy / learning / identity）。只修正一个核心问题：原模块把前序线索默认推导为「职业方向猜想 → 职业尝试」，让已满意当前工作、只想调整生活方式或验证生活念头的人产生"这题不是问我的"的疏离感。本次把模块的认知任务从「验证一个职业方向」升级为「把一个关于自己、生活或工作的猜想，拿到现实里试一次」，职业探索从默认路径降为众多可能路径之一。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| 主标题 | 选一个猜想，设计一个小实验 | 把一个念头拿去试试 |
| 阶段 intro | 方向有了，现实也看清了。现在把前面那些"我猜我可能适合……"的猜想选一个出来，设计一个成本不高的小实验，帮你确认：想象中的工作方式，放到真实体验里是什么感觉。 | 前面整理的是你的线索，不是答案。现在挑一个你还想知道的东西，给它一次低成本的现实尝试。它可以关于工作，也可以关于生活。重点不是证明自己，而是得到一点真实信息。 |
| Q1 label | 回看前面写下的图景和工作线索——哪一条还只是你的猜想？这次尝试，你最想先验证它。 | 回看前面写下的线索——有没有一个关于生活或工作的念头，值得拿到现实里试试看？ |
| Q1 guide | 不是证明自己行不行，而是得到一点真实信息。选前面最不确定、也最想知道的一件事。 | 不必找"正确方向"。挑一件你现在还不确定、但很想知道答案的事。它可以关于工作，也可以只是生活里的一个念头。 |
| Q1 placeholder | 例如：我想知道，自己是否真的喜欢长时间做深度分析… | 例如：我想知道自己是不是更喜欢长时间独处完成一件事；我一直想做点手工，但不知道是真的喜欢还是只是喜欢想象中的感觉；我想看看每周留出半天自主安排，状态会不会更好… |
| Q1 scaffold | 前面写的工作特征里，哪一条你最不确定「是不是真的适合我」？就验证它。 | 前面写下的图景、在意的东西、想经常做的事里，有没有一条你还只是「猜」？把它写成你还想知道的问题。 |
| Q2 label | 接下来 30 天，你想做的具体尝试是什么？ | 接下来 30 天里，你愿意怎么试一次？ |
| Q2 guide | 越具体越好——做什么、做几次、什么时间内。写不出具体动作，就先把规模缩小一半再写。 | 越具体越好——做什么、做几次、什么时候做。它不必是一次职业尝试，也可以只是一次活动、一次体验或一段新的生活安排。写不出具体动作，就先把规模缩小一半再写。 |
| Q2 placeholder | 例如：完成一个独立分析项目、以兼职方式参与同类工作、深度访谈 3 位从业者… | 例如：完成一个小作品；连续两周每周留出半天自主安排；找一位正在做这件事的人聊一次真实状态… |
| Q3 label | 做完后，你怎么判断这次尝试有没有收获？ | 这次尝试之后，你想留意什么？ |
| Q3 guide | 不只看结果好不好。也可以观察：做的时候有没有投入感，遇到困难时愿不愿继续，以及你还想不想再试一次。 | 不是判断成败，而是收集信息。可以观察：做的时候有没有投入感，这次尝试让你发现了什么，以及你还想不想再来一次。 |
| learning_signal placeholder | 例如：是否自然想深入了解、是否遇到困难仍愿意继续… | 例如：这次尝试让我发现了什么、是否自然想继续了解… |
| 阶段反馈（已填） | 你已经把一个猜想变成了一个小实验。它不是用来证明你优秀，而是帮你得到真实信息：这类工作方式是否适合你。 | 你已经把一个猜想变成了一次真实的尝试。它不是用来证明什么，而是帮你换回一点真实信息——关于这个念头，也关于你自己。 |
| 阶段反馈（兜底） | 这里不要求你立刻决定职业方向，而是设计一个成本不高的小尝试，看看什么更适合你。 | 这里不要求你立刻找到答案。只是挑一个你还想知道的念头，设计一次成本不高的小尝试，换回一点真实反馈。 |
| 报告项标签 | 设计的验证实验；验证目标：…；实验动作：… | 想拿去试一次的念头；想试的念头：…；打算怎么试：…（并补入此前遗漏展示的 identity_signal「还想不想继续」） |

### 本次设计要点

1. **认知任务重定义**：模块不再默认「工作线索 → 验证职业方向」，而是「线索 → 也许…… → 把念头拿到现实里试一次 → 得到一点真实信息」。试验不是证明自己正确，也不是考试；"成功"被重新定义为"获得信息"。
2. **概念边界：活动 ≠ 职业**：Q1 不再把前序 activity_types 的丰富活动自动解释为职业线索（"照顾植物"不等于"园艺职业"），placeholder 三类示例分别覆盖工作方式 / 活动兴趣 / 生活安排，且不分类标注。
3. **职业探索被保留而非删除**：工作方式、职业方向、副业、专业方向仍可进入流程；只是从"默认答案"降为"众多可能之一"，与生活类猜想平行。
4. **信号体系原样保留**：能量 / 学习 / 还想不想继续三个信号字段的 key、label、顺序不变，仅学习信号 placeholder 从"是否自然想深入了解"改为"这次尝试让我发现了什么"（表达"获得反馈"而非"证明方向"）。
5. **报告展示层补漏**：buildCompass 原遗漏 identity_signal 的展示（data 中一直有存储），本次补入报告与 Markdown 导出，纯展示层改动，不涉及数据结构。

### 刻意保留 / 未改动

- 三个问题 id、type、顺序、scaffold 结构（Q2 scaffold「最小版本」提示原样保留）
- `STATE_VERSION = 7`、`STORAGE_KEY = inner_compass_v5_3`、保存 / 恢复 / 加密备份 / JSON / Markdown 导出结构均不变
- 信号字段 key（energy_signal / learning_signal / identity_signal）与 label（能量信号 / 学习信号 / 还想不想继续）不变
- `generateInsights` 8 条规则、其余 6 个阶段（vision → reality_bridge）零改动

### 反预设审计结论（对应验收标准）

1. **不再暗示用户一定不满意现在的工作 / 一定在找新工作 / 一定想转行 / 一定想做副业**：全模块无一处要求"验证职业"或"寻找方向"的表述；兜底反馈删除了「决定职业方向」。
2. **不再把前序活动自动解释为职业线索**：Q1 去掉「工作线索」「验证它」，改为"关于生活或工作的念头"。
3. **未走向过度中性而削弱职业探索**：工作类猜想（研究型工作、项目制工作、访谈从业者）仍在示例与 guide 中合法存在，与生活类猜想平行。
4. **核心验收**：已满意当前工作的用户看到"我想知道每周保留半天自主安排，状态会不会更好"→"试两周"→"留意自己的状态"，全程不需要任何职业意图，也能自然走完本模块。


**对照文件**：`最终公开版inner_compass_v5_optimized_v5.8.html` → `最终公开版inner_compass_v5_optimized_v5.9.html`（v5.8 原文件未作改动）

**定位**：不新增问题、不修改数据结构、不改变四个现有输入字段（income / location / time / responsibility）、不重构 Reality Bridge。只解决一个核心问题：v5.6.2 引入的「成长存量」概念此前只存在于模块说明文字里，用户进入四个输入框后仍只会把它理解成「填写现实限制」。本次让存量成为填写时的一个轻量思考视角，同时消除 placeholder 对"有压力 / 没时间 / 有负担"的预设。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| 阶段 intro | 先把现实情况放回来——不只是收入、地点、时间，也包括过去的尝试（哪怕是没走成的）给你留下的认知、能力和判断，它们同样是你现在位置的一部分。这里不用判断对错，只需要分清：哪些必须满足，哪些希望保留，哪些还有调整空间。 | 过去走过的路，不论最后有没有继续，都可能已经改变了你：留下了能力，也留下了判断。现在，把现实情况放回来看——下面要写的不只是限制，如果过去的经历已经改变了你的选择、能力或判断，也可以一起写下来。 |
| 本题 guide | 诚实记下现在的位置就好——包括从过去的尝试、失败和选择中带过来的判断力。这些不是理由，是你已经积累下来的坐标。 | 写下每一项现在的状态就好，不用只写限制。如果过去的经历已经改变了你的能力、判断或选择，也可以顺手写进对应的地方。 |
| income placeholder | 例如：目前需要保持稳定收入；可以接受短期投入学习；正在面对较大的经济压力…… | 例如：希望保持目前的收入；短期内可以接受一定变化；收入暂时不是主要限制；希望先积累一些缓冲；也可以写下一两条不能退让的现实底线…… |
| location placeholder | 例如：希望继续留在当前城市；可以考虑迁移；因为家庭或关系有固定地点需求…… | 例如：希望继续留在当前城市；可以考虑迁移；因为家庭或关系有固定地点需求；地点暂时不是主要限制…… |
| time placeholder | 例如：工作之外每周可以投入 5 小时；目前精力有限，只适合小规模尝试…… | 例如：目前时间比较有限；每周有固定的时间可以投入；可以阶段性集中投入；时间暂时不是主要限制…… |
| responsibility placeholder | 例如：家庭责任、照顾需求、健康状态、其他重要安排…… | 例如：家庭责任；照顾家人、孩子或宠物；当前的工作承诺；学业或长期项目；暂时没有特别需要考虑的责任…… |

### 本次设计要点

1. **认知功能重定义**：这一问从「我的现实限制是什么？」改为「我现在站在哪里？手上已经有什么？这些现实条件和过去的经历会怎样影响下一步？」——现实条件 = 限制 + 空间 + 已有基础；成长存量（能力 / 判断）不是新题，而是附着在四个字段之上的视角。
2. **存量进入机制（轻量、不新增问题）**：通过三层递进而非新输入框——① 阶段 intro 首句（"留下了能力，也留下了判断"）建立视角；② guide 明示"可以顺手写进对应的地方"（直接回答"写在哪"）；③ 已有的默认折叠 scaffold（v5.6.2 的 2 条 prompt）作为需要时的展开入口。四个字段下未增加任何"过去经历留下了什么"的重复提问。
3. **placeholder 去预设化**：删除"较大的经济压力""精力有限"等默认困难表达；每个字段给出 4 种以上平行的现实状态（含"暂时不是主要限制"），不排序、不暗示哪种更成熟；income 保留"不能退让的现实底线"的合法出口，但不默认人人都有底线。
4. **附带消除一处 v5.8.1 遗留的不一致**：intro 旧句「哪些必须满足，哪些希望保留，还有哪些调整空间」是 v5.6 时代 reality_classification 旧标签的排序式表述，与 v5.8.1 新分类标签（现在就需要满足 / 可以暂时让步）冲突，本次随 intro 重写一并移除。

### 刻意保留 / 未改动

- 四个字段的 key、label、type、顺序全部不变；scaffold 2 条 prompt 原样保留（折叠入口，认知负荷不变）
- `STATE_VERSION = 7`、`STORAGE_KEY = inner_compass_v5_3`、保存 / 恢复 / 加密备份 / Markdown 导出结构均不变
- `generateStepFeedback`（阶段反馈）、`generateInsights`（8 条规则）、`buildCompass` 报告「我的现实地图」、`extractRealityClassifyItems` 全部不动
- 其他阶段（vision → exploration_design）零改动

### 认知负荷审查结论（对应验收标准）

1. **填写的是"现实位置"而非价值观**：label 保留"现实情况"，guide 首句"写下每一项现在的状态"，与「在意与取舍」（问什么重要）边界清晰。
2. **存量是视角而非新题**：只在 intro / guide 各出现一次表述，写入位置复用现有四字段，无新增输入框、无字段级重复提问。
3. **无预设误导**：四个 placeholder 均含"暂时不是主要限制 / 暂时没有特别需要考虑的责任"等空态示例；无任何示例暗示用户应有经济压力、时间紧张、家庭负担或失败经历。
4. **与「在意与取舍」不重复**：不再问"什么最重要"（该任务由 core_values / trade_off 承担）；本题只问现状状态。
5. **与「从生活到工作」不重复**：work_environment 问"理想的工作方式形状"，work_constraints 问"工作上不能牺牲的条件"；本题问"现在实际的状态"（是否、多少、有没有），"想要 / 不能牺牲"的语义未进入本题。
6. **为「现实分类」做好准备**：四字段产出 = 现实条件（限制 + 空间）与已有基础（能力 / 判断）的原始地图；reality_classification 在此基础上做"现在可以怎样成立"的判断，正好完成「现实条件 + 已有基础 → 归类」的输入输出衔接。

### 数据兼容性

零结构改动：question id、answer key、保存 / 恢复、Markdown 导出、加密备份均不受影响；旧存档（v5.8 及更早）可直接加载，无需迁移。

---

## v5.8.1（2026-08-11）— `reality_classification` 现实翻译层定向重构

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.8.html`（同文件内修改）

**定位**：仅修改 `reality_classification` 一题的认知功能与文案，把它的任务从「价值重新排序 / 再次判断重要不重要」改成「现实翻译 / 判断可行性」。不新增问题、不修改数据结构、不改动 `STATE_VERSION`（维持 7）。

### 修改前后对比

| 元素 | 修改前 | 修改后 |
|---|---|---|
| 题目标题 | 回看前面你在意的那些东西——放到现实里，哪些必须保留，哪些可以调整？ | 把前面在意的东西放回现实：现在，它们可以怎样成立？ |
| guide | 不用全部重想。前面已经说过什么是重要的，现在只是把它们放回现实里排个队：真的不能放掉的、希望尽量保留的、还有空间的、暂时说不清的。 | 前面已经说过什么对你重要。现在不必重新排序，试着想想：在你现在的现实条件下，它可以用什么方式成立？有些需要现在就满足，有些可以换一种方式实现，有些可以暂时让步，还有些需要更多信息才能判断。 |
| 分类 1 | 必须满足 | 现在就需要满足 |
| 分类 2 | 希望保留 | 可以换一种方式实现 |
| 分类 3 | 可以调整 | 可以暂时让步 |
| 分类 4 | 不确定 | 还需要更多信息 |

### 联动微调（4 处，均属显示文案一致性同步，不改结构）

1. `renderQuestionInput` 中分类按钮文案同步为新的四个标签。
2. `generateStepFeedback`（reality_bridge）中阶段完成反馈文案从「必须保留 / 还有调整空间」同步为「现在就需要满足 / 可以暂时让步」。
3. `generateInsights` 规则 #8（现实感到行动衔接）文案同步为新的分类标签。
4. `buildCompass`「我的现实地图」与 Markdown 导出中的分类显示标签同步为新标签。

### 刻意保留 / 未改动

- `reality_bridge` 阶段 intro、title、`reality_context`、`adaptability`、`exploration_design` 全部不动。
- `reality_classification` 的数据结构仍为 `[{ label, priority }]`；`priority` 取值 `must / prefer / flexible / unknown` 不变，仅 UI 显示标签重映射。
- `extractRealityClassifyItems` 仍自动复用前序阶段（`core_values`、`work_constraints`、`anti_future`）已产生的条目，不在本题再次询问价值清单。
- `STATE_VERSION = 7`、`STORAGE_KEY`、`保存 / 恢复 / 加密备份 / Markdown 导出结构` 均不变。

### 仍存在的显式不一致

- `reality_bridge` 阶段 intro 仍保留「哪些必须满足，哪些希望保留，哪些还有调整空间」的旧排序式表述。因用户明确要求不修改 intro，此处未动；如后续需要完全对齐，可单独重写该句。

---

## v5.8（2026-08-11）— 「从生活到工作」定向文案重构：Q1 / Q2 / Q3 递进重建

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.7.html` → `最终公开版inner_compass_v5_optimized_v5.8.html`（v5.7 原文件未作改动）

**定位**：不改结构、不改数据模型、不新增问题、不动 Compass 报告与 Insight 规则。仅重写 `lifestyle_to_work` 三个问题的标题 / guide / scaffold / placeholder / 1 处字段 label + 同步阶段 intro，解决两个已知问题：Q1 与 Q2 的「条件清单」重复、Q1 与 Q3 的语义边界不清。

### 逻辑链（本次重建的目标）

```
理想生活（前序阶段）
  ↓ Q1 work_environment  工作以什么「形状」进入我的生活（节奏 / 地点 / 协作）
  ↓ Q2 work_constraints  其中哪些条件不能轻易牺牲（从 Q1 里筛选，而非重新列举）
  ↓ Q3 activity_types    我希望把工作时间具体花在什么动作上 → 产出什么 → 为谁创造价值
```

### 具体改动

| 题 | 字段 | 修改前 | 修改后 |
|---|---|---|---|
| Q1 | label | 回看那个星期二的白天：什么样的**工作环境**，更容易让**那样的**生活成立？ | 回看那个星期二：什么样的**工作方式**，更容易让**这样的**生活成立？ |
| Q1 | guide | 从节奏、空间和协作方式想，不用急着想到职位名。 | 从节奏、地点和与人配合的方式想一想。先不用考虑具体职业。 |
| Q1 | field `rhythm` label | 工作节奏 | 工作节奏（不变） |
| Q1 | field `rhythm` placeholder | 例如：需要整块专注时间，还是规律分段… | 例如：需要整块专注时间、一天里有明显的忙闲节奏、时间比较灵活、规律上下班…… |
| Q1 | field `space` label | 物理与数字空间 | **工作地点** |
| Q1 | field `space` placeholder | 例如：安静的独立空间，还是有人的氛围… | 例如：安静独立、有人一起、远程为主、需要到现场…… |
| Q1 | field `collaboration` placeholder | 例如：各自推进后定期同步，或需要较频繁地一起讨论… | 例如：独立推进后定期同步、小团队一起做、需要频繁交流…… |
| Q1 | scaffold | 1 条（仅节奏） | 3 条（节奏 / 地点 / 协作各一条，与字段一一对应） |
| Q2 | guide | 这些不一定是绝对不能变的规则，但会影响你能不能长期过得舒服。 | 刚才写的是"什么样比较适合"。现在再想想，其中哪些如果长期缺失，你会觉得工作正在侵占你想要的生活。 |
| Q2 | scaffold | 无 | 新增 1 条承接式 prompt：「回看刚写下的节奏、地点和协作方式——哪一条你其实最不愿让步？」（选项与 max 均不变） |
| Q3 | label | 基于前面的线索，你想经常做哪些事？ | 基于前面的线索，你希望自己**经常投入时间**做什么？ |
| Q3 | guide | 用动词描述你在做什么，而不是职位名。也可以写你自己的动词组合，不用从示例里挑。 | 用动词描述你希望反复做的事，而不是职位名称。可以回想那个星期二的白天：哪些事情是你希望**工作本身就包含的**？ |
| Q3 | scaffold | 1 条 | 2 条（保留原动作 prompt + 新增「工作本身就包含」边界 prompt） |
| Q3 | fields | — | **三个字段 label 与 placeholder 完全保留**（做饭、照顾植物、手工、一顿饭、一个空间、自己等「世界切片」示例原样保留） |
| 阶段 intro | — | 先不急着找职业名称。回到你想要的日常：什么样的工作节奏、空间、协作方式和日常活动，更容易支持那样的生活？ | 先不急着找职业名称。回到你想要的日常：工作以什么样的方式进入你的生活，哪些条件不想轻易让步，你又希望把时间经常花在什么样的活动上？ |

**联动微调（1 处）**：`generateStepFeedback` 中 `lifestyle_to_work` 阶段反馈文案「工作环境与日常活动」→「工作方式与日常活动」（Q1 已不再使用「工作环境」一词，属口径一致性同步）。

### 刻意保留 / 未改动

- Q2 `work_constraints` 全部 11 个选项与 `max: 5`（用户要求尽量保留现有稳定选项）
- Q3 三个字段的 label 与「世界切片」示例（v5.7 的成果，本次不动）
- `generateInsights()` 全部 8 条规则、`buildCompass` 报告结构与标签（含「匹配的工作环境」「空间：」等展示名，遵守「不改 Compass 报告」边界）
- `extractRealityClassifyItems`（reality_classification 仍从 `work_constraints` 提取条目，逻辑未动）
- `STORAGE_KEY` / `STATE_VERSION = 7`：纯文案改动，无数据迁移

### 数据兼容性

零结构改动：question id、answer key、保存 / 恢复、Markdown 导出（结构化字段仍按 raw key 导出）、加密备份均不受影响。

---

## v5.7（2026-08-11）— lifestyle_to_work 示例「世界切片」化

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.6.html` → `最终公开版inner_compass_v5_optimized_v5.7.html`（v5.6 原文件未作改动）

**定位**：不改结构、不改数据模型、不新增问题。仅重写「从生活到工作」阶段 `activity_types` 三个输入框的 placeholder 示例，把偏向互联网 / 白领工作的「同类词罗列」改为覆盖更广的「世界切片」。

### 改动（仅 placeholder + 1 处 label）

| 字段 | 修改前（偏白领 / 互联网） | 修改后（世界切片，约 8 个采样词） |
|---|---|
| `primary_activity`（你想经常做的事） | 分析研究、创意表达、系统构建、人际连接、教学传授、策略规划… | 分析研究、写东西、做饭、拍摄、动手制作、照顾植物、教别人、解决现实问题…… |
| `output_form`（产出形式） | 文字内容、代码产品、视觉作品、解决方案、课程内容、服务体验… | 文章、视频、软件、视觉作品、一顿饭、一件手工作品、一个空间、一场活动…… |
| `impact_scope`（label + 示例） | 你希望这些事主要服务谁 / 个人客户、小团队、组织内部、行业领域、大众群体… | 你希望这些事主要为谁创造价值 / 自己、家人、朋友、孩子、客户、学生、观众、小圈子…… |

**原则**：每个输入框仍保持「1 行 placeholder + 约 8 个采样词」；采样词刻意跨越不同生活 / 职业领域（脑力、书写、烹饪、影像、手工、自然、教学、现实问题；内容、作品、餐食、空间、活动；自我、亲密关系、社群、受众），避免把用户默认框进白领叙事。

---

## v5.6.2（2026-08-11）— 支点补完：对照设计文档深度审计后的三处缺口修复

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.6.html`（同文件内修改）

**定位**：v5.6.1 已恢复三个核心认知价值，本次对照 ArenaAI 设计过程文档（模块六「外部声音辨认」/ 模块七「横向比较与焦虑识别」/ 模块八「经历存量与能力资产」）逐条审计，补齐三处承载深度不足的缺口。零新增问题、零新增模块、零新增字段，STATE_VERSION 维持 7。

### 审计结论与改动

| 支点 | v5.6.1 状态 | 缺口 | 本次改动 |
|---|---|---|---|
| 外部声音辨认 | 承载充分（muted_version + Insight #5） | 无 | 不改动 |
| 横向比较与评价坐标 | 有「后视镜」隐喻，缺「用谁的坐标评价自己」的反思入口 | 模块七核心问法「起点是否一致/同一赛道」未落地 | muted_version scaffold 新增 prompt：「当你觉得自己『落后』或『不够好』时，你在拿谁的坐标衡量自己？你们在同一条赛道、同一个起点上吗？」；Insight #6 关键词扩充（坐标、跟不上） |
| 经历存量 | 仅 reality_context 一句 guide + Insight #7，是三个支点中最弱的 | 模块八核心动作「把没成的尝试转化为资源」无任何提问承载；reality_context 是唯一没有 scaffold 的开放题 | reality_bridge 阶段 intro 纳入「过去尝试（哪怕没走成的）留下的认知、能力和判断也是现在位置的一部分」；reality_context 新增 scaffold（2 条 prompt：没走成的尝试留下了什么 / 这些存量如何影响现在的收入、地点、时间安排） |

**原则**：所有承载仍在文案与规则层；scaffold 默认折叠，认知负荷不变。

---

## v5.6.1（2026-08-10）— 设计回溯：恢复原始文字稿的三个核心认知价值

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.6.html`（同文件内修改）

**定位**：不增加模块、不扩大问卷长度。在已有问题中恢复原始文字稿最有力量的三个认知转折点。

### 三个核心认知价值恢复

| 价值 | v5.6 状态 | v5.6.1 承载位置 |
|---|---|---|
| 外部声音辨认 | 存在但被稀释（仅 muted_version 单题） | **boundary 阶段**：muted_version guide/scaffold 增强，明确引导区分"家人的期待/同龄比较/社会叙事"；stage intro 更新 |
| 横向比较与焦虑识别 | 几乎消失 | **boundary 阶段**：anti_future scaffold + muted_version scaffold 新增比较维度引导；**Insight 新增规则 #6**：检测 muted/anti 中的比较语言，给出"赛车无后视镜"隐喻 |
| 经历存量 | 已删除（v5.5 移除 available_assets） | **Reality Bridge**：reality_context guide 增强（"从过去的尝试中带过来的判断力"）；**Insight 新增规则 #7**：当现实描述详实且有价值排序时，将现实认知本身重构为"积累" |

### 具体改动

1. **boundary 阶段 intro**：新增"同龄比较不知不觉写进去的"
2. **anti_future scaffold**：新增第 3 条 prompt——"有没有一部分是你见过别人那样活，它成了一种参照？"
3. **muted_version**：label/guide/placeholder/scaffold 全面增强，引导区分不同声音来源（家人/同龄/社会叙事），新增比较维度
4. **reality_context guide**：新增"从过去的尝试、失败和选择中带过来的判断力"
5. **generateInsights()**：
   - 增强规则 #5（外部声音）：扩大触发关键词范围
   - 新增规则 #6（横向比较）：检测比较语言，给出"赛车无后视镜"锚点
   - 新增规则 #7（经历存量）：当现实描述详实+价值清晰时，将经历重新编码为资产
   - 原规则 #6 改为 #8

**原则**：零新增问题、零新增模块、零新增字段。所有承载都在文案和规则层。

---

## v5.6（2026-08-10）— 收尾优化：Reality Bridge 重构 + Insight Layer + 减量

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.5.html` → `最终公开版inner_compass_v5_optimized_v5.6.html`（v5.5 原文件未作改动）

**定位**：v5.5 结构收尾。不增加阶段、不增加问题、不推翻主线。解决三个剩余问题：Reality Bridge 成为真正的判断桥梁、Compass Report 增加跨阶段洞察、Scaffold 减量回到辅助角色。

### 一、Reality Bridge 重构：从「再次填写」到「归类判断」

**修改前**：`available_assets` (textarea) 让用户自由填写已有资源，与前面探索无承接关系。

**修改后**：`available_assets` → `reality_classification`（新 type: `reality_classify`）：
- 自动从 `core_values`、`work_constraints`、`anti_future` 提取用户前面写过的关键条目
- 每个条目标注来源阶段
- 用户对每条归类：**必须满足 / 希望保留 / 可以调整 / 不确定**
- 全部归类完成后显示确认提示

数据结构：
```javascript
[{ label: "自主权", priority: "must" }, { label: "固定收入底线", priority: "flexible" }]
```

Compass Report「我的现实地图」同步展示分类结果（按四个优先级分组显示）。

### 二、Compass Report 新增 Insight Layer

在 Report 顶部新增 **「几个可能值得注意的发现」**卡片。透明 JS 规则生成（不调用 AI），至少生成 3 条：

| # | 规则 | 跨阶段来源 |
|---|------|-----------|
| 1 | 时间自主权模式 | 在意与取舍 + 理想日常 + 工作条件 |
| 2 | 稳定 vs 探索冲突 | 在意与取舍 + 现实地图 + 实验设计 |
| 3 | 边界意识强烈 | 边界 + 工作条件 + 更像我自己的声音 |
| 4 | 协作距离偏好 | 在意与取舍 + 工作条件 |
| 5 | 自我声音分辨 | 更像我自己的声音 + 在意与取舍 |
| 6 | 现实到行动衔接 | 现实地图 + 实验设计 |

每条标注来源阶段，非空条件下生成；不足 1 条时卡片隐藏。

### 三、Scaffold 减量（~40%）

| 问题 | 修改前 | 修改后 | 减少 |
|------|--------|--------|------|
| life_vision | 3 prompts + 1 starter | 2 prompts + 1 starter | -1 |
| ideal_day | 4 prompts | 2 prompts | -2 |
| trade_off | 3 prompts + 2 examples | 1 prompt + 1 example | -3 |
| desired_identity | 2 prompts | 1 prompt (合并) | -1 |
| muted_version | 3 prompts + 1 starter | 2 prompts (去 starter) | -2 |
| work_environment | 3 prompts | 1 prompt | -2 |
| experiment_goal | 1 prompt + 1 starter | 1 prompt (去 starter) | -1 |
| experiment_action | 2 prompts | 1 prompt | -1 |
| success_criteria | 3 prompts | 1 prompt | -2 |

**总计**: ~30 → ~15 prompts，减少约 50%。每个开放题保留 1 句 guide + 1-2 个提示入口。

### 四、中文文案审查

- 删除「觉察」→「分辨」（insight 5）
- Reality Bridge feedback 删除「现实感不是给方向泼冷水」「现实不是阻碍，而是地图」
- Report「我的现实地图」删除结尾解释句
- `trade_off` 示例合并为 1 条

### 五、数据兼容性

- `STATE_VERSION` 升至 **7**
- `reality_classification` 为增量字段（新 type）
- 旧 `available_assets` 答案保留在 answers 中但不再被 Report 引用
- Markdown 导出增加 `reality_classification` 特殊处理（`[object Object]` → `label：优先级`）
- 旧存档可通过 IMPORT_WHITELIST 正常加载

### 六、当前仍存在的三个问题

1. **第一题（life_vision）仍是高认知负荷入口**：虽然 scaffold 已减，但「你想过什么样的生活」作为第一个问题对部分用户仍然太重。未来可考虑提供「先跳出词汇」的轻量热身。
2. **Insight 规则不够丰富**：当前 6 条规则基于跨阶段模式匹配，覆盖面有限。规则增加需要更多用户数据观察。
3. **muted_version 定位漂移**：v5.5 将 external_voices 合并到 muted_version，意图是「分辨回应期待 vs 真心想要」，但实践中容易滑向心理咨询感。本次已降低 scaffold 强度，但长期看可能需要重新设计这一阶段的认知锚点。

---

## v5.5（2026-08-09）— Journey 骨架重构：从「问题集合」回到「认知链」

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.4.html` → `最终公开版inner_compass_v5_optimized_v5.5.html`（v5.4 原文件未作改动）

**定位**：停止文案层优化，基于 Question Dependency Map 审计做结构性收敛。核心目标不是「减少题量」，而是让每个阶段有明确的认知任务、且前一阶段的产出自然成为后一阶段的输入。

### 结构变化：10 阶段 / 24 题 → 7 阶段 / 17 题

| 新阶段 | 认知任务 | 问题（保留 id / 新增 id） |
| --- | --- | --- |
| ① 人生图景（vision，新） | 真正的入口：「你想过什么样的生活」 | `life_vision`（新，textarea） |
| ② 把日子想具体（ideal_day） | 把图景落到一个普通星期二 | `ideal_day`（改写承接）、`end_feeling` |
| ③ 在意与取舍（values） | 价值从图景里长出来，而非从清单里挑 | `core_values`（改写承接）、`trade_off` |
| ④ 边界与自己的声音（boundary，合并原 identity_boundary + voices_compare） | 不想越过的线 + 区分「真心想要」与「回应期待」 | `anti_future`、`desired_identity`（改写承接）、`muted_version`（强化引用） |
| ⑤ 从生活到工作（lifestyle_to_work，合并原 work_characteristics） | 把理想生活翻译成工作形状 | `work_environment`（改写承接）、`work_constraints`（选项并入 preserve_conditions）、`activity_types` |
| ⑥ 把方向放回现实（reality_bridge） | 看见现在的位置、资源与弹性 | `reality_context`、`available_assets`、`adaptability` |
| ⑦ 猜想与尝试（exploration_design） | 选一个猜想，设计小实验验证 | `experiment_goal`（改写承接）、`experiment_action`、`success_criteria` |

### 删除的问题（8 题）与理由

| 问题 id | 审计标记 | 理由 |
| --- | --- | --- |
| `why_now` | WEAK_TRANSITION + LOW_VALUE | 与认知链无输入/输出关系，对 Compass 报告零贡献，且抢占了核心入口位置 |
| `set_aside` | LOW_VALUE | 「状态管理」而非「认知输入」，对报告零贡献 |
| `alive_moment` | 降为 scaffold | 有过去体验锚点价值，但不应占独立问题位；转为 ideal_day 的思考提示 |
| `alive_tags` | REDUNDANT + LOW_VALUE | 依附已降级的问题；元素语义已被 core_values 选项池覆盖 |
| `external_voices` | 定位错误 | 问的是「谁在影响你」（识别声音源），用户真正需要的是「哪些答案在回应期待」——后者由 muted_version 承担 |
| `comparison_split` | LOW_VALUE | 「比较管理」是行为建议而非认知输入，与认知链无关 |
| `assets_bridge` | REDUNDANT | 与 available_assets 重复（scaffold 几乎逐字相同） |
| `preserve_conditions` | REDUNDANT | 与 work_constraints 重复（都在问「不可让步的条件」），选项并入后者 |

### 改写的问题（承接式，不再「课程章节」式）

- `life_vision`（新）：直接问「你想过什么样的生活」，替代原 arrival「为什么今天打开」作为入口。
- `ideal_day`：label 从「想象一个普通工作日」改为「如果那样的生活真的发生了，一个普通的星期二会是什么样？」。
- `core_values`：label 从「从下面选出你最看重的」改为「回看刚才的图景：哪些东西如果被拿掉，就不再是你想要的生活？」。
- `desired_identity`：从「别人怎么形容你」改为「回看生活图景，有没有一种你希望自己成为的人已经隐约出现」。
- `muted_version`：引用关系显式化——「回看前面写下的图景、价值排序和边界」。
- `work_environment` / `experiment_goal`：改为直接引用前文产出（星期二的白天 / 前面写下的方向）。

### 联动改动

- `generateStepFeedback`：阶段 id 全部对齐新结构（vision / ideal_day / values / boundary / lifestyle_to_work / reality_bridge / exploration_design）。
- `buildCompass`：报告坐标调整为 9 项——我想靠近什么生活 / 我在意什么 / 不想活成什么样 / 想成为怎样的人 / 更像我自己的声音（原「哪些声音会影响我」）/ 匹配的工作环境 / 想从事的工作活动 / 我的现实地图 / 设计的验证实验。
- 进度条、章节导航、总览弹窗、Markdown 导出均为数据驱动，自动生效。
- 页脚、meta、head 注释同步更新为 v5.5。

### 数据兼容性

- `STATE_VERSION` 升至 **6**（`STORAGE_KEY` 不变）：旧存档（v5）经白名单迁移自动兼容；已删除问题的旧答案保留在 answers 中但不被读取，无副作用。
- 旧存档 `stepIndex` 越界（10 阶段 → 7 阶段）时钳制到最后一个阶段，不再直接跳 summary；deferred 中残留的已删除问题 id 在渲染时过滤。

---

## v5.4（2026-08-07）— 新增「把方向放回现实」（Reality Bridge）阶段

**对照文件**：`最终公开版inner_compass_v5_optimized_v5.3.html` → `最终公开版inner_compass_v5_optimized_v5.4.html`（v5.3 原文件未作改动）

### 新增阶段

在「想做怎样的事」（work_characteristics）之后、「下一步尝试」（exploration_design）之前，插入新阶段 `reality_bridge`：

- **阶段名**：把方向放回现实（备选名「现实地图」「看见我的现实条件」未启用）
- **定位**：不是让用户列出人生底线，也不是强化现实限制；而是帮助完成「发现想靠近的方向 → 重新看见当前现实位置 → 基于真实条件设计下一步探索」。全程避免「底线 / 限制 / 不可牺牲」类措辞。

### 新增问题（4 题）

| id | 类型 | 说明 |
| --- | --- | --- |
| `reality_context` | structured（4 字段） | 现实位置扫描：收入与经济 / 地点与环境 / 可投入时间 / 其他责任 |
| `available_assets` | textarea + 思考支架 | 现实资源扫描（3 条非诱导提示） |
| `adaptability` | multitag（7 选项，max 5） | 愿意尝试调整的空间 |
| `preserve_conditions` | multitag（7 选项，max 5） | 希望尽量保留的条件（替代「底线」表述） |

### 联动改动

- `generateStepFeedback`：新增 `reality_bridge` 阶段完成反馈（强调「现实感是帮方向找到落点，而非泼冷水」）。
- `buildCompass`：Compass 报告新增「我的现实地图」坐标，位于「想从事的工作活动」与「设计的验证实验」之间；输出结构为「你希望保留的条件 / 你拥有的资源 / 可以尝试调整的空间 / 现实位置四字段」，并以「这些信息可以帮助你设计更符合当前阶段的小实验」收尾。不生成「你的职业限制是……」「你必须选择……」类表述。
- 阶段总数 9 → 10，问题总数 20 → 24；进度条、章节导航、总览弹窗、Markdown 导出均为数据驱动，自动生效，无硬编码改动。

### 数据兼容性

- `STATE_VERSION` 维持 **5**，`STORAGE_KEY` 不变（`inner_compass_v5_3`）：新答案为纯增量键值，旧存档合并后即可直接使用，无需迁移。
- 旧存档中 `stepIndex = 8`（原「下一步尝试」）的用户，升级后将落在新的「把方向放回现实」阶段，已填写的「下一步尝试」答案不丢失，可在总结页或章节导航中回看。

---

## v5.3 / v5.3.1（2026-08-06）— 文案迭代

详见 `v5.3_文案迭代说明.md`：在不改变题目结构、数据模型与交互的前提下，优化引导、示例、阶段反馈与结果页文案，降低强判断与对抗感。

## v5.2（2026-08-06）— 认知负荷优化

详见 `认知负荷审计与思考支架设计_v5.1.md`：为全部开放题增加三层思考支架（持续可见引导 / 点击展开提示 / 非诱导式示例），修复 `muted_version` 指代歧义，替换 4 处过度暗示的示例。问题结构、价值体系、数据模型零改动。
