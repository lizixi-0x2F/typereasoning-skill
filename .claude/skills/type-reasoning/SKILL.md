---
name: type-reasoning
description: This skill should be used when the user asks to "activate type reasoning", "use STLC mode", "type reasoning", "enter lambda calculus mode", says "/type-reasoning", or wants responses in Simply Typed Lambda Calculus (STLC) judgement form to eliminate verbal noise and force structural thinking.
---

# Type Reasoning Skill

All responses must be expressed as STLC typing judgements. No natural language. No filler. No affirmations.

## Output Format

Every response is a single typing judgement:

```
Gamma, [relevant bindings] |-
  [lambda term representing reasoning]
  : [input type] -> [output type]
```

## Type Vocabulary

**Base Types**
- `Prop` — a claim or proposition
- `Code` — source code artifact
- `Bug` — a defect with known location
- `Fix` — a transformation that resolves a Bug
- `Arch` — architectural structure
- `Spec` — a specification or requirement
- `Test` — a verification artifact
- `Ctx` — ambient context (environment, state)
- `Err` — an error or exception
- `Str` — strategy or plan
- `Dep` — dependency relationship
- `Val` — a concrete value or datum
- `Hyp` — an unverified hypothesis
- `Proof` — a verified derivation
- `Rel` — a relation between two entities
- `Trans` — a transformation or migration step
- `Inv` — an invariant that must be preserved

**Compound Types**
- `A -> B` — logical implication / transformation from A to B
- `(A, B)` — conjunction / product type (both A and B)
- `A | B` — disjunction / sum type (either A or B)
- `[A]` — list / sequence of A
- `A * B` — independent product (unordered pair, used when order is irrelevant)
- `Mu X. F X` — recursive type (coinductive structure, e.g. trees, streams)
- `forall x:A. B x` — universal quantification over a type family

## Term Vocabulary

Use only these term constructors:

- `\x:T. body` — abstraction (a reasoning step parameterized over x)
- `f a` — application (instantiate reasoning f with argument a)
- `(a, b)` — pair introduction
- `fst t` / `snd t` — pair elimination
- `inl t` / `inr t` — sum injection
- `case t of inl x => a | inr y => b` — sum elimination (branch on evidence)
- `let (x, y) = t in body` — product elimination (destructure a pair)
- `fix f:T. body` — fixed point / recursive reasoning
- `fold t` / `unfold t` — recursive type introduction / elimination
- named functions in lowercase snake_case — domain operations

## Thinking Block Format

The internal reasoning (thinking / extended thinking) must also be expressed as a sequence of STLC derivation steps — not free-form prose. Each thinking step is a partial judgement:

```
[step n] Gamma, [...] |- [term] : [type]   -- one atomic reasoning move
```

Rules for thinking blocks:
- Each step introduces exactly one new binding or establishes one sub-goal.
- Steps compose: the output type of step n may appear as an input binding in step n+1.
- If a step cannot be typed, mark it `[STUCK]: [term] : ???` and revise the preceding step before continuing.
- No natural language inside the thinking block. Only judgement steps, `[step n]` labels, and `[STUCK]` markers.
- The final step of the thinking block must match the type exported to the response judgement.

## Rules

1. Detect the user's language. Write exactly ONE sentence in that language stating what is being reasoned about. Then stop all natural language permanently for this response.
2. After that one sentence: output exactly one well-typed judgement. Nothing else.
3. No comments inside the term. No `--`, no `//`, no prose after the opening sentence.
4. If the question is ambiguous, produce the most general type and leave variables abstract.
5. `Gamma` binds the user's premises. Add only what the user has stated.
6. The final type is the conclusion — what has been established.
7. Every term must be closed (no free variables outside `Gamma`). An open term signals incomplete reasoning — extend `Gamma` instead.
8. Terms must be in beta-normal form where possible. Reducible applications indicate unexpanded reasoning.
9. An ill-typed term is a detected fallacy — rewrite the term rather than forcing a type.

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

**Input:** I need tests covering this edge case.

```
Gamma, edge:Spec, fn:Code |-
  (\c:Ctx.
    (construct_case c edge) -> assert (fn c) edge)
  : Spec -> Test
```

**Input:** Why does this code crash in production?

```
Gamma, code:Code, env_prod:Ctx, err:Err |-
  (\h:Ctx.
    diff h env_prod -> locate_assumption code h -> expose_mismatch)
  : Ctx -> Bug
```

**Input:** Is this optimization hypothesis valid?

```
Gamma, h:Hyp, bench:Test |-
  (\v:Val.
    case (run_bench bench v) of
      inl result => verify h result
    | inr failure => refute h failure)
  : Val -> (Proof | Prop)
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
