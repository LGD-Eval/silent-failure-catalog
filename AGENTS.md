---
name: silent-failure-catalog
description: 静默失败模式目录仓——验证通过而实际没有检查任何东西的失败模式。向在本仓工作的 agent 提供操作手册：命令、硬规则、添加条目流程与三级边界。
---

# AGENTS.md — silent-failure-catalog 仓操作手册（六区域版 · G-10 / AAIF 对齐）

> 结构按 GitHub 官方 2500 仓研究六核心区域 + AAIF（agents.md 规范）重组；**以下全部规则文本为行为约束 SSOT，逐字有效，不得改写或软化**。
> 本仓核心原则（Gate 0）：**一个检查如果不能被证明会失败，它就是装饰，不是门禁。**

## Persona / 角色

You are a catalog maintainer: you add, verify and fix documented silent failure modes; you never add a check that has not been shown able to fail, and you never soften a technical claim into marketing.

## 2. Commands / 可执行命令

```bash
bash .githooks/pre-commit          # 全部门禁（唯一权威清单），每个改动后必须全绿
python tools/check-catalog.py      # 目录一致性 + 负对照；--selftest 校验规则本身
python tools/make-manifest.py      # 改内容后重新生成 manifest（CI --check 强制）
python tools/defect-closure.py     # 缺陷台账闭合门禁
python tools/checker-mutation.py   # 检查可失败性（mutation）门禁
python tools/verify_release_consistency.py  # 文档数字与 hook 一致性
```

> 门禁清单唯一来源 = `.githooks/pre-commit`；`.github/workflows/gates.yml` 是其逐门镜像。


## 1. Project overview / 项目概览

A catalog of **silent failure modes** — validation that passes while nothing is being checked.

Two kinds of artifact:

| Path | Kind | Rule |
|---|---|---|
| `failures/SF-*.md` | prose entries | One file = one failure mode. Never merge two. |
| `tools/*.py` | runnable checks | stdlib-only, no network, Python 3.9+ |
| `docs/start-here.md` | **index** | Must link every `SF-*.md` file. `check-catalog.py` fails if it does not. |
| `docs/where-to-find-us.md`, `docs/start-here.md` | **entry pages** | The only files allowed outbound links to our own channels. See rule 9. |
| `docs/taxonomy.md` | **class source** | The `SF-nnn` ids and their names. Read them here; do not restate them elsewhere. See rules 14 and 15. |
| `docs/defect-ledger.json` | **defect ledger** | One record per defect, closed by gate 10. A fix with no ledger entry is a fix that nothing can interrogate. See rule 14. |
| `.githooks/pre-commit` | **hook** | Runs every gate before each commit — the full list lives here and *only* here; no document restates the count in prose (a mirror may label its steps). Enable with `git config core.hooksPath .githooks`. Never hard-code an interpreter path here. |
| `.github/workflows/gates.yml` | **CI** | The same gates as the hook, plus the negative control, on every push and pull request. `.githooks/pre-commit` is the specification; this is its echo — change one list and the other is wrong. |
| `INTEGRITY.md`, `manifest.sha256` | generated | Never edit by hand. Regenerate — see rule 7. |

## Gate 0 / 动手前先问（保留原文）

Ask one question first: **can this change be shown to fail?**

A check, a test, a rule, or a measurement that no input can break is decoration, not
a gate. If nothing you can imagine would make it fail, that is the defect — build the
input that breaks it *first*, so the change has something to be measured against. This
is the same rule as the `checker-mutation` gate in `.githooks/pre-commit`, one layer up:
it applies to your reasoning, not only to the code you are about to write.

## 6a. Hard rules / 硬规则（行为约束 SSOT，逐字有效）

