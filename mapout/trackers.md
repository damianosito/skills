# Trackers for mapout

Use the first that applies, unless the map's Ground rules name a tracker.

## GitHub

Applies when the repo has a GitHub remote and `gh auth status` succeeds. Map and tickets are issues; tickets are sub-issues of the map. `<o>/<r>` is owner/repo.

| Operation | How |
|---|---|
| Labels | `gh label create` for `mapout:map`, `mapout:research`, `mapout:prototype`, `mapout:grilling`, `mapout:task` and `mapout:attended` when missing |
| Create map | `gh issue create --label mapout:map --title "<name>" --body-file map.md` |
| Create ticket | `gh issue create --label mapout:<kind> ...`, then attach it: `gh api --method POST repos/<o>/<r>/issues/<map>/sub_issues -F sub_issue_id=<ticket db id>`. The db id comes from `gh api repos/<o>/<r>/issues/<n> --jq .id`. Without sub-issues: `Part of #<map>` as the first line of the ticket body, plus a task-list entry in the map body |
| Block | `gh api --method POST repos/<o>/<r>/issues/<ticket>/dependencies/blocked_by -F issue_id=<blocker db id>`. Without dependencies: a `Blocked by: #<n>` line at the top of the ticket body |
| Frontier | `gh api repos/<o>/<r>/issues/<map>/sub_issues --jq '.[] \| select(.state=="open") \| {number,title,assignee,labels:[.labels[].name],blocked:.issue_dependencies_summary.blocked_by}'`; keep rows with `blocked` 0 and no assignee; unattended runs also drop `mapout:attended` |
| Claim | `gh issue edit <n> --add-assignee @me`, then `gh issue comment <n> --body "claimed <ISO time>"` |
| Release | `gh issue comment <n> --body "<where it got to>"`, then `gh issue edit <n> --remove-assignee @me` |
| Resolve | `gh issue comment <n> --body-file resolution.md`, `gh issue close <n>`, then `gh issue view <map> --json body` immediately before `gh issue edit <map> --body-file` with the gist line added |
| Reconcile | closed sub-issues whose titles are missing from Settled: take the Decision line of the last comment and add a gist line |

Keep output small: `--jq` or `head` on every list and api call.

## Local markdown

Applies when there is no GitHub remote or `gh` is not signed in. Everything lives under a gitignored `.scratch/<effort>/`.

| Operation | How |
|---|---|
| Map | `.scratch/<effort>/map.md`, holding the map body |
| Ticket | `.scratch/<effort>/tickets/NN-<slug>.md`, numbered from 01, with header lines `Kind:`, `Attended: yes/no`, `Status: open/claimed/resolved`, `Claimed: <ISO time>`, `Could redraw the destination: yes/no` |
| Block | a `Blocked by: NN, NN` header line; ready when every listed ticket is `resolved` |
| Frontier | tickets with `Status: open` and no unresolved blockers, ordered as in Advancing a map step 2, then by number |
| Claim | set `Status: claimed` and `Claimed:`; save before any work |
| Release | set `Status: open` and add a `## Progress` note |
| Resolve | append the Resolution block, set `Status: resolved`, add the gist line to `map.md` |
| Reconcile | resolved tickets whose titles are missing from Settled get a gist line |
