# Reject invalid judge sample counts

## Requirement and cause

PRD 34.4 includes privacy measurement. PRD 34.5 defines model judges.
CONTRIBUTING.md section 3.1 accepts corrections to simulations that measure the wrong result.

The CLI converted --judge with Number after running the world, without validating the count.
With 0 or NaN, it selected no messages and called neither judge helper.
It still exited 0 and reported zero privacy leaks, zero errors and 11 facts audited.
The facts count described available facts, although no privacy audit ran.

## Fix and alternatives

Parse the optional count once after help handling and before world setup.
Require a positive safe integer, then pass that value to the existing judge function.
The omitted option keeps its existing behavior. No judge prompt, score threshold or safety policy changes.

Two alternatives were considered:

1. Replace invalid values with one. This hides a configuration error and can create an unrequested model call.
2. Check inside judgeRun. This rejects the same values after the full simulation has already run.

The early CLI check avoids both costs without adding a new parser or dependency.

## Same-command measurement

Base: a0ed7bc6495d9e524afc3d500738212a28dc7bc2. Runtime: Bun 1.4.2.
Date: November 1, 2026, America/Vancouver.
Fixture: 40 seeded synthetic personas, 14 days, seed 1, stub network, --no-log.
Repeat the same command with --judge=0 and --judge=NaN:

```bash
SURPLUS_API_KEY=local-test-only SURPLUS_BASE_URL=http://127.0.0.1:9/v1 \
  OPENAI_API_KEY= JUDGE_PROVIDER=surplus JUDGE_MODEL=gpt-6-luna \
  LLM_MAX_RETRIES=0 LLM_TIMEOUT_MS=100 LLM_DEADLINE_MS=100 \
  bun run packages/sim/src/cli.ts --personas=40 --days=14 --seed=1 \
  --judge=0 --no-log --json
```

| Input | Before | After |
|---|---|---|
| --judge=0 | Exit 0; sampled 0; privacyLeaks 0; factsAudited 11; errors 0 | Exit 1 with a positive-integer configuration error; no world or judge summary |
| --judge=NaN | Exit 0; sampled 0; privacyLeaks 0; factsAudited 11; errors 0 | Same early rejection |

The endpoint and key above are local test settings, not real credentials.
The empty sample invokes no model. The displayed surplus/gpt-6-luna label is configuration text, not execution evidence.
No real provider request, paid call or fake model answer was used.
These two deterministic counterexamples support no population-wide quality claim or statistical estimate.

## Regression and validation

One gate in scripts/sim/audit.ts runs the actual CLI.
It rejects 0, NaN, 1.5, -1, Infinity and 9007199254740992 before simulation or audit output.
A valid --judge=1 control runs a deterministic world with no delivered messages.
That control confirms the option remains valid; it is not model qualification.

Run the same audit command with the new gate present in both versions:

```bash
bun run sim --only audit --json ../judge-audit.json
```

| Check | Before | After |
|---|---|---|
| Audit blocking gates | 14/15; new gate fails because --judge=0 exits 0 | 15/15 pass |
| Existing gate records, excluding timing | All 16 other records present | Identical to before |
| Database-dependent tracked checks | Two skipped | Same two skipped |
| Wall time | 13.1 seconds | 13.2 seconds |

The negative run used the exact main CLI bytes, with the same updated audit gate.
The patched CLI was restored after measurement. The shared baseline checkout was not modified.
Independent probes also reject -Infinity, 9007199254740993, invalid and an empty argument.

The final revision passes all four repository typecheck configurations.
It adds Infinity to the existing gate and includes this report.
No full simulation, database suite, end-to-end, real-model or external-client qualification is claimed for this candidate.
The two database skips are not passing database evidence.

## Limits and agent clients

Positive counts with no delivered messages still produce the existing empty audit result.
This fix rejects invalid configuration; it does not change every zero-sample reporting case.
No member-facing flow, MCP schema, matching decision, model prompt or agent-client integration changed.
No external agent-client acceptance was tested or claimed.
No installs, credentials, account settings, real messages, deployments or publication changed.