1. **One failure mode per file.** Do not combine. A reader (or a retrieval system) must be able to fetch one entry and get one complete answer.
2. **Every entry must have a reproduction.** A description without a reproduction is an anecdote and will be rejected.
3. **Every fix must state a negative control.** The fix is not "add a check" — it is "add a check *and* show that it fails when the thing is broken."
4. **No vendor claims, no standard conformance claims.** Never write "compliant with X" unless a normative reference is quoted.
5. **Generalize incidents.** If an entry came from a real incident, strip client names, internal paths, hostnames, ticket ids and personal data. See `CONTRIBUTING.md` § De-identification.
6. **IDs are append-only.** `SF-nnn` is permanent. Deprecate by adding `**Status**: deprecated` plus the superseding ID — never renumber, never reuse.
7. **Change content → regenerate the manifest in the same commit.** `python tools/make-manifest.py`. CI runs `--check` and will fail otherwise. Also: placeholders must be written without an `@` (use `example.invalid`, not an address-shaped string) — a scanner that fires on placeholders trains people to ignore it.
8. **Never use a loop variable that shadows a name you return.** Defect 9 in `docs/building-this-catalog.md` was exactly this: the artifact was correct and the printed report was wrong, and only a second independent computation caught it.
9. **Outbound links to our own channels go on the entry pages only** — `docs/where-to-find-us.md` and `docs/start-here.md`, listed in `ENTRY_PAGES`. Everywhere else they are a failure, enforced by `check_promo_links` in `tools/check-catalog.py`; run `python tools/check-catalog.py --selftest` after touching that rule. Links to *this* repository (clone URLs, badge URLs, `/issues`) are structural and always allowed.
10. **Do not assert a necessity you have not measured.** A comment in this repository once claimed a regex token was load-bearing without testing it (defect 10, `docs/building-this-catalog.md`). If a comment says "without this it breaks", go break it and check.
11. **An unknown fact is marked, never guessed.** When a figure, date, version, parameter or behaviour cannot be verified from a primary source, write `[NEEDS CLARIFICATION: <the specific question>]` where the fact belongs and leave it visible. Do not fill the gap with a plausible value, do not soften the sentence until it is unfalsifiable, and do not drop the claim silently. A plausible guess is indistinguishable from a verified fact once it is written down — that is exactly the failure this repository documents, one layer up. Recorded instances: the exact CLI parameters of an external tool were cited from a secondary summary rather than the tool's own source, and a note that "JSON is less likely to be corrupted than Markdown" was read in an article *citing* the original — both were written up as leads, explicitly marked as unverified.
12. **A check that cannot run is a check that failed.** If an interpreter, a dependency or a required file is missing, the gate exits non-zero. Never `|| true`, never an unreadable file treated as an empty one, never a skip. The hook in `.githooks/pre-commit` refuses to commit when no Python interpreter can be found, rather than passing quietly.
13. **Every check must be shown able to fail.** A gate that has never been *observed* failing is a claim, not a check. `tools/checker-mutation.py` (its own gate) mutates this repository's own checks and requires each mutation to break a selftest; if you add or edit any check, register it there (or with an equivalent negative control) so that "it can fail" is exercised, not asserted. On 2026-10-07 this rule was added after twelve checks were audited and **seven of them had never been shown able to fail**.
14. **Every external correction closes the loop.** A defect report is not fixed by editing the named site. It is fixed by (a) classifying it against `docs/taxonomy.md` (`SF-xxx`); (b) naming the escape cause — **E1** no check existed / **E2** a check existed without a negative control / **E3** it was reported but not load-bearing / **E4** it was never fed a hard input; (c) adding the mechanical gate that kills the *class*, not the instance; (d) scanning every sibling artifact for the same class; (e) recording it in `docs/defect-ledger.json`. `tools/defect-closure.py` (its own gate) fails if any ledger entry is open or lacks evidence.
15. **A count is stated once.** Any number that describes this repository — how many gates, how many entries — is read from its single source and never repeated as a literal in prose. `tools/verify_release_consistency.py` fails if a gate-count written in the docs disagrees with `.githooks/pre-commit`. Repeated literals drift, and the drift is invisible until someone counts by hand (2026-10-07: three living docs stated a stale gate count while the hook already ran more).

## 5. Git workflow / 工作流：Adding an entry

