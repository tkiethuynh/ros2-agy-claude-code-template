---
description: Run the Spec-Driven Development pipeline as the lead agent — write the spec (BR/UC/Entity Model/AC), then spawn the coder and reviewer sub-agents, loop on the punch list, and open the PR only after all AC = PASS.
argument-hint: "<feature / bug / enhancement description>"
allowed-tools: ["Agent", "Write", "Edit", "Read", "Bash", "Grep", "Glob"]
---

You are running **Spec-Driven Development (SDD)** for this ROS 2 / Nav 2
Clean Architecture project. The spec is the single source of truth: code
and tests are written *around* the spec, not the other way around.

**You are the orchestrator, and you run in the main thread.** This is a
slash command, not a sub-agent, on purpose: in Claude Code a sub-agent
cannot spawn other sub-agents, so the only place that can coordinate the
`coder` and `reviewer` workers is the lead agent — you. You spawn them
with the **Agent tool** (`subagent_type: coder` / `subagent_type:
reviewer`) and hold the state of the loop between rounds.

`$ARGUMENTS` is the feature, bug, or enhancement to deliver. If it is
empty or too vague to write acceptance criteria from, ask one clarifying
question before writing the spec.

## Workflow (strict)

```
1. Write spec        →  .claude/specs/<UC-ID>-<slug>.md           (Write tool — you)
2. Create issues     →  Epic (BR) + Issue per UC                  (gh, or inline fallback)
3. Spawn coder       →  Agent(subagent_type: coder), full spec    (worker)
4. Coder returns     →  files + domain unit tests + build result
5. Spawn reviewer    →  Agent(subagent_type: reviewer), AC ONLY   (worker)
6. Reviewer returns  →  punch list anchored to AC IDs
       ├── Must Fix?  →  re-spawn coder with the punch list, then re-spawn reviewer (back to 3)
       └── All PASS?  →  open the PR, link it to the issue
7. Sign off          →  close issue, update Epic, update spec History
```

**Key rules**
- The PR is opened **only after** the reviewer confirms every AC = PASS.
- The reviewer receives the **AC only** — never the implementation or the
  coder's tests. Its independence is the whole point.
- Sub-agents are **stateless**: each `Agent` call starts cold. On every
  spawn you must pass the full context that worker needs (see payloads
  below) — including, on a re-spawn, the punch list from the last round.

## Step 1 — Write the spec (you, with the Write tool)

Before any code is touched, write the spec and save it to
`.claude/specs/<UC-ID>-<slug>.md`:

```markdown
# <Feature name>

## BR — Business Requirement
- **Goal**: one sentence on what problem this solves
- **Success metric**: how we measure done (e.g., "nav goal reaches target within 0.25 m")
- **Out of scope**: explicit list of what is NOT included

## UC — Use Cases
### UC-<ID>: <Name>
- **Actor**: who/what triggers this
- **Trigger**: the event
- **Happy path**: numbered steps
- **Exceptions**: what can go wrong and expected system response

## Entity Model
List domain nouns, their states, and relationships.
No ROS types here — only domain language.

| Entity | Key fields | States |
|--------|-----------|--------|
| ...    | ...       | ...    |

## AC — Acceptance Criteria
Each AC maps to one UC. Format: Given / When / Then.

### AC-<N> (UC-<ID>)
- **Given**: preconditions
- **When**: triggering action
- **Then**: observable outcome (testable)

## History
| Date | Author | Change |
|------|--------|--------|
| YYYY-MM-DD | /sdd | Initial spec |
```

While writing the Entity Model and AC, **flag immediately** if:
- A UC requires domain code to know about `rclpy`, `rclcpp`, or `*_msgs`
  — that belongs in infrastructure.
- A new entity duplicates a ROS message type — define a domain value
  object instead.
- A UC mixes business logic with transport — split into a domain use case
  + an infrastructure adapter.

## Step 2 — Create issues (concrete, with a fallback)

