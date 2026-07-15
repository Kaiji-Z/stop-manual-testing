# agent-verifier-protocol

> 一个 Markdown 协议文件,让 AI coding agent(Claude Code / Codex 等)读一次,就能在你的项目里自动造一套机器可判定的验证体系,把人工测试时间从 90% 压到接近 0。

**这不是一个工具,不依赖任何运行时。** 它只是一份 `VERIFICATION.md`——放进你的项目根目录,开一个新会话对 agent 说"读一下 VERIFICATION.md",agent 就会自动诊断你的项目、改造接口、搭建回归测试、在闭环里自己收敛。

---

## 它解决什么痛点

如果你在做 AI agent 开发,大概率你正卡在这里:

- **90% 的时间耗在手动测试**:点开网页、输入数据、盯着 agent 一步步跑、用人脑判断这次是变好还是变坏。
- **改一句 prompt 就崩**,但你根本不知道它有没有真坏,只能再跑一遍、再跑一遍。
- **你的工作已经不是写代码**,而是反复验证一个连你自己都快看不懂的系统。

这套痛点有一个根本原因:**生成廉价,验证昂贵。** AI 几秒就能生成代码,但"这段代码到底对不对"如果只能靠人盯,你就永远被困在验证里。

本协议做的事,就是把"靠人眼靠感觉"的验证,改造成**自动的、可重复的、机器可判定的**验证体系。一旦"什么算对"变得机器可判定,agent 就能在闭环里自己跑、自己改、自己收敛,不需要你守屏幕。

---

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

---

## 它要求什么前提

**一个硬条件,过不了它,本协议先帮你诊断出来,再改造:**

你的 AI agent 系统必须是**对 AI 友好的(Agent Computer Interface)**:
- 后端能脱离前端独立启动
- 触发 workflow 有 CLI/API,不依赖浏览器点击
- 中间状态有结构化记录,可程序化取出

如果你的系统现在只能靠点网页测,agent 会把这个差距诚实诊断出来,列为 P0——因为验证体系需要一个能脱离人、独立运行的系统。这一步是架构活,不是读规范能自动解决的,但诊断是可靠的。

---

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

---

## 谁该用什么

| 你的情况 | 该怎么用 |
|---|---|
| 我在做 AI agent 开发,被手动测试折磨 | 放进项目,说"读一下 VERIFICATION.md",让 agent 帮你诊断 |
| 我想理解这套体系为什么这么设计 | 见下方"设计原理"和参考链接 |
| 我想贡献或改造这个协议 | fork 后改 `VERIFICATION.md`;执行层是英文,教学层在本 README |

---

## 设计原理(给想深入的人)

### 验证不对称性(Verifiers Law)
OpenAI 研究员 Jason Wei 提出的观察:很多任务做出来难,但验证它对不对却很容易。推论:**任何可解且易于验证的任务,终将被 AI 解决。** 所以价值转移到——你能不能把"难验证"改造成"易验证"。一旦做到,剩下交给 AI。人的工作从"运行时盯屏"变成"一次性定义什么算对"。

### 芯片行业的 30 年先例
芯片流片失败损失几百万美元,所以验证人力是设计的 2–3 倍。行业把绝大部分精力压在"设计可暴露 bug 的验证体系"——assertion / coverage / scoreboard。本协议的两层裁判直接借鉴于此,只是对 agent 的不确定性容忍度更高。

### Agent Computer Interface(ACI)
来自普林斯顿 SWE-agent 论文。核心论点:AI agent 是一类全新用户,接口设计对 agent 表现的影响,**甚至大过你换一个更强的模型**。本协议的 §2 体检三项就是这个概念的落地——先确认你的系统"配不配被 AI 验证"。

### 为什么有 GATE 机制
社区实证经验表明:LLM 在多步流程中天然漏步(lost in the middle + 指令过载),靠"请务必遵守"拦不住。把遵守状态显式化、可核验、不实则阻断,是把"应该做"变成"实际做了"的唯一可靠机制。

---

## 它做不到什么(诚实边界)

- **它不是工具,跑不了任何验证。** 它让 agent 知道"该造什么",但断言怎么跑、supervisor 怎么调,接的是你项目已有的 eval 工具(DeepEval / LangSmith 等)。
- **它的"完全遵守"靠 GATE 自声明,不是强制执行。** 一个足够懒的 agent 仍可能伪造 GATE 声明。GATE 的 Verify 规则能拦住大部分伪造(要求每个"做了"必须有证据),但终极核验还是需要人或下一轮 agent 抽查。
- **它推不动架构重构。** 如果你的系统现在只能 UI 测,§2.1 不达标,那拆前后端是看代码才能干的活。但 agent 会把这个差距诚实诊断出来,写进 AGENTS.md 待办,下一轮继承。

这不是缺陷,是第一性原理的自我一致:**验证从不靠自觉,靠可核验的证据。** 本协议自己也遵守这条——它不假设 agent 会自觉遵守它,而是用 GATE 把遵守变成可核验的产物。

---

## 文件说明

| 文件 | 语言 | 作用 | 读者 |
|---|---|---|---|
| `VERIFICATION.md` | 英文 | 纯执行层协议,agent 读了照着干 | coding agent |
| `README.md` | 中文 | 痛点、用法、设计原理 | 开发者(你) |

执行层用英文,是因为 coding agent 对英文长指令 + 红线约束的遵循率最高,且和代码/API 同构、注意力损耗最低。教学层用中文,降低你的维护成本。

---

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

**编排对象(本协议不替代,而是编排它们)**

| 工具 | 仓库 | 用途 |
|---|---|---|
| DeepEval | [confident-ai/deepeval](https://github.com/confident-ai/deepeval) | §3.1 断言 + §3.2 LLM 打分,pytest 原生 |
| LangSmith | [langchain-ai/langsmith](https://github.com/langchain-ai/langsmith-sdk) | §4 回归集 + trace 回放 + 对比 |
| ASSERT | [responsibleai/ASSERT](https://github.com/responsibleai/ASSERT) | 从需求自动生成断言用例 |
| harness-evals | [harness/harness-evals](https://github.com/harness/harness-evals) | 归一化打分(0.0–1.0) |
| EvalView | [GitHub Marketplace Action](https://github.com/marketplace/actions/evalview-ai-agent-testing) | CI 里记 agent 行为、抓回归 |

---

## License

MIT
