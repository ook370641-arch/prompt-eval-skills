# Prompt & Evaluation Skills for Claude Code

一套面向**提示词工程 + 评测驱动迭代**的 Claude Code Skill 集合，覆盖从提示词创建/诊断、评测迭代全链路，到报告审查的完整闭环。

核心理念：**提示词是任务设计的「结果」，不是任务设计本身；一个新方案「证明可行」不等于「证明值得采用」。** 这套 skill 把「怎么把提示词迭代到更高一致率、并用证据链证明它值得上线」这件事，固化成可复用的方法论与流程纪律。

---

## Skill 一览

| Skill | 一句话 | 触发时机 |
|---|---|---|
| **auto-eval** | 评测驱动的提示词迭代**全链路引擎**（读背景→数据分析→改提示词→跑测→报告，循环） | 要基于评测/badcase 迭代提示词、对齐人评、提高一致率，且需要**跑得起来的完整闭环** |
| **prompt-creation** | 提示词**创建**：先做任务建模，再写提示词 | 新任务刚启动、目标还停在业务描述层、要决定单提示词还是多阶段 |
| **prompt-diagnosis** | 提示词**诊断**：ABC 三层 + 分桶诊断 + 回归分析定位病因 | 改一条规则输出全漂移、同类 badcase 反复波动、改了没效果 |
| **report-review** | 报告**审查 / 撰写**：证据链 + 双轴质量把关 | A/B 测评、prompt/model 评测、质检、badcase 分析、方案准出报告 |

四个 skill 可独立使用，也彼此咬合：**auto-eval** 编排迭代循环，循环里按环节调用 **prompt-creation**（写）、**prompt-diagnosis**（诊断）、**report-review**（审）。

---

### 1. auto-eval — 评测驱动的提示词迭代全链路引擎

把提示词迭代的整条链路——读背景 → 数据分析 → 生成提示词 → 跑测 → 报告，且反复循环——固化为一套可复用的自动化编排。

**核心设计：skill 通用，项目细节落文档。** 本 skill 只持有**通用流程骨架 + 方法论铁律**，不持有任何具体项目的工作流。具体怎么跑测、数据长什么样、读哪个字段，全写在项目自己的 `CLAUDE.md`（spec：为什么/做什么/红线）和 `README.md`（plan：怎么做）里，启动时去读；没有就依据当前文件夹上下文创建。

**主循环 Part 0→1→2→3→4 按需循环**，每个 Part 都有落盘验收物：

- **Part 0** 展开/改写项目文档（CLAUDE spec + README plan）
- **Part 1** 数据分析（整体→具体的带符号混淆矩阵 + 读模型 COT 归因 + 返回率预判），接一步提示词诊断
- **Part 2** 提示词迭代（Reflector 定效果 → Refiner 落措辞 + **两道闸门**：写前人工挡方向、写后子 agent 挡覆盖）
- **Part 3** 跑测（对接项目脚本，脚手架方法论见 `references/test-harness.md`）
- **Part 4** 评测与归因（三分类迁移 xlsx + 修正率/返回率 + **回归分析抓「沉默丢失」** + report-review 自审）

**防回归核心铁律**：数据分析结论禁止原文焊进提示词（先抽成与措辞无关的效果，再落成可复用原则，永不嵌具体 case）；改提示词默认 EDIT-局部、禁 REWRITE-整段；晋升 baseline 前强制回归分析闸门（有价值旧规定被无意丢失即不晋升）。

**References：**
- `grill-checklist.md` — 启动时盘问的完整 branch 清单
- `data-analysis.md` — 数据层（现象）：混淆矩阵、COT 归因、返回率预判、逐 case 迁移四类
- `diagnosis-method.md` — 提示词层（病因）：ABC 三层 + 入参咬合 + 简洁性 + 分桶诊断
- `prompt-modification.md` — 精修框架：加/删/改决策表、CASE_PATCH 拒收、GT 错前置筛
- `doc-templates.md` — CLAUDE / README 两份文档的结构模板
- `test-harness.md` — 跑测脚手架方法论（三段式、temperature=0、**评分尺子分级+带符号**、**模型输出鲁棒解析**、看板设计、脚手架自检）

