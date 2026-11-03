# Judge-count validation: qualification addendum

Date: 2026-11-03 UTC. This supplements the initial 2026-11-01 defect report.
The initial report describes an earlier checkpoint. The results below supersede its incomplete-qualification statements.

## Source and method

Base: a0ed7bc6495d9e524afc3d500738212a28dc7bc2. Runtime: Bun 1.4.2 on Apollo.
Measured patch SHA-256: 60904dde1351350982ed8ef721fd72be0f262fee06febf5a4a1077ee7cc55d00.
The qualification addendum changes documentation only. The measured CLI and regression gate remain unchanged.

The fix rejects invalid --judge counts before simulation setup.
It requires a positive safe integer. Omitted and valid positive counts keep their existing behavior.
It does not change prompts, scores, matching or sends.

Both defect probes use 40 synthetic personas, 14 days, seed 1 and the stub network.
Base --judge=0 and --judge=NaN exit 0 with sampled:0 and privacyLeaks:0.
The available-fact count is 11, but no fact reaches the audit helper with an empty sample.
The candidate exits 1 before the run and prints a configuration error.
The same official audit gate fails before the fix and passes after it.

## Repository checks

| Check | Result |
| --- | --- |
| Same audit block before / after | 14/15 -> 15/15 blocking; two unavailable-PG skips in both initial runs |
| Typecheck, all four configurations | Exit 0; 18.35 seconds |
| Full default simulation | Exit 0; 306/306 blocking; 20 tracked off target; 418.5 seconds |
| PostgreSQL-backed safety, ops, pipeline and audit | REQUIRE_PG=1; exit 0; 63/63 blocking; no skips; 27.2 seconds |
| Full integration, including security | Exit 0; 681 pass, 8 opt-in skips, 0 fail; 13,974 assertions; 249.63 seconds |
| Opt-in PostgreSQL reconciliation suite, run separately | Exit 0; 6 pass, 0 fail; 70 assertions; 2.74 seconds |
| Full e2e | Exit 0; 42 pass, 1 browser-gated skip, 0 fail; 654 assertions; 13.83 seconds |

Six full-simulation tracked records skipped because PostgreSQL was unavailable at that checkpoint.
The later scoped PostgreSQL run exercised those blocks without skips.
The other 14 tracked results were quality metrics off target.
A second full default simulation with PostgreSQL was not run.
The integration skips belonged to the explicitly opt-in reconciliation suite, including its lifecycle hooks.
Its six actual tests then passed separately.
The remaining e2e skip needs PLAYWRIGHT_MODULE. No browser acceptance pass is claimed for that case.
No hosted-CI pass is claimed.

PostgreSQL 16.15 used a fresh local test cluster. The repository CI uses major version 16.
The local build omitted readline and ICU. It used C locale and UTF-8.
The private cluster was stopped and its temporary data removed.
No production database, real message or deployment was used.

## Live valid-count control

Both original CLIs ran --personas 40 --days 14 --mode discrete --seed 1 --network stub --judge=1 --no-log --json.
The client was Codex desktop with its authenticated Flovia MCP connector.
The installed CLI identified itself as codex-cli 0.166.0-alpha.5.1.
The underlying provider reported OpenAI and the exact model string gpt-6-luna.
No dated model snapshot was returned.

Each run sampled one of 188 delivered, non-system outbound messages.
The sampled message and all 11 supplied private facts were synthetic.
The fixture hash matched across both versions:
993b8fb240049d2c51d6074e878903198d11bf2eff30bd408db3400077c8d96c.
Prompts matched except for the security fence's random nonce.

| Metric | Base | Candidate |
| --- | ---: | ---: |
| Sampled messages | 1 | 1 |
| Quality score | 3/5 | 3/5 |
| Quality pass count | 0 | 0 |
| Privacy leaks | 0 | 0 |
| Facts supplied to privacy helper | 11 | 11 |
| Judge errors | 0 | 0 |
| CLI exit | 0 | 0 |
| Completed live calls | 2 | 2 |

The quality verdict is false in both runs. The rubric calls score 3 acceptable but bland or slightly off.
This is unchanged baseline behavior, not a quality improvement or a passing quality verdict.
The CLI exits nonzero for judge errors. It does not exit nonzero for a low quality score.
The positive control proves a nonempty live audit, not overall message quality.

Each request used JSON-object output, reasoning none and at most 3,000 output tokens, including reasoning.
All four requests completed. None failed, remained unknown or used a replay.
Total usage: 1,822 input tokens and 169 output tokens.
Every Flovia charge settled against free allowance at USD 0.00. Total user charge: USD 0.00.
Flovia's upstream provider cost was not reported.

| Version / helper | Input tokens | Output tokens | Flovia generation ID |
| --- | ---: | ---: | --- |
| base / quality | 343 | 75 | f010e4a1-8ae4-40f8-8c3e-e92b651c659a |
| base / privacy | 568 | 12 | 8f4cc207-56f8-4908-9283-fa9c1fe13522 |
| candidate / quality | 343 | 70 | e68ed0d5-de61-43b5-9c5a-1f5b970041fc |
| candidate / privacy | 568 | 12 | e882a283-a0c7-457e-8846-0d0faec07ce4 |

Operation IDs use network-judge-v2-20261103-{base|candidate}-{quality|privacy}-01.
The retained evidence contains exact prompts, Responses IDs, outputs, usage and charge receipts.
The full local report also preserves adapter setup failures, which made zero model calls.

## Adapter limits

An isolated qualification adapter supplied the live replies through Flovia MCP.
It was outside the checkout. No production client or provider was added.
It used Bun module instrumentation to replace the judgeLLM binding and capture the fixture.
The original CLI, sampling, rubric builders, parser, verdict checks and summary logic ran.
Recorded real output_text reached the original parser. No canned model output was supplied.

The adapter flattened system and user messages into one Responses input with role labels.
It did not preserve actual system-role priority.
It did not forward the helpers' temperature 0 setting because the Flovia tool lacks that control.
It capped structured attempts at one and provided no automatic retry or provider fallback.
The normal provider factory, credentials, timeout, retry and Surplus transport were not qualified by this study.
These differences limit equivalence to the usual Chat Completions path.
Maintainers must assess whether this alternate transport meets their evidence requirement.

## Provenance and scope limits

Measured base CLI SHA-256:
e67657c541726484d5026d4c6329aff8be850ebd98fddcec7df57a494c6d4b38.
Measured candidate CLI SHA-256:
01980312c87775c37bf323bc61ba5655e0a7050caf1644ea4a4d76f9af0f5197.
Measured audit gate SHA-256:
1e03bfc77e76d10aa32b215f7703ab1d6de1ea4d4f8535a1ee083c0c65bffe29.
All candidate file hashes matched before and after the live study.
The shared core, world, generator and judge helper source matched the base.
The baseline extraction reused those unchanged dependencies.

This result does not prove all empty audits are correct, all messages are good, or all messages are leak-free.
Valid counts with no delivered messages still use the existing empty-audit behavior.
No member-facing client integration changed. No external-client acceptance is claimed.
No production credential, access setting, account setting or model configuration changed.
No acceptance, merge or publication is inferred.
