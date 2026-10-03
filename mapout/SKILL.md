---
name: mapout
description: Map out an effort too big for one session as a shared map of decision tickets on the issue tracker, then resolve one ticket per session until the way is clear or the effort is parked. Every question reaches the human as a pop-up with up to four options and a recommendation.
disable-model-invocation: true
---

# Mapout

You have been handed something big and blurry: an effort that will take many sessions, where nobody can yet see the route to the end. Mapout breaks it into a **map**, one issue on the repo's tracker, with **tickets** underneath, where each ticket is a question that ends in a decision. Sessions take one ticket each. The map is finished when nothing is left to decide, or when the evidence says the effort should stop.

## What the map is for

- **Decisions only.** A ticket closes with a decision, never with built work. The urge to start building is the sign that planning is done and the map should close through its finish line. Ground rules can allow building inside the map for a given effort; otherwise do not.
- **The destination comes first.** Name what done looks like before anything else: a spec to hand over, a decision to lock, or a change to make in place. Everything on the map is measured against it.
- **One ticket per session.** Research tickets are the exception, because subagents resolve them on their own.
- **Names, never numbers.** When you speak to the human, call maps and tickets by their titles, with the link inside the title. Bare issue numbers mean nothing to a reader.

## How hard to push

Grill-me is relentless. Mapout pushes less, but it does push:

- Research prior art before asking anything, and put what you found into the first round's question bodies.
- The round that names the destination includes a kill question: given what already exists, why is this journey worth making? Park and redraw sit on that pop-up as real choices.
- Every round contains exactly one challenge: a question that tests something the human has claimed, with the counter-evidence in the body. Recommend against them when the evidence warrants it.
- When the human deflects a question that gates the destination with "later" or "we'll see", ask it once more next round, sharper. If they hold their position, write it under Risk carried in the resolution and move on.
- Between rounds, narrate one line per answered question: what it settled and whether the answer held up. Say when an answer was strong and when it was thin.

## Pop-up rounds

Ask the human through the **AskUserQuestion** tool, never as markdown in the chat.

- A **round** is every question you can ask right now: the ones whose prerequisites are settled. Put up to four questions in one pop-up; a bigger round takes several pop-ups back to back before you recompute. A question that depends on another question in the same round waits for the next round.
- Per question: a header of twelve characters or fewer; a body carrying the evidence you found (repo facts, docs, prior art) so the human decides on facts rather than on your framing; two to four options; the recommended option first with `(Recommended)` on its label; each option's description stating its trade-off in one line. The tool adds **Other** for free text.
- Give concrete candidates even for names and numbers, so Other is the escape hatch and not the default.
- You find facts, the human makes decisions. When a question needs a fact from the environment, send a subagent with an explicit `model` for it and ask the rest of the round now. Only the questions that depend on that fact wait.
- When the human's Other text is a question back to you, answer it in narration and re-ask.
- A ticket's rounds end when no questions remain. Then one **confirm gate** pop-up: record the resolution as stated (Recommended) / amend it / ask more / park this ticket. Write nothing to the tracker before the human passes this gate.

## The map on the tracker

The map is one issue labelled `mapout:map`. Tickets are its child issues. Each decision lives in its ticket; the map holds a one-line gist and a link.

**Which tracker.** See [trackers.md](trackers.md): GitHub issues when the repo has a GitHub remote and `gh` is signed in; local markdown otherwise. If Ground rules name a tracker, that wins.

**Editing the map body.** Other sessions edit the tracker at the same time. Re-read the body immediately before you change it, and change it once per session, at the end. At the start of every session, **reconcile**: every closed ticket missing from Settled gets its gist line added. Research subagents and interrupted sessions leave these gaps.

### Map body

```markdown
## Destination

<What done looks like, in one or two lines. When the map closes, the last line records the outcome with a link: handed off / done in place / parked.>

## Ground rules

<Domain. Skills every session should load. Standing preferences for this effort.>

## Settled

- [<ticket title>](link): <one-line gist of the decision>

## Risks carried

- [<ticket title>](link): <the gating question the human deflected, in one line>

## Still foggy

<!-- In-scope questions you cannot yet phrase sharply. Becomes tickets as decisions land. -->

## Ruled out

<!-- Work beyond the destination. Closed for this effort; never becomes a ticket. -->
```

### Ticket body

A ticket is sized to one session.

```markdown
## Decide

<the question this ticket answers>

Could redraw the destination: <yes | no>
```

Labels: `mapout:<kind>` with kind one of `research`, `prototype`, `grilling`, `task`, plus `mapout:attended` on every ticket that needs the human present, so unattended runs can skip them.

**Claiming.** Before doing anything on a ticket, assign it to the dev running the map and comment `claimed <ISO time>`. Unassigned means free. Assigned with no comment in the last 24 hours means abandoned: read its comments, then claim it. **Releasing.** If your session ends with the ticket unresolved, comment where you got to and unassign.

**Blocking.** Use the tracker's native blocked-by relation. A ticket with no open blockers is ready; the **frontier** is every ready, unclaimed ticket.

**Splitting.** A ticket that will not finish in one session gets split: record the part that is settled, open a child ticket for the remainder, link both ways, and write "split" in the resolution.

### Resolution comment

Post this, then close the ticket. The Decision line becomes the gist on the map.

```markdown
## Resolution
**Decision:** <one line>
**Why:** <evidence and reasoning, short>
**Rejected:** <each option considered, one line each, with why not>
**Strongest objection:** <the challenge raised and how it was answered>
**Risk carried:** <none, or the gating question the human deflected>
**Unlocks / invalidates:** <tickets by name; fog turned into tickets>
```

