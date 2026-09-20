# Agent Behavior Drift

> **Detect behavioral drift in AI agent sessions — tool-call patterns, output quality, decision anomalies. Baseline + alert.**

Agent Behavior Drift (ABD) watches your AI agent's behavior over time and raises an alarm when it changes in ways that matter — without requiring LLM calls or expensive observability infrastructure.

## Problem

AI coding agents (Claude Code, Cursor, Codex, Copilot, etc.) drift silently:

- **Tool-call patterns change** — an agent that used to call `git log` before making changes suddenly stops, or starts calling destructive tools it never used before.
- **Output quality shifts** — the same prompt produces subtly different code style, different error handling, different test coverage.
- **Decision anomalies** — an agent that typically asks before deleting files now deletes without confirmation.
- **Session budget drift** — average tokens per task creeps upward over weeks without anyone noticing.

Existing observability tools (LangFuse, Phoenix, LangSmith) are LLM-focused: they trace model calls and token usage. They don't answer the question **"Is my agent behaving differently than it did last month?"** without expensive re-evaluation.

## Solution

ABD works on **session logs** — the JSONL/JSON records your agent produces. No LLM calls needed. It:

1. **Builds a baseline** from historical sessions (tool-call frequency, output size, decision patterns, token usage, error rates).
2. **Compares new sessions** against the baseline using statistical tests (z-score, chi-squared, Kolmogorov-Smirnov).
3. **Alerts on drift** when metrics exceed configurable thresholds (default: 2σ for 3+ consecutive sessions).
4. **Emits SARIF** for GitHub Code Scanning integration.

Works with any agent that emits structured logs: Claude Code, Cursor, Codex, custom agents.

## Quick Start

```bash
pip install agent-behavior-drift

# Build baseline from historical sessions
abd baseline --sessions ./past-sessions/ --output .abd-baseline.json

# Check a new session against the baseline
abd check --session ./new-session.jsonl --baseline .abd-baseline.json

# CI mode: exit 1 on drift (for GitHub Actions)
abd check --session ./new-session.jsonl --baseline .abd-baseline.json --threshold 2.0
```

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│ Session Logs│────▶│   Baseline   │────▶│   Drift     │
│  (JSONL)    │     │   Builder    │     │  Detector   │
└─────────────┘     └──────────────┘     └──────┬──────┘
                                                │
                    ┌──────────────┐            │
                    │   SARIF /    │◀───────────┘
                    │   Markdown   │
                    │   Report     │
                    └──────────────┘
```

### Detectors

| Detector | What it catches |
|----------|-----------------|
| `tool-frequency` | Tool calls per session deviate from baseline |
| `output-size` | Output token count shifts significantly |
| `error-rate` | Tool error rate increases |
| `decision-pattern` | Agent stops asking before destructive actions |
| `budget-drift` | Average tokens per task creeps up |
| `latency` | Response time anomalies |
| `schema-drift` | Output schema structure changes |

## Output

Default output is human-readable Markdown. With `--sarif`, emits SARIF 2.1.0 for GitHub Code Scanning.

```markdown
# ABD Report: session-2026-09-20.jsonl

## Drift Detected ⚠️

| Metric | Baseline | Current | Z-Score | Status |
|--------|----------|---------|---------|--------|
| tool_frequency | 12.3 ± 3.1 | 28.0 | 5.06 | ⚠️ DRIFT |
| error_rate | 0.02 ± 0.01 | 0.15 | 13.0 | ⚠️ DRIFT |
| output_tokens | 1500 ± 400 | 1200 | -0.75 | ✅ OK |

## Recommendations

- Agent calling tools 2.3x more than baseline — investigate prompt changes
- Error rate 7.5x higher — check tool definitions or permissions
```

## Stack

- **Python 3.10+** — no external dependencies for core detection
- **Click** — CLI interface
- **Rich** — terminal output formatting
- **pytest** — testing

## Roadmap

- [ ] `abd init` — interactive baseline wizard
- [ ] GitHub Action for CI drift checks
- [ ] JSON Schema for session log format validation
- [ ] Session log collectors for Claude Code, Cursor, Codex
- [ ] Trend dashboard (time-series visualization)
- [ ] Slack/Discord webhook alerts
- [ ] Multi-agent session correlation
- [ ] Custom detector plugin system

## Contributing

PRs welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT — see [LICENSE](LICENSE).