---

### 2. prompt-creation — 提示词创建

在写提示词之前，先把任务定义、上下游接口、失败容忍度、方案架构定清楚。通过 9 步流程（信息搜集 → ABC 层理解 → 分轮提问 → 方案决策 → 生成 → 检验 → 精修 → 交付），交付一套能支撑后续迭代和评测的提示词信息栈 + 提示词文件。

**适用**：新任务刚启动目标还停在业务描述层；旧提示词改了很多轮仍没稳定方向；要决定单提示词还是多阶段方案。

---

### 3. prompt-diagnosis — 提示词诊断

诊断现有提示词的结构性问题，分析 badcase，对齐调整策略，产出新提示词。基于 **ABC 三层架构**（任务定义层 / 任务拆分层 / 规则下沉层，C 层含**入参咬合**检验）做系统性诊断，避免「改措辞」式盲目迭代。能独立跑完「数据分析 → 分桶诊断 → 回归分析」全流程。

**适用**：改一条规则输出其他部分一起漂移；同类 badcase 多轮下依然随机波动；不确定该继续调提示词还是改规则、拆阶段。

**References：** `data-analysis.md`（数据层现象）、`diagnosis-method.md`（提示词层病因）。与 auto-eval 各持一套同名 reference，独立演化、不强制同步。

---

### 4. report-review — 报告审查 / 撰写

既能审查已写完的报告找缺口，也能在测评/迭代链路跑完后把成果整理成报告。你是报告质量**把关者**，不是文风润色者。

**两条主轴**：
1. **直观可信度**：主判断、关键证据、对照、边界是否集中可定位。
2. **剖析深度**：现象是否推进到原因、机制、边界、策略，结论强度是否匹配证据。

专门识别结构性问题：主问题不清、样本口径混乱、判定标准缺失、评测工具未验证、A/B 对照缺失、badcase 只陈列未归因、**证明可行但没证明值得**、总指标掩盖边界、结论强度越级、决策未冻结等。

**References（按报告类型选读）：** 方案准出 / 评测器可靠性 / 失败机制 / 对比对照 / 安全关键系统。

---

## 安装

```bash
# 全局安装（推荐）
cp -r auto-eval prompt-creation prompt-diagnosis report-review ~/.claude/skills/

# 项目级安装
cp -r auto-eval prompt-creation prompt-diagnosis report-review ./.claude/skills/
```

安装后在 Claude Code 中说出触发词即可调用。

---

## 文件结构

```
.
├── auto-eval/
│   ├── SKILL.md
│   └── references/
│       ├── grill-checklist.md
│       ├── data-analysis.md
│       ├── diagnosis-method.md
│       ├── prompt-modification.md
│       ├── doc-templates.md
│       └── test-harness.md
├── prompt-creation/
│   └── SKILL.md
├── prompt-diagnosis/
│   ├── SKILL.md
│   └── references/
│       ├── data-analysis.md
│       └── diagnosis-method.md
├── report-review/
│   ├── SKILL.md
│   └── references/
│       ├── comparative-quality-report.md
│       ├── evaluator-reliability-report.md
│       ├── failure-mechanism-report.md
│       ├── safety-critical-system-report.md
│       └── solution-decision-report.md
└── README.md
```

---

## 说明

- 全部为 Claude Code 专属格式（YAML frontmatter + Markdown），不兼容其他 AI 助手框架。
- 示例中的 `任务二`、`V{N}`、`运行与评测/` 等均为占位/示意，使用时替换为你自己项目的实际值。
- 这套 skill **不含任何 api key、内网 host、私有数据或本机绝对路径**——跑测脚手架的具体接入（api/密钥/脚本路径/评测数据）由你在项目自己的 README/CLAUDE 里落地，方法论骨架见 `auto-eval/references/test-harness.md`。
