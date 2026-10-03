---
name: wayfinder2
description: Chart an effort too big for one session as a shared map of decision tickets on the issue tracker, then resolve one ticket per session until the way is clear or the effort is parked. Every question reaches the human as a pop-up with up to four options and a recommendation.
disable-model-invocation: true
---

A loose idea has arrived, too big for one agent session, and wrapped in fog: the way from here to the **destination** is not visible yet. Wayfinding finds that way instead of charging at the destination. This skill charts the way as a **shared map** on the issue tracker, then works its **decision tickets** (questions whose resolution is a decision, not slices of a build) one at a time, until the route is clear or the evidence says the destination is not worth reaching.

The destination varies per effort and naming it is the first act of charting: a spec to hand off, a decision to lock, or a change made in place. The map is domain-agnostic.

Wayfinder2 differs from wayfinder in three ways: every question is a **pop-up** (see Rounds), the map carries a **kill question** and one **challenge** per round (see Posture), and the map has a **finish line** (see Work through the map).

## Plan, don't do

Each ticket resolves a decision. The map is done when nothing is left to decide before someone goes and does the thing. The pull to just do the work is the signal you have reached the edge of the map. An effort can override this in its **Notes**; absent that, produce decisions, not deliverables.

## Posture

Constructive with teeth. Grill-me is relentless; wayfinder2 is measured:

- **Prior art first.** Before the first question, know who has built this or solved it, and what the repo already constrains. Named prior art goes into round 1 bodies.
- **One kill question** when the destination is named: what makes reaching it worth the journey, given what already exists. Park and redraw are real options on that pop-up.
- **One challenge per round.** One question each round tests something the human has asserted, with the evidence against it in the body. Recommend against them when the evidence says so.
- **One pushback per hand-wave.** An answer of "later" or "we'll see" on a question that gates the destination is re-asked once, sharpened, next round. If it stands, record it under **Risk accepted** in the resolution and move on.
- **Narrate honestly between rounds.** One line per answered question: what it settled, and whether the answer held up. Credit a strong answer; name a weak one as weak.

## Rounds

Every question to the human goes through the **AskUserQuestion** tool, never as markdown in the chat.

- A **round** is the whole **frontier**: every question whose prerequisites are already settled. Up to four questions per call; a larger frontier takes several calls back to back, then recompute. A question that depends on another still open this round belongs to a later round.
- Each question: a header of at most twelve characters; a body that carries the evidence (repo facts, docs, prior art) so the human decides on facts, not on your framing; two to four options, the recommended one first with `(Recommended)` on its label; each option's description names its trade-off in one line. The tool adds **Other** for free text.
- Options are concrete candidates, even for names and numbers, so Other is the escape hatch and not the default.
- Facts are yours to find, decisions are theirs. When a question needs a fact from the environment, dispatch a subagent with an explicit `model` and ask the rest of the frontier now; only the questions downstream of that fact wait.
- An Other answer that is a question back to you gets answered in narration, then the question is re-asked.
- Each round carries the challenge (see Posture).
- A ticket's rounds end when its frontier is empty. Then one **confirm gate** pop-up: record the resolution as stated (Recommended) / amend it / keep asking / park this ticket. Nothing is written to the tracker before that gate.

## Refer by name

Every map and ticket is an issue, so it has a name: its title. In everything the human reads, refer to it by name, with the link riding inside the name. A wall of `#42, #43` is illegible.

## The Map

The map is a single issue labelled `wayfinder:map`; its tickets are child issues. The map is an **index**, not a store: a decision lives in exactly one place, its ticket, and the map gists and links.

**Tracker.** If the repo has `docs/agents/issue-tracker.md`, follow its "Wayfinding operations" section. Otherwise read [trackers.md](trackers.md) in this folder: GitHub issues when the repo has a GitHub remote and `gh` is authenticated, local markdown otherwise.

**Map edits.** Other sessions edit the tracker concurrently. Re-read the map body immediately before every edit, and edit it once per session, at the end. On every load, **reconcile**: any closed child with no line in Decisions so far gets its gist appended (research subagents and interrupted sessions leave these).

### The map body

```markdown
## Destination

<what reaching the end looks like: the spec, decision, or change. One or two lines. The last line records the outcome once the map closes: handed off / done in place / parked, with a link.>

## Notes

<domain; skills every session should consult; standing preferences for this effort>

## Decisions so far

- [<closed ticket title>](link): <one-line gist>

## Risks accepted

- [<ticket title>](link): <the gate question that was deflected, in one line>

## Not yet specified

<!-- fog: in-scope questions you cannot yet phrase sharply; graduates to tickets as the frontier advances -->

## Out of scope

<!-- work ruled beyond the destination; closed, never graduates -->
```

### Tickets

Each ticket is a child issue of the map, sized to one agent session:

```markdown
## Question

<the decision or investigation this ticket resolves>

Destination risk: <high | low>  <!-- high when the answer could redraw or kill the destination -->
```

Labels: `wayfinder:<type>` (research, prototype, grilling, task) and `wayfinder:hitl` on every ticket that needs a human, so an unattended run can filter them out of the frontier.

**Claim.** Assign the ticket to the dev driving the map **first**, before any work, and comment `claimed <ISO time>`. An open, unassigned ticket is unclaimed. A ticket assigned with no comment in the last 24 hours is stale: read its comments, then claim it. **Release.** A session that ends with its ticket unresolved comments where it got to and unassigns.

**Blocking** uses the tracker's native dependency relationship. A ticket is **unblocked** when every blocker is closed; the **frontier** is the open, unblocked, unclaimed children.

