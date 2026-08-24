# stop-manual-testing

> **A skill that stops you from manually testing your AI agent. The agent reads it, builds a machine-checkable verification system, and self-converges in a closed loop — collapsing the ~90% of dev time you spend staring at runs and judging by gut feel.**
>
> **[中文](#中文版) · English**

---

# English

A [skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) for AI coding agents (Claude Code / Codex / ZCode / Cursor). Load it once, and instead of you manually clicking through the UI and eyeballing whether the agent "got better or worse this run," the agent builds itself a verification system where correctness is machine-checkable — then iterates inside a closed loop until it converges.

## The pain it solves

If you develop AI agents, you are likely stuck here:

- **~90% of your time goes to manual testing** — opening the web page, typing inputs, watching the agent step by step, judging with your own brain whether it got better or worse this run.
- **One prompt tweak breaks it**, and you cannot even tell at a glance whether it really broke — so you run it again, and again, and again.
- **Your job is no longer writing code**; it is repeatedly verifying a system even you no longer fully understand.

The root cause is a single asymmetry: **generation is cheap, verification is expensive.** What this skill does is turn "by human eye, by gut feel" verification into an **automated, repeatable, machine-checkable** system. Once "what counts as correct" becomes machine-decidable, the agent can run, fix, and converge on its own — you stop watching the screen.

## Install

Copy the skill into your agent's skills directory:

```bash
# Claude Code / ZCode / Codex standard location
git clone https://github.com/Kaiji-Z/stop-manual-testing.git
cp -r stop-manual-testing ~/.agents/skills/stop-manual-testing
# (or your project's .agents/skills/ if you want it scoped to one repo)
```

The skill is self-contained — just `SKILL.md` + `references/VERIFICATION.md`. No dependencies, no runtime.

## Use

```
1. Open a coding agent session in your AI agent project.

2. The skill announces its capabilities when loaded — and then STOPS.
   It does NOT auto-run. Loading a capability ≠ authorizing its execution.

3. Say one of:
   "stop manual testing"
   "start diagnosis" / "开始诊断"
   "read VERIFICATION.md"

4. The agent runs a 7-step diagnosis — read-only on your code, writing exactly two files:
   - reads project context (AGENTS.md, build config, entry code, test entry)
   - ACI audit: can your system run without the UI? are states logged? interfaces programmatic?
   - inventories existing test infrastructure
   - outputs a priority-sorted gap list (P0/P1/P2, with remediation plans)
   - instantiates the protocol into your project root as `VERIFICATION.md` — the
     project-local, customized copy (the globally installed skill stays a generic template)
   - fills §8 parameters in that local copy: auto-fills what code evidence supports;
     ASKS you for the critical items (acceptance criteria, supervisor design) — never guesses those
   - updates AGENTS.md to point at the project-local `VERIFICATION.md`, persisting the diagnosis
   - stops and reports; waits for your go-ahead before touching code
```

No production-code changes happen in the diagnosis round. Exactly two files are written: the project-local `VERIFICATION.md` and `AGENTS.md`. Remediation waits for your explicit confirmation.

## Template vs. instance: how project-local instantiation works

The diagnosis **instantiates** the protocol into your project root. From that moment, there are two artifacts with different jobs:

- **The skill (global template)** — generic, shared by all projects. Upgrade it once and every *new* diagnosis gets the new version. Never contains project-specific data.
- **`VERIFICATION.md` in your project root (instance)** — this project's customized copy: filled §8 parameters, acceptance criteria, supervisor design, evolving gap lists. Your `AGENTS.md` points at **this local file**, not at the skill — because what your project obeys is *your* customized protocol, not the generic template.

So after the first diagnosis, every session in that project auto-reads the protocol from `AGENTS.md` — you never have to re-invoke the skill just to keep the protocol in force. Re-invoking the skill is only for *re-diagnosis* (fresh audit, updated gap list).

Don't want the skill at all? The manual path is equivalent:

```bash
cp VERIFICATION.md /path/to/your-project/VERIFICATION.md
# then add to your project's AGENTS.md (create if absent):
# ## Mandatory protocol
# Before developing any feature or changing any code, read and follow the project-local `VERIFICATION.md`.
```

**Honest note:** a file in the project root lowers the *trigger* cost, but it does NOT raise the *compliance* rate. Both forms rely on the GATE mechanism for checkable compliance — the file form is "background knowledge" the agent can forget mid-task, while the loaded skill is "active instruction" in working memory. If strict adherence matters, spot-check the GATE declarations regardless of which form you use. The protocol text is the same at instantiation time; the local copy then drifts *by design* as the project customizes it.

## Why it actually drives the agent (core mechanisms)

- **Self-boot protocol** — the skill announces capabilities on load, then waits for an explicit trigger. No long prompt to type every session.
- **GATE mechanism (Declare-Verify-Enforce)** — the agent must emit a fixed-format GATE declaration after every step. No evidence = not done; declaration contradicts the artifact = redo. Compliance becomes visible and checkable, not just hoped for.
- **Two-layer judge** — Layer 1 deterministic assertions (absolutely reliable, zero cost; never use an LLM where it can catch it). Layer 2 an LLM supervisor that *scores only, never judges right/wrong*, and whose context must be clean (it must not know how the code is written, or it scores its own people high).
- **Flag-based A/B** — every new feature gets a flag. Run the same regression suite with flag=on/off; compare what got better / what got silently broken.
- **Orchestrate mature tools, don't reinvent** — detects your project's existing eval tools (DeepEval / LangSmith / pytest / jest) and lands on their APIs. If none exist, recommends but does NOT self-install.

## Prerequisite

**One hard condition. If it fails, the skill diagnoses it first, then helps remediate:**

Your AI agent system must be **AI-friendly (Agent Computer Interface)** — backend runs without the frontend, workflows trigger via CLI/API not browser clicks, intermediate states are structured and programmatically retrievable. If your system is currently UI-only, the skill will honestly surface this as P0 — because a verification system needs a system that can run without a human.

## What it is NOT (honest limits)

- **Not a testing framework.** It orchestrates existing eval tools (DeepEval / LangSmith, etc.); it does not replace them.
- **"Full compliance" relies on GATE self-declaration, not enforced execution.** A determined agent could still forge a GATE — the Verify rule catches most, but ultimate checking needs a human or next-round spot-check.
- **It cannot do architecture refactoring autonomously.** If §2.1 fails, splitting front/back is code work requiring your involvement — but the diagnosis will surface it honestly.

This is not a flaw — it is first-principle self-consistency: **verification never relies on goodwill; it relies on checkable evidence.**

## Files

```
stop-manual-testing/
├── SKILL.md                      # Entry point: announces capabilities, waits for trigger
├── references/
│   └── VERIFICATION.md           # Protocol body: 7-step diagnosis + GATE + two-layer judge
└── README.md                     # This file (bilingual)
```

The protocol body (`references/VERIFICATION.md`) is in English for maximum instruction-following reliability. Conversation with you follows your language.

## Design rationale (for the curious)

- **Verifier's Law** — Jason Wei (OpenAI), 2025: some tasks are far easier to verify than to solve, and AI progresses fastest where outputs are easy to check. So value shifts to: *can you turn a hard-to-verify problem into an easy-to-verify one?* Once yes, hand the rest to AI.
- **30 years of chip-industry precedent** — a failed tape-out costs millions, so verification headcount is 2–3× design headcount. The industry pours effort into "designing verification systems that expose bugs" — assertion / coverage / scoreboard. This skill's two-layer judge borrows from this.
- **Agent-Computer Interface (ACI)** — from the SWE-agent paper (Princeton NLP, NeurIPS 2024): interfaces designed for LLM agents affect performance *more than* swapping in a stronger model.
- **Why GATE exists** — LLMs drop steps in long pipelines (lost-in-the-middle + instruction overload); "please comply" cannot stop it. Making compliance visible, checkable, and blocking-on-mismatch is the only reliable way to turn "should do" into "actually did."

## References

Each is annotated with a verifiable original source.

**Core idea sources**

- **Verifier's Law** — Jason Wei, July 2025: [Asymmetry of verification and verifier's law](https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law).
- **Agent-Computer Interface (ACI)** — Yang et al., NeurIPS 2024: [SWE-agent (arXiv 2405.15793)](https://arxiv.org/abs/2405.15793). Code: [princeton-nlp/SWE-agent](https://github.com/princeton-nlp/SWE-agent).

**Mechanism design sources**

- **Visible Checklist Pattern (Declare-Verify-Enforce)** — wharsojo: [The Visible Checklist Pattern](https://dev.to/wharsojo/the-visible-checklist-pattern-enforcing-multi-step-pipeline-compliance-in-llm-agents-j30).
- **Chip-industry verification practice** — assertion / coverage / scoreboard methodology.

**Empirical evidence sources**

- **AGENTS.md / CLAUDE.md writing experience** — progressive disclosure, instruction-overload anti-pattern ([arXiv 2602.11988](https://arxiv.org/html/2602.11988v1)); community summaries from [HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [AI Hero](https://www.aihero.dev/a-complete-guide-to-agents-md).

**Orchestration targets (orchestrated, not replaced)**

| Tool | Repo | Use |
|---|---|---|
| DeepEval | [confident-ai/deepeval](https://github.com/confident-ai/deepeval) | assertions + LLM scoring, pytest-native |
| LangSmith | [langchain-ai/langsmith](https://github.com/langchain-ai/langsmith-sdk) | regression sets + trace replay |
| ASSERT | [responsibleai/ASSERT](https://github.com/responsibleai/ASSERT) | auto-generate assertion cases |
| harness-evals | [harness/harness-evals](https://github.com/harness/harness-evals) | normalized scoring |
| EvalView | [GitHub Marketplace](https://github.com/marketplace/actions/evalview-ai-agent-testing) | CI agent behavior recording |

## License

MIT

---
---

# 中文版

> **[English](#english) · 中文**

一个给 AI coding agent(Claude Code / Codex / ZCode / Cursor)用的 [skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)。加载它,你就不用再手动点网页、用人脑判断 agent 这次是变好还是变坏——agent 会给自己搭一套"对错机器可判定"的验证体系,然后在闭环里自己跑、自己改、自己收敛。

## 它解决什么痛点

如果你在做 AI agent 开发,大概率你正卡在这里:

- **90% 的时间耗在手动测试**——点开网页、输入数据、盯着 agent 一步步跑、用人脑判断这次是变好还是变坏。
- **改一句 prompt 就崩**,但你根本不知道它有没有真坏,只能再跑一遍、再跑一遍。
- **你的工作已经不是写代码**,而是反复验证一个连你自己都快看不懂的系统。

根本原因是一个不对称:**生成廉价,验证昂贵。** 这个 skill 做的事,是把"靠人眼靠感觉"的验证,改造成**自动的、可重复的、机器可判定的**体系。一旦"什么算对"变得机器可判定,agent 就能在闭环里自己收敛,你不用再守屏幕。

## 安装

把 skill 复制进你的 agent skills 目录:

```bash
# Claude Code / ZCode / Codex 标准位置
git clone https://github.com/Kaiji-Z/stop-manual-testing.git
cp -r stop-manual-testing ~/.agents/skills/stop-manual-testing
# (或你项目的 .agents/skills/,如果想限定在单个仓库)
```

skill 是自包含的——只有 `SKILL.md` + `references/VERIFICATION.md`。无依赖,无运行时。

## 用法

```
1. 在你的 AI agent 项目里开一个 coding agent 会话。

2. skill 加载后会展示它的能力,然后停下来。
   它不会自动跑。加载一个能力 ≠ 授权执行它。

3. 说其中一句:
   "stop manual testing"
   "开始诊断" / "start diagnosis"
   "读一下 VERIFICATION.md"

4. agent 跑一套 7 步诊断——对你的代码只读,只写两个文件:
   - 读项目上下文(AGENTS.md、构建配置、入口代码、测试入口)
   - ACI 体检:能脱离 UI 跑吗?状态留痕吗?接口程序化吗?
   - 摸底现有测试基建
   - 输出按优先级排序的差距清单(P0/P1/P2,带改造方案)
   - 把协议**实例化**到你项目根目录的 `VERIFICATION.md`——项目本地、可定制的副本
     (全局安装的 skill 始终保持通用模板)
   - 在本地副本里填 §8 参数:自动填能填的;该问你的(验收标准、supervisor 设计)问你,绝不瞎猜
   - 更新 AGENTS.md,指向项目本地的 `VERIFICATION.md`,把诊断落盘
   - 停下报告,等你点头后才动代码
```

诊断轮次不改业务代码。只写两个文件:项目本地的 `VERIFICATION.md` 和 `AGENTS.md`。改造要等你明确确认。

## 模板与实例:项目本地实例化是怎么工作的

诊断会把协议**实例化**到你项目根目录。从那一刻起,存在两个职责不同的工件:

- **skill(全局模板)**——通用,所有项目共享。升级一次,之后每次*新诊断*拿到的就是新版。永不包含项目数据。
- **你项目根的 `VERIFICATION.md`(实例)**——本项目的定制副本:填好的 §8 参数、验收标准、supervisor 设计、持续演进的差距清单。你的 `AGENTS.md` 指向**这个本地文件**,而不是指向 skill——因为你的项目遵守的是*你的*定制协议,不是通用模板。

所以首次诊断之后,该项目的每个会话都通过 AGENTS.md 自动读到协议——不用为了"让协议持续生效"再调 skill。再调 skill 只为了*重新诊断*(新鲜审计、更新差距清单)。

完全不想用 skill?手动路径等价:

```bash
cp VERIFICATION.md /path/to/your-project/VERIFICATION.md
# 然后在项目的 AGENTS.md(没有就建一个)里加:
# ## 强制规范
# 开发任何功能、改任何代码前,必读并遵守项目本地的 `VERIFICATION.md`。
```

**诚实提醒**:项目根的文件降低了*触发*成本,但**没有提高*遵守率***。两种形态都靠 GATE 机制保证可核验的遵守——文件形态是 agent 干活时可能忘记的"背景知识",而加载后的 skill 是工作记忆里的"活跃指令"。如果严格遵守很重要,不管用哪种形态,都要抽查 GATE 声明。协议文本在实例化那一刻是相同的;之后本地副本*按设计地*随项目定制而分化。

## 为什么它能真正驱动 agent(核心机制)

- **自启动协议**——skill 加载后展示能力,然后等显式触发。不用每轮打长 prompt。
- **GATE 机制(Declare-Verify-Enforce)**——agent 每步必须输出固定格式的 GATE 声明。无证据 = 未做;声明与实际不符 = 重做。遵守状态变得可见、可核验,而不是靠希望。
- **两层裁判**——第一层确定性断言(绝对可靠、零成本,凡能用这层卡的绝不上 LLM)。第二层 LLM supervisor 只打分不判对错,且上下文必须干净(不能知道代码怎么写,否则给自己人打高分)。
- **flag 开关对照**——每个新功能配 flag,跑同一套回归对比"哪些变好/哪些被悄悄弄坏"。
- **编排成熟工具,不重复造轮子**——检测项目已有的 eval 工具(DeepEval / LangSmith / pytest / jest),直接用它的 API。没有就推荐,但不自行安装。

## 前提

**一个硬条件,过不了它,skill 先诊断出来,再帮你改造:**

你的 AI agent 系统必须是**对 AI 友好的(Agent Computer Interface)**——后端脱离前端能跑、workflow 用 CLI/API 触发不用点网页、中间状态结构化可程序化取出。如果你的系统现在只能 UI 测,skill 会诚实把它列为 P0——因为验证体系需要一个能脱离人运行的系统。

## 它不是什么(诚实边界)

- **不是测试框架。** 它编排已有的 eval 工具(DeepEval / LangSmith 等),不替代它们。
- **"完全遵守"靠 GATE 自声明,不是强制执行。** 一个够执着的 agent 仍可能伪造 GATE——Verify 规则能拦住大部分,但终极核验还得靠人或下一轮抽查。
- **不能自动做架构重构。** 如果 §2.1 不达标,拆前后端是要你看代码的活——但诊断会诚实把它暴露出来。

这不是缺陷,是第一性原理的自我一致:**验证从不靠自觉,靠可核验的证据。**

## 文件结构

```
stop-manual-testing/
├── SKILL.md                      # 入口:展示能力,等触发
├── references/
│   └── VERIFICATION.md           # 协议主体:7 步诊断 + GATE + 两层裁判
└── README.md                     # 本文件(双语)
```

协议主体(`references/VERIFICATION.md`)用英文,是为了指令遵循率最高。和你对话用你的语言。

## 设计原理(给想深入的人)

- **验证不对称性 / Verifier's Law** — Jason Wei(OpenAI),2025:有些任务验证远比求解容易,AI 在"输出易于检查"的任务上进步最快。所以价值转移到:你能不能把"难验证"改造成"易验证"。
- **芯片行业的 30 年先例** — 流片失败损失几百万,验证人力是设计的 2–3 倍。行业把精力压在"设计可暴露 bug 的验证体系"。本 skill 的两层裁判借鉴于此。
- **Agent Computer Interface(ACI)** — SWE-agent 论文(Princeton NLP,NeurIPS 2024):为 LLM agent 设计的接口,对表现的影响超过换更强的模型。
- **为什么有 GATE** — LLM 在长流程里天然漏步,靠"请遵守"拦不住。把遵守状态显式化、可核验、不实则阻断,是唯一可靠的办法。

## 参考

每条都标注了可核验的原始出处。

**核心思想来源**

- **Verifier's Law** — Jason Wei,2025 年 7 月:[Asymmetry of verification and verifier's law](https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law)。
- **Agent-Computer Interface(ACI)** — Yang et al.,NeurIPS 2024:[SWE-agent (arXiv 2405.15793)](https://arxiv.org/abs/2405.15793)。代码:[princeton-nlp/SWE-agent](https://github.com/princeton-nlp/SWE-agent)。

**机制设计来源**

- **Visible Checklist Pattern(Declare-Verify-Enforce)** — wharsojo:[The Visible Checklist Pattern](https://dev.to/wharsojo/the-visible-checklist-pattern-enforcing-multi-step-pipeline-compliance-in-llm-agents-j30)。
- **芯片验证工程实践** — assertion / coverage / scoreboard 方法论。

**实证经验来源**

- **AGENTS.md / CLAUDE.md 写作经验** — progressive disclosure、指令过载 anti-pattern([arXiv 2602.11988](https://arxiv.org/html/2602.11988v1));社区总结见 [HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)、[AI Hero](https://www.aihero.dev/a-complete-guide-to-agents-md)。

**编排对象(编排它们,不替代)**

| 工具 | 仓库 | 用途 |
|---|---|---|
| DeepEval | [confident-ai/deepeval](https://github.com/confident-ai/deepeval) | 断言 + LLM 打分,pytest 原生 |
| LangSmith | [langchain-ai/langsmith](https://github.com/langchain-ai/langsmith-sdk) | 回归集 + trace 回放 |
| ASSERT | [responsibleai/ASSERT](https://github.com/responsibleai/ASSERT) | 从需求生成断言用例 |
| harness-evals | [harness/harness-evals](https://github.com/harness/harness-evals) | 归一化打分 |
| EvalView | [GitHub Marketplace](https://github.com/marketplace/actions/evalview-ai-agent-testing) | CI 记 agent 行为 |

## License

MIT
