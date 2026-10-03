# Trackers for wayfinder2

Read this only when the repo has no `docs/agents/issue-tracker.md`. Pick the first that applies.

## GitHub (the repo has a GitHub remote and `gh auth status` succeeds)

The map is a single issue with child issues as tickets.

- **Map**: `gh issue create --label wayfinder:map`. Create missing labels first: `gh label create wayfinder:map`, one per `wayfinder:<type>`, and `wayfinder:hitl`.
- **Child ticket**: a GitHub sub-issue of the map. Add it with the sub-issues endpoint: `gh api --method POST repos/<owner>/<repo>/issues/<map>/sub_issues -F sub_issue_id=<child-db-id>`, where the db id comes from `gh api repos/<owner>/<repo>/issues/<n> --jq .id`. Where sub-issues are off, put `Part of #<map>` at the top of the child body and add the child to a task list in the map body.
- **Blocking**: native issue dependencies. `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` (database id, not `#number`). Open blockers show in `issue_dependencies_summary.blocked_by`. Fallback where dependencies are off: a `Blocked by: #<n>, #<n>` line at the top of the child body.
- **Frontier**: list the map's children with `gh api repos/<owner>/<repo>/issues/<map>/sub_issues --jq '.[] | select(.state=="open") | {number,title,assignee,labels:[.labels[].name],blocked:.issue_dependencies_summary.blocked_by}'`, then drop any with `blocked > 0` or an assignee. Unattended runs also drop `wayfinder:hitl`.
- **Claim**: `gh issue edit <n> --add-assignee @me`, then `gh issue comment <n> --body "claimed <ISO time>"`. Both before any work.
- **Release**: `gh issue comment <n> --body "<where it got to>"`, then `gh issue edit <n> --remove-assignee @me`.
- **Resolve**: `gh issue comment <n> --body-file <resolution.md>`, `gh issue close <n>`, then append the gist line to the map body (`gh issue view <map> --json body` immediately before `gh issue edit <map> --body-file`).
- **Reconcile on load**: closed children (`gh issue list --state closed`, scoped to the map) whose titles are absent from Decisions so far get a gist line from the Decision line of their last comment.

Pipe every `gh issue list` and `gh api` call through `head` or `--jq`; never dump whole bodies you will not read.

## Local markdown (no GitHub remote, or `gh` not authenticated)

The map is a file with one child file per ticket, under a gitignored `.scratch/`.

- **Map**: `.scratch/<effort>/map.md`.
- **Child ticket**: `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`. Lines near the top: `Type: <research|prototype|grilling|task>`, `HITL: <yes|no>`, `Status: <open|claimed|resolved>`, `Claimed: <ISO time>`, `Destination risk: <high|low>`.
- **Blocking**: `Blocked by: NN, NN` near the top. Unblocked when every listed file is `resolved`.
- **Frontier**: files that are `open`, unblocked, and unclaimed; by the order in step 2 of Work through the map, then by number.
- **Claim**: set `Status: claimed` and `Claimed:`; save before any work. **Release**: back to `open`, with a `## Progress` note.
- **Resolve**: append the Resolution block, set `Status: resolved`, then append the gist line to `map.md`.
- **Reconcile on load**: resolved files whose titles are absent from Decisions so far get a gist line.
