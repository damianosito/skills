# Alpine.Dev Public Skills

Free, reusable agent skills published by [Alpine.Dev / damianosito](https://github.com/damianosito).

These skills package detailed workflows, evaluation rules, output formats, and examples so that ChatGPT or Codex can perform a specialised task consistently instead of relying on a one-off prompt.

## Available Skills

| Skill | What it does | Status |
|---|---|---|
| [`business-idea-feasibility`](business-idea-feasibility/) | Produces a researched Red, Amber, or Green feasibility study for a business, app, SaaS, marketplace, product, side-hustle, or startup idea. | Available |

## Business Idea Feasibility

The `business-idea-feasibility` skill is a sceptical business-idea evaluator. It is designed to protect founders from spending time and money on ideas that are exciting but commercially weak, saturated, legally risky, too expensive, or difficult to distribute.

It does not automatically praise an idea. It researches and challenges the idea before recommending whether to abandon it, validate it further, or proceed with a narrow MVP.

### What It Analyses

Every full report covers:

1. Idea summary and one-sentence value proposition.
2. Assumptions made because information was missing.
3. Red, Amber, or Green verdict.
4. Twelve-category feasibility score out of 60.
5. Competitors, substitutes, market saturation, competitor weaknesses, and missing market needs.
6. Market demand, target niche, and realistic potential share.
7. USP strength and whether the proposed difference is genuinely valuable.
8. Build complexity, MVP scope, setup costs, subscriptions, legal costs, fixed costs, variable costs, and break-even.
9. Monetisation and pricing.
10. Legal, ethical, privacy, safety, and compliance risks.
11. Go-to-market plan and route to the first 10 and 100 customers.
12. Numerical kill criteria.
13. Seven-day validation plan.
14. Smallest practical MVP and manual concierge alternative.
15. Direct final recommendation.

The skill also applies a novelty-bias filter to distinguish a painful paid problem from an interesting concept that is unlikely to become a sustainable business.

### Founder Fit

The skill uses relevant information the current user has shared about their experience, skills, customer access, budget, technical ability, constraints, and preferred way of working.

It does not contain the skill author's personal background. If little user context is available, founder fit is scored conservatively and the unknowns are stated rather than invented.

Founder credibility is treated as an execution advantage, not as proof that customers want the product.

## Installation

### Option 1: Install With Codex Skill Installer

This is the simplest method.

1. Open Codex.
2. Invoke the built-in installer:

```text
$skill-installer
```

3. Ask it to install this skill:

```text
Install the business-idea-feasibility skill from:
https://github.com/damianosito/skills/tree/main/business-idea-feasibility
```

Codex should detect the newly installed skill automatically. Restart Codex if it does not appear.

### Option 2: Download and Install Manually

1. Open the [skills repository](https://github.com/damianosito/skills).
2. Select **Code**, then **Download ZIP**.
3. Extract the downloaded ZIP.
4. Find the `business-idea-feasibility` folder inside it.
5. Copy that complete folder, including `SKILL.md`, `agents/`, and `references/`, to one of the locations below.

For a user-wide installation available in all your Codex projects:

```text
$HOME/.agents/skills/business-idea-feasibility/
```

For an installation used only by one repository:

```text
YOUR-REPOSITORY/.agents/skills/business-idea-feasibility/
```

Do not copy only `SKILL.md`. The references contain the required report format, scoring rules, research process, and examples.

## How to Use It

Invoke the skill explicitly by mentioning its name:

```text
Use $business-idea-feasibility to evaluate this idea brutally honestly:

[Describe your idea here]
```

You can provide a rough or badly written idea. The skill will extract the likely user, problem, product, buyer, business model, risks, and missing assumptions.

### Example: Basic Evaluation

```text
Use $business-idea-feasibility to evaluate this idea:

A mobile app that lets museum visitors photograph exhibits and later creates
a personal visual journal, cultural map, and concise explanation of what they saw.
```

### Example: Include Founder Context

```text
Use $business-idea-feasibility to evaluate this idea.

Idea: A stock-count and reorder reminder service for independent food trucks.

My context: I ran two food trucks for six years, know 40 local operators,
can build no-code automations, have a GBP 4,000 budget, and want a low-support
side business.
```

### Example: Challenge a Specific Concern

```text
Use $business-idea-feasibility to evaluate this idea, but pay particular
attention to competitor weaknesses, GDPR risk, variable AI costs, and whether
a stranger would realistically pay for it.
```

### Request a PDF

If the host supports PDF creation, ask for a downloadable version after the report:

```text
Now turn this feasibility report into a polished PDF and verify the layout.
```

## Verdicts

- **Red - Abandon / Park:** the current concept has a structural, legal, economic, technical, or distribution problem that makes it a poor use of resources.
- **Amber - Proceed with Caveats:** a real problem may exist, but the niche, demand, differentiation, economics, scope, or compliance needs validation.
- **Green - Proceed:** evidence supports a painful problem, reachable buyer, useful wedge, manageable MVP, plausible payment, and fast validation.

Green means proceed with lean validation. It does not mean immediately building the full product.

## Research Requirements

The best results require an agent with live web access. The skill instructs the agent to verify current competitors, prices, laws, platform rules, market evidence, and technical constraints.

When live research is unavailable, the agent should disclose that limitation and avoid presenting remembered or estimated facts as confirmed-current information.

The included Red, Amber, and Green reports are calibration examples. Their market facts must not be copied into a real report without fresh verification.

## Updating

To update an installed copy:

1. Download or install the latest version from this repository.
2. Replace the existing `business-idea-feasibility` folder with the new complete folder.
3. Restart Codex if the updated skill is not detected automatically.

## Repository Structure

```text
business-idea-feasibility/
|-- SKILL.md
|-- agents/
|   `-- openai.yaml
`-- references/
    |-- report-contract.md
    |-- research-and-scoring.md
    |-- example-red.md
    |-- example-amber.md
    `-- example-green.md
```

## Limitations

- Feasibility reports are decision support, not legal, financial, medical, tax, or investment advice.
- Market data and competitor details can change and must be checked live.
- A score is diagnostic; one fatal flaw can still make an idea Red.
- Founder enthusiasm, waitlist sign-ups, likes, and compliments are not payment evidence.
- The skill recommends manual or paid validation before substantial development.

## Licence

This repository is released under the [MIT Licence](LICENSE). You may use, modify, and share the skills freely, subject to the licence terms.