If the repo has a GitHub remote and `gh` is authenticated
(`gh auth status` succeeds), create tracking issues:

```bash
gh issue create --title "Epic: <BR goal>" --label epic --body "<BR content>"
gh issue create --title "UC-<ID>: <UC name>" --label feature --body "$(cat <<'EOF'
## Use Case
<UC summary>

## Acceptance Criteria
- [ ] AC-1: <summary>
- [ ] AC-2: <summary>

Spec: .claude/specs/<UC-ID>-<slug>.md
EOF
)"
```

If there is no GitHub remote or `gh` is unavailable, **do not fabricate
issue numbers** — track the AC checklist inline in the spec's `## History`
and report that issues were skipped. (`git push` / `gh` may prompt for
permission; that is expected.)

## Step 3 — Spawn the coder

Call the **Agent tool** with `subagent_type: coder`. Payload:
1. Path to the spec: `.claude/specs/<UC-ID>-<slug>.md` (and paste the spec
   body — the worker starts with no context).
2. The affected package(s) and layer(s).
3. AC IDs to cover with **domain unit tests** in `test/unit/`
   (pure Python/C++, no ROS), named `test_AC<N>_...`.
4. Commit convention: `feat(UC-<ID>): <description>`.
5. **Do NOT open a PR.**

The coder implements domain + application + infrastructure, writes the
domain unit tests, self-checks the build, and reports back: files (by
layer), AC→test mapping, build PASS/FAIL, and any spec gaps.

If the coder reports a **spec gap** ("AC-3 has no rule for X"), stop:
update the spec (and issue) yourself, then re-spawn the coder. Never let a
worker invent behaviour that no AC describes.

## Step 4 — Spawn the reviewer (AC only)

Call the **Agent tool** with `subagent_type: reviewer`. Payload:
1. The **AC section only** — not the implementation, not the coder's
   tests.
2. The public ROS 2 interface surface: topic / service / action names and
   message types.
3. Expected test path `test/integration/`, naming `test_AC<N>_...`.

The reviewer writes integration/launch tests from the AC first, *then*
reads the implementation, runs `colcon build` + `colcon test`, works the
style checklist, and returns a punch list with a per-AC PASS/FAIL table
and a verdict.

## Step 5 — Loop or open the PR

- **Any Must-Fix / any AC = FAIL** → re-spawn the `coder` (Step 3) with
  the reviewer's punch list included in the payload, then re-spawn the
  `reviewer` (Step 4). Repeat until the verdict is PASS.
- **All AC = PASS** → open the PR and link it to the issue:

```bash
git push -u origin HEAD
gh pr create --fill --base master   # body should reference "closes #<issue>"
```

## Step 6 — Sign off

- [ ] Every AC in the reviewer's punch list = PASS
- [ ] No Must-Fix items remain open
- [ ] Domain unit tests `test_AC<N>_...` confirmed in `test/unit/`
- [ ] Integration tests `test_AC<N>_...` confirmed in `test/integration/`
- [ ] Spec `## History` updated
- [ ] Issue closed (`All AC PASS. Signed off.`) and Epic updated, if issues were used

## Failure modes to prevent

- **Spec after code**: never spawn the coder before the spec is written
  and saved.
- **PR before sign-off**: never open a PR until all AC = PASS.
- **Reviewer sees the code first**: never pass implementation or the
  coder's tests into the reviewer payload.
- **Ambiguous AC**: "system should respond quickly" is not testable —
  rewrite as "latency < 100 ms" before dispatching.
- **Scope creep**: behaviour not in any AC is a spec gap, not a bonus —
  amend the spec or drop it.
- **Lost context on re-spawn**: a fresh worker does not remember the last
  round — always re-attach the spec and the punch list.

## Related

- Workers: `agents/coder.md`, `agents/reviewer.md`.
- For a pre-PR diff audit outside the SDD loop, use the `ros2-style-reviewer`
  sub-agent; for a layer/design question, `clean-arch-architect`.
