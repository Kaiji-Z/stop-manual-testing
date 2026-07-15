# agent-verifier-protocol

> **A single Markdown protocol file that an AI coding agent reads once, then auto-builds a machine-checkable verification system in your project — collapsing manual testing time from ~90% toward zero.**
>
> **[中文](#中文版) · English**

---

# English

> This is not a tool and has no runtime dependency. It is just a `VERIFICATION.md` file — drop it in your project root, open a new agent session, and say *"read VERIFICATION.md."* The agent then self-diagnoses your project, surfaces gaps, wires up regression tests, and converges inside a closed loop.

## The pain it solves

If you develop AI agents, you are likely stuck here:

- **~90% of your time goes to manual testing** — opening the web page, typing inputs, watching the agent step by step, judging with your own brain whether it got better or worse this run.
- **One prompt tweak breaks it**, and you cannot even tell at a glance whether it really broke — so you run it again, and again, and again.
- **Your job is no longer writing code**; it is repeatedly verifying a system even you no longer fully understand.

The root cause is a single asymmetry: **generation is cheap, verification is expensive.** An AI generates code in seconds, but if "is this actually correct?" can only be answered by a human staring at the screen, you are permanently trapped in verification.

What this protocol does is turn "by human eye, by gut feel" verification into **an automated, repeatable, machine-checkable** system. Once "what counts as correct" becomes machine-decidable, the agent can run, fix, and converge inside a closed loop on its own — you no longer watch the screen.

## Get started in 3 minutes

```bash
# 1. Put VERIFICATION.md in your project root
cp VERIFICATION.md /path/to/your-project/

# 2. Open a new coding agent session (Claude Code / Codex / ZCode, etc.)

# 3. Tell the agent one thing:
"read VERIFICATION.md"
```

That's it — one sentence. The agent runs a 7-step diagnosis pipeline on its own after reading, because the protocol file declares on its very first line: **opening this file = trigger to execute.**

What it does:
1. Reads your project context (AGENTS.md, build config, entry code, test entry)
2. Runs an **ACI audit**: can your system run without the UI? Are intermediate states logged? Are interfaces programmatic?
3. Inventories the existing test infrastructure
4. Outputs a **priority-sorted gap list** (P0/P1/P2, with remediation plans)
5. **Auto-fills what it can** (start commands, test commands); **asks you what it must** (acceptance criteria, supervisor design)
6. Updates AGENTS.md, persisting the diagnosis
7. Stops and reports; waits for your go-ahead before touching code

After the first round, every subsequent session inherits the updated AGENTS.md — no starting from scratch.

## Prerequisite

**One hard condition. If it fails, the protocol diagnoses it first, then remediates:**

Your AI agent system must be **AI-friendly (Agent Computer Interface):**
- Backend starts independently of the frontend
- Triggering a workflow has a CLI/API, not requiring browser clicks
- Intermediate states are structured and programmatically retrievable

If your system can currently only be tested via a web UI, the agent will honestly surface this gap as P0 — because a verification system needs a system that can run without a human. This is architecture work, not something a spec file can auto-solve — but the *diagnosis* is reliable.

## Why it actually drives the agent (core mechanisms)

### 1. Self-boot protocol
The first line declares "opening = trigger." The user only says "read it," with no long prompt. The execution instructions are welded into the file, not re-typed every session.

### 2. GATE mechanism (Declare-Verify-Enforce)
LLMs naturally skip steps in multi-step pipelines; "please follow strictly" cannot stop it. This protocol requires the agent to **emit a fixed-format GATE declaration after every step**:

```
GATE [step N]: DONE
- Did: [concrete action + artifact location]
- Evidence: [file:line / command output / developer answer]
- Next: [step N+1]
```

No evidence = not done; declaration contradicts the artifact = redo. This pulls the agent's compliance state out of opaque reasoning and makes it **visible, checkable, blocking-on-mismatch.**

### 3. Two-layer judge
- **Layer 1, deterministic assertions**: was the tool triggered? Are the record counts right? — absolutely reliable, zero cost. Never use an LLM where this layer can catch it.
- **Layer 2, LLM judge (supervisor)**: handles fuzzy parts, **scores only, never judges right/wrong**, and its context must be clean (it must not know how the code is written, or it scores its own people high).

### 4. Flag-based A/B
Every new feature gets a flag. Run the *same* regression suite with flag=on and flag=off; compare "what got better / what got silently broken" — the only way to do controlled experiments in a non-deterministic system.

### 5. Orchestrate mature tools, don't reinvent
The protocol detects your project's existing eval tools (DeepEval / LangSmith / pytest / jest) and lands §3/§4 directly on their APIs. If none exist, it recommends but does **not** self-install — introducing a dependency is a decision owned by the developer.

## Who should use it

| Your situation | What to do |
|---|---|
| I build AI agents and manual testing is torturing me | Drop it in the project, say "read VERIFICATION.md," let the agent diagnose |
| I want to understand why it is designed this way | See "Design rationale" and references below |
| I want to contribute or adapt the protocol | Fork and edit `VERIFICATION.md`; the execution layer is English |

## Design rationale (for the curious)

- **Verifier's Law** — Jason Wei (OpenAI), 2025: some tasks are far easier to verify than to solve, and AI progresses fastest where outputs are easy to check. So value shifts to: *can you turn a hard-to-verify problem into an easy-to-verify one?* Once yes, hand the rest to AI.
- **30 years of chip-industry precedent** — a failed tape-out costs millions, so verification headcount is 2–3× design headcount. The industry pours effort into "designing verification systems that expose bugs" — assertion / coverage / scoreboard. This protocol's two-layer judge borrows directly from this.
- **Agent-Computer Interface (ACI)** — from the SWE-agent paper (Princeton NLP, NeurIPS 2024): AI agents are a new class of user, and interface design affects agent performance **more than swapping in a stronger model.** This protocol's §2 audit operationalizes this.
- **Why GATE exists** — community evidence shows LLMs drop steps in long pipelines (lost-in-the-middle + instruction overload); "please comply" cannot stop it. Making compliance visible, checkable, and blocking-on-mismatch is the only reliable way to turn "should do" into "actually did."

## What it cannot do (honest limits)

- **It is not a tool; it runs no verification.** It tells the agent *what to build*; how assertions run and how the supervisor is called lands on your project's existing eval tools (DeepEval / LangSmith, etc.).
- **"Full compliance" relies on GATE self-declaration, not enforced execution.** A sufficiently lazy agent could still forge a GATE. The Verify rule catches most forgery (every "Did" needs evidence), but ultimate checking still requires a human or a next-round agent spot-check.
- **It cannot do architecture refactoring.** If your system is currently UI-only, §2.1 fails, and splitting front/back is work that requires reading code. But the agent will honestly diagnose this gap and write it into the AGENTS.md backlog for the next round to inherit.

This is not a flaw — it is first-principle self-consistency: **verification never relies on goodwill; it relies on checkable evidence.** The protocol itself follows this rule: it does not assume the agent will comply willingly; it uses GATE to turn compliance into a checkable artifact.

## Files

| File | Language | Purpose | Reader |
|---|---|---|---|
| `VERIFICATION.md` | English | Pure execution-layer protocol, agent reads and executes | coding agent |
| `README.md` | Bilingual | Pain points, usage, design rationale, honest limits | developer (you) |

The execution layer is English because coding agents follow English long-instruction + red-line constraints at the highest rate, and it is isomorphic with code/APIs (lowest attention loss). The teaching layer keeps Chinese to lower your maintenance cost.

## References

The protocol's design draws on the following sources. Each is annotated with a verifiable original source.

**Core idea sources**

- **Asymmetry of verification / Verifier's Law** — Jason Wei (OpenAI researcher), July 2025 blog post: [Asymmetry of verification and verifier's law](https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law). Core thesis: some tasks are far easier to verify than to solve; AI progresses fastest where outputs are easy to check.
- **Agent-Computer Interface (ACI)** — Yang et al., Princeton NLP, NeurIPS 2024 paper: [SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering](https://arxiv.org/abs/2405.15793) (~1900+ citations). Thesis: interfaces designed specifically for LLM agents (analogous to HCI) can improve agent performance *more than* swapping in a stronger model. Code: [github.com/princeton-nlp/SWE-agent](https://github.com/princeton-nlp/SWE-agent).

**Mechanism design sources**

- **Visible Checklist Pattern (Declare-Verify-Enforce)** — wharsojo: [The Visible Checklist Pattern: Enforcing Multi-Step Pipeline Compliance in LLM Agents](https://dev.to/wharsojo/the-visible-checklist-pattern-enforcing-multi-step-pipeline-compliance-in-llm-agents-j30). The direct source of this protocol's §0 GATE mechanism — solving "LLMs naturally drop steps in multi-step pipelines."
- **Chip-industry verification practice** — assertion / coverage / scoreboard methodology, a 30-year industry standard (verification headcount is 2–3× design headcount). This protocol's §3 two-layer judge borrows from it.

**Empirical evidence sources**

- **AGENTS.md / CLAUDE.md writing experience** — progressive disclosure (short root file, detail sinks down); instruction-overload anti-pattern (long context files actually reduce agent success rates, see [arXiv 2602.11988](https://arxiv.org/html/2602.11988v1)); short-concrete-command beats long prose. These come from community empirical summaries (e.g. [HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [AI Hero](https://www.aihero.dev/a-complete-guide-to-agents-md)) and are the basis for this protocol's "English execution layer + thin + two-layer structure" design.

**Orchestration targets (this protocol orchestrates, does not replace)**

| Tool | Repo | Use |
|---|---|---|
| DeepEval | [confident-ai/deepeval](https://github.com/confident-ai/deepeval) | §3.1 assertions + §3.2 LLM scoring, pytest-native |
| LangSmith | [langchain-ai/langsmith](https://github.com/langchain-ai/langsmith-sdk) | §4 regression sets + trace replay + comparison |
| ASSERT | [responsibleai/ASSERT](https://github.com/responsibleai/ASSERT) | auto-generate assertion cases from requirements |
| harness-evals | [harness/harness-evals](https://github.com/harness/harness-evals) | normalized scoring (0.0–1.0) |
| EvalView | [GitHub Marketplace Action](https://github.com/marketplace/actions/evalview-ai-agent-testing) | record agent behavior in CI, catch regressions |

## License

MIT

---
---

# 中文版

> **[English](#english) · 中文**

> 这不是一个工具,不依赖任何运行时。它只是一份 `VERIFICATION.md`——放进你的项目根目录,开一个新会话对 agent 说"读一下 VERIFICATION.md",agent 就会自动诊断你的项目、改造接口、搭建回归测试、在闭环里自己收敛。

## 它解决什么痛点

如果你在做 AI agent 开发,大概率你正卡在这里:

- **90% 的时间耗在手动测试**:点开网页、输入数据、盯着 agent 一步步跑、用人脑判断这次是变好还是变坏。
- **改一句 prompt 就崩**,但你根本不知道它有没有真坏,只能再跑一遍、再跑一遍。
- **你的工作已经不是写代码**,而是反复验证一个连你自己都快看不懂的系统。

这套痛点有一个根本原因:**生成廉价,验证昂贵。** AI 几秒就能生成代码,但"这段代码到底对不对"如果只能靠人盯,你就永远被困在验证里。

本协议做的事,就是把"靠人眼靠感觉"的验证,改造成**自动的、可重复的、机器可判定的**验证体系。一旦"什么算对"变得机器可判定,agent 就能在闭环里自己跑、自己改、自己收敛,不需要你守屏幕。

## 三分钟看懂怎么用

```bash
# 1. 把 VERIFICATION.md 放进你的项目根目录
cp VERIFICATION.md /path/to/your-project/

# 2. 开一个新的 coding agent 会话(Claude Code / Codex / ZCode 等)

# 3. 对 agent 说一句:
"读一下 VERIFICATION.md"
```

就这一句。agent 读完后会自动执行一套 7 步诊断流程,因为协议文件第一行就声明了:**打开本文件 = 触发执行**。

它会做什么:
1. 读你的项目上下文(AGENTS.md、构建配置、入口代码、测试入口)
2. 做 **ACI 体检**:你的系统能脱离 UI 跑吗?中间状态留痕吗?接口程序化了吗?
3. 摸底现有测试基建
4. 输出一张**按优先级排序的差距清单**(P0/P1/P2,带改造方案)
5. **自动填能填的**(启动命令、测试命令),**问你该问的**(验收标准、supervisor 设计)
6. 更新 AGENTS.md,把诊断结果落盘
7. 停下来报告,等你确认方向后再动手改代码

第一轮结束后,后续每一轮会话都继承已更新的 AGENTS.md,不会从零开始。

## 它要求什么前提

**一个硬条件,过不了它,本协议先帮你诊断出来,再改造:**

你的 AI agent 系统必须是**对 AI 友好的(Agent Computer Interface)**:
- 后端能脱离前端独立启动
- 触发 workflow 有 CLI/API,不依赖浏览器点击
- 中间状态有结构化记录,可程序化取出

如果你的系统现在只能靠点网页测,agent 会把这个差距诚实诊断出来,列为 P0——因为验证体系需要一个能脱离人、独立运行的系统。这一步是架构活,不是读规范能自动解决的,但诊断是可靠的。

## 核心机制(为什么它能真正驱动 agent 行动)

### 1. 自启动协议
文件第一行声明"打开即触发"。用户只需说"读一下",不需要打一长串 prompt。执行指令焊死在文件里,不是每次靠用户叮嘱。

### 2. GATE 机制(Declare-Verify-Enforce)
LLM 在多步流程里天然会漏步,靠"请务必遵守"拦不住。本协议要求 agent **每一步必须输出固定格式的 GATE 声明**:

```
GATE [step N]: DONE
- Did: [具体动作 + 产物位置]
- Evidence: [文件:行号 / 命令输出 / 开发者回答]
- Next: [step N+1]
```

无证据 = 未做;声明与实际不符 = 重做。这是把 agent 的遵守状态从"不透明推理"里拉出来,变成**可见、可核验、不实则可阻断**的东西。

### 3. 两层裁判
- **第一层 确定性断言**:工具有没有触发、记录条数对不对——绝对可靠、零成本,凡能用这层卡的绝不上 LLM。
- **第二层 LLM 裁判(supervisor)**:处理模糊部分,**只打分不判对错**,且上下文必须干净(不能知道代码怎么写,否则给自己人打高分)。

### 4. flag 开关对照
每个新功能配一个 flag。flag 开/关跑同一套回归测试,对比"哪些变好了、哪些被悄悄弄坏"——这是在不确定系统里做对照实验的唯一手段。

### 5. 编排成熟工具,不重复造轮子
协议会检测你项目已有的 eval 工具(DeepEval / LangSmith / pytest / jest),直接指向它的 API 落地断言和回归。没有工具时推荐但不自行安装——引入依赖的决策归你拍板。

## 谁该用什么

| 你的情况 | 该怎么用 |
|---|---|
| 我在做 AI agent 开发,被手动测试折磨 | 放进项目,说"读一下 VERIFICATION.md",让 agent 帮你诊断 |
| 我想理解这套体系为什么这么设计 | 见下方"设计原理"和参考链接 |
| 我想贡献或改造这个协议 | fork 后改 `VERIFICATION.md`;执行层是英文,教学层在本 README |

## 设计原理(给想深入的人)

- **验证不对称性 / Verifier's Law** — Jason Wei(OpenAI),2025:有些任务验证远比求解容易,AI 在"输出易于检查"的任务上进步最快。所以价值转移到——你能不能把"难验证"改造成"易验证"。一旦做到,剩下交给 AI。
- **芯片行业的 30 年先例** — 芯片流片失败损失几百万美元,所以验证人力是设计的 2–3 倍。行业把绝大部分精力压在"设计可暴露 bug 的验证体系"——assertion / coverage / scoreboard。本协议的两层裁判直接借鉴于此。
- **Agent Computer Interface(ACI)** — 来自 SWE-agent 论文(Princeton NLP,NeurIPS 2024):AI agent 是一类全新用户,接口设计对 agent 表现的影响,**甚至大过你换一个更强的模型**。
- **为什么有 GATE 机制** — 社区实证经验表明:LLM 在多步流程中天然漏步(lost in the middle + 指令过载),靠"请务必遵守"拦不住。把遵守状态显式化、可核验、不实则阻断,是把"应该做"变成"实际做了"的唯一可靠机制。

## 它做不到什么(诚实边界)

- **它不是工具,跑不了任何验证。** 它让 agent 知道"该造什么",但断言怎么跑、supervisor 怎么调,接的是你项目已有的 eval 工具(DeepEval / LangSmith 等)。
- **它的"完全遵守"靠 GATE 自声明,不是强制执行。** 一个足够懒的 agent 仍可能伪造 GATE 声明。GATE 的 Verify 规则能拦住大部分伪造(要求每个"做了"必须有证据),但终极核验还是需要人或下一轮 agent 抽查。
- **它推不动架构重构。** 如果你的系统现在只能 UI 测,§2.1 不达标,那拆前后端是看代码才能干的活。但 agent 会把这个差距诚实诊断出来,写进 AGENTS.md 待办,下一轮继承。

这不是缺陷,是第一性原理的自我一致:**验证从不靠自觉,靠可核验的证据。** 本协议自己也遵守这条——它不假设 agent 会自觉遵守它,而是用 GATE 把遵守变成可核验的产物。

## 文件说明

| 文件 | 语言 | 作用 | 读者 |
|---|---|---|---|
| `VERIFICATION.md` | 英文 | 纯执行层协议,agent 读了照着干 | coding agent |
| `README.md` | 双语 | 痛点、用法、设计原理、诚实边界 | 开发者(你) |

执行层用英文,是因为 coding agent 对英文长指令 + 红线约束的遵循率最高,且和代码/API 同构、注意力损耗最低。教学层保留中文,降低你的维护成本。

## 参考

这套协议的设计综合了以下来源。每条都标注了可核验的原始出处:

**核心思想来源**

- **验证不对称性 / Verifier's Law** — Jason Wei(OpenAI 研究员),2025 年 7 月博客原文:[Asymmetry of verification and verifier's law](https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law)。核心论点:有些任务验证远比求解容易,AI 在"输出易于检查"的任务上进步最快。
- **Agent-Computer Interface(ACI)** — Yang et al., Princeton NLP,NeurIPS 2024 论文:[SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering](https://arxiv.org/abs/2405.15793)(被引约 1900+ 次)。论点:为 LLM agent 专门设计的接口(类比 HCI),对 agent 表现的提升甚至超过换更强的模型。代码:[github.com/princeton-nlp/SWE-agent](https://github.com/princeton-nlp/SWE-agent)。

**机制设计来源**

- **Visible Checklist Pattern(Declare-Verify-Enforce)** — wharsojo:[The Visible Checklist Pattern: Enforcing Multi-Step Pipeline Compliance in LLM Agents](https://dev.to/wharsojo/the-visible-checklist-pattern-enforcing-multi-step-pipeline-compliance-in-llm-agents-j30)。本协议 §0 GATE 机制的直接来源——解决"LLM 在多步流程中天然漏步"的问题。
- **芯片验证工程实践** — assertion / coverage / scoreboard 方法论,芯片行业 30 年标准实践(验证人力是设计的 2–3 倍)。本协议 §3 两层裁判借鉴于此。

**实证经验来源**

- **AGENTS.md / CLAUDE.md 写作经验** — progressive disclosure(根文件短、细节下沉)、指令过载 anti-pattern(长上下文文件反而降低 agent 成功率,见 [arXiv 2602.11988](https://arxiv.org/html/2602.11988v1))、short-concrete-command 胜过长叙述。这些经验来自社区(如 [HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)、[AI Hero](https://www.aihero.dev/a-complete-guide-to-agents-md))的实证总结,也是本协议"执行层英文 + 极薄 + 双层结构"设计的依据。

**编排对象(本协议编排它们,不替代)**

| 工具 | 仓库 | 用途 |
|---|---|---|
| DeepEval | [confident-ai/deepeval](https://github.com/confident-ai/deepeval) | §3.1 断言 + §3.2 LLM 打分,pytest 原生 |
| LangSmith | [langchain-ai/langsmith](https://github.com/langchain-ai/langsmith-sdk) | §4 回归集 + trace 回放 + 对比 |
| ASSERT | [responsibleai/ASSERT](https://github.com/responsibleai/ASSERT) | 从需求自动生成断言用例 |
| harness-evals | [harness/harness-evals](https://github.com/harness/harness-evals) | 归一化打分(0.0–1.0) |
| EvalView | [GitHub Marketplace Action](https://github.com/marketplace/actions/evalview-ai-agent-testing) | CI 里记 agent 行为、抓回归 |

## License

MIT
