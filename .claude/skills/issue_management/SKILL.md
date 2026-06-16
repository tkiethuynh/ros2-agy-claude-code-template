---
name: issue_management
description: Create well-formed tracking issues on GitHub (gh) or GitLab (glab / REST API) using the project's house format — Description / checkbox Acceptance Criteria / Dependencies / Resources / Notes. Maps an SDD spec onto a tracker (BR → milestone, UC → issue, AC → checkboxes). Trigger when creating issues or wiring issue creation into /sdd.
---

# Issue management

One house format for every issue, and one recipe per platform to create it.
The format is the single source of truth and lives in
**`.gitlab/issue_templates/Feature.md`** (GitLab loads it natively; reuse the
same body on GitHub). Five sections, inspired by a clean real-world board:

```
## Description          – what & why, plus constraints
## Acceptance Criteria  – observable, testable; one checkbox each (= "done")
## Dependencies         – blocking issues / hardware / prior work, or "None"
## Resources            – datasheets, docs, links; for SDD, the spec file
## Additional Notes      – priority, timeline, safety, contact (optional)
```

Title convention: `[<Area>] <imperative summary>` (e.g. `[IMU] Publish to ROS 2`).
For SDD-driven work use the spec IDs instead: `UC-<ID>: <name>`.

## Mapping an SDD spec onto a tracker

| Spec element | Tracker artifact |
|--------------|------------------|
| BR (one per feature) | a **milestone** (the "phase") |
| UC | one **issue**, title `UC-<ID>: <name>` |
| AC items | the issue's **Acceptance Criteria** checkboxes (`- [ ] AC-1: …`) |
| UC→UC ordering | **Dependencies** (`- [ ] #<iid>` / `UC-<ID>`) |
| spec file | a **Resources** link (`.claude/specs/<UC-ID>-<slug>.md`) |

## Create issues

Pick the recipe for the project's host. Body comes from the format file so
it never drifts: `BODY="$(cat .gitlab/issue_templates/Feature.md)"`, then fill
in the sections.

### GitHub — `gh`
```bash
gh api repos/:owner/:repo/milestones -f title="<BR goal>" 2>/dev/null || true
gh issue create --title "UC-<ID>: <name>" --milestone "<BR goal>" \
  --label "enhancement" --body "$BODY"
```

### GitLab — `glab` (if installed)
```bash
glab issue create --title "UC-<ID>: <name>" --label "enhancement" \
  --milestone "<BR goal>" --description "$BODY"
```

### GitLab — REST API (universal; no glab needed)
```bash
: "${GITLAB_URL:?e.g. export GITLAB_URL=https://gitlab.example.com}"
: "${GITLAB_TOKEN:?put a PAT in GITLAB_TOKEN — never hardcode or commit it}"
: "${GITLAB_PROJECT:?URL-encoded path, e.g. group%2Fproject}"
API="$GITLAB_URL/api/v4/projects/$GITLAB_PROJECT"

# 1) milestone for the BR (create once; capture its id)
MID=$(curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --data-urlencode "title=<BR goal>" "$API/milestones" | python3 -c 'import sys,json;print(json.load(sys.stdin).get("id",""))')

# 2) one issue per UC
curl -s --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --data-urlencode "title=UC-<ID>: <name>" \
  --data-urlencode "description=$BODY" \
  --data "labels=ROS2,enhancement" \
  --data "milestone_id=$MID" \
  "$API/issues"
```

## Rules

- **Secrets:** the token is read from `$GITLAB_TOKEN` (or `gh`/`glab` auth) —
  never write a `glpat-…` / `gho_…` token into a file, command snippet, or
  commit. If one leaks, rotate it.
- **Idempotency:** before creating, list existing issues and skip if an issue
  with the same `UC-<ID>` title already exists (re-running must not duplicate).
- **No fabrication:** if there is no tracker, or the CLI/API is unreachable,
  do **not** invent issue numbers — track the AC checklist inline in the
  spec's `## History` and report that issues were skipped.
- **Labels:** combine an *area* label (`IMU`, `ROS2`, `Motor`, `Software`…)
  with a *type* (`enhancement`, `documentation`, `good first issue`).

Used by the `/sdd` workflow (Step 2). See `commands/sdd.md` and the format in
`.gitlab/issue_templates/Feature.md`.
