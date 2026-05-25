# Coding Agent Experiment Report

This report summarizes the coding-agent experiments on the same 31 selected R
clinical benchmark tasks. Each agent was asked to read task inputs, write an R
solution script, and leave output files for an automatic evaluator.

The case folders remain separate from this report:

- `case4`, `case5`, `case6`: Codex CLI with GPT-5.5
- `case7`, `case8`: Claude Code with claude-sonnet-4-6

## Experimental Settings

| Case | Agent | Setting | Web access | Extra task hint |
| --- | --- | --- | --- | --- |
| `case4` | Codex CLI / GPT-5.5 | Base coding-agent prompt | No explicit web-doc instruction | None |
| `case5` | Codex CLI / GPT-5.5 | Public R package docs explicitly allowed | CRAN, r-universe, GitHub docs | None |
| `case6` | Codex CLI / GPT-5.5 | Base prompt plus reference package-function list | No explicit web-doc instruction | Function list used by reference solution |
| `case7` | Claude Code / claude-sonnet-4-6 | Base coding-agent prompt | WebFetch/WebSearch disabled | None |
| `case8` | Claude Code / claude-sonnet-4-6 | Public R package docs allowed | WebFetch/WebSearch enabled | None |

## Accuracy Summary

| Case | Agent | Setting | Pass rate | Logged token usage |
| --- | --- | --- | ---: | ---: |
| `case4` | Codex CLI / GPT-5.5 | Base | 19 / 31 = 61.3% | 923,277 |
| `case5` | Codex CLI / GPT-5.5 | Web docs allowed | 19 / 31 = 61.3% | 988,296 |
| `case6` | Codex CLI / GPT-5.5 | Reference function list | 22 / 31 = 71.0% | 915,739 |
| `case7` | Claude Code / claude-sonnet-4-6 | Base, no web | 15 / 31 = 48.4% | 5,480,982 |
| `case8` | Claude Code / claude-sonnet-4-6 | Web access enabled | 13 / 31 = 41.9% | 8,496,469 |

The best run in this batch is `case6`: Codex CLI with GPT-5.5 plus a concise
list of R package functions used by the reference solution.

## Cost and Runtime Summary

Claude Code logs expose observed dollar cost and detailed token accounting.
Codex CLI logs in this run expose only a single per-task `tokens used` number.
They do not expose local dollar cost, nor input/output/cache-token breakdowns.
For that reason, this report does not assign a dollar cost to the Codex runs.

The token totals are still useful for comparing scale: the Codex runs consumed
about 0.92-0.99M logged tokens, while the Claude Code runs logged about
5.48-8.50M tokens. The dollar columns below should only be read for Claude Code,
where the agent log reports observed cost directly.

| Case | Agent | Observed total cost | Observed avg cost / task | Logged token usage | Avg runtime / task | Total runtime |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| `case4` | Codex CLI / GPT-5.5 | not logged | not logged | 923,277 | 60s | 31.0 min |
| `case5` | Codex CLI / GPT-5.5 | not logged | not logged | 988,296 | 60s | 30.8 min |
| `case6` | Codex CLI / GPT-5.5 | not logged | not logged | 915,739 | 58s | 30.1 min |
| `case7` | Claude Code / claude-sonnet-4-6 | $4.50 | $0.15 | 5,480,982 | 108s | 55.7 min |
| `case8` | Claude Code / claude-sonnet-4-6 | $7.58 | $0.24 | 8,496,469 | 109s | 56.1 min |

For Claude Code, enabling web access increased cost from $4.50 to $7.58, about
68% higher, while average runtime stayed essentially unchanged.

## Single-Task Codex Cost Probe

On 2026-05-25, we reran one representative task
(`pharmaverse/aNCA/PKNCA_impute_method_start_c1`) under the three Codex prompt
settings. The Codex CLI again reported total tokens only, with no dollar-cost or
input/output/cache split.

| Public case | Probe run | Tokens used | Observed cost |
| --- | --- | ---: | ---: |
| `case4` | `cost_probe_case4_base` | 51,207 | not logged |
| `case5` | `cost_probe_case5_web_docs` | 28,944 | not logged |
| `case6` | `cost_probe_case6_ref_functions` | 90,350 | not logged |

These single-task probes are useful as a sanity check that the Codex CLI still
does not expose cost accounting locally. They should not be converted into
dollar amounts without billing data or a trustworthy input/output/cache-token
breakdown from the same Codex execution path.

## Token Usage Summary

### Codex CLI

Codex token usage below is the sum of the `tokens used` value recorded in each
Codex CLI task log.

| Case | Setting | Total Codex tokens | Avg tokens / task | Accuracy |
| --- | --- | ---: | ---: | ---: |
| `case4` | Base | 923,277 | 29,783 | 19 / 31 |
| `case5` | Web docs allowed | 988,296 | 31,881 | 19 / 31 |
| `case6` | Reference function list | 915,739 | 29,540 | 22 / 31 |

The Codex web-docs setting used about 7% more total tokens than the base run but
did not improve accuracy. The reference-function setting used slightly fewer
tokens than the base run and achieved the best accuracy.

### Claude Code

Claude token usage below is parsed from Claude Code `modelUsage` records. The
totals include the main Sonnet model and the auxiliary Haiku model reported by
Claude Code.

| Case | Setting | Input tokens | Cache creation tokens | Cache read tokens | Output tokens | Web search requests |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| `case7` | Base, no web | 32,153 | 372,542 | 4,965,759 | 110,528 | 0 |
| `case8` | Web access enabled | 1,519,818 | 410,079 | 6,387,164 | 179,408 | 16 |

For Claude Code, enabling web access greatly increased token use, especially
input tokens and output tokens. The web-enabled run was more expensive and had a
lower pass rate on this task subset.

## Main Takeaways

1. Codex CLI outperformed Claude Code on this 31-task set in all comparable
   settings tested here.
2. Simply allowing public documentation lookup did not improve either agent's
   pass rate in this batch.
3. Giving Codex a concise reference package-function list improved accuracy from
   19/31 to 22/31.
4. Claude Code web access increased cost substantially but reduced accuracy in
   this experiment.
5. The remaining failures in the best Codex run were not execution failures;
   they were mostly precise clinical data-transformation details such as units,
   grouping, row retention, labels, or strict output formatting.

## Source Files

Primary result folders:

```text
case4/
case5/
case6/
case7/
case8/
```

Local run directories used to produce this report:

```text
internal/coding_agent_outputs/case2_31/codex/gpt-5.5/case4_fixed/
internal/coding_agent_outputs/case2_31/codex/gpt-5.5/case6_web_docs/
internal/coding_agent_outputs/case2_31/codex/gpt-5.5/case7_ref_functions/
internal/coding_agent_outputs/case2_31/claude-code/claude-sonnet-4-6/case8_base/
internal/coding_agent_outputs/case2_31/claude-code/claude-sonnet-4-6/case9_web_docs/
```