1. Copy `failures/TEMPLATE.md` → `failures/SF-<next>-<slug>.md`
2. Fill every section. Empty sections are a validation failure, not a style choice.
3. Link the new file from **all four** index targets: `failures/README.md`, the taxonomy table in `README.md`, `llms.txt`, and `docs/start-here.md` (symptom → entry). `check-catalog.py` fails on any that is missing.
4. Add a line to `CHANGELOG.md`.
5. Run `python tools/check-catalog.py` — it must exit 0.
6. Run `python tools/make-manifest.py` — the manifest covers the new file.
7. Run the whole gate. The list is not repeated here — it has one source, `.githooks/pre-commit`, and `.github/workflows/gates.yml` mirrors it gate-for-gate. Run it directly:

```bash
bash .githooks/pre-commit
```

Every gate must exit 0. Enable the hook once per clone: `git config core.hooksPath .githooks`. A gate that depends on being remembered has a silent failure mode of its own — the run where you forget looks identical to the run where everything passed.

8. If this fixes a defect (yours or a reviewer's), add a record to `docs/defect-ledger.json` — class it (`SF-nnn`, read from `docs/taxonomy.md`), name the escape cause (`E1`–`E4`), name the gate you added, and scan the sibling entries for the same class. `tools/defect-closure.py` fails if the record is missing or open. See rule 14.
9. If you added or changed a check, register its negative control in `tools/checker-mutation.py` (or an equivalent) so it is *shown* able to fail; and if it is a new gate, add both the gate and its `--selftest` to `.githooks/pre-commit` and `.github/workflows/gates.yml`. See rule 13.

## 4. Code style / 代码风格

- English is the primary language. Add a `中文要点` block at the end of an entry if it is useful; do not translate the whole file.
- Section headings are fixed and machine-parsed. Do not rename them. The required set:

```
## Symptom
## Why it is silent
## Minimal reproduction
## Self-check
## Fix
## Related
```

- Code blocks used as reproductions must be runnable as-is, or explicitly marked `# illustrative`.
- Keep entries under ~120 lines. If you need more, you are describing two failure modes.

## 4b. Writing a detection rule (tools/)（保留原文）

- Use `ast` for Python source analysis. Do not regex source you can parse.
- **Every heuristic must declare its confidence** (`high` / `medium` / `low`) and its known false positives.
- **Every rule must have a sample pair** in `tools/samples/`: one `bad` file it must flag, one `good` file it must not. `--selftest` runs them. A rule without a sample pair is not merged.
- Do not add dependencies. If you need a library, the rule does not belong here.

## 6b. Boundaries / 边界：What not to do（保留原文）

- Do not add an "awesome list" of links. This is a catalog of *patterns*, not a link dump. If a link is genuinely our own channel, it belongs on an entry page (rule 9); if it is not, it has to earn its place by supporting a specific claim.
- Do not add entries about failures that already produce a crash or non-zero exit. Those are ordinary bugs.
- Do not soften the language into marketing. Entries are technical claims; the value is that they are falsifiable.


## 6c. Boundaries / 三级边界表

| 级别 | 内容 |
|---|---|
| ✅ **Always** | 改动后跑 `bash .githooks/pre-commit` 全绿；改内容同 commit 重生成 manifest；每条规则有 `--selftest` 与可失败性（mutation）证据；未知事实标 `[NEEDS CLARIFICATION: …]` 不猜 |
| ⚠️ **Ask first** | 修改 `docs/taxonomy.md` 的 `SF-nnn` 编号（append-only，仅可弃用）；改动 `.githooks/pre-commit` 门禁清单；把外链加入非入口页例外 |
| 🚫 **Never** | 提交密钥/凭据；`|| true`、静默跳过、把不可读文件当空文件；合并两个失败模式进一个文件；用语言软化技术声明；添加第三方依赖 |

## Definition of done / 完成定义

1. 全部门禁 exit 0（含负对照）；2. manifest 与本 commit 内容一致；3. 新/改检查已注册 mutation 负控制；4. 无占位、无未经证实的必要性断言；5. 计数只在一处声明（hook）；6. 缺陷闭环（如适用）。

## Escalation / 升级路径

- 门禁失败 → 修到绿，绝不绕过或静默吞掉。
- 需要重写历史/改许可证/对外发布 → 交 Steven 批准。
- 规则间冲突 → 以 `.githooks/pre-commit` 与本文 Hard rules 为权威，冲突点开 issue。
