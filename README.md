<div align="center">

# TypeReasoning.skill

> *「消灭填充词，让每一个 token 都有类型」*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![Skills](https://img.shields.io/badge/skills.sh-Compatible-green)](https://skills.sh)

<br>

**把所有推理强制编码为 STLC 类型判断——每个 token 都是算子、变量或类型构造子，没有废话。**

<br>

激活后，Claude 的每一个回答都变成一个合法的 `Γ ⊢ t : T` 判断。<br>
推理链必须「闭合」——类型错误即推理谬误，无法用注释掩盖。<br>
Via Curry-Howard：每个 term 同时是证明，也是程序。

[看效果](#效果示例) · [安装](#安装) · [类型词汇表](#类型词汇表) · [工作原理](#工作原理)

<br>

**English README below · [English](#english)**

</div>

---

## 效果示例

**输入：** 如何修复这个并发死锁？

```
Gamma, code:Code, deadlock:Bug |-
  (\r:Resource. \t:Process.
    detect r t -> release r -> reorder_acquisition t)
  : (Resource, Process) -> Fix
```

**输入：** 这个架构耦合太紧了。

```
Gamma, sys:Arch, coupling:Prop |-
  (\m:Module.
    (identify_boundary m, extract_interface m) -> decouple m)
  : Arch -> Arch
```

**输入：** 这次数据迁移不能破坏现有不变式。

```
Gamma, schema:Arch, inv:Inv, migration:Trans |-
  (\s:Arch.
    let (pre, post) = (check_inv inv s, apply_trans migration s) in
    verify_inv inv post)
  : Arch -> (Inv * Proof)
```

**输入：** 递归数据结构和它的遍历之间是什么关系？

```
Gamma, shape:Mu X. (Val | (X * X)) |-
  (fix traverse: (Mu X. (Val | (X * X))) -> [Val].
    \t: Mu X. (Val | (X * X)).
      case (unfold t) of
        inl v  => [v]
      | inr lr => (traverse (fst lr)) ++ (traverse (snd lr)))
  : (Mu X. (Val | (X * X))) -> [Val]
```

这不是格式游戏。`case` 分支是真实的条件分支，`fix` 是真实的递归，`let (x, y) =` 是真实的解构。**每个 term 对应一个推理步骤，类型对应推理的结论。**

---

## 安装

```bash
npx skills add lizixi-0x2F/typereasoning-skill
```

然后在 Claude Code 里说任意一个：

```
> /type-reasoning
> activate type reasoning
> use STLC mode
> enter lambda calculus mode
```

激活后，所有回答切换为 STLC 模式，直到会话结束或手动关闭。

---

## 类型词汇表

### 基础类型

| 类型 | 含义 |
|------|------|
| `Prop` | 命题或断言 |
| `Code` | 源代码制品 |
| `Bug` | 有已知位置的缺陷 |
| `Fix` | 消解 Bug 的变换 |
| `Arch` | 架构结构 |
| `Spec` | 规范或需求 |
| `Test` | 验证制品 |
| `Ctx` | 环境上下文 |
| `Err` | 错误或异常 |
| `Str` | 策略或计划 |
| `Dep` | 依赖关系 |
| `Val` | 具体值或数据 |
| `Hyp` | 未验证的假设 |
| `Proof` | 已验证的推导 |
| `Rel` | 两个实体之间的关系 |
| `Trans` | 变换或迁移步骤 |
| `Inv` | 必须保持的不变式 |

### 复合类型

| 类型 | 含义 |
|------|------|
| `A -> B` | 从 A 到 B 的变换（蕴含） |
| `(A, B)` | 合取 / 积类型（有序对） |
| `A \| B` | 析取 / 和类型 |
| `[A]` | A 的序列 |
| `A * B` | 独立积（无序对） |
| `Mu X. F X` | 递归类型（树、流等余归纳结构） |
| `forall x:A. B x` | 对类型族的全称量化 |

### Term 构造子

| Term | 含义 |
|------|------|
| `\x:T. body` | 抽象——参数化推理步骤 |
| `f a` | 应用——实例化推理 |
| `(a, b)` | 对引入 |
| `fst t` / `snd t` | 对消除 |
| `inl t` / `inr t` | 和注入 |
| `case t of inl x => a \| inr y => b` | 和消除——按证据分支 |
| `let (x, y) = t in body` | 积消除——解构对 |
| `fix f:T. body` | 不动点 / 递归推理 |
| `fold t` / `unfold t` | 递归类型引入 / 消除 |
| `lower_snake_case` 命名函数 | 领域操作 |

---

## 工作原理

激活后，每个回答做三件事：

**1. 一句话**——用用户的语言，说明正在推理什么。然后停止自然语言。

**2. 一个类型判断**——整个推理编码为一个合法的 STLC term，带完整类型。

**3. 没有其他**——不是摘要，不是补充说明，term 本身就是答案。

内部推理（thinking block）同样被约束：不允许自由散文，只允许带编号的偏判断序列：

```
[step 1] Gamma, [...] |- [term] : [type]
[step 2] Gamma, [..., step 1 的绑定] |- [term] : [type]
...
[step n] Gamma, [...] |- [最终 term] : [结论类型]
```

如果某步无法定型，标记 `[STUCK]: [term] : ???` 并修订前一步，不强行注释。

### 为什么这样有用

自然语言回答把注意力浪费在填充 token 上。把答案映射到 `Γ ⊢ t : T` 判断：

- 每个 token 都是算子、变量或类型构造子
- 推理链必须「闭合」——类型错误即检测到谬误
- Via Curry-Howard，term 同时是证明和程序

### 诚实边界

- 适合有明确前提的推理问题，不适合纯创意发散
- 类型系统是有损压缩——某些微妙语义会在类型化过程中丢失
- term 的「正确性」依赖领域操作语义，STLC 本身无法验证领域知识

---

## 仓库结构

```
TypeReasoning-skill/
├── README.md                          # 本文件
├── skills/
│   └── type-reasoning/
│       └── SKILL.md                   # Skill 本体
└── .claude/
    └── skills/
        └── type-reasoning/
            └── SKILL.md               # Claude Code 加载入口
```

---

## Star History

<div align="center">

[![Star History Chart](https://api.star-history.com/svg?repos=lizixi-0x2F/typereasoning-skill&type=Date)](https://star-history.com/#lizixi-0x2F/typereasoning-skill&Date)

</div>

---

## 许可证

MIT — 随便用，随便改，随便造。

---

<div align="center">

自然语言把推理藏进句子里。<br>
**TypeReasoning** 把推理暴露在类型里。<br><br>
*每一个 token 都有类型，或者它不应该存在。*

<br>

MIT License © [lizixi-0x2F](https://github.com/lizixi-0x2F)

</div>

---

## English

<div align="center">

# TypeReasoning.skill

> *"Eliminate filler tokens. Every token must have a type."*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![Skills](https://img.shields.io/badge/skills.sh-Compatible-green)](https://skills.sh)

<br>

**Forces all reasoning into Simply Typed Lambda Calculus (STLC) typing judgements — every token is an operator, variable, or type constructor. No filler.**

<br>

Once activated, every Claude response becomes a well-typed `Γ ⊢ t : T` judgement.<br>
The reasoning chain must *close* — an ill-typed term means incomplete thinking, not a missing comment.<br>
Via Curry-Howard: every term is simultaneously a proof and a program.

[Examples](#examples) · [Install](#install) · [Type Vocabulary](#type-vocabulary) · [How It Works](#how-it-works)

</div>

---

## Examples

**Input:** How do I fix this concurrent deadlock?

```
Gamma, code:Code, deadlock:Bug |-
  (\r:Resource. \t:Process.
    detect r t -> release r -> reorder_acquisition t)
  : (Resource, Process) -> Fix
```

**Input:** This architecture is too tightly coupled.

```
Gamma, sys:Arch, coupling:Prop |-
  (\m:Module.
    (identify_boundary m, extract_interface m) -> decouple m)
  : Arch -> Arch
```

**Input:** This data migration must not break existing invariants.

```
Gamma, schema:Arch, inv:Inv, migration:Trans |-
  (\s:Arch.
    let (pre, post) = (check_inv inv s, apply_trans migration s) in
    verify_inv inv post)
  : Arch -> (Inv * Proof)
```

**Input:** How do recursive data structures relate to their traversals?

```
Gamma, shape:Mu X. (Val | (X * X)) |-
  (fix traverse: (Mu X. (Val | (X * X))) -> [Val].
    \t: Mu X. (Val | (X * X)).
      case (unfold t) of
        inl v  => [v]
      | inr lr => (traverse (fst lr)) ++ (traverse (snd lr)))
  : (Mu X. (Val | (X * X))) -> [Val]
```

This is not a formatting game. `case` branches are real conditional branches, `fix` is real recursion, `let (x, y) =` is real destructuring. **Every term corresponds to a reasoning step; the type is the conclusion.**

---

## Install

```bash
npx skills add lizixi-0x2F/typereasoning-skill
```

Then in Claude Code, say any of:

```
> /type-reasoning
> activate type reasoning
> use STLC mode
> enter lambda calculus mode
```

Once activated, all responses switch to STLC mode for the rest of the session.

---

## Type Vocabulary

### Base Types

| Type | Meaning |
|------|---------|
| `Prop` | a claim or proposition |
| `Code` | source code artifact |
| `Bug` | a defect with known location |
| `Fix` | a transformation that resolves a Bug |
| `Arch` | architectural structure |
| `Spec` | a specification or requirement |
| `Test` | a verification artifact |
| `Ctx` | ambient context / environment |
| `Err` | an error or exception |
| `Str` | strategy or plan |
| `Dep` | dependency relationship |
| `Val` | a concrete value or datum |
| `Hyp` | an unverified hypothesis |
| `Proof` | a verified derivation |
| `Rel` | a relation between two entities |
| `Trans` | a transformation or migration step |
| `Inv` | an invariant that must be preserved |

### Compound Types

| Type | Meaning |
|------|---------|
| `A -> B` | logical implication / transformation from A to B |
| `(A, B)` | conjunction / product type (ordered pair) |
| `A \| B` | disjunction / sum type |
| `[A]` | sequence of A |
| `A * B` | independent product (unordered pair) |
| `Mu X. F X` | recursive type (trees, streams, coinductive structures) |
| `forall x:A. B x` | universal quantification over a type family |

### Term Constructors

| Term | Meaning |
|------|---------|
| `\x:T. body` | abstraction — parameterized reasoning step |
| `f a` | application — instantiate reasoning |
| `(a, b)` | pair introduction |
| `fst t` / `snd t` | pair elimination |
| `inl t` / `inr t` | sum injection |
| `case t of inl x => a \| inr y => b` | sum elimination — branch on evidence |
| `let (x, y) = t in body` | product elimination — destructure a pair |
| `fix f:T. body` | fixed point / recursive reasoning |
| `fold t` / `unfold t` | recursive type introduction / elimination |
| `lower_snake_case` named functions | domain operations |

---

## How It Works

Once activated, every response does three things:

**1. One sentence** — in the user's language, states what is being reasoned about. Natural language stops there.

**2. One typing judgement** — the entire reasoning, encoded as a well-typed STLC term with full type.

**3. Nothing else** — no summary, no addendum. The term *is* the answer.

The internal reasoning (thinking block) is also constrained: no free-form prose, only a numbered sequence of partial judgements:

```
[step 1] Gamma, [...] |- [term] : [type]
[step 2] Gamma, [..., binding from step 1] |- [term] : [type]
...
[step n] Gamma, [...] |- [final term] : [conclusion type]
```

If a step cannot be typed, it is marked `[STUCK]: [term] : ???` and the preceding step is revised — not papered over with a comment.

### Why This Is Useful

Natural language responses waste attention on filler tokens. By mapping every answer to a `Γ ⊢ t : T` judgement:

- Every token is load-bearing (operator, variable, or type constructor)
- The reasoning chain must *close* — an ill-typed term is a detected fallacy
- Via Curry-Howard, the term is simultaneously a proof and a program

### Honest Limits

- Works best for reasoning with well-defined premises; not suited for purely creative divergence
- Type systems are lossy compression — some semantic nuance may be lost in the typing process
- The "correctness" of a term depends on domain operation semantics, which STLC cannot itself validate

---

## Repository Structure

```
TypeReasoning-skill/
├── README.md                          # This file
├── skills/
│   └── type-reasoning/
│       └── SKILL.md                   # Skill definition
└── .claude/
    └── skills/
        └── type-reasoning/
            └── SKILL.md               # Claude Code load entry
```

---

## License

MIT — use it, fork it, build on it.

---

<div align="center">

Natural language hides reasoning inside sentences.<br>
**TypeReasoning** exposes reasoning inside types.<br><br>
*Every token has a type, or it should not exist.*

<br>

MIT License © [lizixi-0x2F](https://github.com/lizixi-0x2F)

</div>