**Overrun.** A ticket that will not resolve in one session is split: resolve the part that settled, open a child ticket for the rest with a link both ways, and say "split" in the resolution.

### Resolution

Post this comment, then close the ticket. The **Decision** line is the gist copied to the map.

```markdown
## Resolution
**Decision:** <one line>
**Why:** <evidence and reasoning, short>
**Rejected:** <each option considered, one line, why not>
**Strongest objection:** <the challenge raised and how it was answered>
**Risk accepted:** <none, or the gate question that was deflected>
**Unlocks / invalidates:** <tickets by name; fog graduated>
```

Assets (prototypes, research files, branches) are linked from the comment, never pasted in.

**ADRs.** Call the Skill tool with `mattpocock-skills:domain-modeling` only when a resolution changes the domain model (a term coined or redefined, a structural choice). The ADR links the ticket; the ticket stays canonical.

**Supersede, never rewrite.** A closed ticket's resolution is history. When a later decision overturns it, resolve the new ticket and mark the old line in Decisions so far `superseded by <name>`.

## Ticket Types

Every ticket is **HITL** (worked with a human who speaks for themselves) or **AFK** (the agent alone). A HITL ticket resolves only through the live exchange. If the pop-up cannot be shown, or the human has said they are away, comment `waiting on human`, release the claim, and stop.

- **Research** (AFK): facts from outside the working directory. A subagent with an explicit `model` (default `sonnet`; Notes may override) calls the Skill tool with `mattpocock-skills:research`, posts its findings as the ticket's Resolution comment with a link to the file or branch, and closes the ticket. The map line is reconciled on the next load.
- **Prototype** (HITL): raise the fidelity of the discussion with a cheap concrete artifact. Call the Skill tool with `mattpocock-skills:prototype`; link it as an asset. Use when "how should it look" or "how should it behave" is the question.
- **Grilling** (HITL): conversation through Rounds. The default case.
- **Task**: manual work that must happen before a decision can be made. The one type that does rather than decides, and it earns its place by unblocking a decision. AFK only when it touches nothing outside the repo and destroys nothing. Sign-ups, access, data moves, anything with money or credentials is HITL: call the Skill tool with `mattpocock-skills:wizard` to hand the human a scripted walkthrough, or walk them through it step by step with pop-ups. The resolution records what was done and the facts later tickets depend on.

## Fog of war

The map is deliberately incomplete. Beyond the live tickets lies the **fog of war**: decisions you can tell are coming but cannot yet pin down. **Not yet specified** holds that dim view, as loosely or fully as the view allows. Resolving a ticket clears the fog ahead of it; graduate what is now specifiable into fresh tickets and clear it from the section.

**Fog or ticket?** Ticket when you can state the question sharply now, even if it is blocked. Fog when you cannot. One patch of fog may graduate into several tickets, or none.

## Out of scope

The destination fixes the scope. Work beyond it is out of scope, not fog: it gets a line in **Out of scope** with why, and never graduates. When an existing ticket turns out to sit past the destination, close it and leave that line; it stays out of Decisions so far.

## Invocation

Two modes. Either way, resolve **one ticket per session**, research excepted. End every session by releasing any claim you still hold.

### Chart the map

User invokes with a loose idea.

0. **Prior art and constraints** (AFK, before any question). One research subagent with an explicit `model`: who has built this or solved it, how well, and what the repo already fixes (data shape, platform, pricing, legal). It returns a short brief. Ask nothing until it lands.
1. **Name the destination** through Rounds. Round 1 carries the kill question, with the prior art in its body: proceed as framed / narrow it to a smaller destination / redraw it / park. Recommend whichever the evidence supports. Call `mattpocock-skills:domain-modeling` when terms get coined.
2. **Map the frontier** through Rounds, **breadth-first**: fan out across the whole space, surfacing the open decisions and the first steps takeable now. If this surfaces no fog, you do not need a map: one pop-up offers to run grill-me and to-spec in this session instead, or stop.
3. **Create the map** (label `wayfinder:map`): Destination and Notes filled, fog sketched into Not yet specified, the rest empty.
4. **Create the tickets** you can specify now, with type labels, `wayfinder:hitl` where it applies, and the Destination risk line. Wire blocking edges in a second pass.
5. **Fire the research subagents**, one per research ticket, in parallel. They resolve their own tickets.
6. **Stop.** Narrate the map by name: destination, the frontier, the fog. Charting hand-resolves nothing.

### Work through the map

User invokes with a map (URL or number). A ticket is optional.

1. **Load the map** and reconcile closed children into Decisions so far.
2. **Choose the ticket.** The user's, if named. Otherwise from the frontier: Destination risk high first, then the ticket the most others are blocked by, then creation order. **Claim it.**
3. **Resolve it.** Zoom into at most five closed tickets, gist first, body on demand. Rounds for HITL types; the type's skill otherwise. Consult whatever Notes names.
4. **Destination check.** If the answer undermines the destination, stop resolving and ask: redraw / narrow / park the effort / continue with the risk recorded. A redraw edits Destination, re-runs chart step 2 on the affected area, and supersedes or rules out the invalidated tickets.
5. **Confirm gate**, then record: Resolution comment, close, map line.
6. **Advance the frontier.** Create then wire newly surfaced tickets; graduate fog; supersede; rule out of scope.
7. **Finish line.** Frontier empty and Not yet specified empty means the map is done. One pop-up: hand off to to-spec (Recommended when the destination is a spec) / to-tickets / mark the change done in place / park. Record the outcome as the last line of Destination with a link, and close the map. Parked is a close too, with the reason.
