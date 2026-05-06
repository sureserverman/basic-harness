# process-skills

Six domain-agnostic process skills for Claude Code. The four orange ones drive a project from idea → done; the two blue ones review what comes out the other end.

```
brainstorming   →   planning-projects   →   executing-plans   →   peer-review-deliverable
                                                  ↑                       │
                                       evidence-first-investigation       │
                                                  ↑                       │
                                            review-document  ←————————————┘
```

Useful for research, writing, project management, analysis — not just coding.

## Install

See the [top-level basic-harness README](../../README.md#install) for the canonical install flow (Cowork → Customize → upload zip from this repo's GitHub releases).

## Tour

For a calm two-minute read-only walkthrough of this plugin together with `delegation-agents` and `vault-librarian` — the brainstorm-plan-execute pipeline, the parallel-worker model, and the optional vault stack — run:

```text
/process-skills:tour
```

For the marketplace-wide map (where this plugin sits in `basic-harness` overall), run `/welcome:tour` from the `welcome` plugin.

## Skills

| Skill | When it fires | Hands off to |
|---|---|---|
| `brainstorming` | Vague idea → validated design, one question at a time | `planning-projects` |
| `planning-projects` | Validated design → staged plan with gates and rollback notes | `executing-plans` |
| `executing-plans` | Staged plan → completion via Confirm-Adjust loops and stage gates | (done) |
| `evidence-first-investigation` | Something looks wrong — gather evidence before proposing fixes | (informs any of the others) |
| `review-document` | Holistic review of a reader-facing document against the 30-second rule | `document-rewriter` (delegation-agents) |
| `peer-review-deliverable` | Review what's NEW in a deliverable against a baseline (prior version, plan, template) | `document-rewriter` (delegation-agents) |

## Pairs with

- **`vault-librarian`** + upstream **`obsidian-wiki`** — when the deliverable is a vault page; the planning skill knows how to file output into a researcher / writer / generic vault layout.
- **`delegation-agents`** — `executing-plans` and `peer-review-deliverable` both delegate scan-and-edit work to the cheaper agents in that plugin.

## Vocabulary

The plan / execute / review skills share one execution model:

- **Confirm-Adjust loop** — make an attempt, check the evidence-of-done, adjust, re-check. Generalized from Kent Beck's Red-Green from TDD so it works outside coding.
- **Evidence-of-done** — a concrete, checkable criterion that says a task is finished. Replaces "test" so the same plan format works whether the deliverable is code, a chapter, a campaign brief, or a pitch deck.
- **Stage gate** — a synchronization point between phases. The next stage doesn't start until the current one's gate is green.

These are explained in `planning-projects/SKILL.md` and used verbatim by `executing-plans/SKILL.md`.