Link assets (prototypes, research files, branches) from the comment; do not paste them in.

**Terms and structural choices.** When a resolution coins or redefines a term, or fixes a structural choice, record it where the repo already records such things (a `CONTEXT.md`, a `docs/adr/` folder, a decisions section in the README). If the repo has nowhere for it, add one line to Ground rules. The record links the ticket; the ticket remains the source of truth.

**Overturned decisions.** A closed ticket is history: never edit its resolution. When a later decision overturns it, resolve the new ticket and mark the old line in Settled `superseded by <name>`.

## Ticket kinds

A ticket is **attended** (the human is present and answers for themselves) or **unattended** (the agent works alone). An attended ticket resolves only through the live exchange. If the pop-up cannot be shown, or the human has said they are away, comment `waiting on human`, release the claim, and stop.

- **Research** (unattended): a fact from outside the working directory. Send a subagent with an explicit `model` (default `sonnet`; Ground rules may override) and this brief: answer the ticket's question from primary sources (official docs, source code, specs, first-party APIs, the live product, web search for prior art); trace every claim to the source that owns it and cite it; keep verified facts apart from inferences. The subagent posts its findings as the Resolution comment, links any file it wrote, and closes the ticket. The map line is added at the next reconcile.
- **Prototype** (attended): make something cheap and concrete for the human to react to. Send a builder subagent with an explicit `model` (default `opus`) under these rules: throwaway and named as such; one self-contained HTML file for a logic or state question, a few switchable variants on one route for a look-and-feel question; runs with one command; no persistence, tests or polish; the full state shown after every action. The builder commits to a throwaway branch, never main. The resolution links the branch and records the verdict. Use when the question is how something should look or behave.
- **Grilling** (attended): pop-up rounds. The default kind.
- **Task**: work that has to happen before a decision can be made, such as getting access or moving data so its shape can be seen. The only kind that acts instead of deciding, and it exists only to unblock a decision. Unattended only when it touches nothing outside the repo and destroys nothing. Sign-ups, access grants, data moves, money and credentials are attended: walk the human through one pop-up per step (do this now / done / skip / blocked), with the exact URL, click path and value to copy in each body. Secrets go straight from the human into `.env` or the secret store, never into a pop-up answer or a comment. The resolution records only the non-secret facts later tickets need: where a credential lives, new URLs, row counts.

## Fog

Leave the map incomplete on purpose. Beyond the live tickets are the questions you can sense but cannot yet phrase. Write those into Still foggy, as rough or as full as your view allows. Each resolution sharpens some of that fog; turn whatever is now sharp into tickets and delete it from the section.

Ticket or fog? Ticket when you can phrase the question precisely today, even if it is blocked. Fog when you cannot. One patch of fog may become several tickets, or none.

## Ruled out

The destination sets the scope. Anything beyond it is not fog; it is ruled out. Give it one line under Ruled out with the reason. It never becomes a ticket on this map; it would need a new destination and a new map. If an existing ticket turns out to lie beyond the destination, close it and leave that line. It stays out of Settled.

## Starting a map

Invoked with a loose idea.

0. **Prior art and constraints**, unattended, before any question. One research subagent with an explicit `model` and the Research brief: who has built or solved this, how well, and what the repo already fixes (data shape, platform, pricing, legal). It returns a short brief. Ask nothing until it lands.
1. **Name the destination** through pop-up rounds. Round 1 carries the kill question with the prior art in its body: proceed as framed / narrow it / redraw it / park. Recommend what the evidence supports. Record coined terms as the Terms rule says.
2. **Survey the space** through rounds, wide rather than deep: surface every open decision and the first steps available now. If no fog appears, the effort fits one session and needs no map: one pop-up offers `/grill-me` then `/to-prd` right now, or stop.
3. **Create the map** with the `mapout:map` label: Destination and Ground rules filled, fog written into Still foggy, the rest empty.
4. **Create the tickets** you can phrase now, with kind labels, `mapout:attended` where needed, and the redraw line. Then a second pass to add blocking edges, since tickets need ids before they can point at each other.
5. **Launch the research subagents**, one per research ticket, in parallel. They resolve their own tickets.
6. **Stop.** Describe the map by name: destination, the ready tickets, the fog. The starting session resolves nothing by hand.

## Advancing a map

Invoked with a map URL or number, and optionally a ticket.

1. **Load the map** and reconcile.
2. **Pick a ticket.** The named one, if given. Otherwise from the frontier: tickets that could redraw the destination first, then the ticket with the most tickets blocked behind it, then oldest. **Claim it.**
3. **Resolve it.** Read at most five closed tickets for context, gist first, full body only when needed. Rounds for attended kinds; the kind's own rules otherwise. Load whatever Ground rules names.
4. **Destination check.** If the answer undermines the destination, stop and ask: redraw / narrow / park the effort / continue with the risk recorded. A redraw edits Destination, re-runs step 2 of Starting a map on the affected area, and supersedes or rules out the invalidated tickets.
5. **Confirm gate**, then record: Resolution comment, close, gist line on the map.
6. **Move the frontier.** Create then wire any new tickets; turn sharpened fog into tickets; supersede; rule out.
7. **Finish line.** When the frontier is empty and Still foggy is empty, the map is done. One pop-up: hand off to `/to-prd` (Recommended when the destination is a spec) / `/to-issues` / mark the change done in place / park. Write the outcome as the last line of Destination with a link, and close the map. Parked is also a close, with the reason.

Whatever happens, end the session by releasing any claim you still hold.
