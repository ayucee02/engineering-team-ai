# Engineering Team — Multi-Agent Software Delivery Crew

A multi-agent AI system that takes a plain-English software requirement and
takes it all the way to working code: design → implementation → a Gradio demo
UI → unit tests — built with [CrewAI](https://crewai.com).

I built this while working through an agentic AI engineering course, then
extended it by removing unused CrewAI template scaffolding (an unreferenced
`knowledge/` file and empty tool boilerplate) and cleaning up stray files
from the original scaffolding.

## What it does

Four agents run **sequentially**, each handing its output to the next:

1. **Engineering Lead** — takes a natural-language requirements spec and
   produces a detailed module design (classes, methods, responsibilities).
2. **Backend Engineer** — implements the design as a working Python module.
   Runs with `allow_code_execution=True` in a **sandboxed Docker
   environment** (`code_execution_mode="safe"`), with execution timeouts and
   retry limits — the LLM's generated code never runs unsandboxed on the
   host.
3. **Frontend Engineer** — writes a minimal Gradio UI (`app.py`) that
   demonstrates the backend module.
4. **Test Engineer** — writes unit tests for the backend module, also inside
   the Docker sandbox.

The default example (in `main.py`) asks the crew to build a small trading
account management system — deposits, withdrawals, buy/sell of shares,
portfolio valuation, P&L reporting — entirely from a plain-English spec.

## Architecture

```
src/engineering_team/
├── crew.py              # 4 agents, sequential process
├── main.py               # Hardcoded requirements + entry point
├── config/
│   ├── agents.yaml        # Role, goal, backstory per agent
│   └── tasks.yaml         # design_task → code_task → frontend_task/test_task
└── tools/
output/                    # Generated design doc, module, app.py, tests
```

## Setup

Requires Python 3.10–3.12, [uv](https://docs.astral.sh/uv/), and Docker
running locally (needed for the sandboxed code execution agents).

```bash
pip install uv
crewai install
```

Add `OPENAI_API_KEY` to a `.env` file in the project root.

## Running it

```bash
crewai run
```

This runs the crew against the requirements defined in `main.py` and writes
the design doc, generated module, Gradio app, and tests to `output/`.

## Why the Docker sandboxing matters

LLM-generated code shouldn't execute directly on the host — it can hallucinate
destructive operations or simply be buggy. Sandboxing with execution timeouts
and retry limits keeps the "write code and immediately test it" loop safe
while still letting the agents self-correct.

## What I'd improve next

- Parameterize requirements via CLI/file input instead of hardcoding them in
  `main.py`.
- Feed pytest failures back into the crew for a self-correcting loop instead
  of a one-shot test run.
- Add a code-review agent before tests run, to catch design issues earlier.

## Credit

Built while completing an agentic AI engineering course (CrewAI-based
multi-agent projects), then extended as a portfolio project.
