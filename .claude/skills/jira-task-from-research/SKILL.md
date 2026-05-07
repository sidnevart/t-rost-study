---
name: jira-task-from-research
description: Convert a research note, experiment result, RFC, or benchmark into a well-structured Jira task (or backlog of tasks) ready to be created in Jira. Produces title, description with context/goal/scope/acceptance-criteria/test-plan, links to source artifacts, suggested labels, estimate, and a `jira-cli` command to create it. Use when the user says "сделай jira задачу из этого ресерча", "create a Jira ticket from this experiment", "превращай RFC в задачи".
---

# Jira Task From Research

Turn the output of a research session, experiment, benchmark, or RFC into a Jira-ready task. Bridges the gap between "we figured something out" and "engineering is shipping it".

## When to Use

- User says "создай jira задачу из этого ресерча/эксперимента/RFC"
- After completing a sprint artifact (e.g. `05-weekly-sprints/sprint-XX-*.md` in this repo)
- Converting a benchmark result (`06-practice/02-benchmark-template.md`) into follow-up work
- Breaking an RFC into implementable tickets
- Capturing a TODO that surfaced during code review

## Inputs

The skill takes one of:
- A markdown file (research note, RFC, sprint summary, benchmark)
- A user-pasted blob of findings
- A reference to a section of an existing artifact (e.g. "the optimization options in section 4")

If the input is ambiguous (long doc, multiple distinct asks), **ask the user** which slice to convert before generating tasks. Use `AskUserQuestion` for this.

## How It Works

### Phase 1 — Extract intent

Read the source and identify:

1. **Discovery** — what did the research find?
2. **Decision** — what is now decided / recommended?
3. **Action** — what needs to happen as a result?

A Jira task represents an **action**. If the source has multiple actions, produce multiple tasks (a backlog). If it has zero actions ("we just learned X, no follow-up needed"), say so and stop — do not invent work.

### Phase 2 — Shape each task

For every action, fill this template:

```markdown
### Task <N>: <Imperative title, ≤80 chars>

**Type**: Story | Task | Spike | Bug | Tech debt
**Priority**: P0 | P1 | P2 | P3 — <one-line justification>
**Estimate**: <T-shirt: XS / S / M / L / XL>  or  <story points: 1 / 2 / 3 / 5 / 8 / 13>
**Labels**: <comma-separated, e.g. `audience-platform, performance, q2`>
**Components**: <if known>

#### Context
<Why are we doing this? Link the source artifact. 2-4 sentences max.>
Source: <path/to/research.md#section> or <URL>

#### Goal
<One sentence: what the world looks like when this is done.>

#### Scope
**In scope**
- <bullet>
- <bullet>

**Out of scope**
- <bullet — explicitly call out adjacent things this task does NOT cover>

#### Acceptance criteria
- [ ] <Concrete, verifiable. "Endpoint returns 200 with payload X under load Y", not "is fast">
- [ ] <Each criterion is independently testable>
- [ ] <No more than 5-7 — split the task if you need more>

#### Test plan
- <How will we know it works? Unit / integration / load / manual smoke?>
- <Specific scenarios, not "write tests">

#### Risks / open questions
- <Things that could derail this — call them out so they get addressed in refinement>

#### Links
- Source research: <link>
- Related tickets: <if known>
- Design / RFC: <if applicable>
```

### Phase 3 — Title discipline

Titles must be:
- **Imperative** — "Add bitmap index to audience compute", not "Bitmap index"
- **Specific** — name the component, not the system
- **≤80 characters** — Jira list views truncate
- **Free of jargon a PM can't parse** — internal acronyms only when unavoidable

Bad: `Performance` · `Refactor things` · `BMP idx for AP svc`
Good: `Add bitmap index to AudiencePlatform compute path` · `Spike: evaluate ClickHouse vs DuckDB for export aggregation`

### Phase 4 — Estimate honestly

If the source research already produced a benchmark or prototype, the task is usually S–M. If the action is "design then build", split into a **spike** (XS–S, time-boxed) and a **build** task (size after spike).

If you cannot estimate confidently, write `Estimate: TBD — needs refinement` rather than guessing. A wrong estimate is worse than no estimate.

### Phase 5 — Emit the task

Output the markdown task spec **and** a ready-to-run create command. Default to `jira-cli` (the `ankitpokhrel/jira-cli` tool) since it's the most common CLI; fall back to a `curl` against the Jira REST API if the user has a different setup.

```bash
# jira-cli (https://github.com/ankitpokhrel/jira-cli)
jira issue create \
  --project <KEY> \
  --type "<Type>" \
  --summary "<Title>" \
  --priority "<Priority>" \
  --label "<label1>" --label "<label2>" \
  --body "$(cat <<'EOF'
<full description in Jira-flavored markdown>
EOF
)"
```

If the user has not configured `jira-cli`, also include a one-line note pointing them at `jira init`.

If multiple tasks: emit them as a **backlog** — a numbered list at the top with titles + estimates, then the full spec for each. This lets the user skim before creating.

## Best Practices

1. **One action = one task.** If a description contains "and", split it.
2. **Acceptance criteria are contracts.** Vague ACs ("works well", "is fast") are unverifiable; reject them.
3. **Always link the source.** A Jira task with no link back to the research is unmoored — reviewers can't see the reasoning.
4. **Distinguish spike from build.** A spike has a time-box and a deliverable (a doc, a recommendation), not a feature.
5. **Don't pad.** A 30-line task is fine. A 200-line task is a design doc — link the doc, don't duplicate it in Jira.

## Anti-Patterns to Avoid

- Generating a task for every section of an RFC — most sections are context, not action
- "Implement findings of the research" — meaningless title, not actionable
- Acceptance criteria that restate the title — must add specificity
- Estimating an unfamiliar codebase confidently — write TBD
- Producing the `jira-cli` command but with placeholder `<PROJECT_KEY>` — ask the user for the project key if unknown

## Configuration

If the project has `.claude/jira-defaults.json`, read it for defaults:

```json
{
  "project": "AP",
  "defaultLabels": ["audience-platform"],
  "defaultComponents": ["compute"],
  "defaultAssignee": "@me"
}
```

If no config exists, ask the user once for project key and reusable labels, then offer to save them to `.claude/jira-defaults.json` for next time.

## Examples

### Example 1: Single experiment → single task
**Input**: `06-practice/benchmark-bitmap-vs-postgres.md` showing bitmap is 12× faster
**Action**: Phase 1 finds one action ("adopt bitmap for the hot path"). Phase 2 produces a Story-type task with ACs around latency targets, fallback behavior, and rollout plan. Phase 5 emits jira-cli command.

### Example 2: RFC → backlog
**Input**: `07-final-rfc/01-final-rfc-outline.md` — 7 sections, 3 of which describe new components
**Action**: Ask user "convert all sections or just the implementation chapter?". On "implementation only" → produce 3-5 tasks (one per component) plus 1 spike for the unresolved trade-off. Output as ranked backlog.

### Example 3: Sprint summary → follow-up tasks
**Input**: `05-weekly-sprints/sprint-04-graph-vs-alternatives.md` ending with "open questions"
**Action**: Each open question becomes a Spike task (time-boxed, deliverable = a decision doc). The "decided" items become Story tasks.
